# AWS – EBS Encryption (Chiffrement)

## Qu'est-ce que le chiffrement EBS ?

Le **chiffrement EBS** protège vos données en les rendant illisibles sans la clé de déchiffrement.

---

## Avantages du chiffrement EBS

### Quand vous créez un volume EBS chiffré

```
✅ Données au repos (at rest) chiffrées sur le volume
✅ Données en transit (in flight) chiffrées entre instance et volume
✅ Snapshots automatiquement chiffrés
✅ Volumes créés depuis snapshots chiffrés → automatiquement chiffrés
```

### Gestion transparente

```
Vous créez un volume chiffré
  ↓
AWS gère AUTOMATIQUEMENT :
  ├─ Chiffrement lors de l'écriture
  ├─ Déchiffrement lors de la lecture
  └─ Aucune action de votre part ✅

Impact sur les performances : NÉGLIGEABLE (~0%)
```

---

## Technologie de chiffrement

| Paramètre               | Détail                                 |
| ----------------------- | -------------------------------------- |
| **Algorithme**          | AES-256 (Advanced Encryption Standard) |
| **Gestion des clés**    | AWS KMS (Key Management Service)       |
| **Clé par défaut**      | `aws/ebs` (clé gérée par AWS)          |
| **Clés personnalisées** | CMK (Customer Master Keys) possibles   |
| **Impact performance**  | Quasi nul                              |

---

## Règles de propagation du chiffrement

### Volumes → Snapshots → Volumes

```
Volume NON chiffré
  ↓ Create Snapshot
Snapshot NON chiffré
  ↓ Create Volume
Volume NON chiffré


Volume CHIFFRÉ
  ↓ Create Snapshot
Snapshot CHIFFRÉ
  ↓ Create Volume
Volume CHIFFRÉ ✅
```

**Règle** : Le chiffrement se **propage automatiquement** dans toute la chaîne.

---

## Problème : Chiffrer un volume existant non chiffré

### Situation

```
[Volume EBS existant]
  └─ Encryption: NOT encrypted
  └─ Contient des données importantes
  └─ Objectif: Le chiffrer sans perte de données
```

⚠️ **Impossible de chiffrer directement un volume existant !**

Il faut passer par un processus de migration.

---

## Processus de chiffrement d'un volume non chiffré

### Méthode complète (4 étapes)

```
ÉTAPE 1 : CRÉER SNAPSHOT
────────────────────────
[Volume NON chiffré]
  ↓ Create Snapshot
[Snapshot NON chiffré]


ÉTAPE 2 : COPIER EN CHIFFRANT
──────────────────────────────
[Snapshot NON chiffré]
  ↓ Copy Snapshot + Enable Encryption
[Snapshot CHIFFRÉ] ✅


ÉTAPE 3 : CRÉER VOLUME DEPUIS SNAPSHOT CHIFFRÉ
───────────────────────────────────────────────
[Snapshot CHIFFRÉ]
  ↓ Create Volume
[Volume CHIFFRÉ] ✅


ÉTAPE 4 : REMPLACER L'ANCIEN VOLUME
────────────────────────────────────
[Instance EC2]
  ├─ Détacher volume NON chiffré
  └─ Attacher volume CHIFFRÉ ✅
```

---

## Méthode raccourcie (3 étapes)

**Vous pouvez sauter l'étape de copie de snapshot !**

```
ÉTAPE 1 : CRÉER SNAPSHOT
────────────────────────
[Volume NON chiffré]
  ↓ Create Snapshot
[Snapshot NON chiffré]


ÉTAPE 2 : CRÉER VOLUME CHIFFRÉ DIRECTEMENT
───────────────────────────────────────────
[Snapshot NON chiffré]
  ↓ Create Volume + Enable Encryption (à la volée)
[Volume CHIFFRÉ] ✅


ÉTAPE 3 : REMPLACER
───────────────────
[Instance EC2]
  ├─ Détacher volume NON chiffré
  └─ Attacher volume CHIFFRÉ ✅
```

**Avantage** : Plus rapide, moins d'étapes.

---

## Démonstration pratique : Méthode complète

### Étape 1 : Créer un volume non chiffré

**Volumes** → **Create volume**

```
┌────────────────────────────────────────┐
│ Size: 1 GB                             │
│ Volume type: gp3                       │
│ Encryption: ☐ (NON coché)              │
└────────────────────────────────────────┘
```

**Create volume**

**Résultat** :

```
Volume vol-abc123
  └─ Encryption: Not encrypted ❌
```

---

### Étape 2 : Créer un snapshot (non chiffré)

**Volumes** → Sélectionner vol-abc123 → **Actions** → **Create snapshot**

```
Description: Snapshot unencrypted
Create snapshot
```

**Résultat** :

```
Snapshot snap-xyz789
  └─ Encryption: Not encrypted ❌
```

---

### Étape 3 : Copier le snapshot en activant le chiffrement

**Snapshots** → Sélectionner snap-xyz789 → **Actions** → **Copy snapshot**

```
┌────────────────────────────────────────┐
│ Destination region: Same region       │
│ Encryption: ✅ Encrypt this snapshot  │
│ KMS key: aws/ebs (default)             │
└────────────────────────────────────────┘
```

**Copy snapshot**

**Résultat** :

```
Snapshot snap-encrypted
  └─ Encryption: Encrypted ✅
```

---

### Étape 4 : Créer un volume depuis le snapshot chiffré

**Snapshots** → Sélectionner snap-encrypted → **Actions** → **Create volume from snapshot**

```
┌────────────────────────────────────────┐
│ Size: 1 GB                             │
│ Encryption: ✅ Encrypted (automatique) │
│ KMS key: aws/ebs                       │
└────────────────────────────────────────┘
```

**Create volume**

**Résultat** :

```
Volume vol-encrypted
  └─ Encryption: Encrypted ✅
  └─ Contient les mêmes données que vol-abc123
```

---

### Étape 5 : Remplacer le volume (si attaché à une instance)

```bash
# Arrêter l'instance (recommandé)
aws ec2 stop-instances --instance-ids i-instance

# Détacher ancien volume
aws ec2 detach-volume --volume-id vol-abc123

# Attacher nouveau volume chiffré
aws ec2 attach-volume \
  --volume-id vol-encrypted \
  --instance-id i-instance \
  --device /dev/xvda

# Redémarrer l'instance
aws ec2 start-instances --instance-ids i-instance
```

---

## Démonstration pratique : Méthode raccourcie

### Étape 1 : Créer snapshot (non chiffré)

Identique à la méthode complète.

```
[Volume NON chiffré]
  ↓ Create Snapshot
[Snapshot NON chiffré]
```

---

### Étape 2 : Créer volume chiffré DIRECTEMENT

**Snapshots** → Sélectionner snap-xyz789 (non chiffré) → **Actions** → **Create volume from snapshot**

```
┌────────────────────────────────────────┐
│ Size: 1 GB                             │
│ Encryption: ✅ Enable (activer ici !)  │
│ KMS key: aws/ebs                       │
└────────────────────────────────────────┘
```

**Create volume**

**Résultat** :

```
Volume vol-encrypted-direct
  └─ Encryption: Encrypted ✅
  └─ Créé DIRECTEMENT depuis snapshot non chiffré
```

**Avantage** : Pas besoin de copier le snapshot !

---

## Schéma comparatif des deux méthodes

```
MÉTHODE COMPLÈTE (4 étapes)
───────────────────────────
Volume NON chiffré
  ↓ Snapshot
Snapshot NON chiffré
  ↓ Copy + Encrypt
Snapshot CHIFFRÉ
  ↓ Create Volume
Volume CHIFFRÉ ✅


MÉTHODE RACCOURCIE (3 étapes)
──────────────────────────────
Volume NON chiffré
  ↓ Snapshot
Snapshot NON chiffré
  ↓ Create Volume + Encrypt (à la volée)
Volume CHIFFRÉ ✅

Gain : 1 étape en moins
```

---

## Clés de chiffrement (KMS)

### Clé par défaut : `aws/ebs`

```
Gérée automatiquement par AWS
  ├─ Pas de configuration nécessaire
  ├─ Gratuite
  └─ Rotation automatique
```

### Clé personnalisée (CMK)

```
Créée par vous dans KMS
  ├─ Contrôle total des permissions
  ├─ Audit via CloudTrail
  ├─ Rotation manuelle ou automatique
  └─ Coût : ~1 $/mois par clé
```

**Cas d'usage CMK** :

- Conformité réglementaire stricte
- Besoin de partager des volumes chiffrés entre comptes
- Audit détaillé requis

---

## Partage de volumes/snapshots chiffrés

### Snapshots NON chiffrés

```
[Snapshot public]
  └─ Partageable avec n'importe qui ✅
```

### Snapshots CHIFFRÉS

```
[Snapshot chiffré avec aws/ebs]
  └─ ❌ Impossible de partager (clé gérée AWS, privée)

[Snapshot chiffré avec CMK]
  └─ ✅ Partageable SI vous donnez accès à la clé CMK
```

**Process de partage avec CMK** :

1. Partager le snapshot avec le compte cible
2. Donner accès à la CMK au compte cible (via KMS policy)

---

## Bonnes pratiques

### ✅ À faire

| Pratique                                | Raison                |
| --------------------------------------- | --------------------- |
| **Chiffrer tous les nouveaux volumes**  | Sécurité par défaut   |
| **Utiliser aws/ebs pour usage général** | Simplicité, gratuit   |
| **Utiliser CMK pour conformité**        | Contrôle et audit     |
| **Chiffrer snapshots cross-region**     | Sécurité des backups  |
| **Tester la restauration**              | Vérifier le processus |

### ❌ À éviter

| Pratique                                    | Raison             |
| ------------------------------------------- | ------------------ |
| Laisser des volumes critiques non chiffrés  | Risque de sécurité |
| Partager des CMK sans restriction           | Fuite potentielle  |
| Oublier de supprimer snapshots de migration | Coûts inutiles     |

---

## Coûts

### Chiffrement lui-même

```
Coût du chiffrement EBS : GRATUIT ✅
  └─ Aucun surcoût par rapport à un volume non chiffré
```

### Clés KMS

```
aws/ebs (clé par défaut) : GRATUIT
CMK (clé personnalisée) : ~1 $/mois/clé + appels API KMS
```

### Snapshots

```
Snapshot chiffré = même prix que snapshot non chiffré
  └─ 0.05 $/GB/mois
```

---

## Workflow complet : Migration production

### Scénario réel

```
Production DB sur volume NON chiffré (500 GB)
  └─ Objectif : Migrer vers volume chiffré sans downtime
```

**Plan** :

```
PRÉPARATION (pendant heures creuses)
────────────────────────────────────
1. Créer snapshot du volume prod (500 GB)
   ⏱️ ~30 minutes
2. Créer volume chiffré depuis snapshot
   ⏱️ ~10 minutes


MIGRATION (fenêtre de maintenance)
──────────────────────────────────
3. Arrêter l'application DB
   ⏱️ ~1 minute
4. Détacher ancien volume
   ⏱️ ~30 secondes
5. Attacher nouveau volume chiffré
   ⏱️ ~30 secondes
6. Redémarrer l'application DB
   ⏱️ ~2 minutes

DOWNTIME TOTAL : ~4 minutes ✅
```

---

## Nettoyage après démonstration

### Supprimer les ressources

```
1. SNAPSHOTS
   ├─ snap-xyz789 (non chiffré) → Delete
   └─ snap-encrypted → Delete

2. VOLUMES
   ├─ vol-abc123 (non chiffré) → Delete
   └─ vol-encrypted → Delete
```

**Actions** → **Delete** pour chaque ressource.

---

## Points clés à retenir

✅ **Chiffrement EBS** = AES-256 via KMS  
✅ **Impact performance** = quasi nul  
✅ **Transparent** = AWS gère chiffrement/déchiffrement  
✅ **Propagation** = Volume chiffré → Snapshot chiffré → Volume chiffré  
✅ **Migration** = Snapshot → Copy/Encrypt → New Volume  
✅ **Raccourci** = Snapshot non chiffré → Volume chiffré direct  
✅ **Clé par défaut** = `aws/ebs` (gratuit)  
✅ **Partage** = Snapshots chiffrés nécessitent CMK

---

## Pour l'examen AWS

**Question typique** : "Vous avez un volume EBS non chiffré avec des données. Comment le chiffrer ?"

- ❌ "Activer le chiffrement directement sur le volume"
- ❌ "Copier le volume vers un volume chiffré"
- ✅ "Créer un snapshot, puis créer un volume chiffré depuis le snapshot"

**Question typique 2** : "Un volume EBS chiffré crée un snapshot. Le snapshot est-il chiffré ?"

- ❌ "Non, il faut chiffrer manuellement"
- ✅ "Oui, automatiquement chiffré"

**Mot-clé de l'examen** : "encrypt existing unencrypted volume" = **Snapshot → Copy/Encrypt → New Volume**
