# AWS – EBS Snapshots

## Qu'est-ce qu'un EBS Snapshot ?

Un **snapshot** est une **sauvegarde à un instant T** de votre volume EBS.

C'est comme prendre une **photo** de votre disque à un moment donné.

---

## Caractéristiques principales

### 1. Création de snapshot

| Question                         | Réponse                                             |
| -------------------------------- | --------------------------------------------------- |
| **Faut-il détacher le volume ?** | ❌ Non, pas obligatoire                             |
| **Recommandé de détacher ?**     | ✅ Oui, pour garantir la cohérence des données      |
| **L'instance peut tourner ?**    | ✅ Oui, mais préférable de l'arrêter temporairement |

---

### 2. Portabilité : Cross-AZ et Cross-Region

Les snapshots permettent de **copier/restaurer des volumes entre AZ et régions**.

```
PROBLÈME
────────
Volume EBS en us-east-1a
  ↓
Besoin d'un volume identique en us-east-1b
  ↓
❌ Impossible de déplacer directement (volumes liés à l'AZ)


SOLUTION AVEC SNAPSHOT
──────────────────────
1. Créer un snapshot du volume (us-east-1a)
2. Restaurer le snapshot dans us-east-1b
3. ✅ Nouveau volume créé avec les mêmes données
```

---

## Schéma : Transfert de volume entre AZ

```
╔═══════════════════════════════╗    ╔═══════════════════════════════╗
║  AZ : us-east-1a              ║    ║  AZ : us-east-1b              ║
║                               ║    ║                               ║
║  [Instance EC2 #1]            ║    ║  [Instance EC2 #2]            ║
║       ↕                       ║    ║       ↕                       ║
║  [EBS Volume A]               ║    ║  [EBS Volume B]               ║
║   10 GB, données importantes  ║    ║   (vide au départ)            ║
║                               ║    ║                               ║
╚═══════════════════════════════╝    ╚═══════════════════════════════╝
         │                                        ↑
         │ 1. Créer snapshot                     │
         ↓                                        │
    [Snapshot]                                    │
    Sauvegarde de Volume A                        │
         │                                        │
         └─────── 2. Restaurer snapshot ─────────┘

RÉSULTAT : Volume B contient maintenant les mêmes données que Volume A
```

---

## Cross-Region : Même principe

```
[Volume en eu-west-1]
        ↓
[Créer Snapshot]
        ↓
[Copier Snapshot vers us-east-1]
        ↓
[Restaurer Snapshot en us-east-1]
        ↓
[Nouveau volume avec mêmes données en us-east-1] ✅
```

**Cas d'usage** : Disaster Recovery (sauvegarde dans une autre région)

---

## Fonctionnalités avancées des Snapshots

### 1. EBS Snapshot Archive (Archivage)

**Principe** : Déplacer un snapshot vers un **tier d'archivage** moins cher.

| Caractéristique           | Détail                              |
| ------------------------- | ----------------------------------- |
| **Économies**             | Jusqu'à **75% moins cher**          |
| **Temps de restauration** | **24 à 72 heures** ⏱️               |
| **Cas d'usage**           | Snapshots anciens rarement utilisés |

```
SNAPSHOT STANDARD
─────────────────
Coût : 0.05 $/GB/mois
Restauration : Immédiate


SNAPSHOT ARCHIVE
────────────────
Coût : 0.0125 $/GB/mois (75% moins cher)
Restauration : 24-72 heures
```

**Exemple** :

```
Snapshot de 100 GB
  ├─ Standard : 5 $/mois, restauration immédiate
  └─ Archive : 1.25 $/mois, restauration 24-72h

→ Archiver si vous n'avez pas besoin d'accès rapide
```

---

### 2. Recycle Bin (Corbeille)

**Principe** : Protection contre les suppressions accidentelles.

```
SANS RECYCLE BIN
────────────────
Supprimer snapshot
  ↓
❌ PERDU DÉFINITIVEMENT


AVEC RECYCLE BIN
────────────────
Supprimer snapshot
  ↓
Snapshot va dans la Recycle Bin
  ↓
Rétention : 1 jour à 1 an (configurable)
  ↓
Possibilité de restaurer ✅
```

**Configuration** :

- **Retention** : De 1 jour à 365 jours
- Après expiration → suppression définitive

**Cas d'usage** :

```
Admin supprime accidentellement un snapshot critique
  ↓
Le snapshot est dans la Recycle Bin (retention 30 jours)
  ↓
Admin se rend compte de l'erreur le lendemain
  ↓
Restaure le snapshot depuis la Recycle Bin ✅
```

---

### 3. Fast Snapshot Restore (FSR)

**Problème** : Les snapshots ont une **latence lors de la première utilisation**.

Quand vous créez un volume depuis un snapshot, les données sont **chargées à la demande** (lazy loading).

```
SANS FSR (comportement par défaut)
──────────────────────────────────
Créer volume depuis snapshot (1 TB)
  ↓
Volume créé instantanément
  ↓
Première lecture des données → LENT (chargement depuis S3)
  ↓
Après quelques heures → performances normales


AVEC FSR
────────
Activer FSR sur le snapshot
  ↓
Créer volume depuis snapshot (1 TB)
  ↓
Volume COMPLÈTEMENT initialisé (toutes les données pré-chargées)
  ↓
Première lecture → ✅ RAPIDE (aucune latence)
```

**Caractéristiques** :

| Paramètre       | Détail                                              |
| --------------- | --------------------------------------------------- |
| **Fonction**    | Pré-initialise TOUTES les données                   |
| **Performance** | ✅ Aucune latence dès la première utilisation       |
| **Coût**        | 💰💰💰 TRÈS CHER (environ 0.75 $/snapshot/AZ/heure) |
| **Cas d'usage** | Snapshots très gros + besoin de perf immédiate      |

**Exemple de coût** :

```
FSR activé sur 1 snapshot dans 1 AZ pendant 1 mois
  ↓
0.75 $ × 24h × 30 jours = 540 $ ! 💸

→ À utiliser UNIQUEMENT si absolument nécessaire
```

**Quand utiliser FSR ?**

✅ **Oui** :

- Base de données critique de 10 TB
- Besoin de restauration immédiate avec performances maximales
- Budget confortable

❌ **Non** :

- Cas d'usage standard
- Snapshot < 100 GB (latence négligeable)
- Budget limité

---

## Tableau comparatif des fonctionnalités

| Fonctionnalité            | Avantage               | Inconvénient              | Cas d'usage                  |
| ------------------------- | ---------------------- | ------------------------- | ---------------------------- |
| **Snapshot Standard**     | Restauration rapide    | Coût standard             | Usage quotidien              |
| **Snapshot Archive**      | 75% moins cher         | Restauration 24-72h       | Snapshots anciens/conformité |
| **Recycle Bin**           | Protection suppression | Coût de stockage prolongé | Tous les snapshots critiques |
| **Fast Snapshot Restore** | Aucune latence         | Très cher                 | Snapshots énormes + urgence  |

---

## Workflow typique avec snapshots

```
1. CRÉATION
───────────
[Volume EBS 100 GB en us-east-1a]
  ↓
Actions → Create Snapshot
  ↓
[Snapshot créé]


2. SAUVEGARDE CROSS-REGION (Disaster Recovery)
───────────────────────────────────────────────
[Snapshot en us-east-1]
  ↓
Actions → Copy Snapshot → us-west-2
  ↓
[Snapshot copié en us-west-2] ✅


3. ARCHIVAGE (économies)
─────────────────────────
[Snapshot de 6 mois (rarement utilisé)]
  ↓
Actions → Archive Snapshot
  ↓
[Snapshot archivé] → 75% moins cher


4. PROTECTION (Recycle Bin)
────────────────────────────
Configurer Recycle Bin (retention 30 jours)
  ↓
Supprimer snapshot accidentellement
  ↓
Recycle Bin → Recover → ✅ Restauré


5. RESTAURATION RAPIDE (si besoin critique)
────────────────────────────────────────────
[Snapshot 5 TB]
  ↓
Activer FSR
  ↓
Créer volume → Performances immédiates ✅
```

---

## Points clés à retenir

✅ **Snapshot** = sauvegarde à un instant T  
✅ **Détacher recommandé** mais pas obligatoire  
✅ **Cross-AZ / Cross-Region** = possible via snapshots  
✅ **Archive** = 75% moins cher, restauration 24-72h  
✅ **Recycle Bin** = protection contre suppression (1-365 jours)  
✅ **FSR** = performances immédiates mais TRÈS cher

---

## Pour l'examen AWS

**Question typique** : "Comment déplacer un volume EBS de us-east-1a vers us-east-1b ?"

- ❌ Mauvaise réponse : "Détacher le volume et l'attacher dans l'autre AZ"
- ✅ Bonne réponse : "Créer un snapshot, puis restaurer le snapshot dans us-east-1b"

**Question typique 2** : "Vous avez un snapshot de 10 TB et besoin de performances maximales immédiatement. Que faire ?"

- ❌ "Utiliser Snapshot Archive"
- ✅ "Activer Fast Snapshot Restore (FSR)"
