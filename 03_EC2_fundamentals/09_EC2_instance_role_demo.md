# AWS – IAM Roles pour EC2

## Problème : Comment donner des permissions AWS à une instance EC2 ?

### ❌ La mauvaise approche : `aws configure`

Il est techniquement possible de lancer `aws configure` sur l'instance et d'entrer ses clés personnelles (Access Key ID + Secret Access Key), mais c'est **une très mauvaise pratique**.

> **Règle absolue : Ne jamais entrer vos clés IAM dans une instance EC2.**

N'importe quelle personne ayant accès à l'instance (via SSH ou EC2 Instance Connect) pourrait récupérer ces credentials.

---

### ✅ La bonne approche : IAM Role

Attacher un **IAM Role** à l'instance EC2 pour lui fournir des permissions, sans jamais exposer de clés.

---

## Démonstration

### Situation initiale

```bash
aws iam list-users
# → "Unable to locate credentials"
```

Sans rôle attaché, l'instance n'a aucune permission.

### Attacher un IAM Role à l'instance

1. Sélectionnez votre instance EC2
2. **Actions** → **Security** → **Modify IAM Role**
3. Sélectionnez le rôle souhaité (ex. `DemoRoleForEC2`)
4. Cliquez sur **Save**

### Résultat

```bash
aws iam list-users
# → Liste des utilisateurs IAM ✅
```

L'instance utilise automatiquement les permissions du rôle attaché.

---

## Vérification du lien Role ↔ Instance

| Action                                | Résultat de `aws iam list-users` |
| ------------------------------------- | -------------------------------- |
| Rôle attaché avec `IAMReadOnlyAccess` | ✅ Retourne la liste             |
| Rôle détaché                          | ❌ Access Denied                 |
| Politique retirée du rôle             | ❌ Access Denied                 |

> **Note :** Les changements de permissions IAM peuvent prendre quelques secondes à se propager.

---

## Résumé

Pour fournir des credentials AWS à une instance EC2 → **toujours utiliser un IAM Role**, jamais `aws configure`.
