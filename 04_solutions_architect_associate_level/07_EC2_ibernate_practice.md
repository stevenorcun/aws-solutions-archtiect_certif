# AWS – EC2 Hibernate : Démonstration Pratique

## Configuration de l'instance avec Hibernate

### Étape 1 : Lancer l'instance avec les bons paramètres

**Configuration de base :**

1. AMI : **Amazon Linux 2**
2. Type : **t2.micro** (1 GB RAM)
3. Key pair : Au choix
4. Security Group : Existant (ex: launch-wizard-1)

---

### Étape 2 : Activer Hibernate

**Dans la section "Advanced details" :**

1. Descendre jusqu'à **Stop - Hibernate behavior**
2. ✅ Cocher **Enable hibernation**

⚠️ **Message d'avertissement** :

> Pour activer l'hibernation, le volume EBS root doit :
>
> - Être **chiffré**
> - Avoir **assez d'espace** pour contenir la RAM

---

### Étape 3 : Configurer le stockage

**Dans la section "Configure storage" :**

1. Cliquer sur **Advanced**
2. Sélectionner le volume EBS root
3. ✅ **Encrypt this volume** : Oui
4. **KMS key** : `aws/ebs` (clé par défaut)
5. **Size** : 8 GB (suffisant car RAM = 1 GB)

**Calcul :**

```
Instance t2.micro :
  ├─ RAM : 1 GB
  └─ EBS requis : ≥ 1 GB

Volume configuré : 8 GB ✅
→ 8 GB > 1 GB donc OK
```

---

### Étape 4 : Lancer l'instance

1. **Launch instance**
2. L'instance démarre avec **hibernation activée**

---

## Test de l'hibernation

### Étape 1 : Se connecter et vérifier l'uptime

**Commande `uptime`** : Indique depuis combien de temps l'instance tourne **sans interruption** du point de vue de l'OS.

```bash
# Connexion via EC2 Instance Connect
[ec2-user@ip-10-0-1-5 ~]$ uptime
 10:42:15 up 0 min,  1 user,  load average: 0.00, 0.00, 0.00
           └─ Instance démarrée il y a 0 minute

# Attendre 1 minute...
[ec2-user@ip-10-0-1-5 ~]$ uptime
 10:43:20 up 1 min,  1 user,  load average: 0.05, 0.01, 0.00
           └─ Instance démarrée il y a 1 minute
```

**Uptime initial** : 1 minute ✅

---

### Étape 2 : Hiberner l'instance

1. Déconnecter de l'instance
2. Dans la console EC2 → **Instance state** → **Hibernate instance**
3. L'instance passe en état **Stopping** puis **Stopped**

**Ce qui se passe en coulisses :**

```
[Instance Running]
  ├─ RAM (1 GB) → Écrit sur EBS
  └─ Uptime : 1 minute (conservé en mémoire)
       ↓
[Instance Stopped]
  └─ EBS contient : OS + données + dump RAM (uptime inclus)
```

---

### Étape 3 : Redémarrer l'instance

1. **Instance state** → **Start instance**
2. Attendre que l'instance soit **Running**
3. Se reconnecter via **EC2 Instance Connect**

---

### Étape 4 : Vérifier l'uptime après redémarrage

```bash
[ec2-user@ip-10-0-1-5 ~]$ uptime
 10:48:30 up 2 min,  1 user,  load average: 0.02, 0.01, 0.00
           └─ Uptime = 2 minutes (pas remis à zéro !)
```

**Résultat attendu** : L'uptime **ne repart PAS à zéro** ! Il continue à compter depuis le démarrage initial.

---

## Comparaison : Stop classique vs Hibernate

### Scénario 1 : Stop classique (SANS hibernation)

```bash
# Démarrage initial
uptime → 1 min

# Stop l'instance
# ...attendre...
# Start l'instance

# Reconnexion
uptime → 0 min (puis 1 min, 2 min...)
         └─ Compteur REMIS À ZÉRO ❌
```

**Pourquoi ?** L'OS a complètement redémarré. C'est un nouveau cycle de vie.

---

### Scénario 2 : Hibernate (AVEC hibernation)

```bash
# Démarrage initial
uptime → 1 min

# Hibernate l'instance
# ...attendre...
# Start l'instance

# Reconnexion
uptime → 2 min (ou 3 min selon le temps écoulé)
         └─ Compteur CONTINUE ✅
```

**Pourquoi ?** L'OS n'a **jamais vraiment redémarré**. Il a été gelé puis restauré.

---

## Schéma chronologique du test

```
T = 0 min
────────────
Instance lancée
uptime = 0 min


T = 1 min
────────────
uptime = 1 min ✅


T = 2 min
────────────
Hibernate déclenchée
  ↓
RAM → EBS (uptime sauvegardé)


T = 3 min
────────────
Instance Stopped


T = 4 min
────────────
Start instance
  ↓
EBS → RAM (uptime restauré)


T = 5 min
────────────
uptime = 2-3 min ✅
(pas remis à zéro !)
```

---

## Preuve que l'hibernation a fonctionné

| Comportement                 | Stop classique | Hibernate     |
| ---------------------------- | -------------- | ------------- |
| **Uptime après redémarrage** | ⏱️ Repart à 0  | ⏱️ Continue   |
| **Sessions actives**         | ❌ Perdues     | ✅ Conservées |
| **Processus en cours**       | ❌ Tués        | ✅ Reprennent |
| **Caches en RAM**            | ❌ Vidés       | ✅ Intacts    |

---

## Commandes utiles pour tester

### Vérifier l'uptime

```bash
uptime
# Résultat : 10:48:30 up 2 min, 1 user, load average: 0.02, 0.01, 0.00
```

### Créer un processus test (optionnel)

```bash
# Avant hibernation : lancer un processus
sleep 3600 &
ps aux | grep sleep

# Après redémarrage : vérifier s'il tourne encore
ps aux | grep sleep
# Si hibernate a fonctionné → le processus est toujours là ✅
```

### Vérifier la RAM utilisée

```bash
free -h
# Avant hibernation : noter la RAM utilisée
# Après redémarrage : la RAM utilisée devrait être identique
```

---

## Points clés du test

✅ **Uptime ne repart pas à zéro** → Preuve que l'OS n'a pas redémarré  
✅ **Les processus continuent** → Preuve que la RAM a été restaurée  
✅ **Temps de démarrage rapide** → ~30-60 secondes vs 2-5 minutes  
✅ **L'état de l'instance est préservé** → Comme si elle n'avait jamais été arrêtée

---

## Nettoyage

1. **Terminate l'instance** pour éviter les frais
2. Le volume EBS chiffré sera supprimé automatiquement

---

## Récapitulatif du test

**Question** : Comment prouver que l'hibernation fonctionne ?

**Réponse** : La commande `uptime` continue de compter après le redémarrage, prouvant que l'OS n'a jamais vraiment été arrêté du point de vue du système d'exploitation.

**Analogie** : C'est comme mettre votre ordinateur en veille prolongée. Quand vous le rallumez, votre musique reprend exactement où elle s'était arrêtée, vos fenêtres sont toujours ouvertes. Pour l'OS, le temps ne s'est jamais arrêté.
