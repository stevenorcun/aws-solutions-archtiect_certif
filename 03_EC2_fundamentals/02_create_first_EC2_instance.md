# AWS – Lancer sa première instance EC2

## Objectif

Lancer une instance EC2 sous Amazon Linux, y déployer un serveur web via user data, puis apprendre à démarrer, arrêter et terminer l'instance.

---

## 1. Créer l'instance

### Nommage

- Aller dans **EC2 Console → Instances → Launch Instances**
- Donner un nom : `My First Instance`

### Choisir une AMI (système d'exploitation)

- Utiliser **Amazon Linux 2 AMI** (fournie par AWS)
- Architecture : **64-bits x86**
- ✅ Free tier eligible

### Choisir un type d'instance

- Utiliser **t2.micro** (free tier eligible – 750h/mois)
- Si indisponible dans votre région → **t3.micro**

---

## 2. Créer une paire de clés (Key Pair)

Nécessaire pour se connecter via SSH.

| Paramètre                          | Valeur              |
| ---------------------------------- | ------------------- |
| Nom                                | EC2 Tutorial        |
| Type                               | RSA                 |
| Format (Mac / Linux / Windows 10+) | `.pem`              |
| Format (Windows 7 / 8)             | `.ppk` (pour PuTTY) |

> La clé est téléchargée automatiquement à la création.

---

## 3. Configurer le réseau et le pare-feu

Un **Security Group** est créé automatiquement (`launch-wizard-1`).

| Règle | Port | Source   |
| ----- | ---- | -------- |
| SSH   | 22   | Anywhere |
| HTTP  | 80   | Anywhere |

> HTTPS n'est pas nécessaire pour ce TP.

---

## 4. Configurer le stockage

- Volume : **8 Go, gp2** (free tier : jusqu'à 30 Go)
- **Delete on termination** : activé par défaut → le volume est supprimé avec l'instance

---

## 5. Ajouter le User Data (script de démarrage)

Dans **Advanced Details → User Data**, coller le script fourni dans le cours (`EC2 user data` file).

Ce script s'exécute **une seule fois**, au premier démarrage. Il va :

1. Mettre à jour le système
2. Installer le serveur web **HTTPD**
3. Créer une page HTML **Hello World**

---

## 6. Lancer et accéder à l'instance

1. Cliquer sur **Launch Instance**
2. Aller dans **View all Instances**
3. Attendre que l'état passe à **Running** (~10-15 secondes)

### Informations clés de l'instance

| Champ        | Description                          |
| ------------ | ------------------------------------ |
| Instance ID  | Identifiant unique                   |
| Public IPv4  | Adresse pour accéder depuis internet |
| Private IPv4 | Adresse interne au réseau AWS (fixe) |
| AMI          | Amazon Linux 2                       |
| Key pair     | EC2 Tutorial                         |

### Accéder au serveur web

Copier le **Public IPv4** et l'ouvrir dans un navigateur en utilisant **HTTP** :

```
http://<public-ip>
```

> ⚠️ Ne pas utiliser `https://` → chargement infini.  
> Si rien ne s'affiche, patienter 5 minutes et rafraîchir.

---

## 7. Gérer l'état de l'instance

| Action        | Menu                       | Effet                                     |
| ------------- | -------------------------- | ----------------------------------------- |
| **Stop**      | Instance State → Stop      | Arrête l'instance, pas de facturation CPU |
| **Start**     | Instance State → Start     | Redémarre l'instance                      |
| **Terminate** | Instance State → Terminate | Supprime définitivement l'instance        |

> ⚠️ **Le Public IPv4 change** à chaque redémarrage.  
> Le **Private IPv4 reste fixe**.

---

## Récapitulatif

- ✅ Première instance EC2 lancée sous Amazon Linux 2
- ✅ Serveur web déployé automatiquement via User Data
- ✅ Accès via HTTP sur le Public IPv4
- ✅ Start / Stop / Terminate maîtrisés
