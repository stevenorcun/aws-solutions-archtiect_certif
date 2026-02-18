# AWS – EC2 Purchase Options : Cas d'usage concrets

## 🔵 On-Demand

**Qui ?** Une startup qui teste une nouvelle application

**Scénario :**

- Vous lancez un MVP (Minimum Viable Product)
- Vous ne savez pas si l'app va marcher
- Le trafic est imprévisible
- Vous voulez arrêter les serveurs le week-end pour économiser

**Pourquoi On-Demand ?**

- Pas d'engagement → vous pouvez tout arrêter si ça ne marche pas
- Facturation à la seconde → vous payez uniquement ce que vous utilisez
- Flexibilité totale

---

## 🟢 Reserved Instances

**Qui ?** Une banque qui héberge sa base de données clients

**Scénario :**

- La base de données tourne **24/7/365** depuis des années
- Aucune variation de charge
- Budget IT stable et prévisible
- Engagement sur 3 ans acceptable

**Pourquoi Reserved ?**

- **72% d'économie** sur 3 ans
- Workload stable et prévisible
- ROI garanti sur le long terme

---

## 🟢 Savings Plan

**Qui ?** Une entreprise SaaS en croissance

**Scénario :**

- Vous savez que vous allez dépenser **minimum 500 $/mois** en compute
- Vous voulez tester différents types d'instances (m5, c5, r5...)
- Vous prévoyez d'étendre à d'autres régions dans 6 mois
- Vous voulez des économies sans vous enfermer dans un type d'instance précis

**Pourquoi Savings Plan ?**

- Flexibilité sur le type d'instance
- Vous vous engagez sur un montant, pas sur une config
- Toute dépense au-delà reste en On-Demand

---

## 🔴 Spot Instances

**Qui ?** Une entreprise de bioinformatique qui analyse des génomes

**Scénario :**

- Traitement de données massives par batch
- Chaque analyse prend 2-6 heures
- Si une instance est coupée, le job reprend depuis le dernier checkpoint
- Budget limité, objectif : analyser le max de données au moindre coût

**Pourquoi Spot ?**

- **90% d'économie** vs On-Demand
- Workload interruptible et tolérant aux pannes
- Volume de calcul > rapidité d'exécution

**Autre exemple :** Rendering vidéo, entraînement de modèles ML (avec checkpoints), analyse de logs

---

## ⚫ Dedicated Host

**Qui ?** Une banque avec des licences Microsoft Windows Server existantes

**Scénario :**

- Vous avez acheté 50 licences Windows Server avec vos propres contrats
- Ces licences sont liées au **nombre de sockets physiques** du serveur
- Obligation légale de savoir sur quel matériel vos données sont stockées
- Audit de conformité exigé

**Pourquoi Dedicated Host ?**

- BYOL (Bring Your Own License) → réutiliser vos licences existantes
- Visibilité sur le matériel physique (sockets, cores)
- Conformité réglementaire stricte

**Autre exemple :** Entreprise pharmaceutique avec contraintes HIPAA, cabinet d'avocats avec secret professionnel

---

## ⚫ Dedicated Instance

**Qui ?** Une fintech qui veut isoler son infrastructure sans gérer le hardware

**Scénario :**

- Exigence contractuelle : aucun autre client AWS ne doit partager votre matériel
- Mais vous n'avez **pas besoin de visibilité sur les sockets/cores**
- Pas de licences spéciales à gérer
- Juste besoin d'isolation hardware

**Pourquoi Dedicated Instance ?**

- Isolation garantie sans la complexité du Dedicated Host
- Moins cher qu'un Dedicated Host
- AWS gère le hardware pour vous

---

## 🟡 Capacity Reservation

**Qui ?** Un e-commerce avant le Black Friday

**Scénario :**

- Votre événement de vente a lieu **vendredi 29 novembre à 00h00**
- Vous devez être **sûr à 100%** que 200 instances m5.xlarge seront disponibles en us-east-1a
- Vous ne voulez pas risquer un message "insufficient capacity" à minuit pile
- Vous ne voulez pas payer pour des Reserved Instances toute l'année

**Pourquoi Capacity Reservation ?**

- Garantie de capacité dans une AZ spécifique
- Pas d'engagement long terme (on/off quand vous voulez)
- Combinable avec Savings Plan pour obtenir des remises

**Note :** Vous payez même si vous n'utilisez pas la capacité réservée

---

## 📊 Tableau récapitulatif

| Option                   | Qui ?                          | Pourquoi ?                                  |
| ------------------------ | ------------------------------ | ------------------------------------------- |
| **On-Demand**            | Startup testant un MVP         | Pas d'engagement, trafic imprévisible       |
| **Reserved**             | Banque avec DB 24/7            | Workload stable, économies max sur 1-3 ans  |
| **Savings Plan**         | SaaS en croissance             | Engagement financier, flexibilité technique |
| **Spot**                 | Analyse génomique par batch    | 90% d'économie, workload interruptible      |
| **Dedicated Host**       | Banque avec licences Windows   | BYOL, conformité, visibilité hardware       |
| **Dedicated Instance**   | Fintech avec isolation requise | Isolation sans gestion du hardware          |
| **Capacity Reservation** | E-commerce avant Black Friday  | Garantie de capacité à un moment précis     |
