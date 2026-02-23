# AWS – AMI (Amazon Machine Image)

## Qu'est-ce qu'une AMI ?

**AMI** = **Amazon Machine Image**

C'est un **modèle personnalisé** de votre instance EC2 qui contient :

- La configuration logicielle
- Le système d'exploitation
- Les applications installées
- Les outils de monitoring
- Toute configuration personnalisée

---

## Analogie simple

Une AMI c'est comme une **photo de votre ordinateur configuré** :

```
Votre PC fraîchement configuré
  ├─ Windows 11 installé
  ├─ Chrome, VS Code, Spotify installés
  ├─ Vos préférences système
  └─ Vos logiciels métier

Vous prenez une "photo" (AMI)
  ↓
Vous pouvez recréer EXACTEMENT le même PC
sur n'importe quelle autre machine
```

---

## Avantages d'une AMI personnalisée

### Sans AMI (processus manuel)

```
Lancer EC2 avec Amazon Linux 2
  ↓
Attendre le boot (2-3 min)
  ↓
Installer Apache (5 min)
  ↓
Installer MySQL (5 min)
  ↓
Configurer les services (10 min)
  ↓
Installer monitoring (5 min)

⏱️ TOTAL : ~30 minutes par instance
```

---

### Avec AMI personnalisée

```
Créer UNE FOIS l'AMI avec tout préinstallé
  ↓
Lancer EC2 depuis votre AMI
  ↓
✅ TOUT est déjà là (Apache, MySQL, config, monitoring)

⏱️ TOTAL : 2-3 minutes (juste le boot)
```

**Gain de temps énorme quand vous lancez 10, 50, 100+ instances !**

---

## Types d'AMI

### 1. Public AMI (fournie par AWS)

```
[Amazon Linux 2 AMI]
  ├─ Maintenue par AWS
  ├─ Gratuite
  ├─ Mise à jour régulière
  └─ Utilisée dans ce cours jusqu'à présent
```

**Exemples** :

- Amazon Linux 2
- Ubuntu Server
- Windows Server
- Red Hat Enterprise Linux

---

### 2. Custom AMI (Votre propre AMI)

```
[Votre AMI personnalisée]
  ├─ Créée par VOUS
  ├─ Contient VOS applications
  ├─ VOS configurations
  └─ VOUS devez la maintenir
```

**Cas d'usage** :

- Application web avec stack préconfigurée (LAMP, MEAN...)
- Serveur avec vos outils internes
- Configuration corporate standardisée

---

### 3. AWS Marketplace AMI (Vendeurs tiers)

```
[AMI WordPress par Bitnami]
  ├─ Créée par Bitnami (vendeur)
  ├─ WordPress préinstallé et configuré
  ├─ Payante (ou gratuite selon vendeur)
  └─ Support du vendeur
```

**Exemples** :

- WordPress préconfigured par Bitnami
- GitLab Server
- Firewall Palo Alto
- SAP HANA

> 💡 **Vous pouvez aussi VENDRE vos AMI sur le Marketplace AWS !**

---

## Processus de création d'une AMI

### Étape 1 : Lancer et personnaliser une instance

```
[Instance EC2 standard]
  ↓
Installer Apache
Installer MySQL
Configurer firewall
Installer scripts de backup
Configurer monitoring
  ↓
[Instance EC2 personnalisée] ✅
```

---

### Étape 2 : Arrêter l'instance (recommandé)

```
[Instance Running]
  ↓
Stop Instance (pour garantir l'intégrité des données)
  ↓
[Instance Stopped]
```

> **Pourquoi arrêter ?** Pour s'assurer que toutes les données sont bien écrites sur le disque.

---

### Étape 3 : Créer l'AMI

```
[Instance Stopped]
  ↓
Actions → Create Image (AMI)
  ↓
AWS crée automatiquement :
  ├─ AMI
  └─ EBS Snapshots (de tous les volumes attachés)
```

**Important** : La création d'AMI génère des **snapshots EBS** en arrière-plan.

---

### Étape 4 : Lancer des instances depuis l'AMI

```
[Votre AMI personnalisée]
  ↓
Launch Instance from AMI
  ↓
[Nouvelle instance identique] ✅
  └─ Même config, mêmes apps, même tout !
```

---

## Schéma complet du processus

```
ÉTAPE 1 : PERSONNALISATION
──────────────────────────
╔═══════════════════════════════╗
║  AZ : us-east-1a              ║
║                               ║
║  [Instance EC2 #1]            ║
║    ├─ Amazon Linux 2          ║
║    ├─ Apache installé         ║
║    ├─ MySQL installé          ║
║    └─ Scripts custom          ║
║                               ║
╚═══════════════════════════════╝
         ↓
    Stop Instance
         ↓


ÉTAPE 2 : CRÉATION AMI
──────────────────────
[Instance Stopped]
  ↓
Create Image
  ↓
[Custom AMI créée]
  └─ Snapshots EBS créés automatiquement


ÉTAPE 3 : DÉPLOIEMENT
─────────────────────
╔═══════════════════════════════╗
║  AZ : us-east-1b              ║
║                               ║
║  [Lancer depuis Custom AMI]   ║
║         ↓                     ║
║  [Instance EC2 #2]            ║
║    ├─ Amazon Linux 2          ║
║    ├─ Apache DÉJÀ installé ✅ ║
║    ├─ MySQL DÉJÀ installé ✅  ║
║    └─ Scripts DÉJÀ là ✅      ║
║                               ║
╚═══════════════════════════════╝

RÉSULTAT : Instance #2 est une COPIE EXACTE de Instance #1
```

---

## AMI et Régions

### Scope régional

Les AMI sont **liées à une région** par défaut.

```
[AMI créée en us-east-1]
  ↓
Utilisable uniquement dans us-east-1
```

---

### Copier une AMI vers une autre région

```
[AMI en us-east-1]
  ↓
Actions → Copy AMI → us-west-2
  ↓
[AMI copiée en us-west-2] ✅
```

**Cas d'usage** :

- Déploiement multi-régions
- Disaster Recovery
- Expansion globale

---

## Exemple concret : Architecture multi-AZ

### Objectif

Déployer la même application dans 2 AZ différentes.

```
SANS AMI
────────
AZ-1a : Installer manuellement (30 min)
AZ-1b : Réinstaller manuellement (30 min)
⏱️ TOTAL : 1 heure


AVEC AMI
────────
AZ-1a : Installer manuellement (30 min)
  ↓
Créer AMI (5 min)
  ↓
AZ-1b : Lancer depuis AMI (3 min)
⏱️ TOTAL : 38 minutes
```

**Gain : 22 minutes pour 2 instances. Pour 100 instances → gain de 48 heures !**

---

## Contenu d'une AMI

| Élément           | Description                                        |
| ----------------- | -------------------------------------------------- |
| **OS**            | Système d'exploitation (Linux, Windows...)         |
| **Applications**  | Logiciels installés (Apache, MySQL, Node.js...)    |
| **Configuration** | Fichiers de config, scripts, cron jobs             |
| **Volumes EBS**   | Snapshots des volumes (root + data)                |
| **Permissions**   | Public, privé, ou partagé avec comptes spécifiques |

---

## Permissions des AMI

### AMI Privée (par défaut)

```
[Votre AMI]
  └─ Utilisable uniquement par VOTRE compte AWS
```

### AMI Publique

```
[Votre AMI]
  └─ Visible et utilisable par TOUS les utilisateurs AWS
```

### AMI Partagée

```
[Votre AMI]
  └─ Partagée avec des comptes AWS spécifiques (ex: comptes de votre entreprise)
```

---

## Facturation

| Type d'AMI           | Coût AMI                    | Coût Snapshots                  |
| -------------------- | --------------------------- | ------------------------------- |
| **Public AMI (AWS)** | Gratuit                     | N/A                             |
| **Custom AMI**       | Gratuit                     | 💰 Oui (snapshots EBS facturés) |
| **Marketplace AMI**  | 💰 Variable (selon vendeur) | 💰 Oui (snapshots EBS)          |

> **Important** : Vous ne payez pas l'AMI elle-même, mais les **snapshots EBS** qui la composent.

---

## Cas d'usage réels

### 1. Auto Scaling

```
[AMI avec app web préconfigurée]
  ↓
Auto Scaling Group
  ↓
Charge augmente → Lance 10 instances depuis AMI
  ↓
✅ Toutes identiques, prêtes en 2-3 minutes
```

---

### 2. Golden Image (Image de référence)

```
Entreprise : Standard corporate pour tous les serveurs
  ├─ AMI avec :
  │   ├─ Antivirus installé
  │   ├─ Agents de monitoring
  │   ├─ Outils de conformité
  │   └─ Configurations de sécurité
  │
  └─ Tous les serveurs lancés depuis cette AMI
      → Conformité garantie ✅
```

---

### 3. Disaster Recovery

```
[Production en us-east-1]
  ↓
Créer AMI quotidiennement
  ↓
Copier AMI vers us-west-2
  ↓
En cas de désastre en us-east-1
  ↓
Lancer depuis AMI en us-west-2 ✅
```

---

## Workflow typique

```
1. DEV
──────
Développeur configure serveur manuellement
  ↓
Teste que tout fonctionne
  ↓
Crée AMI "app-v1.0"


2. STAGING
──────────
Lance instance depuis AMI "app-v1.0"
  ↓
Tests de validation
  ↓
OK ✅


3. PRODUCTION
─────────────
Lance 50 instances depuis AMI "app-v1.0"
  ↓
Toutes identiques, déploiement en 10 minutes ✅
```

---

## Points clés à retenir

✅ **AMI** = modèle personnalisé d'instance EC2  
✅ **Gain de temps** : boot + config pré-faits  
✅ **3 types** : Public (AWS), Custom (vous), Marketplace (vendeurs)  
✅ **Processus** : Personnaliser → Arrêter → Créer AMI → Lancer  
✅ **Snapshots EBS** créés automatiquement lors de la création AMI  
✅ **Régionale** par défaut, mais copiable entre régions  
✅ **Facturation** : snapshots EBS uniquement (pas l'AMI elle-même)

---

## Pour l'examen AWS

**Question typique** : "Comment déployer rapidement 100 serveurs web identiques ?"

- ❌ "Lancer 100 instances et installer manuellement"
- ✅ "Créer une AMI avec la config complète, puis lancer 100 instances depuis cette AMI"

**Question typique 2** : "Vous voulez déployer votre app dans une nouvelle région. Comment faire ?"

- ❌ "Reconfigurer manuellement dans la nouvelle région"
- ✅ "Copier l'AMI vers la nouvelle région, puis lancer des instances"
