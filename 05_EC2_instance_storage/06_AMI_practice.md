# AWS – AMI : Démonstration Pratique

## Objectif de la démo

Créer une AMI personnalisée avec Apache préinstallé pour accélérer le déploiement de futures instances.

---

## Étape 1 : Lancer l'instance de base

### Configuration

**Paramètres de base :**

- **AMI** : Amazon Linux 2
- **Instance type** : t2.micro
- **Key pair** : Au choix
- **Security Group** : Utiliser existant (launch-wizard-1)

### User Data (Script de démarrage)

**Copier UNIQUEMENT les 4 premières lignes** (installation Apache) :

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
```

**❌ NE PAS copier la dernière ligne** :

```bash
# Cette ligne sera ajoutée APRÈS création de l'AMI
echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
```

**Pourquoi ?** Nous voulons créer une AMI avec **Apache installé**, mais **sans contenu spécifique**. Le contenu sera ajouté plus tard.

### Lancer l'instance

**Launch instance**

---

## Étape 2 : Attendre l'installation d'Apache

### Vérification (trop rapide = erreur)

```
Instance lancée → Status: Running
  ↓
Essayer d'accéder immédiatement à http://<IP_PUBLIQUE>
  ↓
❌ Connection refused / Timeout

Pourquoi ?
  └─ Le User Data script est encore en cours d'exécution
```

**Il faut attendre 1-2 minutes** pour que le script User Data termine l'installation.

---

### Vérification (après 2 minutes)

```
Rafraîchir http://<IP_PUBLIQUE>
  ↓
✅ Page de test Apache s'affiche !
```

**Résultat** : Apache est installé et fonctionne.

---

## Étape 3 : Créer l'AMI

### Actions

1. Sélectionner l'instance
2. **Actions** → **Image and templates** → **Create image**

### Configuration

```
┌────────────────────────────────────────┐
│ Image name: demo-image                 │
│ Image description: Apache préinstallé  │
│                                        │
│ No reboot: ☐ (recommandé de laisser)  │
│                                        │
│ Instance volumes: (automatique)        │
└────────────────────────────────────────┘
```

3. **Create image**

---

### Vérifier la création

**Menu gauche** → **Images** → **AMIs**

```
┌─────────────┬──────────┬──────────────┬────────────┐
│ AMI ID      │ Name     │ Status       │ Visibility │
├─────────────┼──────────┼──────────────┼────────────┤
│ ami-0abc... │ demo-im..│ pending      │ private    │
└─────────────┴──────────┴──────────────┴────────────┘

Attendre quelques minutes...

┌─────────────┬──────────┬──────────────┬────────────┐
│ AMI ID      │ Name     │ Status       │ Visibility │
├─────────────┼──────────┼──────────────┼────────────┤
│ ami-0abc... │ demo-im..│ available ✅ │ private    │
└─────────────┴──────────┴──────────────┴────────────┘
```

**L'AMI est prête !**

---

## Étape 4 : Lancer une instance depuis l'AMI

### Méthode 1 : Depuis AMIs

1. **AMIs** → Sélectionner `demo-image`
2. **Launch instance from AMI**

### Méthode 2 : Depuis Launch Instance

1. **Launch instance**
2. Onglet **My AMIs**
3. Sélectionner `demo-image`

---

### Configuration de la nouvelle instance

**Paramètres :**

- AMI : `demo-image` (déjà sélectionnée)
- Instance type : t2.micro
- Key pair : Au choix (ou aucun)
- Security Group : launch-wizard-1

**User Data (cette fois) :**

```bash
#!/bin/bash
echo "<h1>Hello World from $(hostname -f)</h1>" > /var/www/html/index.html
```

**Pourquoi seulement cette ligne ?**

```
Apache est DÉJÀ installé dans l'AMI ✅
  ├─ yum install httpd → SKIP (déjà fait)
  ├─ systemctl start httpd → SKIP (déjà fait)
  └─ Juste créer le fichier HTML → RAPIDE
```

### Lancer l'instance

**Launch instance**

---

## Étape 5 : Vérifier la rapidité

### Comparaison des temps

```
INSTANCE 1 (sans AMI)
─────────────────────
User Data complet :
  ├─ yum update → 30s
  ├─ yum install httpd → 45s
  ├─ systemctl start → 10s
  └─ echo HTML → 1s

⏱️ TOTAL : ~90 secondes


INSTANCE 2 (avec AMI)
─────────────────────
User Data réduit :
  └─ echo HTML → 1s

⏱️ TOTAL : ~10 secondes
```

**Gain de temps : 80 secondes !**

---

### Résultat

```
Accéder à http://<IP_PUBLIQUE_INSTANCE_2>
  ↓
✅ "Hello World from ip-10-0-1-23.ec2.internal"

Beaucoup plus rapide car Apache était déjà installé !
```

---

## Schéma du processus complet

```
ÉTAPE 1 : INSTANCE DE BASE
──────────────────────────
[Lancer Instance]
  ├─ Amazon Linux 2
  └─ User Data: Installer Apache (4 lignes)
       ↓
[Instance Running]
  └─ Apache installé ✅


ÉTAPE 2 : CRÉATION AMI
──────────────────────
[Instance avec Apache]
  ↓
Actions → Create Image
  ↓
[AMI: demo-image]
  └─ Contient: OS + Apache préinstallé


ÉTAPE 3 : DÉPLOIEMENT RAPIDE
─────────────────────────────
[Lancer depuis AMI demo-image]
  ├─ Apache DÉJÀ là ✅
  └─ User Data: Juste créer index.html
       ↓
[Instance prête en 10 secondes] 🚀
```

---

## Cas d'usage réaliste étendu

### Scénario : 100 serveurs web identiques

**Sans AMI :**

```
100 instances × 90 secondes = 9000 secondes (2h30)
  └─ Chaque instance installe Apache individuellement
```

**Avec AMI :**

```
1. Créer AMI une fois (5 min)
2. Lancer 100 instances depuis AMI
   └─ 100 × 10 secondes = 1000 secondes (17 min)

⏱️ Gain total : 2h30 → 22 minutes
```

---

## Contenu de l'AMI créée

| Élément                     | Inclus dans AMI ?                   |
| --------------------------- | ----------------------------------- |
| **Amazon Linux 2**          | ✅ Oui                              |
| **Apache (httpd)**          | ✅ Oui (installé et configuré)      |
| **httpd.service**           | ✅ Oui (enabled au démarrage)       |
| **index.html**              | ❌ Non (ajouté via User Data après) |
| **Packages système à jour** | ✅ Oui (yum update effectué)        |

---

## Extension : AMI plus complète

Dans un vrai projet, l'AMI contiendrait :

```bash
#!/bin/bash
# User Data pour créer une "Golden Image" complète

# Mise à jour système
yum update -y

# Installer stack web
yum install -y httpd php mysql

# Installer monitoring
wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
rpm -U ./amazon-cloudwatch-agent.rpm

# Installer Docker
yum install -y docker
systemctl start docker
systemctl enable docker

# Installer Node.js
curl -sL https://rpm.nodesource.com/setup_18.x | bash -
yum install -y nodejs

# Installer outils de sécurité
yum install -y fail2ban

# Configurer Nginx
yum install -y nginx
systemctl enable nginx

# Scripts de déploiement
mkdir -p /opt/scripts
cat > /opt/scripts/deploy.sh << 'EOF'
#!/bin/bash
cd /var/www/html
git pull origin main
npm install
systemctl restart nginx
EOF
chmod +x /opt/scripts/deploy.sh
```

**Temps d'installation** : 5-10 minutes

**AMI créée** : Tout est préinstallé

**Nouvelle instance** : Prête en 30 secondes 🚀

---

## Nettoyage

Pour éviter les coûts :

### 1. Terminer les instances

```
Instances:
  ├─ Instance 1 (base) → Terminate
  └─ Instance 2 (depuis AMI) → Terminate
```

### 2. (Optionnel) Supprimer l'AMI

```
AMIs → Sélectionner demo-image
  ↓
Actions → Deregister AMI
  ↓
Confirmer
```

**Note** : Deregister l'AMI ne supprime pas les snapshots automatiquement.

### 3. (Optionnel) Supprimer les snapshots

```
Snapshots → Filtrer par AMI ID
  ↓
Sélectionner les snapshots associés
  ↓
Actions → Delete snapshot
```

---

## Comparaison : Avant/Après AMI

```
AVANT (déploiement manuel)
──────────────────────────
Besoin de 10 serveurs web
  ↓
Lancer 10 instances
  ↓
Chacune installe Apache (90s × 10)
  ↓
⏱️ 15 minutes + risque d'erreurs


APRÈS (avec AMI)
────────────────
Créer AMI golden image (une fois)
  ↓
Lancer 10 instances depuis AMI
  ↓
Apache déjà là sur toutes
  ↓
⏱️ 2 minutes + zéro erreur
```

---

## Points clés de la démo

✅ **User Data partiel** pour créer l'AMI (installations seulement)  
✅ **Attendre 1-2 min** après lancement pour que User Data se termine  
✅ **Create Image** depuis instance configurée  
✅ **AMI status** doit être "available" avant utilisation  
✅ **Nouvelle instance** depuis AMI = boot ultra-rapide  
✅ **User Data minimal** sur instance depuis AMI (juste personnalisation)  
✅ **Gain de temps majeur** pour déploiements multiples

---

## Pour l'examen AWS

**Question typique** : "Vous devez déployer 50 serveurs web identiques rapidement. Quelle approche ?"

- ❌ "Lancer 50 instances et installer manuellement"
- ❌ "Utiliser des snapshots EBS"
- ✅ "Créer une AMI avec tout préinstallé, puis lancer 50 instances depuis cette AMI"

**Question typique 2** : "Quelle est la différence entre Snapshot et AMI ?"

- **Snapshot** : Sauvegarde d'un volume EBS (données seulement)
- **AMI** : Image complète (OS + config + apps) pour lancer des instances
