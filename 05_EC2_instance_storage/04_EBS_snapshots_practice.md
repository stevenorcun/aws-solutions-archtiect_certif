# AWS – EBS Snapshots : Démonstration Pratique

## Créer un snapshot

### Prérequis

Avoir un volume EBS disponible (ex: 2 GB, gp2)

### Étapes

1. **Volumes** → Sélectionner le volume
2. **Actions** → **Create snapshot**
3. **Description** : `DemoSnapshot`
4. **Create snapshot**

**Résultat** :

```
[Volume EBS 2 GB]
      ↓
[Snapshot créé]
  ├─ Status: Completed
  ├─ Progress: 100%
  └─ Size: 2 GB
```

---

## Voir les snapshots

**Menu gauche** → **Elastic Block Store** → **Snapshots**

Liste de tous vos snapshots :

- Snapshot ID
- Description
- Size
- Status (Completed, Pending, Error)
- Progress (0-100%)

---

## Fonctionnalité 1 : Copier un snapshot vers une autre région

### Cas d'usage : Disaster Recovery

```
OBJECTIF
────────
Snapshot en eu-west-1
  ↓
Besoin de backup en us-east-1
  ↓
En cas de panne région EU → données disponibles en US
```

### Étapes

1. Sélectionner le snapshot
2. **Actions** → **Copy snapshot**
3. **Destination region** : Choisir n'importe quelle région (ex: `us-east-1`)
4. (Optionnel) Chiffrer la copie
5. **Copy snapshot**

**Résultat** :

```
[Snapshot en eu-west-1]
        ↓
    (Copie)
        ↓
[Snapshot en us-east-1] ✅
```

> **Note** : Dans la démo, cette étape n'est pas réalisée pour économiser du temps/argent.

---

## Fonctionnalité 2 : Créer un volume depuis un snapshot

### Objectif : Transférer un volume entre AZ

```
SITUATION INITIALE
──────────────────
Volume original : eu-west-1a (2 GB)
  ↓
Snapshot créé
  ↓
OBJECTIF : Créer un volume identique en eu-west-1b
```

### Étapes

1. **Snapshots** → Sélectionner le snapshot
2. **Actions** → **Create volume from snapshot**
3. Paramètres :
   - **Volume type** : gp2 (même que l'original)
   - **Size** : 2 GB
   - **Availability Zone** : `eu-west-1b` ← **Différent de l'original !**
   - (Optionnel) **Encrypt this volume** : ✅
   - (Optionnel) **Tags** : Ajouter des tags
4. **Create volume**

**Résultat** :

```
AVANT
─────
Volumes:
  └─ vol-abc (2 GB, eu-west-1a) - available


APRÈS
─────
Volumes:
  ├─ vol-abc (2 GB, eu-west-1a) - available
  └─ vol-def (2 GB, eu-west-1b) - available ✅ Nouveau !
      └─ Source: Snapshot de vol-abc
```

**Le nouveau volume en eu-west-1b contient les mêmes données que l'original !**

---

## Schéma : Cross-AZ via Snapshot

```
╔═══════════════════════════════╗
║  AZ : eu-west-1a              ║
║                               ║
║  [Volume A - 2 GB]            ║
║   Données importantes         ║
╚═══════════════════════════════╝
         │
         │ 1. Create Snapshot
         ↓
    [Snapshot]
    Sauvegarde de Volume A
         │
         │ 2. Create Volume from Snapshot
         │    Target AZ: eu-west-1b
         ↓
╔═══════════════════════════════╗
║  AZ : eu-west-1b              ║
║                               ║
║  [Volume B - 2 GB]            ║
║   Mêmes données que Volume A  ║
╚═══════════════════════════════╝
```

**C'est ainsi qu'on "déplace" un volume entre AZ !**

---

## Fonctionnalité 3 : Storage Tiers (Tiers de stockage)

### Vérifier le tier actuel

1. **Snapshots** → Sélectionner le snapshot
2. Onglet **Description** ou colonne **Storage Tier**

**Par défaut** : `Standard`

### Archiver un snapshot

1. Sélectionner le snapshot
2. **Actions** → **Archive snapshot**
3. Confirmer

**Résultat** :

```
AVANT
─────
Storage Tier: Standard
  ├─ Coût: 0.05 $/GB/mois
  └─ Restauration: Immédiate


APRÈS
─────
Storage Tier: Archive
  ├─ Coût: 0.0125 $/GB/mois (75% moins cher)
  └─ Restauration: 24-72 heures ⏱️
```

**Restaurer depuis Archive** :

```
Actions → Restore snapshot
  ↓
Attendre 24-72 heures
  ↓
Snapshot redevient Standard ✅
```

---

## Fonctionnalité 4 : Recycle Bin (Corbeille)

### Créer une règle de rétention

**Menu gauche** → **Elastic Block Store** → **Recycle Bin** → **Create retention rule**

**Configuration** :

```
┌────────────────────────────────────────────┐
│ Rule name: DemoRetentionRule              │
│ Resource type: EBS Snapshots               │
│ Apply to: All resources                    │
│ Retention period: 1 day                    │
│ Rule lock: Unlocked (modifiable/supprimable)│
└────────────────────────────────────────────┘
```

**Create retention rule**

---

### Test : Supprimer un snapshot

```
SANS RECYCLE BIN
────────────────
Delete snapshot
  ↓
❌ Snapshot supprimé DÉFINITIVEMENT


AVEC RECYCLE BIN
────────────────
Delete snapshot
  ↓
Snapshot disparaît de la liste
  ↓
✅ Mais il est dans Recycle Bin !
```

**Étapes du test** :

1. **Snapshots** → Sélectionner le snapshot
2. **Actions** → **Delete snapshot**
3. Confirmer
4. Le snapshot **disparaît** de la liste

---

### Récupérer depuis Recycle Bin

1. **Recycle Bin** → **Resources**
2. Rafraîchir la page
3. Le snapshot apparaît dans la liste !

```
┌─────────────────┬──────────┬────────────────────┐
│ Resource ID     │ Type     │ Deletion date      │
├─────────────────┼──────────┼────────────────────┤
│ snap-0abc123... │ Snapshot │ 2025-02-23 14:30   │
└─────────────────┴──────────┴────────────────────┘
```

4. Sélectionner le snapshot
5. **Recover**
6. Confirmer : **Recover resources**

**Résultat** :

```
Le snapshot réapparaît dans Snapshots → EC2 Console ✅
```

---

## Schéma : Flux Recycle Bin

```
CONFIGURATION
─────────────
Créer Retention Rule
  └─ EBS Snapshots, 1 jour de rétention


SUPPRESSION
───────────
[Snapshot dans Snapshots]
      ↓ Delete
[Snapshot disparaît]
      ↓ Transfert automatique
[Snapshot dans Recycle Bin]
  └─ Rétention: 1 jour


RÉCUPÉRATION (dans les 24h)
────────────────────────────
[Recycle Bin → Resources]
      ↓ Recover
[Snapshot restauré dans Snapshots] ✅


EXPIRATION (après 24h)
──────────────────────
[Snapshot dans Recycle Bin]
      ↓ Rétention expirée
❌ Snapshot supprimé DÉFINITIVEMENT
```

---

## Récapitulatif des actions

| Action                          | Résultat                          | Cas d'usage                      |
| ------------------------------- | --------------------------------- | -------------------------------- |
| **Create snapshot**             | Sauvegarde du volume              | Backup régulier                  |
| **Copy snapshot**               | Snapshot dans autre région        | Disaster Recovery                |
| **Create volume from snapshot** | Nouveau volume (même AZ ou autre) | Migration, restauration          |
| **Archive snapshot**            | 75% moins cher, 24-72h restore    | Snapshots anciens                |
| **Delete snapshot**             | Va dans Recycle Bin               | Nettoyage                        |
| **Recover from Recycle Bin**    | Restauration snapshot             | Annuler suppression accidentelle |

---

## Workflow complet : Migration EU → US

```
1. CRÉATION SNAPSHOT
────────────────────
[Volume en eu-west-1a]
  ↓
Create Snapshot
  ↓
[Snapshot en eu-west-1]


2. COPIE VERS US
────────────────
[Snapshot en eu-west-1]
  ↓
Copy Snapshot → us-east-1
  ↓
[Snapshot en us-east-1]


3. RESTAURATION
───────────────
[Snapshot en us-east-1]
  ↓
Create Volume from Snapshot
  ↓
[Volume en us-east-1a] ✅

RÉSULTAT : Volume avec mêmes données, maintenant aux USA
```

---

## Points clés de la démo

✅ **Snapshot créé** depuis un volume disponible  
✅ **Copy snapshot** vers autre région (DR)  
✅ **Create volume from snapshot** dans autre AZ (migration)  
✅ **Archive tier** pour économiser 75% (non testé dans la démo)  
✅ **Recycle Bin** protège contre suppression accidentelle  
✅ **Recovery** fonctionne dans le délai de rétention

---

## Nettoyage

Pour éviter les coûts :

1. **Supprimer les volumes** créés :
   - vol-abc (2 GB, eu-west-1a)
   - vol-def (2 GB, eu-west-1b)

2. **Supprimer le snapshot** (ira dans Recycle Bin)

3. **Supprimer la Retention Rule** si vous ne l'utilisez plus

4. **(Optionnel)** Vider le Recycle Bin manuellement pour suppression immédiate
