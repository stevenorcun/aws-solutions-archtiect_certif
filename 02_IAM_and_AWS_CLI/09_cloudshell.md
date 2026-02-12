# AWS CloudShell

## 🌐 Définition

**CloudShell** = Terminal intégré directement dans la console AWS (accessible via l'icône en haut à droite de la console)

> **Alternative** au terminal local pour exécuter des commandes AWS CLI

---

## ⚠️ Disponibilité

- **N'est pas disponible dans toutes les régions**
- Vérifier la liste des régions compatibles dans la documentation AWS
- Recommandation : choisir une région où CloudShell est disponible si vous souhaitez l'utiliser

> **Note** : Si votre terminal local fonctionne déjà, CloudShell n'est pas obligatoire

---

## ✅ Caractéristiques principales

### Authentification automatique

- Les commandes CLI s'exécutent **avec les credentials du compte AWS connecté**
- Pas besoin de configurer `aws configure`
- La **région par défaut** = région actuellement sélectionnée dans la console

```bash
# Exemple - fonctionne directement sans configuration
aws iam list-users
```

### Version AWS CLI

```bash
# Vérifier la version installée
aws --version
# → aws-cli/2.x.x (version 2 installée par défaut)
```

---

## 💾 Persistance des fichiers

- Les fichiers créés dans CloudShell **sont conservés** après redémarrage
- L'environnement persiste entre les sessions

```bash
# Créer un fichier
echo "tests" > demo.txt

# Ce fichier sera toujours là après un redémarrage de CloudShell
```

---

## ⚙️ Options de configuration

| Option         | Valeurs disponibles  |
| -------------- | -------------------- |
| **Font size**  | Small, Medium, Large |
| **Theme**      | Light, Dark          |
| **Safe paste** | Activé / Désactivé   |

---

## 📁 Upload & Download de fichiers

**Fonctionnalité très utile** pour transférer des fichiers entre votre machine locale et CloudShell

### Download (Télécharger depuis CloudShell)

1. Actions → **Download file**
2. Entrer le chemin complet du fichier (ex: `/home/cloudshell-user/demo.txt`)
3. Le fichier est téléchargé sur votre machine locale

### Upload (Envoyer vers CloudShell)

1. Actions → **Upload file**
2. Sélectionner le fichier depuis votre machine locale
3. Fichier disponible dans l'environnement CloudShell

---

## 🖥️ Gestion des terminaux

- Possibilité d'ouvrir **plusieurs onglets** simultanément
- Possibilité de **diviser en colonnes** (split view)

```
CloudShell
├── Terminal 1 (onglet principal)
├── Terminal 2 (nouvel onglet)
└── Terminal 3 (split en colonne)
```

---

## 📊 CloudShell vs Terminal Local

| Fonctionnalité           | CloudShell           | Terminal Local             |
| ------------------------ | -------------------- | -------------------------- |
| **Configuration**        | ✅ Automatique       | ⚙️ `aws configure` requis  |
| **Credentials**          | ✅ Automatiques      | 🔑 Access Keys requises    |
| **Région par défaut**    | ✅ Région console    | ⚙️ Configurée manuellement |
| **Disponibilité**        | ⚠️ Certaines régions | ✅ Partout                 |
| **Persistance fichiers** | ✅ Oui               | ✅ Oui                     |
| **Coût**                 | 🆓 Gratuit           | 🆓 Gratuit                 |

---

## 📝 Points clés à retenir

> ✅ CloudShell = terminal **intégré à la console AWS**  
> ✅ **Gratuit** à utiliser  
> ✅ **Pas disponible** dans toutes les régions  
> ✅ Credentials **automatiques** (pas de configuration nécessaire)  
> ✅ Région par défaut = **région actuellement sélectionnée** dans la console  
> ✅ Fichiers **persistants** entre les sessions  
> ✅ Fonctionnalité **upload/download** très pratique  
> ✅ Terminal local + Access Keys = **alternative valide** à CloudShell

---

**Prochaine étape** : IAM Roles (Rôles IAM)
