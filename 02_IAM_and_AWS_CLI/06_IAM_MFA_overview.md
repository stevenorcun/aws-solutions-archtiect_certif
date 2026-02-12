# IAM Security - Password Policy & MFA

## 🔐 Mécanismes de protection des comptes IAM

AWS propose **2 mécanismes de défense** principaux pour protéger les users et groupes :

1. **Password Policy** (Politique de mot de passe)
2. **MFA** (Multi-Factor Authentication)

---

## 🔑 Password Policy (Politique de mot de passe)

**Principe** : Plus le mot de passe est fort, plus le compte est sécurisé

### Options configurables

| Option                           | Description                                            | Exemple                                    |
| -------------------------------- | ------------------------------------------------------ | ------------------------------------------ |
| **Longueur minimale**            | Nombre minimum de caractères                           | Minimum 8 caractères                       |
| **Types de caractères**          | Exiger des caractères spécifiques                      | Majuscules, minuscules, chiffres, symboles |
| **Changement par l'utilisateur** | Autoriser les users à changer leur propre mot de passe | Activé/Désactivé                           |
| **Expiration du mot de passe**   | Forcer le changement après X jours                     | Tous les 90 jours                          |
| **Prévention de réutilisation**  | Empêcher de réutiliser d'anciens mots de passe         | Interdit de réutiliser les 5 derniers      |

### Types de caractères exigibles

- ✅ Lettres **MAJUSCULES** (A-Z)
- ✅ Lettres **minuscules** (a-z)
- ✅ **Chiffres** (0-9)
- ✅ Caractères **non-alphanumériques** (!, ?, @, #, etc.)

### Avantages

> 🛡️ **Protection contre les attaques par force brute** (brute force attacks)

---

## 📱 MFA - Multi-Factor Authentication

**Définition** : Authentification à deux facteurs combinant :

1. **Ce que vous savez** : Votre mot de passe
2. **Ce que vous possédez** : Un appareil physique/virtuel

### Pourquoi utiliser MFA ?

**Scénario** : Protéger les comptes avec des privilèges élevés

- ✅ **Root account** : OBLIGATOIRE
- ✅ **IAM users administrateurs** : FORTEMENT RECOMMANDÉ
- ✅ **Tous les IAM users** : Recommandé

**Risques sans MFA** :

- Changement de configurations critiques
- Suppression de ressources
- Accès non autorisé aux données

### Comment fonctionne MFA ?

**Exemple avec Alice** :

```
Login réussi = Password + MFA Token
              └─────┬─────┘   └────┬────┘
              Ce qu'elle sait   Ce qu'elle possède
```

**Étapes de connexion** :

1. Alice entre son **mot de passe**
2. Alice entre le **code MFA** généré par son appareil
3. ✅ Connexion réussie

### Avantage principal

> 🔒 **Même si le mot de passe est volé ou hacké**, le compte reste protégé car le hacker a également besoin de l'appareil physique d'Alice (ex: son téléphone)

---

## 🔧 Options d'appareils MFA dans AWS

### 1️⃣ Virtual MFA Device (Appareil MFA virtuel)

**Applications supportées** :

| Application              | Caractéristiques                                                     |
| ------------------------ | -------------------------------------------------------------------- |
| **Google Authenticator** | 📱 Fonctionne sur un seul téléphone à la fois                        |
| **Authy**                | 📱 Support de **plusieurs tokens sur un seul appareil** (RECOMMANDÉ) |

**Avantages** :

- ✅ Un seul appareil peut gérer **plusieurs comptes AWS**
- ✅ Un seul appareil peut gérer **plusieurs IAM users**
- ✅ Solution la plus simple et gratuite
- ✅ Utilisé dans le hands-on du cours

**Exemple d'utilisation** :

```
Authy (1 téléphone)
├── Root Account AWS
├── IAM User 1
├── IAM User 2
└── Autre compte AWS
```

---

### 2️⃣ U2F Security Key (Clé de sécurité U2F)

**Exemple** : **YubiKey** par Yubico (tiers externe à AWS)

**Caractéristiques** :

- 🔑 **Appareil physique** (comme une clé USB)
- 🔑 Peut être attaché à un porte-clés
- 🔑 **Une seule clé** peut supporter **plusieurs root accounts et IAM users**

**Avantages** :

- ✅ Très pratique (toujours sur vous)
- ✅ Pas besoin d'une clé par utilisateur
- ✅ Support multi-comptes

---

### 3️⃣ Hardware Key Fob MFA Device

**Exemple** : Appareil fourni par **Gemalto** (tiers externe à AWS)

**Caractéristiques** :

- 🔐 Appareil physique générant des codes
- 🔐 Similaire aux tokens bancaires

---

### 4️⃣ Hardware Key Fob for AWS GovCloud

**Exemple** : Appareil fourni par **SurePassID** (tiers externe à AWS)

**Utilisation** :

- 🏛️ **Réservé aux clients AWS GovCloud** (cloud gouvernemental US)
- 🏛️ Contraintes de sécurité gouvernementales

---

## 📊 Comparaison des appareils MFA

| Type                  | Coût        | Praticité  | Multi-comptes | Recommandé pour        |
| --------------------- | ----------- | ---------- | ------------- | ---------------------- |
| **Virtual MFA**       | 🆓 Gratuit  | ⭐⭐⭐⭐⭐ | ✅ Oui        | Tous les utilisateurs  |
| **U2F Key (YubiKey)** | 💰 ~50€     | ⭐⭐⭐⭐   | ✅ Oui        | Utilisateurs fréquents |
| **Hardware Key Fob**  | 💰💰 Payant | ⭐⭐⭐     | ❌ Non        | Entreprises            |
| **GovCloud Key Fob**  | 💰💰 Payant | ⭐⭐⭐     | ❌ Non        | Gouvernement US        |

---

## 📝 Points clés à retenir pour l'examen

> ✅ **Password Policy** = Protection contre les attaques par force brute  
> ✅ **MFA** = Obligatoire pour le root account, recommandé pour tous les users  
> ✅ **MFA = Password + Device** (ce que vous savez + ce que vous possédez)  
> ✅ **Virtual MFA** (Google Authenticator, Authy) = Solution la plus courante  
> ✅ **U2F Security Key** (YubiKey) = Appareil physique USB  
> ✅ **Hardware Key Fob** = Appareil physique générant des codes  
> ✅ **GovCloud Key Fob** = Spécial pour AWS GovCloud (gouvernement US)  
> ✅ Tous les appareils MFA (sauf Virtual) sont fournis par des **tiers externes** à AWS

---

## 🎯 Bonnes pratiques de sécurité

| Pratique            | Description                                             |
| ------------------- | ------------------------------------------------------- |
| **Root account**    | ⚠️ Toujours activer MFA                                 |
| **IAM admins**      | ⚠️ Toujours activer MFA                                 |
| **Password Policy** | Configurer dès la création du compte                    |
| **Expiration**      | Forcer le changement tous les 90 jours                  |
| **Complexité**      | Minimum 8 caractères + majuscules + chiffres + symboles |

---

**Prochaine étape** : Hands-on - Configuration du Password Policy et activation de MFA dans la console AWS

** Setting up : On va pouvoir manage le type de password soir celuit par défaut ou personnalisé**

On peut par exemple exiger la réinitialisation du mot de passe tous les 90 jours, exiger des caractères spéciaux, etc.

** MFA : On va pouvoir activer MFA pour le root account et les IAM users. On peut choisir entre un appareil virtuel (Google Authenticator, Authy) ou un appareil physique (YubiKey, Gemalto, SurePassID). Le virtual MFA est la solution la plus courante et gratuite, tandis que les appareils physiques sont payants mais offrent une sécurité renforcée.**
