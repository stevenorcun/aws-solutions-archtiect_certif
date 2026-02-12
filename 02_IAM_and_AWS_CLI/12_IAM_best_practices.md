# IAM - Best Practices (Bonnes Pratiques)

## 🏆 Règles d'or IAM

### 1️⃣ Root Account

- ⚠️ Utiliser **UNIQUEMENT** pour la configuration initiale du compte
- ✅ Ensuite, toujours utiliser son **IAM user personnel**

### 2️⃣ Un User = Une Personne Physique

- ❌ Ne **jamais** partager ses credentials avec quelqu'un d'autre
- ✅ Créer un **nouvel IAM user** pour chaque personne qui a besoin d'accès

### 3️⃣ Gestion des permissions via les Groupes

- ✅ Assigner les users à des **groupes**
- ✅ Assigner les permissions aux **groupes** (pas aux users individuellement)
- ✅ Sécurité gérée au **niveau du groupe**

### 4️⃣ Password Policy

- ✅ Créer une **politique de mot de passe forte**
- ✅ Complexité, expiration, non-réutilisation

### 5️⃣ MFA (Multi-Factor Authentication)

- ✅ Utiliser et **imposer le MFA** sur tous les comptes
- ✅ Garantit la sécurité même en cas de mot de passe compromis

### 6️⃣ IAM Roles pour les Services AWS

- ✅ Toujours utiliser des **IAM Roles** pour donner des permissions aux services AWS
- ✅ Inclut les **EC2 instances** (serveurs virtuels)
- ❌ Ne jamais utiliser des Access Keys personnelles pour les services AWS

### 7️⃣ Access Keys pour l'accès programmatique

- ✅ Générer des **Access Keys** pour CLI et SDK
- ✅ Traiter les Access Keys comme des **mots de passe** (très secrets)
- ❌ Ne **jamais** partager ses Access Keys

### 8️⃣ Audit de sécurité

- ✅ Utiliser le **IAM Credentials Report** pour auditer les permissions du compte
- ✅ Utiliser l'**IAM Access Advisor** pour identifier les permissions inutilisées

---

## 📋 Récapitulatif des Best Practices

| Règle            | ✅ À faire                          | ❌ À ne pas faire                   |
| ---------------- | ----------------------------------- | ----------------------------------- |
| **Root Account** | Setup initial uniquement            | Utiliser au quotidien               |
| **Users**        | 1 user par personne                 | Partager ses credentials            |
| **Permissions**  | Gérer via les groupes               | Assigner directement aux users      |
| **Password**     | Policy forte activée                | Mots de passe faibles               |
| **MFA**          | Activer sur tous les comptes        | Laisser des comptes sans MFA        |
| **Services AWS** | Utiliser des IAM Roles              | Utiliser des Access Keys perso      |
| **Access Keys**  | Garder pour soi                     | Partager avec d'autres              |
| **Audit**        | Credentials Report + Access Advisor | Ignorer les permissions inutilisées |

---

## 🔒 Les 3 choses à ne JAMAIS faire

> ❌ **Ne jamais** utiliser le root account au quotidien  
> ❌ **Ne jamais** partager ses IAM Users credentials  
> ❌ **Ne jamais** partager ses Access Keys

---

## 📝 Récapitulatif de la section IAM

```
IAM
├── Users → 1 personne = 1 user
├── Groups → Regrouper les users, assigner les permissions
├── Policies → Documents JSON définissant les permissions
├── Roles → Permissions pour les services AWS
├── Security
│   ├── Password Policy
│   └── MFA
├── Access Methods
│   ├── Console (Username + Password)
│   ├── CLI (Access Keys)
│   └── SDK (Access Keys)
└── Security Tools
    ├── Credentials Report (account-level)
    └── Access Advisor (user-level)
```

---

## 📝 Points clés à retenir pour l'examen

> ✅ Root account = setup uniquement  
> ✅ 1 user = 1 personne physique  
> ✅ Permissions → toujours via les **groupes**  
> ✅ **MFA** sur tous les comptes  
> ✅ **IAM Roles** pour les services AWS  
> ✅ **Access Keys** = secrets, jamais partagés  
> ✅ **Principe du moindre privilège** = toujours

---

**🎉 Fin de la section IAM !**  
**Prochaine section** : Amazon EC2 (Elastic Compute Cloud)
