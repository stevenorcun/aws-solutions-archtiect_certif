# Amazon EC2 – Introduction

## Qu'est-ce qu'EC2 ?

EC2 (**Elastic Compute Cloud**) est l'un des services les plus populaires d'AWS. Il représente l'**Infrastructure as a Service (IaaS)** sur AWS et illustre parfaitement le concept du cloud : louer des ressources de calcul à la demande.

EC2 n'est pas un service unique, il regroupe plusieurs composants :

- **EC2 Instances** – machines virtuelles à louer
- **EBS Volumes** – stockage sur des disques virtuels
- **Elastic Load Balancer** – distribution de charge entre machines
- **Auto-Scaling Group (ASG)** – mise à l'échelle automatique des services

---

## Options de configuration d'une instance EC2

### Système d'exploitation (OS)

- Linux _(le plus populaire)_
- Windows
- macOS

### Puissance de calcul

- **CPU** – nombre de cœurs
- **RAM** – mémoire vive

### Stockage

- Via le réseau : **EBS** ou **EFS**
- Attaché physiquement à la machine : **EC2 Instance Store**

### Réseau

- Type et vitesse de la carte réseau
- Type d'adresse IP publique

### Sécurité

- **Security Group** – règles de pare-feu de l'instance

### Script de démarrage

- **EC2 User Data** – script de configuration au premier lancement

---

## EC2 User Data (Bootstrap Script)

Le **bootstrapping** consiste à lancer des commandes automatiquement au démarrage de la machine.

Caractéristiques importantes :

- Le script ne s'exécute qu'**une seule fois**, au premier démarrage
- Il s'exécute avec les droits **root** (sudo)

Cas d'usage typiques :

- Installer des mises à jour
- Installer des logiciels
- Télécharger des fichiers depuis Internet
- Toute autre tâche d'initialisation

> **Note :** Plus le script User Data est long, plus le démarrage de l'instance sera long.

---

## Résumé

| Composant          | Rôle                                         |
| ------------------ | -------------------------------------------- |
| **EC2 Instance**   | Machine virtuelle (OS, CPU, RAM, stockage)   |
| **Security Group** | Pare-feu / règles réseau                     |
| **EC2 User Data**  | Script d'automatisation au premier démarrage |
| **EBS / EFS**      | Stockage réseau                              |
| **Instance Store** | Stockage physique attaché                    |
| **ELB / ASG**      | Distribution de charge et mise à l'échelle   |

> EC2 est fondamental pour comprendre le cloud AWS. Maîtriser ce service, c'est maîtriser le concept même de cloud computing.
