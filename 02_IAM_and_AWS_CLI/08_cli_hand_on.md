# IAM - Configuration du CLI & Access Keys (Hands-On)

## 🎯 Objectif

Créer des Access Keys et configurer le AWS CLI pour accéder à AWS depuis un terminal

---

## 🔑 Création des Access Keys

### Étapes dans la Console AWS

1. Cliquer sur son **username** (ex: Stephane) en haut à droite
2. Aller dans **"Security credentials"**
3. Scroller vers le bas → **"Create access key"**
4. Sélectionner le **cas d'usage** (ex: CLI)

### ⚠️ Recommandations AWS lors de la création

| Cas d'usage             | Alternative recommandée par AWS            |
| ----------------------- | ------------------------------------------ |
| **CLI**                 | CloudShell ou CLI V2 + IAM Identity Center |
| **Application locale**  | Variables d'environnement                  |
| **Application sur AWS** | IAM Roles                                  |

> **Note** : Pour ce cours, on utilise les Access Keys directement pour bien comprendre leur fonctionnement

### ⚠️ Point critique

> 🔒 Les Access Keys ne sont visibles qu'**UNE SEULE FOIS** lors de leur création  
> → Les télécharger ou les copier immédiatement !

---

## 💻 Configuration du AWS CLI

### Commande de configuration

```bash
aws configure
```

### Informations demandées

```bash
AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name: eu-west-1        # Choisir la région la plus proche
Default output format:                 # Laisser vide = format par défaut
```

> **Comment trouver le nom de sa région ?**  
> Dans la console AWS → Menu déroulant des régions → Code de région visible (ex: `eu-west-1`)

---

## 🧪 Test du CLI

### Lister les utilisateurs IAM

```bash
aws iam list-users
```

### Résultat retourné

```json
{
  "Users": [
    {
      "UserName": "Stephane",
      "UserId": "AIDIODR4TAW7CSEXAMPLE",
      "Arn": "arn:aws:iam::123456789012:user/Stephane",
      "CreateDate": "2023-01-01T00:00:00Z",
      "PasswordLastUsed": "2023-06-01T00:00:00Z"
    }
  ]
}
```

> **Note** : Résultat identique à ce qu'on voit dans la Management Console → Les deux méthodes donnent les mêmes informations

---

## 🔒 Démonstration : Permissions CLI = Permissions Console

### Test de suppression des permissions

**Action** : Retrait du user "Stephane" du groupe "admin"

**Résultats** :

| Interface              | Résultat                                    |
| ---------------------- | ------------------------------------------- |
| **Management Console** | ❌ Erreur `Access Denied` (après refresh)   |
| **CLI**                | ❌ Requête refusée, aucun résultat retourné |

```bash
# Après suppression des permissions
aws iam list-users
# → Aucun résultat (accès refusé)
```

> **Conclusion** : Les permissions IAM s'appliquent **de la même façon** que ce soit via la Console ou le CLI

---

## 🔄 Restauration des permissions

> ⚠️ **Ne pas oublier** de ré-ajouter le user dans le groupe admin !

**Étapes** :

1. IAM → **User groups** → admins
2. **Add users** → Sélectionner Stephane
3. ✅ Permissions restaurées

---

## 📊 Récapitulatif - Console vs CLI

```
Management Console
└── Accès via navigateur web
└── Protégé par Username + Password + MFA
└── Interface graphique

AWS CLI
└── Accès via terminal
└── Protégé par Access Key ID + Secret Access Key
└── Mêmes permissions que la Console
└── Résultats identiques (format JSON)
```

---

## 📝 Points clés à retenir

> ✅ Access Keys créées dans **Security Credentials** du user  
> ✅ Access Keys visibles **une seule fois** → à sauvegarder immédiatement  
> ✅ Commande de config : **`aws configure`**  
> ✅ CLI et Console ont les **mêmes permissions**  
> ✅ Permissions retirées dans IAM = **accès refusé aussi dans le CLI**  
> ✅ Toujours **ré-ajouter les permissions** après un test de suppression  
> ✅ Ne jamais oublier : **Access Keys = privées, ne pas partager**
