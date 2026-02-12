# IAM Introduction - Users, Groups, Policies

## 🔐 IAM - Identity and Access Management

**Définition** : Service de gestion des identités et des accès AWS

**Caractéristique clé** : Service **GLOBAL** (non lié à une région spécifique)

---

## 👤 Root Account vs Users

### Root Account (Compte racine)

- Créé **automatiquement** lors de l'ouverture du compte AWS
- **⚠️ À utiliser UNIQUEMENT pour** :
  - Configuration initiale du compte
- **NE PAS** :
  - Utiliser au quotidien
  - Partager avec qui que ce soit

### Users (Utilisateurs)

- Représente **une personne physique** dans l'organisation
- À créer pour chaque personne qui utilise AWS
- **Pratique recommandée** : ne pas utiliser le root account, créer des users

---

## 👥 Groups (Groupes)

**Définition** : Regroupement logique d'utilisateurs

### Règles importantes

- Un groupe contient **UNIQUEMENT des utilisateurs** (pas d'autres groupes)
- Un utilisateur peut **ne pas appartenir à un groupe** (non recommandé)
- Un utilisateur peut **appartenir à plusieurs groupes**

### Exemple d'organisation

```
Organisation (6 personnes)
│
├── Groupe "Developers"
│   ├── Alice
│   ├── Bob
│   └── Charles
│
├── Groupe "Operations"
│   ├── David
│   └── Edward
│
├── Groupe "Audit"
│   ├── Charles (appartient aussi à Developers)
│   └── David (appartient aussi à Operations)
│
└── Fred (aucun groupe - non recommandé)
```

---

## 📜 Policies (Politiques IAM)

**Définition** : Document JSON qui définit les **permissions** accordées aux users/groups

**Format** : Document JSON (pas de programmation requise, langage descriptif)

### Exemple de policy

```json
{
  "Allow": ["EC2:Describe", "ElasticLoadBalancing:Describe", "CloudWatch:*"]
}
```

### Ce qu'elle fait

Autorise l'utilisateur/groupe à :

- Utiliser EC2 (voir les instances)
- Utiliser Elastic Load Balancing (voir les load balancers)
- Utiliser CloudWatch (toutes actions)

---

## 🛡️ Principe du moindre privilège (Least Privilege Principle)

**Règle d'or IAM** : Ne donner QUE les permissions nécessaires, rien de plus

### Pourquoi ?

- **Sécurité** : Limiter les dégâts en cas de compte compromis
- **Coûts** : Éviter qu'un utilisateur lance des services coûteux par erreur
- **Conformité** : Respecter les bonnes pratiques de sécurité

### Exemple

- Si un utilisateur a besoin de 3 services uniquement
- → Créer une policy avec accès à ces 3 services SEULEMENT

---

## 📋 Points clés à retenir

> ✅ IAM = service global  
> ✅ Root account = setup uniquement, puis ne plus utiliser  
> ✅ 1 User = 1 personne physique  
> ✅ Groups contiennent uniquement des Users  
> ✅ Users peuvent appartenir à 0, 1 ou plusieurs Groups  
> ✅ Policies (JSON) = définissent les permissions  
> ✅ Appliquer le principe du moindre privilège
