# AWS – EC2 Hibernate

## Comportement standard : Stop vs Terminate

### Stop (Arrêt)

```
[Instance Running]
  ├─ RAM : Données perdues ❌
  └─ EBS : Données conservées ✅

→ Au redémarrage :
  1. L'OS démarre (boot)
  2. EC2 User Data s'exécute
  3. Applications démarrent
  4. Caches se réchauffent

⏱️ Temps : 2-5 minutes
```

### Terminate (Terminaison)

```
[Instance Running]
  ├─ RAM : Données perdues ❌
  └─ EBS root : Supprimé par défaut ❌
  └─ EBS secondaires : Conservés si configuré ✅
```

---

## EC2 Hibernate : Une nouvelle approche

### Principe

Au lieu d'**arrêter** l'OS, on le **gèle** (freeze). Le contenu de la RAM est **sauvegardé sur le disque EBS**.

### Fonctionnement

```
ÉTAPE 1 : Instance en cours d'exécution
──────────────────────────────────────
[Instance EC2]
  ├─ RAM : 8 GB de données (applications, caches, sessions)
  └─ EBS : 30 GB (OS + données)


ÉTAPE 2 : Hibernation déclenchée
─────────────────────────────────
[Instance EC2] Status: Stopping
  ↓
RAM (8 GB) → Écrit dans un fichier sur EBS root
  ↓
[EBS Volume]
  ├─ OS + données : 22 GB
  └─ Dump RAM : 8 GB
  └─ Total utilisé : 30 GB


ÉTAPE 3 : Instance arrêtée
──────────────────────────
[Instance EC2] ❌ Arrêtée
  ├─ RAM : Vide (instance éteinte)
  └─ EBS : Contient le dump de la RAM ✅


ÉTAPE 4 : Redémarrage
─────────────────────
[Instance EC2] Démarrage
  ↓
EBS → Charge le dump de RAM dans la mémoire
  ↓
[Instance EC2] ✅ Running
  ├─ RAM : 8 GB restaurée (même état qu'avant)
  └─ Applications : Reprennent exactement où elles étaient

⏱️ Temps : 30-60 secondes (au lieu de 2-5 minutes)
```

---

## Différence clé

| Action             | OS Boot         | RAM          | Applications                 | Temps de démarrage |
| ------------------ | --------------- | ------------ | ---------------------------- | ------------------ |
| **Stop classique** | ✅ Redémarre    | ❌ Perdue    | 🔄 Redémarrent de zéro       | 2-5 min            |
| **Hibernate**      | ❌ Reprend gelé | ✅ Restaurée | ✅ Reprennent instantanément | 30-60 sec          |

---

## Analogie simple

### Stop classique = Éteindre votre ordinateur

```
Vous éteignez votre PC
  ↓
Le lendemain vous le rallumez
  ↓
Windows/Linux démarre
  ↓
Vous réouvrez Chrome, VS Code, Spotify...
  ↓
Vous reconnectez vos sessions

⏱️ 3-5 minutes pour retrouver votre environnement
```

### Hibernate = Mettre en veille prolongée

```
Vous mettez en veille prolongée
  ↓
Le lendemain vous le rallumez
  ↓
Tous vos programmes sont EXACTEMENT comme vous les avez laissés
  ↓
Chrome a vos 50 onglets ouverts
  ↓
VS Code a votre code au même endroit

⏱️ 30 secondes, tout est là
```

---

## Cas d'usage

### ✅ Quand utiliser Hibernate

| Scénario                                    | Pourquoi Hibernate aide                                        |
| ------------------------------------------- | -------------------------------------------------------------- |
| **Processus long (analyse de données, ML)** | Le processus reprend exactement où il était                    |
| **Applications avec cache volumineux**      | Pas besoin de réchauffer les caches                            |
| **Services avec initialisation lente**      | Pas besoin de réinitialiser (connexions DB, chargement config) |
| **Sessions utilisateur actives**            | Les sessions restent ouvertes                                  |
| **Développement/test**                      | Arrêt rapide le soir, redémarrage rapide le matin              |

### Exemple concret

```
Vous entraînez un modèle ML qui prend 12 heures
  ├─ Après 8 heures : vous devez arrêter l'instance (économies)
  │
  ├─ SANS Hibernate :
  │   └─ Stop → Le processus s'arrête
  │   └─ Redémarrage → Le processus recommence à zéro ❌
  │
  └─ AVEC Hibernate :
      └─ Hibernate → Le processus est gelé
      └─ Redémarrage → Le processus reprend à 8 heures ✅
```

---

## Prérequis techniques

### Configuration requise

| Prérequis            | Détail                             |
| -------------------- | ---------------------------------- |
| **EBS root volume**  | Obligatoire (pas d'instance store) |
| **Volume chiffré**   | Le root EBS **DOIT être chiffré**  |
| **Taille du volume** | Doit être ≥ taille de la RAM       |
| **Taille RAM max**   | ≤ 150 GB                           |
| **Type d'instance**  | ❌ Pas de bare metal               |

### Exemple de configuration valide

```
[Instance m5.2xlarge]
  ├─ RAM : 32 GB
  └─ EBS root : 50 GB, chiffré ✅

→ Hibernate fonctionne ✅
```

### Exemple de configuration invalide

```
[Instance r5.4xlarge]
  ├─ RAM : 128 GB
  └─ EBS root : 100 GB, chiffré ✅

→ Hibernate ne fonctionne pas ❌ (100 GB < 128 GB)
```

---

## Limites importantes

| Limite                      | Valeur                          | Note                                |
| --------------------------- | ------------------------------- | ----------------------------------- |
| **Durée max d'hibernation** | 60 jours                        | Au-delà, l'instance doit redémarrer |
| **RAM max**                 | 150 GB                          | Peut évoluer                        |
| **Familles supportées**     | C, M, R, T, I (la plupart)      | Vérifier la doc AWS                 |
| **OS supportés**            | Amazon Linux 2, Ubuntu, Windows | Liste non exhaustive                |
| **Type de pricing**         | On-Demand, Reserved, Spot       | ✅ Tous supportés                   |

---

## Schéma récapitulatif : Stop vs Hibernate

```
STOP CLASSIQUE
──────────────
Running → Stopping (RAM perdue) → Stopped
                ↓
         EBS garde OS + données
                ↓
Stopped → Starting (OS boot) → Running
                ↓
     Applications redémarrent de zéro

⏱️ Temps total : 2-5 minutes


HIBERNATE
─────────
Running → Stopping (RAM → EBS) → Stopped
                ↓
         EBS garde OS + données + RAM
                ↓
Stopped → Starting (RAM restaurée) → Running
                ↓
     Applications reprennent instantanément

⏱️ Temps total : 30-60 secondes
```

---

## Points clés à retenir

✅ Hibernate **sauvegarde la RAM** sur le disque EBS  
✅ Le redémarrage est **beaucoup plus rapide** (pas de boot OS)  
✅ Les applications reprennent **exactement où elles étaient**  
✅ Nécessite un **EBS root chiffré** avec **assez d'espace**  
✅ Utile pour les **processus longs** ou **caches volumineux**  
✅ Limité à **60 jours** d'hibernation maximum  
✅ Ne fonctionne **pas** avec les instances bare metal

---

## À l'examen AWS

**Question typique** : "Vous avez une application avec un cache de 30 GB qui prend 10 minutes à se réchauffer. Comment réduire le temps de démarrage ?"

- ❌ Mauvaise réponse : "Utiliser un Stop classique"
- ✅ Bonne réponse : "Utiliser EC2 Hibernate pour préserver le cache en RAM"
