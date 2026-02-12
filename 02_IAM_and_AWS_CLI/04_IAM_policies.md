# IAM Policies - Guide Complet

## 📋 Types de policies et leur application

### 1️⃣ Group Policy (Politique de groupe)

- Policy attachée **au niveau du groupe**
- **Tous les membres héritent** automatiquement de cette policy
- **Exemple** : Groupe "Developers" (Alice, Bob, Charles) → tous ont les mêmes permissions

### 2️⃣ Inline Policy (Politique inline)

- Policy attachée **directement à un utilisateur spécifique**
- Indépendante de l'appartenance à un groupe
- L'utilisateur peut ou non appartenir à un groupe
- **Exemple** : Fred a une inline policy même s'il n'est dans aucun groupe

### 3️⃣ Héritage multiple

- Un utilisateur appartenant à **plusieurs groupes** hérite de **toutes leurs policies**
- **Exemple** :
  - Charles : Groupe "Developers" + Groupe "Audit Team" → 2 policies
  - David : Groupe "Operations" + Groupe "Audit Team" → 2 policies

---

## 🏗️ Structure d'une IAM Policy (Document JSON)

### Composants principaux

```json
{
  "Version": "2012-10-17",          // Version du langage de policy
  "Id": "S3-Account-Permissions",    // Identifiant de la policy (optionnel)
  "Statement": [                     // Liste des instructions
    {
      "Sid": "1",                    // Statement ID (optionnel)
      "Effect": "Allow",             // Allow ou Deny
      "Principal": {                 // À qui s'applique la policy
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": [                    // Quelles actions API
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [                  // Sur quelles ressources
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {                 // Conditions d'application (optionnel)
        ...
      }
    }
  ]
}
```

---

## 🔑 Éléments clés d'une policy (à connaître pour l'examen)

### Structure générale d'une Policy

| Élément       | Description                     | Obligatoire | Valeurs possibles         |
| ------------- | ------------------------------- | :---------: | ------------------------- |
| **Version**   | Version du langage de policy    |     ✅      | `"2012-10-17"` (standard) |
| **Id**        | Identifiant unique de la policy |     ❌      | Texte libre               |
| **Statement** | Liste des règles de permission  |     ✅      | Array d'objets            |

### Dans chaque Statement

| Élément       | Description                             | Obligatoire | Exemple                     |
| ------------- | --------------------------------------- | :---------: | --------------------------- |
| **Sid**       | Statement ID (identifiant)              |     ❌      | `"1"`, `"AllowS3Access"`    |
| **Effect**    | Autoriser ou refuser                    |     ✅      | `Allow` ou `Deny`           |
| **Principal** | À qui s'applique la policy              |     ✅      | Account, User, Role         |
| **Action**    | Liste des API calls autorisées/refusées |     ✅      | `"s3:GetObject"`, `"ec2:*"` |
| **Resource**  | Ressources concernées                   |     ✅      | ARN des ressources          |
| **Condition** | Conditions d'application                |     ❌      | IP, date, MFA...            |

---

## 💡 Exemple concret

### Scénario : Autoriser l'accès à un bucket S3

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "1",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

### Traduction

- ✅ **Autoriser** (`Allow`)
- 👤 Le compte AWS root (`Principal`)
- 📥 À télécharger des objets (`s3:GetObject`)
- 📦 Dans le bucket `my-bucket` (`Resource`)

---

## 📝 Points clés à retenir pour l'examen

> ✅ **Effect, Principal, Action, Resource** = éléments critiques à maîtriser  
> ✅ **Version** = toujours `"2012-10-17"`  
> ✅ Policy au niveau groupe → tous les membres héritent  
> ✅ Inline policy → attachée directement à un user  
> ✅ User dans plusieurs groupes → hérite de toutes les policies  
> ✅ Format JSON (pas de programmation, juste descriptif)

---

## 🎯 Schéma récapitulatif

```
Organisation
│
├── Groupe "Developers" (Policy A)
│   ├── Alice → hérite Policy A
│   ├── Bob → hérite Policy A
│   └── Charles → hérite Policy A + Policy C (Audit)
│
├── Groupe "Operations" (Policy B)
│   ├── David → hérite Policy B + Policy C (Audit)
│   └── Edward → hérite Policy B
│
├── Groupe "Audit" (Policy C)
│   ├── Charles → déjà listé
│   └── David → déjà listé
│
└── Fred (Inline Policy D) → pas de groupe
```

---

**Note** : Ces concepts seront répétés tout au long du cours. Tu deviendras de plus en plus à l'aise avec le temps
