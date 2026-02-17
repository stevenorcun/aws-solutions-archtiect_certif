# AWS – Security Groups : Démonstration Pratique

## Accéder aux Security Groups

Deux façons d'y accéder :

- Onglet **Security** directement sur votre instance EC2
- Menu gauche → **Networking & Security** → **Security Groups** (vue complète)

---

## Lire les règles d'un Security Group

Par défaut, après la création d'une instance, deux règles inbound sont présentes :

| Type | Port | Source    | Rôle                   |
| ---- | ---- | --------- | ---------------------- |
| SSH  | 22   | 0.0.0.0/0 | Connexion à l'instance |
| HTTP | 80   | 0.0.0.0/0 | Accès au serveur web   |

---

## Démonstration : Impact d'une règle

### Suppression de la règle HTTP (port 80)

→ La page web ne charge plus : **timeout infini**

### Ré-ajout de la règle HTTP (port 80)

→ La page web redevient **immédiatement accessible**

> **Règle d'or :** Un **timeout** lors d'une connexion à une instance EC2 est **toujours** causé par un problème de security group. Vérifiez vos règles en premier.

---

## Ajouter une règle Inbound

Lors de l'ajout d'une règle, vous pouvez configurer :

- **Type** : choisir dans le menu déroulant (SSH, HTTP, HTTPS…) — le port se remplit automatiquement
- **Port** : personnalisé si besoin (ex. 443 pour HTTPS)
- **Source** :
  - `0.0.0.0/0` → tout le monde (IPv4)
  - **My IP** → uniquement votre IP actuelle ⚠️ Si votre IP change, vous aurez un timeout

---

## Règles Outbound

Par défaut, **tout le trafic sortant est autorisé** (IPv4, vers n'importe quelle destination).

Cela permet à l'instance EC2 d'accéder librement à internet.

---

## Rappels importants

- Un security group peut être attaché à **plusieurs instances EC2**
- Une instance EC2 peut avoir **plusieurs security groups** simultanément
- Les règles de plusieurs security groups **s'additionnent**
