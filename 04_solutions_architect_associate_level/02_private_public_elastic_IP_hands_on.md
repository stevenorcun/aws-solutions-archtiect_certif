# AWS – Elastic IP : Démonstration Pratique

## Comportement standard d'une instance EC2

### IP Publique

- Utilisée pour se connecter en SSH depuis Internet
- ✅ Fonctionne : `ssh -i key.pem ec2-user@<IP_PUBLIQUE>`

### IP Privée

- Visible une fois connecté à l'instance (hostname)
- ❌ Ne fonctionne PAS pour SSH depuis Internet : `ssh -i key.pem ec2-user@<IP_PRIVÉE>`
- Raison : vous n'êtes pas sur le réseau privé AWS

---

## Problème : L'IP publique change après Stop/Start

### Test

1. Instance en cours → IP publique : `54.123.45.67`
2. **Stop** l'instance
3. **Start** l'instance (≠ reboot)
4. Nouvelle IP publique : `18.234.56.78`

> ⚠️ **Reboot ≠ Stop/Start** — Un simple reboot ne change pas l'IP publique.

### Conséquence

- L'ancienne commande SSH ne fonctionne plus
- Il faut utiliser la nouvelle IP
- L'**IP privée reste identique**

---

## Solution : Elastic IP

### Créer une Elastic IP

1. Menu gauche → **Elastic IPs**
2. **Allocate Elastic IP address**
3. Allouer depuis le pool Amazon IPv4 → **Allocate**

### Associer l'Elastic IP à l'instance

1. Clic droit sur l'Elastic IP → **Associate Elastic IP address**
2. Choisir **Instance**
3. Sélectionner votre instance en cours
4. Choisir l'IP privée associée
5. **Associate**

### Résultat

- L'**IP publique** de l'instance = l'**Elastic IP**
- Cette IP reste fixe même après Stop/Start

---

## Test : Stop/Start avec Elastic IP

1. **Stop** l'instance → l'Elastic IP reste attachée
2. **Start** l'instance → l'Elastic IP est toujours là
3. Même commande SSH fonctionne : `ssh -i key.pem ec2-user@<ELASTIC_IP>`

---

## ⚠️ Tarification IPv4 (Important)

Depuis 2024, AWS facture **toutes les IP publiques** :

| Situation                     | Coût                             |
| ----------------------------- | -------------------------------- |
| IP publique (utilisée ou non) | ~0,005 $/heure (~3,50 $/mois)    |
| Elastic IP (attachée)         | ~0,005 $/heure (~3,50 $/mois)    |
| Elastic IP (non attachée)     | Facturée même si non utilisée ❗ |

### Free Tier

- **750 heures/mois** d'adresses IPv4 publiques gratuites
- Si vous avez **1 instance** tournant 24/7 → couvert (24×30 = 720h)
- Si vous avez **2 instances** → dépassement du quota

> **Conseil :** Toujours terminer les instances et libérer les Elastic IP non utilisées.

---

## Nettoyer les ressources

### Supprimer une Elastic IP

1. Clic droit → **Disassociate Elastic IP address** (détacher de l'instance)
2. Clic droit → **Release Elastic IP address** (libérer définitivement)

> ⚠️ Si vous ne **libérez pas** l'Elastic IP, vous continuez d'être facturé même si elle n'est pas attachée.

### Après suppression de l'Elastic IP

- L'instance récupère automatiquement une nouvelle IP publique standard
- Cette IP changera au prochain Stop/Start
