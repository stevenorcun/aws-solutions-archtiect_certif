# IAM Security Tools

## 🛡️ Les 2 outils de sécurité IAM

| Outil                      | Niveau        | Objectif                                                |
| -------------------------- | ------------- | ------------------------------------------------------- |
| **IAM Credentials Report** | Account-level | Vue globale de tous les users et leurs credentials      |
| **IAM Access Advisor**     | User-level    | Vue des permissions utilisées/non utilisées par un user |

---

## 📊 1. IAM Credentials Report

**Niveau** : Account-level (tout le compte AWS)

**Contenu** :

- Liste de **tous les utilisateurs** du compte
- **Statut** de leurs différents credentials :
  - Mot de passe (actif, dernière utilisation, dernière rotation...)
  - Access Keys (actives, dernière utilisation...)
  - MFA (activé ou non)

**Utilité** :

- ✅ Vue d'ensemble de la sécurité de tous les users
- ✅ Identifier les users avec des credentials non sécurisés
- ✅ Audit de sécurité global du compte

---

## 🔍 2. IAM Access Advisor

**Niveau** : User-level (par utilisateur)

**Contenu** :

- Liste des **permissions accordées** à un user
- **Date de dernière utilisation** de chaque service

**Utilité** :

- ✅ Identifier les permissions **jamais ou rarement utilisées**
- ✅ **Réduire les permissions** inutiles
- ✅ Appliquer le **principe du moindre privilège**

**Exemple** :

```
User: Stephane
├── EC2 → Dernière utilisation : aujourd'hui ✅
├── S3 → Dernière utilisation : il y a 6 mois ⚠️
├── DynamoDB → Jamais utilisé ❌ → Peut être retiré
└── CloudWatch → Dernière utilisation : hier ✅
```

---

## 🔗 Lien avec le Principe du Moindre Privilège

```
IAM Access Advisor
│
├── Montre les permissions NON utilisées
│
├── Permet d'identifier les permissions inutiles
│
└── → Supprimer ces permissions
    → ✅ Respecter le principe du moindre privilège
```

---

## 📝 Points clés à retenir pour l'examen

> ✅ **Credentials Report** = niveau **compte** → statut de tous les users  
> ✅ **Access Advisor** = niveau **user** → services utilisés/non utilisés  
> ✅ Access Advisor = outil clé pour appliquer le **principe du moindre privilège**  
> ✅ Ces outils sont utilisés pour l'**audit de sécurité** IAM
