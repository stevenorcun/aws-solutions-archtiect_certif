# AWS – Placement Groups (Groupes de Placement)

## Qu'est-ce qu'un Placement Group ?

Un placement group vous permet de **contrôler comment vos instances EC2 sont placées** dans l'infrastructure AWS, sans accès direct au matériel. Vous indiquez simplement à AWS comment positionner vos instances les unes par rapport aux autres.

---

## Les 3 stratégies

| Stratégie     | Objectif                         | Cas d'usage                                 |
| ------------- | -------------------------------- | ------------------------------------------- |
| **Cluster**   | Regrouper dans une même AZ       | Performance réseau maximale                 |
| **Spread**    | Isoler sur du matériel différent | Haute disponibilité, applications critiques |
| **Partition** | Répartir sur plusieurs racks     | Big Data (Hadoop, Kafka, Cassandra)         |

---

## 1. Cluster Placement Group

### Architecture

Toutes les instances sont **dans la même AZ**, sur du matériel proche.

```
[AZ us-east-1a]
  Rack 1 : [Instance 1] [Instance 2] [Instance 3]
  Rack 2 : [Instance 4] [Instance 5] [Instance 6]
```

### Avantages

- **Réseau ultra-rapide** : ~10 Gbps entre instances
- **Latence très faible**
- **Débit réseau élevé**

### Inconvénients

- Si l'AZ tombe → **toutes les instances tombent**
- Risque élevé de panne simultanée

### Cas d'usage

- Jobs Big Data nécessitant une **complétion rapide**
- Applications nécessitant une **latence extrêmement faible** entre instances
- Calcul haute performance (HPC)

---

## 2. Spread Placement Group

### Architecture

Chaque instance est sur un **matériel différent**, répartie sur plusieurs AZ.

```
[AZ us-east-1a]       [AZ us-east-1b]       [AZ us-east-1c]
  Hardware 1            Hardware 3            Hardware 5
  [Instance 1]          [Instance 3]          [Instance 5]

  Hardware 2            Hardware 4            Hardware 6
  [Instance 2]          [Instance 4]          [Instance 6]
```

### Avantages

- **Risque de panne simultanée réduit**
- Peut s'étendre sur **plusieurs AZ**
- Isolation matérielle maximale

### Inconvénients

- **Limite stricte : 7 instances par AZ par placement group**

### Cas d'usage

- Applications **critiques** où chaque instance doit être isolée
- Maximiser la **haute disponibilité**
- Réduire le risque de défaillance corrélée

---

## 3. Partition Placement Group

### Architecture

Les instances sont réparties sur plusieurs **partitions** (= racks AWS), chaque partition pouvant contenir plusieurs instances.

```
[AZ us-east-1a]
  Partition 1 (Rack 1) : [Inst 1] [Inst 2] [Inst 3] [Inst 4]
  Partition 2 (Rack 2) : [Inst 5] [Inst 6] [Inst 7] [Inst 8]

[AZ us-east-1b]
  Partition 3 (Rack 3) : [Inst 9] [Inst 10] [Inst 11] [Inst 12]
```

### Caractéristiques

- **Jusqu'à 7 partitions par AZ**
- Chaque partition = un rack physique distinct
- Les partitions sont **isolées les unes des autres** en cas de panne
- Peut s'étendre sur **plusieurs AZ**
- **Centaines d'instances** possibles (contrairement à Spread)

### Avantages

- Scalabilité bien supérieure au Spread
- Isolation au niveau rack
- Les métadonnées EC2 permettent de savoir sur quelle partition une instance se trouve

### Inconvénients

- Moins d'isolation qu'avec Spread (plusieurs instances par partition)

### Cas d'usage

- Applications **Big Data partition-aware** :
  - HDFS
  - HBase
  - Cassandra
  - Apache Kafka
- Applications pouvant répartir les données entre partitions

---

## Récapitulatif

|                 | Cluster              | Spread                     | Partition                    |
| --------------- | -------------------- | -------------------------- | ---------------------------- |
| **Périmètre**   | 1 AZ                 | Multi-AZ                   | Multi-AZ                     |
| **Objectif**    | Performance réseau   | Isolation maximale         | Scalabilité + isolation rack |
| **Limite**      | Risque AZ unique     | 7 instances/AZ             | 7 partitions/AZ              |
| **Capacité**    | Illimitée            | ~49 instances max (7×7 AZ) | Centaines d'instances        |
| **Cas d'usage** | HPC, calcul intensif | Apps critiques             | Big Data distribué           |

---

# Example

## 1. Cluster Placement Group

**Objectif :** Performance réseau maximale, toutes les instances proches

```
╔═══════════════════════════════════════════════════════════╗
║  AVAILABILITY ZONE : us-east-1a                           ║
║                                                            ║
║  ┌──────────────────────────────────────────────────┐    ║
║  │  RACK (même infrastructure réseau/électrique)    │    ║
║  │                                                   │    ║
║  │  [Instance 1]  [Instance 2]  [Instance 3]        │    ║
║  │                                                   │    ║
║  │  [Instance 4]  [Instance 5]  [Instance 6]        │    ║
║  │                                                   │    ║
║  │  ↕️ Latence ultra-faible (10 Gbps)               │    ║
║  └──────────────────────────────────────────────────┘    ║
║                                                            ║
║  ⚠️  Si ce rack tombe → TOUTES les instances tombent     ║
╚═══════════════════════════════════════════════════════════╝
```

**Résumé :** Toutes dans la même AZ, très proches = super rapide mais risqué

---

## 2. Spread Placement Group

**Objectif :** Isolation maximale, chaque instance sur du matériel différent

```
╔════════════════════════╗  ╔════════════════════════╗  ╔════════════════════════╗
║  AZ : us-east-1a       ║  ║  AZ : us-east-1b       ║  ║  AZ : us-east-1c       ║
║                        ║  ║                        ║  ║                        ║
║  ┌──────────────────┐ ║  ║  ┌──────────────────┐ ║  ║  ┌──────────────────┐ ║
║  │ RACK 1           │ ║  ║  │ RACK 3           │ ║  ║  │ RACK 5           │ ║
║  │                  │ ║  ║  │                  │ ║  ║  │                  │ ║
║  │ [Instance 1] ✅  │ ║  ║  │ [Instance 3] ✅  │ ║  ║  │ [Instance 5] ✅  │ ║
║  │                  │ ║  ║  │                  │ ║  ║  │                  │ ║
║  └──────────────────┘ ║  ║  └──────────────────┘ ║  ║  └──────────────────┘ ║
║                        ║  ║                        ║  ║                        ║
║  ┌──────────────────┐ ║  ║  ┌──────────────────┐ ║  ║  ┌──────────────────┐ ║
║  │ RACK 2           │ ║  ║  │ RACK 4           │ ║  ║  │ RACK 6           │ ║
║  │                  │ ║  ║  │                  │ ║  ║  │                  │ ║
║  │ [Instance 2] ✅  │ ║  ║  │ [Instance 4] ✅  │ ║  ║  │ [Instance 6] ✅  │ ║
║  │                  │ ║  ║  │                  │ ║  ║  │                  │ ║
║  └──────────────────┘ ║  ║  └──────────────────┘ ║  ║  └──────────────────┘ ║
║                        ║  ║                        ║  ║                        ║
╚════════════════════════╝  ╚════════════════════════╝  ╚════════════════════════╝

✅ Si RACK 1 tombe → seule Instance 1 est affectée
✅ Chaque instance sur un matériel DIFFÉRENT
⚠️  Limite : 7 instances par AZ maximum
```

**Résumé :** 1 instance = 1 rack dédié = isolation maximale

---

## 3. Partition Placement Group

**Objectif :** Scalabilité + isolation au niveau rack (plusieurs instances par rack)

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║  AVAILABILITY ZONE : us-east-1a                                               ║
║                                                                                ║
║  ┌────────────────────────────┐  ┌────────────────────────────┐              ║
║  │  PARTITION 1 (RACK A)      │  │  PARTITION 2 (RACK B)      │              ║
║  │                            │  │                            │              ║
║  │  [Instance 1] [Instance 2] │  │  [Instance 3] [Instance 4] │              ║
║  │                            │  │                            │              ║
║  │  • Alimentation X          │  │  • Alimentation Y          │              ║
║  │  • Réseau X                │  │  • Réseau Y                │              ║
║  └────────────────────────────┘  └────────────────────────────┘              ║
║                                                                                ║
╚═══════════════════════════════════════════════════════════════════════════════╝

╔═══════════════════════════════════════════════════════════════════════════════╗
║  AVAILABILITY ZONE : us-east-1b                                               ║
║                                                                                ║
║  ┌────────────────────────────┐                                               ║
║  │  PARTITION 3 (RACK C)      │                                               ║
║  │                            │                                               ║
║  │  [Instance 5] [Instance 6] │                                               ║
║  │                            │                                               ║
║  │  • Alimentation Z          │                                               ║
║  │  • Réseau Z                │                                               ║
║  └────────────────────────────┘                                               ║
║                                                                                ║
╚═══════════════════════════════════════════════════════════════════════════════╝

✅ Si RACK A tombe → Instances 1 et 2 tombent, MAIS 3, 4, 5, 6 continuent
✅ Jusqu'à 7 partitions par AZ
✅ Centaines d'instances possibles (pas de limite stricte comme Spread)
```

**Résumé :** Plusieurs instances par rack, mais les racks sont isolés entre eux

---

## Tableau comparatif visuel

| Stratégie     | Distribution des 6 instances        | Isolation       | Scalabilité      |
| ------------- | ----------------------------------- | --------------- | ---------------- |
| **Cluster**   | Toutes dans 1 seul rack             | ❌ Aucune       | ✅✅✅ Illimitée |
| **Spread**    | 6 racks différents (1 par instance) | ✅✅✅ Maximale | ⚠️ Max 7/AZ      |
| **Partition** | 3 racks (2 instances par rack)      | ✅✅ Bonne      | ✅✅✅ Centaines |

---

## Scénario de panne

### Si un RACK tombe avec chaque stratégie :

**Cluster :**

```
RACK tombe ⚡❌
└─ Instances perdues : ❌❌❌❌❌❌ (toutes les 6)
```

**Spread :**

```
RACK 1 tombe ⚡❌
├─ Instance 1 : ❌ DOWN
└─ Instances 2, 3, 4, 5, 6 : ✅ UP (5/6 survivent)
```

**Partition :**

```
RACK A (Partition 1) tombe ⚡❌
├─ Instances 1 et 2 : ❌ DOWN
└─ Instances 3, 4, 5, 6 : ✅ UP (4/6 survivent)
```
