# AWS – Les Types d'Instances EC2

## Convention de nommage

Un nom d'instance comme **m5.2xlarge** se décompose ainsi :

| Partie    | Signification          | Exemple                            |
| --------- | ---------------------- | ---------------------------------- |
| `m`       | Classe de l'instance   | General Purpose                    |
| `5`       | Génération du matériel | m5 → m6 si amélioration            |
| `2xlarge` | Taille (CPU/RAM)       | small < large < 2xlarge < 4xlarge… |

> Plus la taille est grande, plus vous avez de CPU et de mémoire.

---

## Les 4 familles d'instances

### 1. General Purpose (Usage général)

- **Idéal pour :** serveurs web, dépôts de code
- **Équilibre :** compute / mémoire / réseau
- **Exemples :** `t2.micro` (inclus dans le Free Tier ✅)

### 2. Compute Optimized (Optimisé CPU)

- **Idéal pour :** batch processing, transcodage vidéo, HPC, machine learning, serveurs de jeux
- **Noms :** série `C` (C5, C6…)

### 3. Memory Optimized (Optimisé RAM)

- **Idéal pour :** bases de données en mémoire, cache distribué (ElastiCache), BI, traitement temps réel de grandes données
- **Noms :** série `R` (RAM), `X1`, `High Memory`, `Z1`

### 4. Storage Optimized (Optimisé stockage)

- **Idéal pour :** OLTP, bases SQL/NoSQL, Redis, data warehousing, systèmes de fichiers distribués
- **Noms :** série `I`, `D`, `H1`

---

## Comparaison d'instances

| Instance      | vCPU | RAM    |
| ------------- | ---- | ------ |
| `t2.micro`    | 1    | 1 Go   |
| `r5.16xlarge` | 16   | 512 Go |
| `c5d.4xlarge` | 16   | 32 Go  |

---

## Ressources utiles

- **AWS :** [aws.amazon.com/ec2/instance-types](https://aws.amazon.com/ec2/instance-types)
- **Comparateur :** [ec2instances.info](https://ec2instances.info) — liste complète avec prix On-Demand, Reserved, vCPU, RAM, etc.
