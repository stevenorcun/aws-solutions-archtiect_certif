# AWS Access Methods - CLI, SDK & Access Keys

## 🌐 Les 3 méthodes d'accès à AWS

| Méthode                | Description                        | Protection                |
| ---------------------- | ---------------------------------- | ------------------------- |
| **Management Console** | Interface web (utilisée jusqu'ici) | Username + Password + MFA |
| **CLI**                | Command Line Interface (terminal)  | Access Keys               |
| **SDK**                | Software Development Kit (code)    | Access Keys               |

---

## 🔑 Access Keys (Clés d'accès)

**Définition** : Credentials permettant d'accéder à AWS via CLI ou SDK

### Génération des Access Keys

- Générées via la **Management Console**
- Chaque user est **responsable de ses propres access keys**
- Peuvent être téléchargées **une seule fois** lors de la création

### Structure d'une Access Key

| Composant             | Équivalent        | Exemple (fictif)                           |
| --------------------- | ----------------- | ------------------------------------------ |
| **Access Key ID**     | Comme un username | `AKIAIOSFODNN7EXAMPLE`                     |
| **Secret Access Key** | Comme un password | `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` |

### ⚠️ Règles de sécurité IMPORTANTES

> ❌ **NE JAMAIS partager ses Access Keys**  
> ❌ **NE JAMAIS committer ses Access Keys dans du code**  
> ✅ Chaque utilisateur doit générer **ses propres** Access Keys  
> ✅ Traiter l'Access Key ID comme un **username**  
> ✅ Traiter le Secret Access Key comme un **password**

---

## 💻 CLI - Command Line Interface

**Définition** : Outil permettant d'interagir avec les services AWS via des commandes dans un terminal

### Caractéristiques

- Chaque commande commence par le mot **`aws`**
- Accès direct aux **APIs publiques** des services AWS
- Permet de **créer des scripts** pour automatiser des tâches
- **Open-source** (code source disponible sur GitHub)
- Alternative à la Management Console

### Exemple de commande CLI

```bash
# Copier un fichier vers S3
aws s3 cp mon-fichier.txt s3://mon-bucket/

# Lister les utilisateurs IAM
aws iam list-users

# Lister les buckets S3
aws s3 ls
```

### Avantages

- ✅ Automatisation des tâches répétitives
- ✅ Gestion des ressources par scripts
- ✅ Certains utilisateurs n'utilisent **que** le CLI (sans Management Console)

---

## 📦 SDK - Software Development Kit

**Définition** : Ensemble de librairies permettant d'accéder aux services AWS **depuis le code de votre application**

### Différence avec le CLI

| CLI                            | SDK                              |
| ------------------------------ | -------------------------------- |
| Utilisé dans un **terminal**   | Intégré dans une **application** |
| Commandes manuelles ou scripts | Code programmatique              |
| Usage interactif               | Usage applicatif                 |

### Langages supportés

| Catégorie   | Langages                                    |
| ----------- | ------------------------------------------- |
| **Web**     | JavaScript, PHP, Ruby                       |
| **Backend** | Python, Java, Go, .NET, Node.js, C++        |
| **Mobile**  | Android, iOS                                |
| **IoT**     | Internet of Things devices (capteurs, etc.) |

### Exemple concret

> 💡 **Fun fact** : Le AWS CLI est lui-même construit sur le **AWS SDK pour Python**, appelé **Boto** !

---

## 🔄 Comparaison des 3 méthodes d'accès

```
AWS
├── Management Console (navigateur web)
│   └── Protégé par : Username + Password + MFA
│
├── CLI (terminal)
│   └── Protégé par : Access Keys
│   └── Usage : Commandes manuelles / Scripts d'automatisation
│
└── SDK (dans votre application)
    └── Protégé par : Access Keys
    └── Usage : Intégration dans du code applicatif
```

---

## 📝 Points clés à retenir pour l'examen

> ✅ **3 méthodes d'accès** : Console, CLI, SDK  
> ✅ **CLI et SDK** = protégés par des **Access Keys**  
> ✅ **Access Key ID** = comme un username  
> ✅ **Secret Access Key** = comme un password  
> ✅ **Ne jamais partager** ses Access Keys  
> ✅ **AWS CLI** est open-source et basé sur le SDK Python (Boto)  
> ✅ **SDK** = embarqué dans le code d'une application  
> ✅ **CLI** = utilisé dans un terminal
