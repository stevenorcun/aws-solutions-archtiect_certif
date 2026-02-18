# AWS – Private vs Public IP (IPv4)

## IPv4 vs IPv6

| Version  | Format                                    | Usage actuel                              |
| -------- | ----------------------------------------- | ----------------------------------------- |
| **IPv4** | `192.168.1.1` (4 nombres de 0-255)        | Standard actuel, 3,7 milliards d'adresses |
| **IPv6** | `2001:0db8:85a3:0000:0000:8a2e:0370:7334` | IoT, adresses quasi illimitées            |

> AWS supporte les deux, mais ce cours utilise **IPv4**.

---

## Public IP vs Private IP

### Public IP

- Accessible depuis **Internet**
- Doit être **unique au monde** (pas deux machines avec la même IP publique)
- Permet la géolocalisation
- Exemple : `54.239.28.85`

### Private IP

- Accessible uniquement au sein du **réseau privé**
- Peut être **dupliquée entre différents réseaux privés** (Entreprise A et B peuvent avoir `10.0.0.5`)
- Se connecte à Internet via un **NAT + Internet Gateway**
- Plages réservées : `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`

---

## Schéma réseau typique

```
[Serveur Web Public]  IP: 203.0.113.10
         ↕ Internet ↕

[Entreprise A - Réseau privé]
  ├─ Serveur 1: 10.0.0.5
  ├─ Serveur 2: 10.0.0.6
  └─ [Internet Gateway] → IP publique: 198.51.100.20

[Entreprise B - Réseau privé]
  ├─ Serveur 1: 10.0.0.5  ← Même IP privée, pas de conflit
  ├─ Serveur 2: 10.0.0.7
  └─ [Internet Gateway] → IP publique: 203.0.113.50
```

---

## Elastic IP (IP Élastique)

### Problème

Quand vous **arrêtez puis redémarrez** une instance EC2, son **IP publique change**.

### Solution : Elastic IP

- IP publique **fixe** que vous possédez
- Attachable à **une seule instance à la fois**
- Permet de masquer une panne en la déplaçant rapidement vers une autre instance
- Limite : **5 Elastic IP par compte** (extensible sur demande)

### ⚠️ Bonne pratique : Éviter les Elastic IP

Les Elastic IP sont souvent le signe d'une mauvaise architecture. Préférez :

| Au lieu de...      | Utilisez...                                |
| ------------------ | ------------------------------------------ |
| Elastic IP fixe    | DNS (Route 53) pointant vers l'IP publique |
| Elastic IP pour HA | Load Balancer (pas d'IP publique exposée)  |

---

## EC2 : Comportement par défaut

Chaque instance EC2 reçoit automatiquement :

- Une **IP privée** (réseau interne AWS)
- Une **IP publique** (accès Internet)

### Connexion SSH

- ✅ Vous devez utiliser l'**IP publique** (sauf si VPN configuré)
- ❌ L'IP privée ne fonctionne pas depuis Internet

### Si vous arrêtez/redémarrez l'instance

- L'**IP privée** reste la même
- L'**IP publique** change (sauf Elastic IP)
