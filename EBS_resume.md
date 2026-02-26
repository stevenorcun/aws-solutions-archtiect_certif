# RÉSUMÉ : EC2 Instance Storage (EBS → Multi-Attach)

## 🔑 Mots-clés essentiels

**EBS, Snapshot, AMI, Instance Store, gp2/gp3, io1/io2, st1, sc1, Multi-Attach, Encryption, KMS, Network Drive, Ephemeral, Boot Volume, IOPS, Throughput, Delete on Termination, Availability Zone, Cross-Region**

---

## 1️⃣ EBS Volumes (Elastic Block Store)

### Concept clé

**Disque réseau** (network drive) attaché aux instances EC2.

### Points importants

✅ **Persistance** : Données survivent aux arrêts/redémarrages  
✅ **1 volume = 1 instance** (niveau CCP, sauf Multi-Attach)  
✅ **Lié à une AZ** : Volume en us-east-1a → instance en us-east-1a uniquement  
✅ **Capacité provisionnée** : Taille + IOPS définis à l'avance  
✅ **Delete on Termination** :

- Root volume : ✅ Supprimé par défaut
- Data volume : ❌ Conservé par défaut  
  ✅ **Détachable/réattachable** rapidement (failover)

### Analogie

**Clé USB réseau** que vous pouvez débrancher et rebrancher ailleurs.

---

## 2️⃣ EBS Snapshots

### Concept clé

**Sauvegarde à un instant T** d'un volume EBS.

### Points importants

✅ **Backup** : Prendre une "photo" du volume  
✅ **Détacher recommandé** mais pas obligatoire  
✅ **Cross-AZ / Cross-Region** : Copier/restaurer entre AZ/régions  
✅ **Propagation automatique** : Volume chiffré → Snapshot chiffré

### Fonctionnalités avancées

| Fonctionnalité            | Détail                               | Mot-clé examen          |
| ------------------------- | ------------------------------------ | ----------------------- |
| **Archive**               | 75% moins cher, restauration 24-72h  | "lowest cost backup"    |
| **Recycle Bin**           | Protection suppression (1-365 jours) | "accidental deletion"   |
| **Fast Snapshot Restore** | Aucune latence, très cher            | "immediate performance" |

### Workflow clé

```
Volume A (us-east-1a)
  ↓ Create Snapshot
Snapshot
  ↓ Restore in us-east-1b
Volume B (us-east-1b) ✅

→ Seule façon de "déplacer" un volume entre AZ
```

---

## 3️⃣ AMI (Amazon Machine Image)

### Concept clé

**Template personnalisé** d'instance EC2 (OS + apps + config).

### Points importants

✅ **Golden Image** : Pré-installer tout (Apache, MySQL, monitoring...)  
✅ **Boot rapide** : Instance prête en 2-3 min vs 30 min manuel  
✅ **Régional** : AMI liée à une région (copiable cross-region)  
✅ **Types** :

- Public (AWS) : Amazon Linux 2, Ubuntu...
- Custom (vous) : Vos propres configs
- Marketplace : Vendeurs tiers

### Processus création

```
1. Instance EC2 configurée (installer logiciels)
2. Stop instance (recommandé)
3. Create Image (AMI)
4. Launch instances depuis AMI → Rapide ✅
```

### Cas d'usage

- Auto Scaling (lancer 100 instances identiques)
- Golden Image corporate
- Disaster Recovery (AMI cross-region)

---

## 4️⃣ EC2 Instance Store

### Concept clé

**Disque physique local** attaché au serveur physique (pas réseau).

### Points importants

✅ **Performance MAXIMALE** : Jusqu'à 3.3 millions d'IOPS (vs 32k EBS)  
❌ **Éphémère** : Données perdues si stop/terminate/panne serveur  
✅ **Reboot OK** : Données conservées si simple reboot  
✅ **Inclus dans prix** instance (pas de surcoût)  
❌ **Backup = votre responsabilité**

### Cas d'usage

✅ Cache, buffer, scratch data, Big Data temporaire  
❌ Base de données, fichiers critiques

### Analogie

**Disque dur soudé sur carte mère** : ultra-rapide mais non détachable et perdu si PC meurt.

---

## 5️⃣ EBS Volume Types

### 6 types de volumes

| Type       | Techno | IOPS max | Boot ? | Prix    | Cas d'usage                    |
| ---------- | ------ | -------- | ------ | ------- | ------------------------------ |
| **gp3**    | SSD    | 16,000   | ✅     | €€      | Général, dev/test (recommandé) |
| **gp2**    | SSD    | 16,000   | ✅     | €€€     | Legacy (IOPS liées à taille)   |
| **io1**    | SSD    | 64,000   | ✅     | €€€€    | DB critique                    |
| **io2 BE** | SSD    | 256,000  | ✅     | €€€€€   | DB ultra-critique              |
| **st1**    | HDD    | 500      | ❌     | €       | Big Data, throughput           |
| **sc1**    | HDD    | 250      | ❌     | € (min) | Archive, lowest cost           |

---

### Points clés par type

#### gp3 (General Purpose SSD - Nouveau)

- IOPS et throughput **indépendants** de la taille
- 3,000 IOPS base → 16,000 max
- 125 MB/s → 1,000 MB/s max
- **Meilleur rapport qualité/prix**

#### gp2 (General Purpose SSD - Legacy)

- IOPS **liées à la taille** : 3 IOPS × GB
- 100 GB → 300 IOPS
- 5,334 GB → 16,000 IOPS (max)

#### io1/io2 (Provisioned IOPS SSD)

- DB critiques, > 16,000 IOPS
- IOPS provisionnées indépendamment
- io2 BE : jusqu'à 256,000 IOPS
- **Seuls à supporter Multi-Attach**

#### st1 (Throughput Optimized HDD)

- Big Data, log processing
- Max 500 MB/s throughput
- ❌ Pas de boot volume

#### sc1 (Cold HDD)

- Archive, données rarement accédées
- **Le moins cher**
- ❌ Pas de boot volume

---

### Règle de sélection (Examen)

```
Boot volume (OS) ?
  └─ OUI → gp3, gp2, io1, io2

DB critique sensible aux perfs ?
  └─ OUI → io1 ou io2

Besoin > 16,000 IOPS ?
  └─ OUI → io1 (64k) ou io2 (256k) + Instance Nitro

Big Data / Throughput élevé ?
  └─ OUI → st1 (HDD)

Archive / Lowest cost ?
  └─ OUI → sc1 (HDD)

Défaut général ?
  └─ gp3
```

---

## 6️⃣ EBS Multi-Attach

### Concept clé

**1 volume io1/io2** attaché à **jusqu'à 16 instances** simultanément (même AZ).

### Points importants

✅ **Volumes supportés** : io1, io2 uniquement  
✅ **Max instances** : 16 simultanées  
✅ **Full read/write** pour toutes  
❌ **Même AZ uniquement** (pas cross-AZ)  
✅ **File system cluster-aware obligatoire** (GFS2, OCFS2, pas XFS/EXT4)

### Cas d'usage

- Applications clustered (Teradata, Oracle RAC)
- Haute disponibilité
- Écritures concurrentes

### Schéma clé

```
[Instance 1] ←─┐
[Instance 2] ←─┼─→ [io2 Volume Multi-Attach]
[Instance 3] ←─┘
```

---

## 7️⃣ EBS Encryption

### Points importants

✅ **At rest + In transit** chiffrés  
✅ **Snapshots chiffrés automatiquement**  
✅ **Transparence totale** : AWS gère tout  
✅ **Impact performance** : Quasi nul  
✅ **Technologie** : AES-256 via KMS  
✅ **Clé par défaut** : `aws/ebs` (gratuit)

### Chiffrer volume existant non chiffré

```
Volume NON chiffré
  ↓ Create Snapshot
Snapshot NON chiffré
  ↓ Copy + Enable Encryption (ou Create Volume + Encrypt)
Volume CHIFFRÉ ✅
```

⚠️ **Impossible de chiffrer directement un volume existant.**

---

## 📊 Tableau Comparatif Global

|                    | EBS          | Snapshot  | AMI       | Instance Store | EFS         |
| ------------------ | ------------ | --------- | --------- | -------------- | ----------- |
| **Type**           | Block        | Backup    | Template  | Block local    | File system |
| **Persistance**    | ✅           | ✅        | ✅        | ❌             | ✅          |
| **Multi-instance** | ❌ (sauf MA) | N/A       | N/A       | ❌             | ✅          |
| **Multi-AZ**       | ❌           | ✅ (copy) | ✅ (copy) | ❌             | ✅          |
| **Performance**    | Élevée       | N/A       | N/A       | Maximale       | Moyenne     |
| **Prix**           | €€           | €         | Snapshots | Inclus         | €€€         |

---

## 🎯 Points CRITIQUES pour l'examen

### EBS Volumes

- Lié à **1 AZ**
- Delete on Termination : Root=Oui, Data=Non
- Détachable/réattachable (failover)

### Snapshots

- **Cross-AZ/Region** = snapshot puis restore
- Archive = 75% moins cher, 24-72h restore
- Recycle Bin = protection suppression

### AMI

- Template pour lancer instances rapidement
- Régional (copiable cross-region)
- Boot rapide (tout préinstallé)

### Instance Store

- **Éphémère** = perdu si stop/terminate
- Performance MAXIMALE (3.3M IOPS)
- Cache, buffer, scratch data uniquement

### Volume Types

- **gp3** = défaut général
- **io1/io2** = DB critique, Multi-Attach
- **st1** = Big Data throughput
- **sc1** = Archive lowest cost

### Multi-Attach

- **io1/io2 uniquement**
- Max **16 instances**
- **Même AZ** uniquement
- FS cluster-aware obligatoire

### Encryption

- Transparence totale
- Snapshot chiffré si volume chiffré
- Migration : Snapshot → Copy/Encrypt → Volume

---

## 🔥 Mots-clés EXAMEN

**Network drive, Availability Zone, Snapshot, Cross-Region, AMI, Golden Image, Ephemeral, Instance Store, IOPS, Throughput, gp3, io2, Multi-Attach, 16 instances, Cluster-aware, Encryption, KMS, Delete on Termination, Boot Volume, Lowest cost = sc1, DB critical = io1/io2, > 32k IOPS = Nitro + io2**

---

## 💡 Décisions rapides (Examen)

**"Déplacer volume entre AZ"** → Snapshot + Restore  
**"DB critique 50k IOPS"** → io1/io2 + Nitro  
**"Archive moins cher"** → sc1  
**"Partager entre instances"** → Multi-Attach (io2) ou EFS  
**"Performance maximale éphémère"** → Instance Store  
**"Boot rapide 100 instances"** → AMI  
**"Chiffrer volume existant"** → Snapshot → Copy/Encrypt  
**"Backup DR cross-region"** → Snapshot copy
