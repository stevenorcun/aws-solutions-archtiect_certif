# AWS – EC2 Instance Store

## Qu'est-ce qu'un EC2 Instance Store ?

Un **disque dur physique** directement attaché au serveur physique qui héberge votre instance EC2.

---

## Comparaison : EBS vs Instance Store

### Architecture

```
EBS VOLUME (Network Drive)
──────────────────────────
[Serveur Physique AWS]
  └─ [Instance EC2 virtuelle]
         ↕ RÉSEAU
     [EBS Volume]
     (sur autre serveur)


INSTANCE STORE (Local Disk)
───────────────────────────
[Serveur Physique AWS]
  ├─ Disque dur SSD/NVMe physique
  │       ↕ CONNEXION DIRECTE (pas de réseau)
  └─ [Instance EC2 virtuelle]
```

---

## Analogie simple

### EBS = Disque dur externe USB

```
Votre ordinateur
  ↔ câble USB ↔
Disque dur externe

✅ Vous pouvez le débrancher et le rebrancher ailleurs
✅ Les données persistent même si vous éteignez le PC
⚠️ Légère latence (câble USB)
```

### Instance Store = Disque dur interne soudé

```
Votre ordinateur
  └─ Disque dur NVMe soudé sur la carte mère

✅ Performance MAXIMALE (connexion directe)
❌ Si le PC meurt → disque perdu
❌ Impossible de le débrancher et le mettre ailleurs
```

---

## Caractéristiques de l'Instance Store

| Caractéristique | Détail                                                |
| --------------- | ----------------------------------------------------- |
| **Performance** | ✅✅✅ TRÈS élevée (jusqu'à 3,3 millions d'IOPS)      |
| **Latence**     | ✅ Très faible (connexion physique)                   |
| **Persistance** | ❌ Éphémère (données perdues si stop/terminate)       |
| **Coût**        | ✅ Inclus dans le prix de l'instance (pas de surcoût) |
| **Sauvegarde**  | ❌ Votre responsabilité                               |
| **Détachable**  | ❌ Non (lié au hardware)                              |

---

## ⚠️ CRITIQUE : Données éphémères

### Ce qui provoque la perte de données

```
PERTE DE DONNÉES ❌
───────────────────
1. Stop de l'instance → Données PERDUES
2. Terminate de l'instance → Données PERDUES
3. Panne du serveur physique → Données PERDUES
4. Migration de l'instance → Données PERDUES


PAS DE PERTE ✅
───────────────
1. Reboot de l'instance → Données CONSERVÉES
```

### Schéma du cycle de vie

```
LANCEMENT
─────────
[Instance i3.large lancée]
  └─ Instance Store : vide
       ↓
  Écrire des données (cache, buffers)
       ↓
  Instance Store : 100 GB de données


STOP (arrêt)
────────────
[Instance stopped]
  └─ Instance Store : ❌ TOUTES LES DONNÉES PERDUES


RESTART (redémarrage)
─────────────────────
[Instance running]
  └─ Instance Store : vide (comme au premier lancement)
```

---

## Performance comparée

### EBS gp3 (Network Drive)

```
Performance maximale :
  ├─ IOPS : 16,000 (ou 32,000 avec optimisation)
  └─ Throughput : 1,000 MB/s

Latence : ~1-5 ms (réseau)
```

### Instance Store i3.8xlarge (Local Disk)

```
Performance maximale :
  ├─ Random Read IOPS : 3,300,000 (3.3 millions) 🚀
  ├─ Random Write IOPS : 1,400,000 (1.4 millions) 🚀
  └─ Throughput : 13,000 MB/s

Latence : <0.1 ms (connexion directe)
```

**Ratio de performance : Instance Store = 100x plus rapide qu'EBS !**

---

## Cas d'usage

### ✅ Bon usage (Instance Store)

| Cas d'usage                  | Pourquoi                                     |
| ---------------------------- | -------------------------------------------- |
| **Cache** (Redis, Memcached) | Données temporaires, reconstructibles        |
| **Buffer**                   | Données de transit, pas critiques            |
| **Scratch data**             | Calculs temporaires, fichiers intermédiaires |
| **Traitement Big Data**      | Données temporaires (Hadoop, Spark)          |
| **Logs temporaires**         | Avant envoi vers S3/CloudWatch               |
| **Session storage**          | Sessions web (avec réplication)              |

**Principe** : Données que vous pouvez **recréer facilement** si perdues.

---

### ❌ Mauvais usage (Instance Store)

| Cas d'usage                  | Pourquoi NE PAS utiliser        |
| ---------------------------- | ------------------------------- |
| **Base de données**          | Perte = désastre                |
| **Fichiers utilisateurs**    | Perte = perte client            |
| **Données critiques**        | Pas de persistance              |
| **Backups**                  | Ironique de backup sur éphémère |
| **Configuration importante** | Perte = downtime                |

**Principe** : Si la perte est **inacceptable**, utilisez EBS ou S3.

---

## Exemple concret : Application de streaming vidéo

### Architecture avec Instance Store

```
[Instance i3.2xlarge]
  ├─ EBS Volume (20 GB)
  │   ├─ OS (Amazon Linux)
  │   ├─ Application (Node.js)
  │   └─ Configuration
  │
  └─ Instance Store (1.9 TB NVMe)
      └─ Cache des vidéos populaires
          ├─ video-trending-001.mp4
          ├─ video-trending-002.mp4
          └─ ...milliers de vidéos en cache
```

**Workflow :**

```
1. Utilisateur demande vidéo populaire
   ↓
2. Check Instance Store (cache local)
   ├─ Vidéo présente ? → ✅ Servir depuis cache (ultra rapide)
   └─ Vidéo absente ? → Récupérer depuis S3 + mettre en cache
   ↓
3. Performance : 3,300,000 IOPS vs 16,000 avec EBS
   → Peut servir 200x plus de requêtes simultanées
```

**Si l'instance crash :**

```
Instance redémarrée
  ↓
Instance Store vide
  ↓
Cache se reconstruit progressivement
  └─ Pas de perte de données critiques (vidéos dans S3)
  └─ Performance temporairement réduite (pas de cache)
```

---

## Exemple concret : Big Data avec Hadoop

### Architecture typique

```
[Cluster Hadoop - 10 instances i3.8xlarge]

Chaque instance :
  ├─ EBS (50 GB) : OS + Hadoop binaries
  └─ Instance Store (8 × 1.9 TB = 15 TB)
      └─ HDFS data blocks (données temporaires)

Workflow :
  1. Job Hadoop démarre
  2. Données lues depuis S3 → écrites sur Instance Store
  3. Calculs sur Instance Store (3.3M IOPS 🚀)
  4. Résultats écrits vers S3
  5. Job terminé → données Instance Store peuvent disparaître
```

**Pourquoi ça fonctionne ?**

- Données sources : **S3** (persistant)
- Données résultats : **S3** (persistant)
- Données intermédiaires : **Instance Store** (éphémère OK)

---

## Stratégie de sauvegarde (obligatoire)

Si vous utilisez Instance Store pour des données importantes, **VOUS** devez gérer la sauvegarde.

### Option 1 : Réplication applicative

```
[Instance 1 - Instance Store]
     ↕ Réplication continue
[Instance 2 - Instance Store]
     ↕ Réplication continue
[Instance 3 - Instance Store]

Si Instance 1 meurt → Données sur Instance 2 et 3 ✅
```

**Exemples** : Redis Cluster, Cassandra, MongoDB avec replica sets

---

### Option 2 : Backup régulier vers S3

```
Cron job toutes les heures :
  ├─ tar -czf /tmp/backup.tar.gz /instance-store-data
  ├─ aws s3 cp /tmp/backup.tar.gz s3://backups/
  └─ rm /tmp/backup.tar.gz

Si perte → Restaurer depuis dernier backup S3
```

---

### Option 3 : Architecture hybride

```
[Instance avec Instance Store]
  ├─ Instance Store (cache chaud, ultra-rapide)
  │   └─ Données fréquemment accédées
  │
  └─ EBS Volume (stockage persistant)
      └─ Données critiques + backup

Synchronisation périodique : Instance Store → EBS
```

---

## Types d'instances avec Instance Store

**Familles principales :**

| Famille           | Cas d'usage                            | Taille Instance Store |
| ----------------- | -------------------------------------- | --------------------- |
| **i3/i4i**        | I/O intensif (databases NoSQL)         | Jusqu'à 60 TB         |
| **d2/d3**         | Dense storage (Hadoop, data warehouse) | Jusqu'à 48 TB         |
| **h1**            | HDD (throughput élevé)                 | Jusqu'à 16 TB         |
| **c5d, m5d, r5d** | Compute/Memory + storage local         | Variable              |

**Exemple :** `i3.8xlarge`

- 32 vCPU
- 244 GB RAM
- **4 × 1.9 TB NVMe SSD** (Instance Store)
- Coût : ~2.50 $/heure

---

## Comparaison finale

```
BESOIN DE PERSISTANCE ?
───────────────────────
OUI → EBS Volume
  └─ Base de données, fichiers utilisateurs, config


BESOIN DE PERFORMANCE MAXIMALE ?
─────────────────────────────────
OUI + données éphémères OK → Instance Store
  └─ Cache, buffer, scratch, Big Data temporaire


BESOIN DES DEUX ?
─────────────────
Architecture hybride :
  ├─ EBS : données critiques
  └─ Instance Store : cache/accélération
```

---

## Points clés à retenir

✅ **Instance Store** = disque physique local, performance maximale  
✅ **Éphémère** = données perdues si stop/terminate/panne  
✅ **Performance** = jusqu'à 3.3 millions d'IOPS (vs 32k pour EBS)  
✅ **Cas d'usage** = cache, buffer, scratch data, Big Data temporaire  
✅ **Backup** = votre responsabilité (réplication ou S3)  
✅ **Coût** = inclus dans le prix de l'instance  
✅ **Reboot** = données conservées (seul cas où elles survivent)

---

## Pour l'examen AWS

**Question typique** : "Vous avez besoin de performances I/O extrêmement élevées pour un cache temporaire. Quelle solution ?"

- ❌ "EBS gp3 avec IOPS provisionnées"
- ❌ "EBS io2 Block Express"
- ✅ "EC2 Instance Store"

**Question typique 2** : "Vous arrêtez une instance avec Instance Store. Que se passe-t-il ?"

- ❌ "Les données sont préservées"
- ❌ "Les données sont sauvegardées automatiquement sur S3"
- ✅ "Les données sont définitivement perdues"

**Mot-clé de l'examen** : "Very high performance hardware-attached volume" = **Instance Store**
