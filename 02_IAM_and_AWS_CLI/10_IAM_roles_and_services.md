# IAM Roles (Rôles IAM)

## 🎭 Définition

**IAM Role** = Permissions assignées à des **services AWS** (et non à des personnes physiques)

> Comme un IAM User, mais destiné à être utilisé par des **services AWS** plutôt que par des humains

---

## 🤔 Pourquoi les IAM Roles ?

Certains services AWS ont besoin d'effectuer des actions **en notre nom** sur notre compte AWS.

Ces services ont donc besoin de **permissions**, tout comme les users humains.

**Solution** → Créer un **IAM Role** et l'assigner au service concerné

---

## 💡 Exemple concret - EC2 Instance

```
EC2 Instance (serveur virtuel)
│
├── Veut accéder à un service AWS (ex: S3, DynamoDB...)
│
├── A besoin de permissions pour le faire
│
├── On lui assigne un IAM Role
│
└── ✅ EC2 + IAM Role = une seule entité avec des permissions
```

**Fonctionnement** :

1. EC2 Instance tente d'accéder à une ressource AWS
2. Elle utilise son **IAM Role** pour s'authentifier
3. Si les permissions du Role sont correctes → ✅ Accès accordé
4. Si les permissions sont insuffisantes → ❌ Accès refusé

---

## 🔧 Roles les plus courants

| Service            | Role associé         | Usage                                                            |
| ------------------ | -------------------- | ---------------------------------------------------------------- |
| **EC2**            | EC2 Instance Role    | Permettre à un serveur virtuel d'accéder à d'autres services AWS |
| **Lambda**         | Lambda Function Role | Permettre à une fonction Lambda d'accéder à des ressources AWS   |
| **CloudFormation** | CloudFormation Role  | Permettre à CloudFormation de créer/modifier des ressources AWS  |

---

## 📊 IAM User vs IAM Role

| Caractéristique | IAM User                          | IAM Role                             |
| --------------- | --------------------------------- | ------------------------------------ |
| **Utilisé par** | Personne physique                 | Service AWS                          |
| **Credentials** | Username + Password / Access Keys | Temporaires (assumés par le service) |
| **Durée**       | Permanents                        | Temporaires                          |
| **Exemple**     | Développeur, Admin                | EC2, Lambda, CloudFormation          |

---

## 📝 Points clés à retenir

> ✅ **IAM Role** = permissions pour les **services AWS** (pas pour les humains)  
> ✅ Un service AWS + IAM Role = **une seule entité** avec des permissions  
> ✅ Roles les plus courants : **EC2**, **Lambda**, **CloudFormation**  
> ✅ Sans IAM Role, un service AWS **ne peut pas** accéder à d'autres services  
> ✅ Même principe que les IAM Users : **principe du moindre privilège**
