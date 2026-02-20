# AWS – ENI : Démonstration Pratique

## Configuration initiale

### Lancer 2 instances EC2

1. Amazon Linux 2, t2.micro
2. Security Group existant (launch-wizard-1)
3. Lancer **2 instances**

---

## Observer les ENI par défaut

### Depuis les instances EC2

1. Sélectionner une instance → onglet **Networking**
2. Voir la section **Network interfaces**
   - Chaque instance a **1 ENI par défaut**
   - Chaque ENI a un **Interface ID** (ex: `eni-0abc123def456`)
   - Contient : IP publique, IP privée, DNS privé

### Depuis le menu ENI

1. Menu gauche → **Network & Security** → **Network Interfaces**
2. Voir les **2 ENI créés automatiquement**
   - Status : **in-use**
   - Attaché à : **Instance ID**

---

## Créer un ENI manuellement

### Étapes de création

1. **Network Interfaces** → **Create network interface**
2. Paramètres :
   - **Description** : `demo ENI`
   - **Subnet** : Choisir la même AZ que vos instances (ex: `us-east-2a`)
   - **Private IPv4** : Auto-assign (ou choisir une IP spécifique)
   - **Security group** : Sélectionner un SG

3. **Create network interface**

### Résultat

- Nouvel ENI créé avec status **available** (pas encore attaché)
- Possède une IP privée secondaire

---

## Attacher l'ENI à une instance

1. Sélectionner le **demo ENI**
2. **Actions** → **Attach**
3. Choisir une instance → **Attach**

### Vérification

```
[Instance EC2 #1]
  ├─ eth0 (ENI primaire) : 10.0.1.5 + IP publique
  └─ eth1 (demo ENI) : 10.0.1.20 (IP privée secondaire)

[Instance EC2 #2]
  └─ eth0 (ENI primaire) : 10.0.1.8 + IP publique
```

L'instance #1 a maintenant **2 interfaces réseau** et **2 IPs privées**.

---

## Déplacer l'ENI entre instances (Failover)

### Étape 1 : Détacher l'ENI

1. Sélectionner le **demo ENI**
2. **Actions** → **Detach**
3. Si erreur → **Actions** → **Detach** avec **Force detach** ✅
4. Attendre que le status devienne **available**

### Étape 2 : Attacher à l'autre instance

1. **Actions** → **Attach**
2. Choisir **Instance #2** → **Attach**

### Résultat après transfert

```
[Instance EC2 #1]
  └─ eth0 (ENI primaire) : 10.0.1.5

[Instance EC2 #2]
  ├─ eth0 (ENI primaire) : 10.0.1.8
  └─ eth1 (demo ENI transféré) : 10.0.1.20 ✅
```

L'IP privée `10.0.1.20` a été **transférée** de l'instance #1 à l'instance #2.

---

## Cas d'usage du transfert

Si vos 2 instances exécutent la **même application** et que vous accédez à l'app via l'IP privée `10.0.1.20` :

```
AVANT
─────
Application sur Instance #1 (accessible via 10.0.1.20)
Instance #2 en standby


APRÈS (failover)
────────────────
Instance #1 crash ou maintenance
Application sur Instance #2 (TOUJOURS accessible via 10.0.1.20) ✅
```

**Avantage** : Failover réseau rapide sans changer les configurations clients.

---

## Comportement lors de la terminaison des instances

### Test

1. Terminer les **2 instances EC2**
2. Observer les ENI

### Résultat

| ENI                                              | Statut après terminaison     |
| ------------------------------------------------ | ---------------------------- |
| **ENI créés automatiquement** avec les instances | ❌ Supprimés automatiquement |
| **ENI créé manuellement** (`demo ENI`)           | ✅ Reste disponible          |

```
AVANT terminaison
─────────────────
3 ENI au total :
  ├─ ENI instance #1 (auto)
  ├─ ENI instance #2 (auto)
  └─ demo ENI (manuel)


APRÈS terminaison
─────────────────
1 ENI restant :
  └─ demo ENI (manuel) ← Toujours là !
```

---

## Avantages de créer un ENI manuellement

| Caractéristique             | ENI auto (défaut) | ENI manuel               |
| --------------------------- | ----------------- | ------------------------ |
| **Cycle de vie**            | Lié à l'instance  | Indépendant              |
| **Survit à la terminaison** | ❌ Non            | ✅ Oui                   |
| **Contrôle de l'IP privée** | ⚠️ Auto-assignée  | ✅ Choix manuel possible |
| **Transférable**            | ❌ Non            | ✅ Oui                   |
| **Cas d'usage**             | Usage standard    | Failover, IP fixe        |

---

## Nettoyage des ressources

1. **Terminer les instances** (déjà fait)
2. **Supprimer le demo ENI** :
   - Sélectionner le demo ENI
   - **Actions** → **Delete**

> ⚠️ Un ENI non attaché ne coûte rien, vous pouvez le garder si vous voulez.

---

## Points clés à retenir

✅ Chaque instance EC2 a au moins **1 ENI par défaut** (eth0)  
✅ Vous pouvez créer des **ENI supplémentaires** indépendamment  
✅ Les ENI peuvent être **détachés et attachés** à d'autres instances  
✅ Les ENI manuels **survivent** à la terminaison des instances  
✅ Les ENI auto créés sont **supprimés** avec l'instance  
✅ Utile pour : failover rapide, IP privée fixe, isolation réseau

---

## Rappel : C'est un concept avancé

Cette fonctionnalité est **rarement utilisée en production moderne**. Les architectures actuelles préfèrent :

- **Load Balancer** pour le failover automatique
- **Auto Scaling** pour la haute disponibilité
- **Route 53** pour le DNS failover

Mais vous devez **connaître les ENI pour l'examen AWS** ! 📚
