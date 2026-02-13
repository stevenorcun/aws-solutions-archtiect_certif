# AWS – Gestion de la Facturation et des Budgets

## Objectif

Configurer un budget et une alerte afin de ne pas dépenser trop d'argent sur votre compte AWS pendant ce cours.

---

## 1. Accéder à la console de facturation

1. Cliquez sur votre nom de compte en haut à droite de l'écran.
2. Sélectionnez **Billing and Cost Management**.

> **Problème fréquent :** Si vous êtes connecté en tant qu'utilisateur IAM, vous risquez d'obtenir des erreurs _Access Denied_, même avec des droits administrateur.

### Résoudre le problème d'accès IAM

1. Connectez-vous à votre **compte root**.
2. Cliquez sur votre nom de compte → **Accounts**.
3. Faites défiler jusqu'à **IAM user and role access to billing information**.
4. **Activez** l'accès IAM à la facturation.
5. Retournez dans la console de facturation et rafraîchissez la page.

---

## 2. Explorer les informations de facturation

### Tableau de bord principal

- Coût du mois en cours
- Prévision du coût total pour le mois
- Total du mois précédent

### Détail des factures (Bills)

1. Cliquez sur **Bills** dans le menu de gauche.
2. Sélectionnez le mois souhaité (ex. : décembre 2023).
3. Faites défiler jusqu'à **Charges by service**.

Exemple de détail pour **EC2 (EU Ireland)** : 43 $

- NAT Gateway : 35 $
- EBS, Elastic IP, etc.

> Cette vue permet de diagnostiquer précisément l'origine de vos coûts.

### Free Tier

Cliquez sur **Free Tier** dans le menu de gauche pour voir l'utilisation actuelle vs la limite gratuite, ainsi que la prévision d'utilisation.

> Si une prévision apparaît **en rouge**, vous risquez d'être facturé. Désactivez immédiatement les ressources concernées.

---

## 3. Configurer un budget

1. Cliquez sur **Budgets** dans le menu de gauche.
2. Cliquez sur **Create a budget**.
3. Choisissez **Template (Simplified)**.

### Option A – Budget zéro dépense

| Paramètre | Valeur               |
| --------- | -------------------- |
| Template  | Zero Spend Budget    |
| Nom       | My Zero Spend Budget |
| Email     | votre@email.com      |

→ Alerte dès le premier centime dépensé.

### Option B – Budget mensuel (ex. 10 $)

| Paramètre | Valeur              |
| --------- | ------------------- |
| Template  | Monthly Cost Budget |
| Montant   | 10 $                |
| Email     | votre@email.com     |

Alertes déclenchées :

- Dépenses réelles à **85 %** du budget
- Dépenses réelles à **100 %** du budget
- Dépenses prévisionnelles à **100 %** du budget

---

## Récapitulatif

| Outil         | Utilité                                                   |
| ------------- | --------------------------------------------------------- |
| **Bills**     | Détail des factures par service et par mois               |
| **Free Tier** | Suivi de l'utilisation gratuite et alertes de dépassement |
| **Budgets**   | Alertes email selon des seuils de dépense définis         |

> **Conseil :** Si vous éteignez vos ressources après chaque TP, vous ne devriez pas être facturé. Les budgets restent une bonne pratique en cas d'oubli.
