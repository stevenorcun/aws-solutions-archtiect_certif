# IAM Policies - Hands-On Practice

## 🎯 Démonstration pratique des permissions IAM

### Scénario 1 : Retrait des permissions

**Étapes** :

1. User "Stephane" appartient au groupe "admin"
2. Possède les permissions **AdministratorAccess**
3. Peut voir tous les users IAM dans la console

**Action** : Retrait du user "Stephane" du groupe "admin"

**Résultat** :

- ❌ Perte immédiate des permissions
- ❌ Message d'erreur : `Access Denied`
- ❌ API call `iam:ListUsers` refusée
- ❌ Impossible de voir la liste des users

> **Conclusion** : Les permissions sont héritées du groupe. Sans groupe = pas de permissions !

---

## 🔧 Scénario 2 : Ajout de permissions directes (Inline Policy)

### Méthode 1 : Attacher une policy directement à un user

**Étapes** :

1. Aller dans IAM → Users → Stephane
2. Cliquer sur **"Add permissions"**
3. Sélectionner **"Attach policies directly"**
4. Choisir la policy **IAMReadOnlyAccess**
5. Ajouter la permission

**Résultat** :

- ✅ User peut maintenant **lire** les informations IAM
- ✅ API call `iam:ListUsers` fonctionne
- ✅ Peut voir les users, groups, policies
- ❌ **MAIS** ne peut PAS créer de groups (read-only)

### Test de limitation des permissions

**Action** : Tentative de création d'un groupe "developers"

**Résultat** :

- ❌ Échec : Permission refusée
- **Raison** : IAMReadOnlyAccess = **lecture seule**, pas de création/modification

---

## 👥 Scénario 3 : Héritage multiple de permissions

### Configuration

**Création de groupes et assignation** :

1. **Groupe "developers"**
   - Ajout du user "Stephane"
   - Policy attachée : `AlexaForBusiness` (exemple)

2. **Groupe "admin"**
   - Ré-ajout du user "Stephane"
   - Policy attachée : `AdministratorAccess`

3. **Permission directe**
   - Policy : `IAMReadOnlyAccess` (déjà attachée)

### Résultat final

Le user "Stephane" possède maintenant **3 policies** :

| Policy                | Source               | Type            |
| --------------------- | -------------------- | --------------- |
| `AdministratorAccess` | Groupe "admin"       | Héritage groupe |
| `AlexaForBusiness`    | Groupe "developers"  | Héritage groupe |
| `IAMReadOnlyAccess`   | Attachée directement | Inline policy   |

> **Note** : Toutes les permissions sont **cumulatives** (effet additif)

---

## 📄 Analyse détaillée des Policies

### 1️⃣ AdministratorAccess Policy

#### Vue résumée (Summary)

- ✅ Accès complet à **tous les services AWS**
- ✅ Toutes les actions autorisées

#### Structure JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
```

**Explication** :

- `"Action": "*"` = **Toutes les actions** possibles
- `"Resource": "*"` = Sur **toutes les ressources**
- `*` (étoile) = **wildcard** = "n'importe quoi"

---

### 2️⃣ IAMReadOnlyAccess Policy

#### Vue résumée (Summary)

- ✅ IAM : **Full: List** + **Limited: Read**
- ✅ Peut lister et lire les informations IAM
- ❌ Ne peut PAS créer/modifier/supprimer

#### Structure JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:GenerateCredentialReport",
        "iam:GenerateServiceLastAccessedDetails",
        "iam:Get*",
        "iam:List*",
        "iam:SimulateCustomPolicy",
        "iam:SimulatePrincipalPolicy"
      ],
      "Resource": "*"
    }
  ]
}
```

**Explication des wildcards** :

- `"iam:Get*"` = Toutes les actions commençant par "Get"
  - Exemples : `GetUser`, `GetGroup`, `GetRole`, `GetPolicy`
- `"iam:List*"` = Toutes les actions commençant par "List"
  - Exemples : `ListUsers`, `ListGroups`, `ListRoles`

---

## 🛠️ Création d'une Policy personnalisée

### Méthode 1 : Visual Editor (Éditeur visuel)

**Étapes** :

1. IAM → Policies → **Create policy**
2. Sélectionner le **Visual editor**
3. **Service** : IAM
4. **Actions** :
   - `ListUsers` (1/38 dans "List")
   - `GetUser` (1/32 dans "Read")
5. **Resources** : `All resources` ou spécifiques
6. Cliquer sur **Next**
7. Nom de la policy : `MyIAMPermissions`
8. **Create policy**

### Méthode 2 : JSON Editor

**Policy créée (vue JSON)** :

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["iam:ListUsers", "iam:GetUser"],
      "Resource": "*"
    }
  ]
}
```

**Utilisation** :

- Cette policy peut être attachée à des **users** ou **groups**
- Permissions très précises et limitées

---

## 🧹 Nettoyage final (dans la démo)

**Actions effectuées** :

1. **Suppression du groupe "developers"**
   - IAM → User groups → developers → Delete

2. **Retrait de la policy inline**
   - IAM → Users → Stephane
   - Retirer `IAMReadOnlyAccess`

3. **État final**
   - User "Stephane" appartient uniquement au groupe "admin"
   - Possède uniquement `AdministratorAccess`
   - Peut tout faire dans la console IAM ✅

---

## 📝 Points clés à retenir

> ✅ **Permissions héritées** : User dans un groupe = hérite des policies du groupe  
> ✅ **Permissions directes** : Policy attachée directement au user (inline)  
> ✅ **Cumul des permissions** : Toutes les policies s'additionnent  
> ✅ **Wildcard (\*)** : Représente "tout" (actions ou ressources)  
> ✅ **Visual Editor** : Création de policies sans écrire de JSON  
> ✅ **JSON Editor** : Contrôle précis pour utilisateurs avancés  
> ✅ **Test des permissions** : Toujours tester après modification

---

## 💡 Bonnes pratiques démontrées

| Pratique                          | Description                                                    |
| --------------------------------- | -------------------------------------------------------------- |
| **Groupes plutôt qu'inline**      | Utiliser les groupes pour gérer les permissions collectives    |
| **Principe du moindre privilège** | IAMReadOnlyAccess au lieu d'AdministratorAccess quand possible |
| **Tester les permissions**        | Vérifier immédiatement après modification (refresh de la page) |
| **Visual Editor**                 | Utiliser l'éditeur visuel pour éviter les erreurs JSON         |
