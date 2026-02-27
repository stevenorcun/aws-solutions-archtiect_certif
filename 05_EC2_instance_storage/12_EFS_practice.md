# AWS – EFS : Démonstration Pratique

## Objectif

Créer un système de fichiers EFS et le monter sur **2 instances EC2 dans différentes AZ** pour partager des fichiers.

---

## Étape 1 : Créer le système de fichiers EFS

### Configuration de base

**EFS** → **Create file system** → **Customize**

---

### 1.1 General Settings

```
┌────────────────────────────────────────┐
│ Name: (optionnel, laisser vide)       │
└────────────────────────────────────────┘
```

---

### 1.2 File System Type

| Option                  | Détail                        | Recommandation         |
| ----------------------- | ----------------------------- | ---------------------- |
| **Regional** (Multi-AZ) | Haute disponibilité, multi-AZ | ✅ Production          |
| **One Zone**            | 1 seule AZ, ~50% moins cher   | ⚠️ Dev/Test uniquement |

**Choix : Regional** ✅

---

### 1.3 Automatic Backups

```
✅ Enable automatic backups (recommandé)
```

---

### 1.4 Lifecycle Management

**Configuration des transitions automatiques :**

```
┌────────────────────────────────────────────────────┐
│ Transition to IA: 30 days (sans accès)            │
│ Transition to Archive: 90 days (sans accès)       │
│ Transition to Standard: On first access           │
└────────────────────────────────────────────────────┘
```

**Workflow :**

```
Fichier créé
  ↓ 30 jours sans accès
Fichier → IA (moins cher)
  ↓ 90 jours sans accès
Fichier → Archive (encore moins cher)
  ↓ Premier accès
Fichier → Standard (supposé être réutilisé)
```

---

### 1.5 Encryption

```
✅ Enable encryption at rest (recommandé)
```

---

### 1.6 Performance Settings : Throughput Mode

⚠️ **Attention à la présentation AWS (confuse)** : Il y a **3 modes**, pas 2 catégories.

| Mode                     | Description                         | Cas d'usage                |
| ------------------------ | ----------------------------------- | -------------------------- |
| **Bursting**             | Throughput croît avec la taille EFS | Petits/moyens volumes      |
| **Elastic** (recommandé) | Scale automatiquement selon besoin  | ✅ Workloads imprévisibles |
| **Provisioned**          | Throughput fixe défini à l'avance   | Besoin de throughput connu |

---

#### Bursting Mode

```
Taille EFS : 1 GB → Throughput baseline calculé
Taille EFS : 1 TB → Throughput plus élevé + burst

Throughput lié à la TAILLE
```

---

#### Elastic Mode (Recommandé ✅)

```
Workload léger : 0 MB/s → EFS ajuste automatiquement
Workload lourd : Scale jusqu'à 100+ MB/s

AUCUNE configuration requise
Payez uniquement ce que vous utilisez
```

**Choix : Elastic** ✅

---

#### Provisioned Mode

```
Vous définissez : 100 MB/s garanti
  └─ Payez pour ce throughput même si non utilisé

Besoin de performance prévisible et garantie
```

---

### 1.7 Performance Mode

| Mode                         | Latence     | Cas d'usage                                 |
| ---------------------------- | ----------- | ------------------------------------------- |
| **General Purpose** (défaut) | Faible      | ✅ Web servers, CMS, applications générales |
| **Max I/O**                  | Plus élevée | Big Data, parallèle massif                  |

⚠️ **Avec Elastic Throughput, seul General Purpose est disponible** (et c'est ce qu'on veut).

**Choix : General Purpose** ✅

---

### Résumé configuration choisie

```
✅ Regional (Multi-AZ)
✅ Automatic backups enabled
✅ Lifecycle: 30 days → IA, 90 days → Archive
✅ Encryption enabled
✅ Throughput: Elastic
✅ Performance: General Purpose
```

**Next**

---

## Étape 2 : Network Access (CRITIQUE)

### 2.1 VPC

```
VPC: Default VPC
```

---

### 2.2 Mount Targets

**Parce qu'on a choisi Regional, on a 3 AZ disponibles :**

```
┌─────────────┬──────────────┬─────────────┬────────────────┐
│ AZ          │ Subnet       │ IP          │ Security Group │
├─────────────┼──────────────┼─────────────┼────────────────┤
│ eu-west-1a  │ (default)    │ Automatic   │ À définir      │
│ eu-west-1b  │ (default)    │ Automatic   │ À définir      │
│ eu-west-1c  │ (default)    │ Automatic   │ À définir      │
└─────────────┴──────────────┴─────────────┴────────────────┘
```

---

### 2.3 Créer Security Group pour EFS

**EC2 Console** → **Security Groups** → **Create security group**

```
┌────────────────────────────────────────┐
│ Name: sg-efs-demo                      │
│ Description: EFS Demo SG               │
│ VPC: Default                           │
│ Inbound rules: (aucune pour l'instant)│
└────────────────────────────────────────┘
```

**Create security group**

---

### 2.4 Attacher le SG aux Mount Targets

Retour sur l'écran EFS Network Access → **Rafraîchir la page** (pour voir le nouveau SG)

```
Pour chaque AZ :
  └─ Supprimer le SG par défaut
  └─ Sélectionner : sg-efs-demo
```

**Next** → **Next** (skip file system policy) → **Create**

---

## Étape 3 : Créer Instance EC2 #1 (Instance A)

### Configuration

**Launch Instance**

```
┌────────────────────────────────────────┐
│ Name: Instance A                       │
│ AMI: Amazon Linux 2                    │
│ Instance type: t2.micro                │
│ Key pair: Proceed without (EC2 IC)    │
└────────────────────────────────────────┘
```

---

### Network Settings

```
┌────────────────────────────────────────┐
│ VPC: Default                           │
│ Subnet: eu-west-1a ← IMPORTANT         │
│ Auto-assign public IP: Enable          │
│ Security group: Create new             │
│   └─ Allow SSH from anywhere           │
└────────────────────────────────────────┘
```

⚠️ **Il FAUT choisir un subnet avant de pouvoir ajouter EFS.**

---

### Configure Storage : Ajouter EFS

**Advanced** → **File systems** → **Add shared file system**

```
┌────────────────────────────────────────────────────┐
│ File system: (sélectionner votre EFS)             │
│ Mount point: /mnt/efs/fs1                         │
│                                                    │
│ ✅ Automatically create and attach security groups│
│ ✅ Automatically mount shared file system         │
│    (via user data script)                         │
└────────────────────────────────────────────────────┘
```

**Ce que ça fait automatiquement :**

1. Crée un nouveau Security Group (ex: `efs-sg-1`)
2. Configure les règles inbound (NFS port 2049)
3. Attache ce SG à l'instance EC2
4. Attache ce SG au EFS
5. Génère le User Data pour monter EFS au boot

**Launch instance**

---

## Étape 4 : Créer Instance EC2 #2 (Instance B)

### Configuration identique

```
┌────────────────────────────────────────┐
│ Name: Instance B                       │
│ AMI: Amazon Linux 2                    │
│ Instance type: t2.micro                │
│ Key pair: Proceed without              │
│ Subnet: eu-west-1b ← Différent de A   │
│ Security group: launch-wizard-2 (créé) │
└────────────────────────────────────────┘
```

**File systems** → **Add shared file system**

```
Même EFS
Même mount point: /mnt/efs/fs1
✅ Auto-create SG (créera efs-sg-2)
```

**Launch instance**

---

## Étape 5 : Vérifier la configuration réseau

### EFS Console → Network Tab

```
╔═══════════════════════════════════════════════════╗
║  AZ : eu-west-1a                                  ║
║    Security Groups:                               ║
║      ├─ sg-efs-demo (créé manuellement)           ║
║      └─ efs-sg-1 (auto-créé pour Instance A)     ║
╚═══════════════════════════════════════════════════╝

╔═══════════════════════════════════════════════════╗
║  AZ : eu-west-1b                                  ║
║    Security Groups:                               ║
║      ├─ sg-efs-demo (créé manuellement)           ║
║      └─ efs-sg-2 (auto-créé pour Instance B)     ║
╚═══════════════════════════════════════════════════╝
```

---

### Vérifier les règles Inbound

**efs-sg-2** (Security Group créé pour Instance B) :

```
Inbound rules:
┌──────────┬──────┬────────────────────────────┐
│ Protocol │ Port │ Source                     │
├──────────┼──────┼────────────────────────────┤
│ NFS      │ 2049 │ sg-launch-wizard-2 (EC2 B) │
└──────────┴──────┴────────────────────────────┘
```

**Ce que ça signifie :**

```
Instance B (sg-launch-wizard-2)
  ↓ Autorisée à accéder
EFS Mount Target (efs-sg-2)
  ↓ Qui est dans
eu-west-1b
```

---

## Étape 6 : Tester le partage de fichiers

### Connexion aux instances

**Instance A** → **Connect** (EC2 Instance Connect)  
**Instance B** → **Connect** (EC2 Instance Connect)

---

### Sur Instance A (eu-west-1a)

```bash
# Vérifier que EFS est monté
ls /mnt/efs/fs1/
# → Vide pour l'instant

# Élever les privilèges
sudo su

# Créer un fichier
echo "hello world" > /mnt/efs/fs1/hello.txt

# Vérifier
cat /mnt/efs/fs1/hello.txt
# → hello world ✅
```

---

### Sur Instance B (eu-west-1b)

```bash
# Vérifier que EFS est monté
ls /mnt/efs/fs1/
# → hello.txt ✅ (fichier créé depuis Instance A !)

# Lire le fichier
cat /mnt/efs/fs1/hello.txt
# → hello world ✅
```

---

## Résultat : Partage réussi !

```
╔═══════════════════════════════╗    ╔═══════════════════════════════╗
║  Instance A (eu-west-1a)      ║    ║  Instance B (eu-west-1b)      ║
║                               ║    ║                               ║
║  echo "hello" > hello.txt     ║    ║  cat hello.txt                ║
║                               ║    ║  → hello world ✅             ║
╚═══════════════════════════════╝    ╚═══════════════════════════════╝
                ↓                                   ↓
                └───────────────┬───────────────────┘
                                ↓
                       [EFS File System]
                       /mnt/efs/fs1/
                         └─ hello.txt

Les 2 instances partagent le MÊME système de fichiers
Données synchronisées en temps réel ✅
```

---

## Étape 7 : Nettoyage

### 7.1 Terminer les instances EC2

```
Instances → Sélectionner Instance A et B
  → Instance State → Terminate
```

---

### 7.2 Supprimer le système de fichiers EFS

```
EFS → Sélectionner le file system
  → Delete
  → Entrer le File System ID pour confirmer
  → Delete
```

---

### 7.3 Supprimer les Security Groups

```
EC2 → Security Groups
  ├─ sg-efs-demo → Delete
  ├─ efs-sg-1 → Delete
  ├─ efs-sg-2 → Delete
  └─ launch-wizard-1, launch-wizard-2 → Delete
```

⚠️ **Attendre que les instances et EFS soient complètement supprimés avant de supprimer les SG.**

---

## Points clés de la démo

✅ **Regional EFS** = Multi-AZ, haute disponibilité  
✅ **Lifecycle policies** = Économies automatiques (IA, Archive)  
✅ **Elastic Throughput** = Scale automatique, recommandé  
✅ **Security Groups** = Essentiels pour autoriser NFS (port 2049)  
✅ **Auto-configuration EC2** = AWS crée/attache les SG automatiquement  
✅ **Partage temps réel** = Fichier créé sur A visible instantanément sur B  
✅ **Multi-AZ** = Instances dans différentes AZ accèdent au même FS  
✅ **Mount point** = `/mnt/efs/fs1` (personnalisable)  
✅ **Pay-per-use** = 6 KB utilisés → coût ~0

---

## Architecture finale

```
╔═══════════════════════════════════════════════════════════╗
║                     REGION : eu-west-1                    ║
║                                                            ║
║  ╔════════════════════╗         ╔════════════════════╗   ║
║  ║  AZ : eu-west-1a   ║         ║  AZ : eu-west-1b   ║   ║
║  ║                    ║         ║                    ║   ║
║  ║  [Instance A]      ║         ║  [Instance B]      ║   ║
║  ║   /mnt/efs/fs1 ←───╫─────────╫───→ /mnt/efs/fs1  ║   ║
║  ║                    ║         ║                    ║   ║
║  ╚════════════════════╝         ╚════════════════════╝   ║
║           ↓                              ↓                ║
║           └──────────────┬───────────────┘                ║
║                          ↓                                ║
║                 [EFS File System]                         ║
║                   Regional (Multi-AZ)                     ║
║                   Elastic Throughput                      ║
║                   General Purpose                         ║
║                                                            ║
╚═══════════════════════════════════════════════════════════╝
```

---

## Différence clé avec EBS Multi-Attach

| Critère               | EBS Multi-Attach     | EFS                |
| --------------------- | -------------------- | ------------------ |
| **Volumes supportés** | io1/io2 uniquement   | Tous (c'est un FS) |
| **Max instances**     | 16                   | Milliers           |
| **Multi-AZ**          | ❌ Non (même AZ)     | ✅ Oui             |
| **File system**       | Cluster-aware requis | POSIX standard     |
| **Configuration**     | Complexe             | Simple ✅          |
| **Prix**              | io2 cher             | EFS plus cher      |

**Pour partage multi-AZ simple → EFS est le meilleur choix.**

---

## Pour l'examen AWS

**Question typique** : "Vous avez des instances dans plusieurs AZ qui doivent partager des fichiers. Quelle solution ?"

- ❌ "EBS Multi-Attach"
- ❌ "Instance Store partagé"
- ✅ "EFS (Elastic File System)"

**Mots-clés** : "shared files", "multi-AZ", "Linux", "NFS" = **EFS**
