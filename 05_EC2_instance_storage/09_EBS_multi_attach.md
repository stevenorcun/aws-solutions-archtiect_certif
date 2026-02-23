# AWS – EBS Multi-Attach

## Qu'est-ce que Multi-Attach ?

Une fonctionnalité qui permet d'attacher **le même volume EBS** à **plusieurs instances EC2 simultanément** dans la même AZ.

---

## Disponibilité

⚠️ **Uniquement pour les volumes io1 et io2 (Provisioned IOPS SSD)**

| Type de volume        | Multi-Attach supporté ? |
| --------------------- | ----------------------- |
| **io1**               | ✅ Oui                  |
| **io2 Block Express** | ✅ Oui                  |
| gp2                   | ❌ Non                  |
| gp3                   | ❌ Non                  |
| st1                   | ❌ Non                  |
| sc1                   | ❌ Non                  |

---

## Architecture

### Sans Multi-Attach (comportement classique)

```
╔═══════════════════════════════╗
║  AZ : us-east-1a              ║
║                               ║
║  [Instance EC2 #1]            ║
║       ↕                       ║
║  [EBS Volume gp3]             ║
║                               ║
║  [Instance EC2 #2]            ║
║       ↕                       ║
║  [EBS Volume gp3] (différent) ║
║                               ║
╚═══════════════════════════════╝

❌ Un volume = Une instance maximum
```

---

### Avec Multi-Attach (io1/io2 uniquement)

```
╔═══════════════════════════════════════════════╗
║  AZ : us-east-1a                              ║
║                                               ║
║  [Instance EC2 #1] ←─┐                       ║
║                      │                        ║
║  [Instance EC2 #2] ←─┼─→ [EBS io2 Volume]    ║
║                      │    (Multi-Attach ON)   ║
║  [Instance EC2 #3] ←─┘                       ║
║                                               ║
║  ...jusqu'à 16 instances                      ║
║                                               ║
╚═══════════════════════════════════════════════╝

✅ Un volume = Jusqu'à 16 instances simultanément
```

---

## Caractéristiques clés

| Caractéristique       | Détail                                   |
| --------------------- | ---------------------------------------- |
| **Volumes supportés** | io1, io2 Block Express uniquement        |
| **Max instances**     | **16 instances** simultanément           |
| **Permissions**       | Full read/write sur toutes les instances |
| **Scope**             | Même AZ uniquement (pas cross-AZ)        |
| **File system**       | Doit être cluster-aware (pas XFS/EXT4)   |

---

## Permissions et accès concurrent

### Toutes les instances ont un accès complet

```
[Instance #1]
  ├─ Read → ✅ Autorisé
  └─ Write → ✅ Autorisé

[Instance #2]
  ├─ Read → ✅ Autorisé
  └─ Write → ✅ Autorisé

[Instance #3]
  ├─ Read → ✅ Autorisé
  └─ Write → ✅ Autorisé

Toutes peuvent lire/écrire SIMULTANÉMENT
```

---

## ⚠️ Limitation importante : Même AZ uniquement

```
╔══════════════════════╗    ╔══════════════════════╗
║  AZ : us-east-1a     ║    ║  AZ : us-east-1b     ║
║                      ║    ║                      ║
║  [Instance #1] ←─┐   ║    ║  [Instance #3]       ║
║                  │   ║    ║       ↕              ║
║  [Instance #2] ←─┤   ║    ║  ❌ Impossible       ║
║                  │   ║    ║                      ║
║  [io2 Volume] ───┘   ║    ║                      ║
║                      ║    ║                      ║
╚══════════════════════╝    ╚══════════════════════╝

✅ Multi-Attach fonctionne dans la MÊME AZ
❌ Impossible d'attacher à une instance dans une autre AZ
```

---

## Cas d'usage

### 1. Application en cluster (Haute disponibilité)

**Exemple : Teradata (base de données MPP)**

```
[Teradata Cluster]
  ├─ Node 1 (Instance EC2)
  ├─ Node 2 (Instance EC2)
  ├─ Node 3 (Instance EC2)
  └─ Node 4 (Instance EC2)
       ↕ Tous attachés à
  [io2 Volume 10 TB - Multi-Attach]
    └─ Données partagées du cluster
```

**Pourquoi ?**

- Les 4 nœuds doivent **lire/écrire** les mêmes données simultanément
- Si un nœud tombe, les autres continuent (HA)
- Performance élevée (io2 + accès parallèle)

---

### 2. Application clustered Linux

**Exemple : Oracle RAC (Real Application Cluster)**

```
[Oracle RAC Cluster]
  ├─ Oracle Instance 1 (EC2)
  ├─ Oracle Instance 2 (EC2)
  └─ Oracle Instance 3 (EC2)
       ↕
  [io2 Volume - Multi-Attach]
    └─ Fichiers de base de données partagés

Si Instance 1 crash :
  └─ Instances 2 et 3 continuent à servir les requêtes ✅
```

---

### 3. Applications avec écritures concurrentes

**Exemple : Système de fichiers distribué (GFS2, OCFS2)**

```
[Application distribuée]
  ├─ Worker 1 → Écrit bloc A
  ├─ Worker 2 → Écrit bloc B
  └─ Worker 3 → Écrit bloc C
       ↓ Simultanément
  [io2 Volume - Multi-Attach]
    └─ Fichier partagé géré par file system cluster-aware
```

---

## File System : Cluster-Aware obligatoire

### ❌ File Systems standards (NE PAS utiliser)

```
XFS, EXT4, NTFS
  └─ Conçus pour UNE seule instance
  └─ Corruption de données si Multi-Attach activé
```

**Problème** :

```
Instance 1 écrit → Cache local XFS
Instance 2 écrit → Cache local XFS (différent)
  ↓
Conflit ! Corruption des données ❌
```

---

### ✅ File Systems cluster-aware (À utiliser)

| File System        | OS      | Description                        |
| ------------------ | ------- | ---------------------------------- |
| **GFS2**           | Linux   | Global File System 2 (Red Hat)     |
| **OCFS2**          | Linux   | Oracle Cluster File System 2       |
| **Clustered NTFS** | Windows | Windows Server Failover Clustering |

**Ces FS savent gérer les écritures concurrentes via des locks distribués.**

---

## Limites à connaître pour l'examen

| Limite                 | Valeur                                |
| ---------------------- | ------------------------------------- |
| **Max instances**      | **16** (très important pour l'examen) |
| **Scope géographique** | Même AZ uniquement                    |
| **Types de volumes**   | io1, io2 uniquement                   |
| **File system**        | Cluster-aware obligatoire             |

---

## Configuration (étapes simplifiées)

### 1. Créer un volume io2 avec Multi-Attach

```bash
aws ec2 create-volume \
  --volume-type io2 \
  --size 100 \
  --iops 10000 \
  --availability-zone us-east-1a \
  --multi-attach-enabled
```

### 2. Attacher à la première instance

```bash
aws ec2 attach-volume \
  --volume-id vol-0abc123 \
  --instance-id i-instance1 \
  --device /dev/sdf
```

### 3. Attacher à la deuxième instance

```bash
aws ec2 attach-volume \
  --volume-id vol-0abc123 \
  --instance-id i-instance2 \
  --device /dev/sdf
```

### 4. Formater avec FS cluster-aware (sur UNE instance seulement)

```bash
# Sur Instance 1 UNIQUEMENT
sudo mkfs.gfs2 -p lock_dlm -t mycluster:myfs -j 4 /dev/sdf
```

### 5. Monter sur toutes les instances

```bash
# Sur chaque instance
sudo mkdir /mnt/shared
sudo mount -t gfs2 /dev/sdf /mnt/shared
```

---

## Schéma : Workflow complet

```
ÉTAPE 1 : CRÉATION
──────────────────
[Créer volume io2 avec Multi-Attach enabled]
  └─ 100 GB, 10,000 IOPS, us-east-1a


ÉTAPE 2 : ATTACHEMENT
─────────────────────
[Volume io2]
     ↓
Attach → [Instance 1]
Attach → [Instance 2]
Attach → [Instance 3]


ÉTAPE 3 : FORMATAGE (Instance 1 seulement)
───────────────────────────────────────────
sudo mkfs.gfs2 /dev/sdf


ÉTAPE 4 : MONTAGE (toutes les instances)
─────────────────────────────────────────
sudo mount /dev/sdf /mnt/shared


RÉSULTAT
────────
[Instance 1] → /mnt/shared (read/write ✅)
[Instance 2] → /mnt/shared (read/write ✅)
[Instance 3] → /mnt/shared (read/write ✅)

Toutes partagent les mêmes données
```

---

## Comparaison : Multi-Attach vs EFS

**Question courante** : Pourquoi ne pas utiliser EFS (Elastic File System) ?

| Critère           | EBS Multi-Attach   | EFS                         |
| ----------------- | ------------------ | --------------------------- |
| **Type**          | Block storage      | Network file system         |
| **Performance**   | Très élevée (io2)  | Modérée                     |
| **Max instances** | 16                 | Milliers                    |
| **Cross-AZ**      | ❌ Non             | ✅ Oui                      |
| **Cas d'usage**   | Cluster haute perf | Partage de fichiers général |
| **Latence**       | Très faible        | Plus élevée                 |

**Choix** :

- **Multi-Attach** : Performance maximale, cluster dans 1 AZ
- **EFS** : Partage multi-AZ, scalabilité, simplicité

---

## Exemple concret : Oracle RAC sur AWS

### Architecture

```
╔═══════════════════════════════════════════════╗
║  AZ : us-east-1a                              ║
║                                               ║
║  [EC2 - Oracle Node 1]                        ║
║       ↕                                       ║
║  [EC2 - Oracle Node 2] ←→ [io2 Volume 2 TB]   ║
║       ↕                   (Multi-Attach)      ║
║  [EC2 - Oracle Node 3]    - Data files        ║
║                           - Redo logs         ║
║                                               ║
║  [Application Load Balancer]                  ║
║       ↓                                       ║
║  Distribue les requêtes sur les 3 nœuds       ║
║                                               ║
╚═══════════════════════════════════════════════╝
```

**Bénéfices** :

- **Haute disponibilité** : Si Node 1 crash, Node 2 et 3 continuent
- **Performance** : Accès parallèle, io2 avec 50,000 IOPS
- **Scalabilité** : Ajouter des nœuds (jusqu'à 16)

---

## Points clés à retenir

✅ **Multi-Attach** = 1 volume io1/io2 → jusqu'à 16 instances  
✅ **Uniquement io1 et io2** (pas gp2, gp3, st1, sc1)  
✅ **Max 16 instances** simultanément (important pour l'examen)  
✅ **Même AZ uniquement** (pas cross-AZ)  
✅ **Full read/write** pour toutes les instances  
✅ **File system cluster-aware obligatoire** (GFS2, OCFS2...)  
✅ **Cas d'usage** : Clusters HA (Teradata, Oracle RAC)

---

## Pour l'examen AWS

**Question typique** : "Vous avez une application clustered qui nécessite que 10 instances partagent le même volume avec accès read/write. Quelle solution ?"

- ❌ "Utiliser 10 volumes gp3 synchronisés"
- ❌ "Utiliser un volume gp3 avec Multi-Attach"
- ✅ "Utiliser un volume io2 avec Multi-Attach activé"

**Question typique 2** : "Combien d'instances maximum peuvent être attachées simultanément à un volume EBS Multi-Attach ?"

- ❌ "Illimité"
- ❌ "32"
- ✅ "16"

**Mot-clé de l'examen** : "clustered application", "concurrent writes", "high availability within same AZ" = **Multi-Attach io1/io2**
