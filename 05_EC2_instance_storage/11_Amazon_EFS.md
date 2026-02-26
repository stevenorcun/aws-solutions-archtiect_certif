# AWS – EFS (Elastic File System)

## Qu'est-ce qu'EFS ?

**EFS** = **Elastic File System**

Un **système de fichiers réseau (NFS)** géré par AWS qui peut être **monté sur plusieurs instances EC2 simultanément**, même dans **différentes AZ**.

---

## Différence clé : EBS vs EFS

### EBS (Block Storage)

```
╔═══════════════════════════════╗
║  AZ : us-east-1a              ║
║                               ║
║  [Instance EC2 #1]            ║
║       ↕                       ║
║  [EBS Volume]                 ║
║                               ║
║  ❌ Instance #2 ne peut pas   ║
║     utiliser le même volume   ║
║                               ║
╚═══════════════════════════════╝

1 volume EBS = 1 instance EC2 (au niveau CCP)
```

---

### EFS (Network File System)

```
╔═══════════════════════╗  ╔═══════════════════════╗  ╔═══════════════════════╗
║  AZ : us-east-1a      ║  ║  AZ : us-east-1b      ║  ║  AZ : us-east-1c      ║
║                       ║  ║                       ║  ║                       ║
║  [Instance #1] ───┐   ║  ║  [Instance #3] ───┐   ║  ║  [Instance #5] ───┐   ║
║                   │   ║  ║                   │   ║  ║                   │   ║
║  [Instance #2] ───┤   ║  ║  [Instance #4] ───┤   ║  ║  [Instance #6] ───┤   ║
║                   │   ║  ║                   │   ║  ║                   │   ║
╚═══════════════════┼═══╝  ╚═══════════════════┼═══╝  ╚═══════════════════┼═══╝
                    │                          │                          │
                    └──────────────┬───────────┴──────────────────────────┘
                                   ↓
                          [EFS File System]
                          (Security Group)

1 système de fichiers EFS = N instances EC2 (multi-AZ ✅)
```

---

## Caractéristiques principales

| Caractéristique    | Détail                               |
| ------------------ | ------------------------------------ |
| **Type**           | Network File System (NFS) managé     |
| **Multi-AZ**       | ✅ Oui (haute disponibilité)         |
| **Multi-instance** | ✅ Milliers d'instances simultanées  |
| **Scalabilité**    | Automatique (pas de provisionnement) |
| **OS supportés**   | ✅ Linux uniquement (AMI Linux)      |
|                    | ❌ Windows NON supporté              |
| **Protocole**      | NFSv4.1                              |
| **Encryption**     | ✅ At rest (KMS) + in transit        |
| **Prix**           | ~3× plus cher que gp2 EBS            |
| **Facturation**    | Pay-per-use (par GB utilisé)         |

---

## Cas d'usage

### ✅ Quand utiliser EFS

| Cas d'usage                   | Pourquoi EFS                         |
| ----------------------------- | ------------------------------------ |
| **Content Management System** | Fichiers partagés entre serveurs web |
| **WordPress**                 | Plugins, thèmes, uploads partagés    |
| **Web serving**               | Assets statiques partagés            |
| **Data sharing**              | Fichiers communs entre applications  |
| **Home directories**          | Dossiers utilisateurs partagés       |
| **Dev/test environments**     | Code source partagé entre devs       |

---

### Exemple concret : WordPress multi-serveurs

```
╔═══════════════════════════════════════════════════════╗
║                [Load Balancer]                        ║
║                       ↓                               ║
║         ┌─────────────┼─────────────┐                ║
║         ↓                           ↓                 ║
║  ╔════════════════╗          ╔════════════════╗      ║
║  ║ Web Server 1   ║          ║ Web Server 2   ║      ║
║  ║ (us-east-1a)   ║          ║ (us-east-1b)   ║      ║
║  ╚════════════════╝          ╚════════════════╝      ║
║         ↓                           ↓                 ║
║         └─────────────┬─────────────┘                ║
║                       ↓                               ║
║              [EFS File System]                        ║
║                /var/www/html                          ║
║                ├─ wp-content/                         ║
║                │   ├─ uploads/ (images partagées)    ║
║                │   ├─ themes/  (thèmes partagés)     ║
║                │   └─ plugins/ (plugins partagés)    ║
║                                                       ║
╚═══════════════════════════════════════════════════════╝

✅ Si Server 1 tombe, Server 2 continue avec les mêmes fichiers
✅ Upload sur Server 1 visible instantanément sur Server 2
```

---

## Sécurité : Security Group obligatoire

### Configuration réseau

```
[EFS File System]
  └─ Security Group: efs-sg
      Inbound rules:
        ├─ NFS (Port 2049) depuis sg-web-servers ✅
        └─ Tout le reste bloqué

[EC2 Instances]
  └─ Security Group: sg-web-servers
      Outbound rules:
        └─ NFS (Port 2049) vers efs-sg ✅
```

**Sans Security Group correctement configuré → instances ne peuvent pas monter EFS.**

---

## Scalabilité automatique

### Pas de provisionnement de capacité

```
EBS
───
Créer volume 100 GB
  └─ Taille FIXE (sauf si vous l'augmentez manuellement)


EFS
───
Créer file system
  └─ Taille AUTOMATIQUE (croît/décroît selon utilisation)

Aujourd'hui : 10 GB → Payez 10 GB
Demain : 500 GB → Payez 500 GB
Après-demain : 50 GB → Payez 50 GB
```

**Facturation : Pay-per-use uniquement pour ce que vous utilisez réellement.**

---

## Performance

### Capacité

| Métrique                   | Valeur                       |
| -------------------------- | ---------------------------- |
| **Clients NFS simultanés** | Milliers                     |
| **Throughput**             | 10+ GB/s                     |
| **Taille maximale**        | Petabyte scale (automatique) |

---

### Performance Mode (choisi à la création)

| Mode                         | Latence     | Throughput | Cas d'usage                                  |
| ---------------------------- | ----------- | ---------- | -------------------------------------------- |
| **General Purpose** (défaut) | Faible      | Standard   | Web servers, CMS, applications générales     |
| **Max I/O**                  | Plus élevée | Très élevé | Big Data, media processing, parallèle massif |

⚠️ **Décision à la création, non modifiable après.**

---

### Throughput Mode

| Mode                     | Fonctionnement                           | Cas d'usage                                     |
| ------------------------ | ---------------------------------------- | ----------------------------------------------- |
| **Bursting**             | Throughput croît avec la taille          | Petits/moyens volumes (< 1 TB)                  |
| **Provisioned**          | Throughput fixe indépendant de la taille | Besoin de throughput élevé même si petit volume |
| **Elastic** (recommandé) | Scale automatiquement selon workload     | Workloads imprévisibles                         |

---

#### Bursting Mode

```
Taille EFS : 1 TB
  └─ Throughput : 50 MB/s baseline
  └─ Burst : jusqu'à 100 MB/s

Taille EFS : 5 TB
  └─ Throughput : 250 MB/s baseline
  └─ Burst : jusqu'à 500 MB/s
```

**Formule** : 50 MB/s par TB + burst

---

#### Provisioned Mode

```
Taille EFS : 100 GB
  └─ Throughput provisionné : 1 GB/s ✅

Taille et throughput DÉCORRÉLÉS
```

**Cas d'usage** : Petit volume mais besoin de performance élevée.

---

#### Elastic Mode (nouveau, recommandé)

```
Workload léger → 100 MB/s automatiquement
Workload lourd → Scale jusqu'à 3 GB/s (lecture) / 1 GB/s (écriture)

Pas de configuration, scaling automatique ✅
```

---

## Storage Classes (Tiers de stockage)

### Standard vs Infrequent Access vs Archive

| Storage Class              | Accès                        | Prix stockage | Prix récupération | Cas d'usage          |
| -------------------------- | ---------------------------- | ------------- | ----------------- | -------------------- |
| **Standard**               | Fréquent                     | $$$           | Gratuit           | Fichiers actifs      |
| **IA** (Infrequent Access) | Rare (< 1×/mois)             | $             | Par GB récupéré   | Anciens fichiers     |
| **Archive**                | Très rare (quelques fois/an) | € (min)       | Par GB récupéré   | Conformité, archives |

---

### Lifecycle Management

**Déplacer automatiquement les fichiers entre tiers selon leur âge.**

```
Configuration Lifecycle Policy
──────────────────────────────
Transition to IA : après 30 jours sans accès
Transition to Archive : après 90 jours sans accès


Workflow
────────
Jour 0 : fichier.txt créé → EFS Standard
Jour 30 : pas accédé → Déplacement automatique vers EFS-IA ✅
Jour 90 : toujours pas accédé → Déplacement vers Archive ✅

Si accès à Jour 45 :
  └─ Fichier reste en IA (compteur remis à zéro)
```

---

### Exemple de politique

```
[EFS File System]
  └─ Lifecycle Policy:
      ├─ Move to IA after 60 days
      └─ Move to Archive after 180 days

/data/reports/
  ├─ report-2025-02.pdf (accédé hier) → Standard
  ├─ report-2024-12.pdf (70 jours) → IA
  └─ report-2023-08.pdf (200 jours) → Archive
```

---

## Availability & Durability

### Standard (Multi-AZ)

```
╔════════════╗  ╔════════════╗  ╔════════════╗
║  AZ-1a     ║  ║  AZ-1b     ║  ║  AZ-1c     ║
║            ║  ║            ║  ║            ║
║  [Data]    ║  ║  [Data]    ║  ║  [Data]    ║
║  Replica   ║  ║  Replica   ║  ║  Replica   ║
║            ║  ║            ║  ║            ║
╚════════════╝  ╚════════════╝  ╚════════════╝

✅ Résiste à la panne d'une AZ entière
✅ Recommandé pour PRODUCTION
```

**Prix** : Standard

---

### One Zone (Single AZ)

```
╔════════════╗
║  AZ-1a     ║
║            ║
║  [Data]    ║
║  (unique)  ║
║            ║
╚════════════╝

⚠️ Si AZ tombe → perte d'accès
✅ ~50% moins cher que Multi-AZ
✅ Backups disponibles
✅ Recommandé pour DEV/TEST
```

**Prix** : ~50% moins cher

---

### One Zone-IA (encore moins cher)

```
One Zone + Infrequent Access
  └─ ~90% moins cher que Standard Multi-AZ ✅

Idéal pour :
  ├─ Environnements dev/test
  ├─ Backups rarement accédés
  └─ Données non critiques
```

---

## Comparaison : Coûts

```
Exemple : 1 TB de données

EBS gp3 (1 instance, 1 AZ)
  └─ ~80 $/mois


EFS Standard (Multi-AZ)
  └─ ~300 $/mois (3× EBS)


EFS One Zone
  └─ ~150 $/mois (1.5× EBS)


EFS One Zone-IA
  └─ ~25 $/mois (beaucoup moins cher ✅)
```

**Économies possibles avec bonnes Storage Classes : jusqu'à 90%**

---

## Encryption

### At Rest (chiffrement au repos)

```
EFS File System
  └─ Encryption at rest: ✅ Enabled
      └─ KMS key: aws/elasticfilesystem (ou CMK)
```

**Activé à la création, non modifiable après.**

---

### In Transit (chiffrement en transit)

```
Instance EC2
  ↕ TLS (HTTPS)
EFS Mount Target

Activation lors du montage :
  └─ mount -t efs -o tls fs-abc123:/ /mnt/efs
```

---

## File System Type (POSIX)

EFS utilise le **standard POSIX** :

- Permissions Unix (chmod, chown)
- Symbolic links
- Hard links
- File locking

```bash
# Exemple sur instance montée
cd /mnt/efs
touch file.txt
chmod 644 file.txt
chown ec2-user:ec2-user file.txt

# Visible sur TOUTES les instances montées ✅
```

---

## Tableau comparatif : EBS vs EFS vs Instance Store

| Critère         | EBS              | EFS              | Instance Store      |
| --------------- | ---------------- | ---------------- | ------------------- |
| **Type**        | Block storage    | File system      | Block storage local |
| **Attachement** | 1 instance (CCP) | N instances      | 1 instance (fixe)   |
| **Multi-AZ**    | ❌ Non           | ✅ Oui           | ❌ Non              |
| **Persistance** | ✅ Oui           | ✅ Oui           | ❌ Éphémère         |
| **Performance** | Élevée           | Moyenne          | Très élevée         |
| **Prix**        | €                | €€€              | Inclus              |
| **Scalabilité** | Manuelle         | Automatique      | Fixe                |
| **OS**          | Linux + Windows  | Linux uniquement | Linux + Windows     |

---

## Points clés à retenir

✅ **EFS** = Network File System (NFS) managé  
✅ **Multi-AZ** = Haute disponibilité  
✅ **Multi-instance** = Milliers d'instances simultanées  
✅ **Linux uniquement** = Pas de support Windows  
✅ **Pay-per-use** = Pas de provisionnement de capacité  
✅ **3× plus cher** que EBS gp2  
✅ **Storage Classes** = Standard, IA, Archive (économies jusqu'à 90%)  
✅ **Lifecycle Policies** = Déplacement automatique entre tiers  
✅ **Performance Modes** = General Purpose (défaut) vs Max I/O  
✅ **Throughput Modes** = Bursting, Provisioned, Elastic  
✅ **One Zone** = ~50% moins cher (dev/test)  
✅ **Encryption** = KMS (at rest) + TLS (in transit)

---

## Pour l'examen AWS

**Question typique** : "Vous avez besoin de partager des fichiers entre plusieurs instances EC2 dans différentes AZ. Quelle solution ?"

- ❌ "EBS avec Multi-Attach"
- ❌ "Instance Store partagé"
- ✅ "EFS (Elastic File System)"

**Question typique 2** : "Vous voulez le stockage partagé le moins cher possible pour environnement dev. Quelle option EFS ?"

- ❌ "EFS Standard Multi-AZ"
- ❌ "EFS Standard One Zone"
- ✅ "EFS One Zone-IA"

**Mot-clé de l'examen** : "shared file system", "multi-AZ", "Linux", "NFS" = **EFS**
