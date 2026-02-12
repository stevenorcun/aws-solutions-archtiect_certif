# Résumé Global - AWS Certified Solutions Architect Associate 2026

## 📚 Section 1 : Getting Started with AWS

### 🌍 Infrastructure Globale AWS

#### AWS Régions

- Cluster de datacenters géographiquement groupés
- Nomenclature : `us-east-1`, `eu-west-3`, `ap-southeast-2`
- La plupart des services AWS sont **liés à une région spécifique**
- **4 critères de choix** d'une région :

| Critère                  | Description                                          |
| ------------------------ | ---------------------------------------------------- |
| **Compliance**           | Conformité légale (ex: données françaises en France) |
| **Latency**              | Déployer proche des utilisateurs                     |
| **Service availability** | Tous les services ne sont pas partout                |
| **Pricing**              | Prix variable selon les régions                      |

#### AWS Availability Zones (AZ)

- **3 à 6 AZ** par région (généralement 3)
- Chaque AZ = 1 ou plusieurs datacenters isolés
- Isolées pour résister aux catastrophes
- Connectées par réseau **haute bande passante et ultra-faible latence**

#### AWS Points of Presence (Edge Locations)

- **400+** points de présence dans **90+ villes** et **40+ pays**
- Objectif : délivrer du contenu avec la **latence la plus faible possible**
- Utilisés par des services comme **CloudFront (CDN)**

#### Portée des services AWS

| Type          | Exemples                                    |
| ------------- | ------------------------------------------- |
| **Globaux**   | IAM, Route 53, CloudFront, WAF              |
| **Régionaux** | EC2, Lambda, Elastic Beanstalk, Rekognition |

---

## 📚 Section 2 : IAM & AWS CLI

### 🔐 IAM - Identity and Access Management

**Service GLOBAL** de gestion des identités et accès AWS

#### Composants principaux

```
IAM
├── Users (1 user = 1 personne physique)
├── Groups (contiennent uniquement des users)
├── Policies (documents JSON définissant les permissions)
└── Roles (permissions pour les services AWS)
```

#### Users & Groups

- **Root account** → setup uniquement, ne plus utiliser ensuite
- **1 User = 1 personne physique** (ne jamais partager)
- Un user peut appartenir à **0, 1 ou plusieurs groupes**
- Les groupes contiennent **UNIQUEMENT des users** (pas d'autres groupes)

#### Policies (Documents JSON)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "1",                      // Optionnel
      "Effect": "Allow",               // Allow ou Deny
      "Principal": { "AWS": "..." },   // À qui s'applique
      "Action": ["s3:GetObject"],      // Quelles actions
      "Resource": "arn:aws:s3:::...",  // Sur quelles ressources
      "Condition": { ... }             // Optionnel
    }
  ]
}
```

**Types de policies** :

| Type                  | Description                                                 |
| --------------------- | ----------------------------------------------------------- |
| **Group Policy**      | Attachée au groupe, héritée par tous les membres            |
| **Inline Policy**     | Attachée directement à un user spécifique                   |
| **Héritage multiple** | User dans plusieurs groupes = hérite de toutes les policies |

**Wildcards** :

- `"Action": "*"` = toutes les actions
- `"iam:Get*"` = toutes les actions commençant par "Get"
- `"Resource": "*"` = toutes les ressources

#### 🛡️ Sécurité IAM

**Password Policy** :

- Longueur minimale, types de caractères
- Expiration (ex: tous les 90 jours)
- Prévention de réutilisation
- Protection contre les **attaques par force brute**

**MFA (Multi-Factor Authentication)** :

- Password + Device physique/virtuel
- **Obligatoire** pour le root account
- **Recommandé** pour tous les IAM users

| Type MFA             | Exemple                     | Caractéristiques        |
| -------------------- | --------------------------- | ----------------------- |
| **Virtual MFA**      | Google Authenticator, Authy | Gratuit, multi-comptes  |
| **U2F Security Key** | YubiKey (Yubico)            | Clé USB physique        |
| **Hardware Key Fob** | Gemalto                     | Appareil physique       |
| **GovCloud Key Fob** | SurePassID                  | AWS GovCloud uniquement |

#### 🌐 Méthodes d'accès à AWS

| Méthode                | Protection                | Usage              |
| ---------------------- | ------------------------- | ------------------ |
| **Management Console** | Username + Password + MFA | Interface web      |
| **CLI**                | Access Keys               | Terminal / Scripts |
| **SDK**                | Access Keys               | Code applicatif    |

**Access Keys** :

- Générées dans **Security Credentials**
- Visibles **une seule fois** lors de la création
- Access Key ID = comme un username
- Secret Access Key = comme un password
- **Ne jamais partager**

**Configuration CLI** :

```bash
aws configure
# → AWS Access Key ID
# → AWS Secret Access Key
# → Default region name (ex: eu-west-1)
# → Default output format
```

**Commandes CLI utiles** :

```bash
aws iam list-users      # Lister les users IAM
aws s3 ls               # Lister les buckets S3
aws --version           # Vérifier la version CLI
```

#### ☁️ AWS CloudShell

- Terminal **intégré à la console AWS**
- **Gratuit**, credentials automatiques
- Région par défaut = région console actuelle
- Fichiers **persistants** entre les sessions
- Fonctionnalité **upload/download** de fichiers
- ⚠️ Disponible uniquement dans **certaines régions**

#### 🎭 IAM Roles

- Permissions assignées aux **services AWS** (pas aux humains)
- Service AWS + IAM Role = une seule entité avec permissions

| Role commun              | Service                      |
| ------------------------ | ---------------------------- |
| **EC2 Instance Role**    | Serveurs virtuels EC2        |
| **Lambda Function Role** | Fonctions Lambda             |
| **CloudFormation Role**  | Déploiement d'infrastructure |

#### 🔍 Outils de Sécurité IAM

| Outil                  | Niveau        | Utilité                                                 |
| ---------------------- | ------------- | ------------------------------------------------------- |
| **Credentials Report** | Account-level | Statut de tous les users et credentials                 |
| **Access Advisor**     | User-level    | Permissions utilisées/non utilisées → moindre privilège |

---

## 🏆 IAM Best Practices

```
✅ Root account → setup uniquement
✅ 1 user = 1 personne physique
✅ Permissions → via les groupes
✅ Password Policy forte
✅ MFA sur tous les comptes
✅ IAM Roles pour les services AWS
✅ Access Keys → secrets, jamais partagés
✅ Principe du moindre privilège
✅ Audit régulier (Credentials Report + Access Advisor)
```

**Les 3 règles absolues** :

> ❌ Ne jamais utiliser le root account au quotidien  
> ❌ Ne jamais partager ses IAM User credentials  
> ❌ Ne jamais partager ses Access Keys

---

## 🗺️ Vue d'ensemble - Ce qu'on a appris

```
AWS
│
├── Infrastructure Globale
│   ├── Régions (clusters de datacenters)
│   ├── Availability Zones (isolation + haute dispo)
│   └── Edge Locations (distribution de contenu)
│
└── IAM (Identity & Access Management)
    ├── Users & Groups (qui peut accéder)
    ├── Policies JSON (ce qu'ils peuvent faire)
    ├── Roles (permissions pour les services AWS)
    ├── Sécurité (Password Policy + MFA)
    ├── Accès (Console, CLI, SDK + Access Keys)
    ├── CloudShell (terminal intégré AWS)
    └── Outils (Credentials Report + Access Advisor)
```

---

**🎉 Prochaine section : Amazon EC2 (Elastic Compute Cloud)**
