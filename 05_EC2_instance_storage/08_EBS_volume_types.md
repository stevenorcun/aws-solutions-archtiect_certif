# AWS – EBS Volume Types (Types de Volumes)

## Vue d'ensemble des 6 types de volumes EBS

Les volumes EBS se divisent en **2 grandes catégories** :

| Catégorie                   | Types                            | Technologie               |
| --------------------------- | -------------------------------- | ------------------------- |
| **SSD** (Solid State Drive) | gp2, gp3, io1, io2 Block Express | Disque à semi-conducteurs |
| **HDD** (Hard Disk Drive)   | st1, sc1                         | Disque mécanique          |

---

## Critères de sélection d'un volume EBS

Lorsque vous choisissez un volume, vous devez considérer :

| Critère         | Description                                    |
| --------------- | ---------------------------------------------- |
| **Size**        | Taille (1 GB → 16 TB selon le type)            |
| **Throughput**  | Débit (MB/s)                                   |
| **IOPS**        | I/O Operations Per Second (opérations/seconde) |
| **Cas d'usage** | Database, boot, archive...                     |
| **Prix**        | Coût par GB/mois                               |

> 💡 **En cas de doute, consultez toujours la documentation AWS officielle**

---

## ⚠️ Boot Volumes (Volumes de démarrage)

**Seuls certains types peuvent être utilisés comme volume root (OS) :**

| Type                  | Boot Volume ? |
| --------------------- | ------------- |
| **gp2**               | ✅ Oui        |
| **gp3**               | ✅ Oui        |
| **io1**               | ✅ Oui        |
| **io2 Block Express** | ✅ Oui        |
| **st1**               | ❌ Non        |
| **sc1**               | ❌ Non        |

---

## 1. General Purpose SSD (gp2 / gp3)

### Usage

```
✅ Cas d'usage :
  ├─ Boot volumes (OS)
  ├─ Virtual desktops
  ├─ Environnements dev/test
  └─ Applications générales

❌ Ne PAS utiliser pour :
  └─ Bases de données critiques nécessitant > 16,000 IOPS
```

### Caractéristiques communes

- **Technologie** : SSD
- **Taille** : 1 GB → 16 TB
- **Latence** : Faible
- **Prix** : Économique

---

### gp3 (Nouvelle génération - Recommandé)

```
PERFORMANCE DE BASE
───────────────────
3,000 IOPS (garanti)
125 MB/s throughput (garanti)

PERFORMANCE MAXIMALE
────────────────────
Jusqu'à 16,000 IOPS (indépendamment)
Jusqu'à 1,000 MB/s throughput (indépendamment)
```

**Avantage clé** : IOPS et throughput **indépendants** de la taille du volume.

**Exemple :**

```
Volume gp3 de 100 GB
  ├─ IOPS : 3,000 (base) ou jusqu'à 16,000 (si provisionné)
  └─ Throughput : 125 MB/s (base) ou jusqu'à 1,000 MB/s (si provisionné)

Volume gp3 de 1 TB
  ├─ IOPS : IDENTIQUE (3,000 base ou jusqu'à 16,000)
  └─ Throughput : IDENTIQUE (125 MB/s ou jusqu'à 1,000 MB/s)
```

**Prix** : ~0.08 $/GB/mois + coût IOPS/throughput supplémentaires

---

### gp2 (Ancienne génération)

```
PERFORMANCE LIÉE À LA TAILLE
────────────────────────────
3 IOPS par GB (minimum 100 IOPS)
Burst jusqu'à 3,000 IOPS pour petits volumes

PERFORMANCE MAXIMALE
────────────────────
16,000 IOPS maximum
250 MB/s throughput maximum
```

**Caractéristique clé** : IOPS et taille **liées**.

**Formule** : `IOPS = Taille_GB × 3`

**Exemples :**

```
Volume gp2 de 100 GB
  └─ IOPS : 100 × 3 = 300 IOPS (peut burst à 3,000)

Volume gp2 de 1,000 GB (1 TB)
  └─ IOPS : 1,000 × 3 = 3,000 IOPS

Volume gp2 de 5,334 GB
  └─ IOPS : 5,334 × 3 = 16,000 IOPS (maximum atteint)

Volume gp2 de 10 TB
  └─ IOPS : toujours plafonné à 16,000 IOPS
```

**Prix** : ~0.10 $/GB/mois

---

### Comparaison gp2 vs gp3

| Critère                    | gp2                  | gp3             |
| -------------------------- | -------------------- | --------------- |
| **IOPS base**              | 3 × taille (min 100) | 3,000 (fixe)    |
| **IOPS max**               | 16,000               | 16,000          |
| **Throughput max**         | 250 MB/s             | 1,000 MB/s      |
| **Indépendance IOPS/Size** | ❌ Non (liés)        | ✅ Oui          |
| **Prix**                   | Plus cher            | ~20% moins cher |
| **Recommandation**         | Legacy               | ✅ Préférer gp3 |

---

## 2. Provisioned IOPS SSD (io1 / io2 Block Express)

### Usage

```
✅ Cas d'usage :
  ├─ Bases de données critiques (Oracle, SQL Server, MySQL)
  ├─ Applications nécessitant > 16,000 IOPS
  ├─ Workloads sensibles à la latence
  └─ Performance IOPS constante et prévisible

⚠️ Mot-clé examen : "database workload sensitive to storage performance"
```

### Caractéristiques communes

- **Technologie** : SSD haute performance
- **IOPS** : Provisionnées indépendamment de la taille
- **Latence** : Très faible
- **Prix** : Élevé (le plus cher)

---

### io1 (Standard Provisioned IOPS)

```
TAILLE
──────
4 GB → 16 TB

PERFORMANCE
───────────
Max IOPS :
  ├─ 64,000 IOPS (instances Nitro)
  └─ 32,000 IOPS (instances non-Nitro)

Ratio : 50 IOPS par GB maximum
```

**Exemple :**

```
Volume io1 de 1 TB
  ├─ IOPS provisionnées : 10,000 IOPS
  └─ Indépendant de la taille (contrairement à gp2)

Volume io1 de 100 GB
  ├─ IOPS provisionnées : 5,000 IOPS (50 × 100 = max possible)
```

**Prix** : ~0.125 $/GB/mois + ~0.065 $/IOPS provisionné/mois

---

### io2 Block Express (Ultra haute performance)

```
TAILLE
──────
4 GB → 64 TB (4× plus que io1)

PERFORMANCE
───────────
Max IOPS : 256,000 IOPS (4× plus que io1)
Latence : Sub-milliseconde (< 1 ms)
Ratio : 1,000 IOPS par GB maximum
```

**Exemple :**

```
Volume io2 Block Express de 10 TB
  └─ IOPS provisionnées possibles : jusqu'à 256,000 IOPS

Performance :
  └─ 256,000 opérations/seconde 🚀
```

**Prix** : ~0.125 $/GB/mois + coût IOPS provisionné (plus cher que io1)

---

### Fonctionnalité spéciale : EBS Multi-Attach

**Uniquement pour io1/io2** : Un volume peut être attaché à **plusieurs instances EC2 simultanément** (dans la même AZ).

```
[Volume io2]
    ↕
┌───┼───┐
│   │   │
▼   ▼   ▼
[EC2-1] [EC2-2] [EC2-3]

Toutes les instances peuvent lire/écrire simultanément
```

**Cas d'usage** : Clustering, applications hautement disponibles

---

## 3. Throughput Optimized HDD (st1)

### Usage

```
✅ Cas d'usage :
  ├─ Big Data
  ├─ Data warehousing (Redshift, analyse)
  ├─ Log processing
  └─ Workloads séquentiels (lecture/écriture continue)

❌ Ne PAS utiliser pour :
  └─ Boot volume (OS)
```

### Caractéristiques

```
TAILLE
──────
125 GB → 16 TB

PERFORMANCE
───────────
Max Throughput : 500 MB/s
Max IOPS : 500
```

**Technologie** : HDD (disque mécanique)

**Prix** : ~0.045 $/GB/mois (moitié prix de gp3)

**Analogie** : Bon pour lire un gros fichier du début à la fin (streaming), mauvais pour accès aléatoires.

---

## 4. Cold HDD (sc1)

### Usage

```
✅ Cas d'usage :
  ├─ Archive data (données rarement accédées)
  ├─ Backup secondaires
  └─ Stockage le moins cher possible

❌ Ne PAS utiliser pour :
  └─ Boot volume (OS)
```

### Caractéristiques

```
TAILLE
──────
125 GB → 16 TB

PERFORMANCE
───────────
Max Throughput : 250 MB/s
Max IOPS : 250
```

**Technologie** : HDD (disque mécanique)

**Prix** : ~0.015 $/GB/mois (le moins cher de tous)

**Mot-clé examen** : "lowest cost" = **sc1**

---

## Tableau comparatif complet

| Type       | Techno | Taille     | IOPS max | Throughput max | Boot ? | Prix    | Cas d'usage       |
| ---------- | ------ | ---------- | -------- | -------------- | ------ | ------- | ----------------- |
| **gp3**    | SSD    | 1GB-16TB   | 16,000   | 1,000 MB/s     | ✅     | €€      | Général, dev/test |
| **gp2**    | SSD    | 1GB-16TB   | 16,000   | 250 MB/s       | ✅     | €€€     | Legacy            |
| **io2 BE** | SSD    | 4GB-64TB   | 256,000  | 4,000 MB/s     | ✅     | €€€€€   | DB ultra-critique |
| **io1**    | SSD    | 4GB-16TB   | 64,000   | 1,000 MB/s     | ✅     | €€€€    | DB critique       |
| **st1**    | HDD    | 125GB-16TB | 500      | 500 MB/s       | ❌     | €       | Big Data          |
| **sc1**    | HDD    | 125GB-16TB | 250      | 250 MB/s       | ❌     | € (min) | Archive           |

---

## Règles de sélection (Examen AWS)

### Question : Quel type de volume choisir ?

```
DÉCISION
────────

Boot volume (OS) ?
  └─ OUI → gp3 (ou gp2, io1, io2)
  └─ NON → Continuer...

Base de données critique ?
  └─ OUI → io1 ou io2 Block Express
  └─ NON → Continuer...

Besoin > 16,000 IOPS ?
  └─ OUI → io1 (64k) ou io2 (256k)
  └─ NON → Continuer...

Big Data / Log processing ?
  └─ OUI → st1 (throughput HDD)
  └─ NON → Continuer...

Archive / Lowest cost ?
  └─ OUI → sc1 (cold HDD)
  └─ NON → gp3 (default général)
```

---

## Instances Nitro (Important pour > 32,000 IOPS)

**Pour dépasser 32,000 IOPS, vous DEVEZ utiliser :**

- Instance **Nitro** (nouvelle génération : m5, c5, r5, t3...)
- Volume **io1** ou **io2**

```
Instance non-Nitro (t2, m4, c4...)
  └─ Max 32,000 IOPS même avec io1/io2

Instance Nitro (t3, m5, c5...)
  └─ Jusqu'à 64,000 IOPS (io1) ou 256,000 IOPS (io2)
```

---

## Exemples de scénarios d'examen

### Scénario 1

**"Vous avez besoin d'un volume pour une base de données Oracle critique avec 50,000 IOPS"**

- ❌ gp3 (max 16,000 IOPS)
- ❌ st1 (max 500 IOPS)
- ✅ **io1 avec instance Nitro** (64,000 IOPS possible)

---

### Scénario 2

**"Vous voulez stocker des logs rarement accédés au coût le plus bas"**

- ❌ gp3 (trop cher)
- ❌ st1 (pas le moins cher)
- ✅ **sc1** (cold HDD, lowest cost)

---

### Scénario 3

**"Environnement dev/test avec budget limité, performances correctes"**

- ❌ io2 (trop cher)
- ❌ sc1 (trop lent)
- ✅ **gp3** (bon rapport qualité/prix)

---

### Scénario 4

**"Big Data processing avec lecture séquentielle de fichiers volumineux"**

- ❌ io1 (trop cher, IOPS pas nécessaires)
- ❌ sc1 (pas assez de throughput)
- ✅ **st1** (optimisé throughput, bon prix)

---

## Points clés à retenir

✅ **6 types** : gp2, gp3, io1, io2 BE, st1, sc1  
✅ **Boot volumes** : Seulement gp2, gp3, io1, io2  
✅ **gp3** : Meilleur rapport qualité/prix, IOPS indépendantes  
✅ **gp2** : Legacy, IOPS liées à la taille  
✅ **io1/io2** : Bases de données critiques, > 16,000 IOPS  
✅ **st1** : Big Data, throughput élevé, HDD  
✅ **sc1** : Archive, lowest cost, HDD  
✅ **> 32,000 IOPS** : Nécessite Nitro + io1/io2  
✅ **Multi-Attach** : Uniquement io1/io2

---

## Ressource officielle

Toutes les spécifications détaillées sont disponibles sur :
**AWS EBS Volume Types Documentation**

> Vous n'avez PAS besoin de mémoriser tous les chiffres exacts pour l'examen, mais comprendre les **différences conceptuelles** entre les types.
