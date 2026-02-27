# AWS – EBS vs EFS vs Instance Store : Comparaison complète

## Tableau comparatif global

| Critère          | EBS                            | EFS                       | Instance Store                        |
| ---------------- | ------------------------------ | ------------------------- | ------------------------------------- |
| **Type**         | Block storage (réseau)         | Network File System       | Block storage (local)                 |
| **Attachement**  | 1 instance (sauf Multi-Attach) | Centaines d'instances     | 1 instance (fixe)                     |
| **Multi-AZ**     | ❌ Non (lié à 1 AZ)            | ✅ Oui (Regional)         | ❌ Non (lié au serveur)               |
| **Persistance**  | ✅ Données persistent          | ✅ Données persistent     | ❌ Éphémère (perdu si stop/terminate) |
| **OS supportés** | Linux + Windows                | Linux uniquement          | Linux + Windows                       |
| **Performance**  | Élevée (io2: 256k IOPS)        | Moyenne (10+ GB/s)        | Très élevée (3.3M IOPS)               |
| **Scalabilité**  | Manuelle (resize)              | Automatique               | Fixe (taille du disque physique)      |
| **Prix**         | €€ (0.08-0.125 $/GB)           | €€€ (0.30 $/GB)           | Inclus dans instance                  |
| **Backup**       | Snapshots EBS                  | AWS Backup                | Votre responsabilité                  |
| **Cas d'usage**  | Boot volumes, DB               | Partage fichiers multi-AZ | Cache, buffer, scratch                |

---

## 1️⃣ EBS (Elastic Block Store)

### Caractéristiques clés

```
✅ Attaché à 1 instance à la fois (défaut)
✅ Lié à 1 AZ spécifique
✅ Persistant (données survivent)
✅ Snapshots pour backup/migration
⚠️ Backups consomment des IO
```

### Architecture typique

```
╔═══════════════════════════════╗
║  AZ : us-east-1a              ║
║                               ║
║  [Instance EC2 #1]            ║
║       ↕                       ║
║  [EBS Volume 100 GB]          ║
║                               ║
╚═══════════════════════════════╝

╔═══════════════════════════════╗
║  AZ : us-east-1b              ║
║                               ║
║  [Instance EC2 #2]            ║
║       ❌                      ║
║  Ne peut PAS utiliser         ║
║  le volume de AZ-1a           ║
║                               ║
╚═══════════════════════════════╝
```

---

### Performance selon le type

#### gp2 (IO liée à la taille)

```
100 GB → 300 IOPS
500 GB → 1,500 IOPS
1 TB → 3,000 IOPS
5.334 TB → 16,000 IOPS (max)

IO augmente SI vous augmentez la taille
```

#### gp3, io1, io2 (IO indépendante)

```
100 GB → 3,000 IOPS (gp3) ou 10,000 IOPS (io1)
1 TB → 3,000 IOPS (gp3) ou 50,000 IOPS (io1)

IO indépendante de la taille ✅
```

---

### Migration entre AZ (via Snapshot)

```
ÉTAPE 1 : SNAPSHOT
──────────────────
[Volume en us-east-1a]
  ↓ Create Snapshot
[Snapshot EBS]


ÉTAPE 2 : RESTAURATION
──────────────────────
[Snapshot EBS]
  ↓ Create Volume in us-east-1b
[Volume en us-east-1b] ✅

C'est la SEULE façon de déplacer un volume entre AZ
```

---

### ⚠️ Impact des Backups (Snapshots)

```
Snapshot en cours
  ↓
Consomme des IO du volume
  ↓
⚠️ Performance réduite pendant le snapshot

Recommandation :
  └─ Faire les snapshots pendant les heures creuses
  └─ Ou sur un volume de backup (pas le volume principal)
```

**Exemple :**

```
Volume de production avec DB
  ├─ Trafic élevé : 10:00 - 18:00
  └─ Snapshot recommandé : 02:00 (faible trafic)
```

---

### Delete on Termination

```
[Instance EC2]
  ├─ Root EBS (OS) : Delete on Termination = ✅ Yes (défaut)
  └─ Data EBS : Delete on Termination = ❌ No (défaut)

Terminaison de l'instance :
  ├─ Root volume → ❌ Supprimé
  └─ Data volume → ✅ Conservé

Modifiable : Décocher "Delete on Termination" pour conserver le root
```

---

## 2️⃣ EFS (Elastic File System)

### Caractéristiques clés

```
✅ Network File System (NFS)
✅ Multi-instance (centaines simultanées)
✅ Multi-AZ (Regional)
✅ Scalabilité automatique
✅ Pay-per-use (pas de provisionnement)
❌ Linux uniquement (POSIX)
```

### Architecture typique

```
╔════════════════════╗  ╔════════════════════╗  ╔════════════════════╗
║  AZ : us-east-1a   ║  ║  AZ : us-east-1b   ║  ║  AZ : us-east-1c   ║
║                    ║  ║                    ║  ║                    ║
║  [EC2 #1] ───┐     ║  ║  [EC2 #3] ───┐     ║  ║  [EC2 #5] ───┐     ║
║  [EC2 #2] ───┤     ║  ║  [EC2 #4] ───┤     ║  ║  [EC2 #6] ───┤     ║
║              │     ║  ║              │     ║  ║              │     ║
╚══════════════┼═════╝  ╚══════════════┼═════╝  ╚══════════════┼═════╝
               │                       │                       │
               └───────────┬───────────┴───────────────────────┘
                           ↓
                  [EFS File System]
                   Regional (Multi-AZ)

Toutes les instances partagent le MÊME système de fichiers ✅
```

---

### Différence clé avec EBS

```
EBS
───
Volume en us-east-1a
  └─ Accessible UNIQUEMENT depuis us-east-1a ❌


EFS
───
File System Regional
  ├─ Mount Target en us-east-1a → Instances en 1a ✅
  ├─ Mount Target en us-east-1b → Instances en 1b ✅
  └─ Mount Target en us-east-1c → Instances en 1c ✅

TOUS partagent les mêmes données ✅
```

---

### Prix : EFS vs EBS

```
Exemple : 1 TB de données

EBS gp3 (1 instance, 1 AZ)
  └─ ~80 $/mois


EFS Standard (Multi-AZ)
  └─ ~300 $/mois (3-4× plus cher que EBS)


MAIS avec Storage Tiers :
  ├─ EFS-IA : ~25 $/mois
  └─ EFS Archive : ~12 $/mois

Économies possibles : jusqu'à 90% ✅
```

---

### Cas d'usage typique : WordPress

```
╔═══════════════════════════════════════════════════════╗
║              [Application Load Balancer]              ║
║                       ↓                               ║
║         ┌─────────────┼─────────────┐                ║
║         ↓                           ↓                 ║
║  [Web Server 1]              [Web Server 2]          ║
║  (us-east-1a)                (us-east-1b)            ║
║         ↓                           ↓                 ║
║         └─────────────┬─────────────┘                ║
║                       ↓                               ║
║              [EFS File System]                        ║
║                /var/www/html                          ║
║                ├─ wp-content/uploads/ (partagé)      ║
║                ├─ themes/ (partagé)                  ║
║                └─ plugins/ (partagé)                 ║
║                                                       ║
╚═══════════════════════════════════════════════════════╝

✅ Upload sur Server 1 → visible sur Server 2 immédiatement
✅ Si Server 1 tombe → Server 2 continue avec les mêmes fichiers
```

---

## 3️⃣ Instance Store

### Caractéristiques clés

```
✅ Disque physique LOCAL (attaché au serveur physique)
✅ Performance MAXIMALE (jusqu'à 3.3M IOPS)
❌ Éphémère (perdu si stop/terminate/panne serveur)
✅ Inclus dans prix de l'instance (pas de surcoût)
✅ Reboot OK (données conservées)
```

### Architecture

```
[Serveur Physique AWS]
  ├─ Disque NVMe SSD (Instance Store)
  │       ↕ Connexion physique directe
  └─ [Instance EC2 virtuelle]

Si serveur physique meurt → disque perdu ❌
Si vous stop l'instance → disque effacé ❌
Si vous reboot → disque conservé ✅
```

---

### Cas d'usage

```
✅ Cache (Redis, Memcached)
✅ Buffer temporaire
✅ Scratch data
✅ Big Data temporaire (Hadoop, Spark)
✅ Logs avant envoi vers S3

❌ Base de données
❌ Fichiers utilisateurs
❌ Toute donnée critique
```

---

## Comparaison détaillée : Scénarios d'usage

### Scénario 1 : Boot volume (OS)

| Solution           | Compatible ?                | Recommandation               |
| ------------------ | --------------------------- | ---------------------------- |
| **EBS**            | ✅ Oui (gp2, gp3, io1, io2) | ✅ Choix standard            |
| **EFS**            | ❌ Non                      | N/A                          |
| **Instance Store** | ⚠️ Possible mais rare       | ❌ Pas recommandé (éphémère) |

**Réponse : EBS gp3** ✅

---

### Scénario 2 : Base de données critique

| Solution           | Adapté ?                     | Raison                                    |
| ------------------ | ---------------------------- | ----------------------------------------- |
| **EBS io2**        | ✅✅✅ Excellent             | Performance élevée, persistant, snapshots |
| **EFS**            | ⚠️ Possible mais non optimal | Performance moyenne, coût élevé           |
| **Instance Store** | ❌ NON                       | Perte de données = désastre               |

**Réponse : EBS io2** ✅

---

### Scénario 3 : Partage de fichiers entre 10 instances dans 3 AZ

| Solution           | Adapté ?       | Raison                              |
| ------------------ | -------------- | ----------------------------------- |
| **EBS**            | ❌ NON         | 1 volume = 1 instance, lié à 1 AZ   |
| **EFS**            | ✅✅✅ Parfait | Multi-instance, multi-AZ nativement |
| **Instance Store** | ❌ NON         | Local, pas de partage               |

**Réponse : EFS** ✅

---

### Scénario 4 : Cache Redis avec performance maximale

| Solution           | Adapté ?         | Raison                          |
| ------------------ | ---------------- | ------------------------------- |
| **EBS io2**        | ✅ Bon           | 256k IOPS, persistant           |
| **EFS**            | ❌ NON           | Performance insuffisante        |
| **Instance Store** | ✅✅✅ Excellent | 3.3M IOPS, cache OK si éphémère |

**Réponse : Instance Store ou EBS io2** (selon besoin de persistance)

---

### Scénario 5 : Environnement dev/test partagé entre équipe

| Solution           | Adapté ?                  | Raison                         |
| ------------------ | ------------------------- | ------------------------------ |
| **EBS**            | ⚠️ Possible mais complexe | Pas de partage natif           |
| **EFS**            | ✅✅✅ Parfait            | Partage simple, multi-instance |
| **Instance Store** | ❌ NON                    | Pas de partage, éphémère       |

**Réponse : EFS** ✅

---

## Tableau décisionnel rapide

```
BESOIN DE BOOT VOLUME ?
  └─ OUI → EBS gp3

BESOIN DE PARTAGE MULTI-AZ ?
  └─ OUI → EFS

BESOIN DE PERFORMANCE MAXIMALE + DONNÉES ÉPHÉMÈRES OK ?
  └─ OUI → Instance Store

BASE DE DONNÉES CRITIQUE ?
  └─ OUI → EBS io1/io2

DONNÉES TEMPORAIRES (CACHE, BUFFER) ?
  └─ OUI → Instance Store

PARTAGE FICHIERS LINUX ENTRE INSTANCES ?
  └─ OUI → EFS

WINDOWS + PARTAGE FICHIERS ?
  └─ OUI → FSx for Windows (pas EFS)

DÉFAUT GÉNÉRAL ?
  └─ EBS gp3
```

---

## Points clés à retenir

### EBS

✅ 1 volume = 1 instance (sauf Multi-Attach io1/io2)  
✅ Lié à 1 AZ (migration via snapshot)  
✅ Performance variable (gp2 vs gp3 vs io2)  
✅ Snapshots = backups (consomment IO)  
✅ Root volume supprimé par défaut à terminaison

### EFS

✅ Multi-instance, multi-AZ  
✅ Linux uniquement (POSIX)  
✅ 3× plus cher que EBS (mais Storage Tiers = économies)  
✅ Scalabilité automatique  
✅ Parfait pour partage de fichiers

### Instance Store

✅ Performance maximale (3.3M IOPS)  
❌ Éphémère (perdu si stop/terminate)  
✅ Gratuit (inclus dans instance)  
✅ Cache, buffer, scratch uniquement

---

## Pour l'examen AWS

**Question typique 1** : "Vous avez besoin de partager des fichiers entre instances dans plusieurs AZ. Quelle solution ?"

- ❌ EBS Multi-Attach
- ✅ **EFS**

**Question typique 2** : "Vous avez une DB critique nécessitant 50,000 IOPS. Quelle solution ?"

- ❌ EFS
- ❌ Instance Store
- ✅ **EBS io2 + Instance Nitro**

**Question typique 3** : "Vous avez besoin de performance maximale pour un cache temporaire. Quelle solution ?"

- ❌ EFS
- ❌ EBS gp3
- ✅ **Instance Store**

**Mots-clés examen :**

- **"multi-AZ sharing"** = EFS
- **"highest IOPS"** + **"ephemeral OK"** = Instance Store
- **"database critical"** = EBS io1/io2
- **"boot volume"** = EBS gp3
- **"WordPress multi-server"** = EFS
