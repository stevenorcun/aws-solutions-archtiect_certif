# AWS Infrastructure Globale - Régions, AZ & Points of Presence

## 🌍 AWS Régions

**Définition** : Cluster de datacenters géographiquement groupés dans une zone géographique

### Nomenclature

- **Nom** : `us-east-1`, `eu-west-3`, `ap-southeast-2`
- **Correspond à** une localisation géographique (Ohio, Paris, Sydney...)

### Caractéristiques

- La plupart des services AWS sont **liés à une région spécifique**
- Utiliser un service dans une région = **nouvelle instance** dans une autre région
- Régions connectées entre elles par un **réseau privé AWS**

### 4 critères de choix d'une région

| Critère                  | Description                | Exemple                                               |
| ------------------------ | -------------------------- | ----------------------------------------------------- |
| **Compliance**           | Conformité légale          | Données françaises doivent rester en France           |
| **Latency**              | Latence réseau             | Déployer proche des utilisateurs finaux               |
| **Service availability** | Disponibilité des services | Tous les services ne sont pas dans toutes les régions |
| **Pricing**              | Prix                       | Varie selon les régions                               |

---

## 📍 AWS Availability Zones (AZ)

**Définition** : Un ou plusieurs datacenters discrets avec alimentation, réseau et connectivité redondants

### Structure

- Chaque région contient **3 à 6 AZ** (généralement 3)
- **Minimum** = 3 AZ | **Maximum** = 6 AZ

### Exemple - Région Sydney (`ap-southeast-2`)

```
ap-southeast-2
├── ap-southeast-2a
├── ap-southeast-2b
└── ap-southeast-2c
```

### Caractéristiques clés

✅ **Isolées** les unes des autres = protection contre les catastrophes  
✅ Si une AZ tombe, les autres ne sont **pas affectées**  
✅ Connectées entre elles par **réseau haute bande passante et ultra-faible latence**  
✅ Ensemble des AZ d'une zone = forme une **région**

---

## 🚀 AWS Points of Presence (Edge Locations)

### Chiffres clés

- **400+** points de présence
- **90+** villes
- **40+** pays

### Objectif

Délivrer du contenu aux utilisateurs finaux avec la **latence la plus faible possible**

### Utilisation

Services de distribution de contenu (ex: **CloudFront** - CDN)

> **Note** : Détails approfondis vus plus tard dans le cours (section Global Infrastructure)

---

## 📊 Portée des services AWS

### Services Globaux (non liés à une région)

| Service        | Description                  |
| -------------- | ---------------------------- |
| **IAM**        | Identity & Access Management |
| **Route 53**   | DNS                          |
| **CloudFront** | CDN                          |
| **WAF**        | Web Application Firewall     |

### Services Régionaux (liés à une région spécifique)

| Service                          | Description                |
| -------------------------------- | -------------------------- |
| **EC2**                          | Serveurs virtuels          |
| **Elastic Beanstalk**            | Déploiement d'applications |
| **Lambda**                       | Fonctions serverless       |
| **Rekognition**                  | Reconnaissance d'images    |
| **La majorité des services AWS** | -                          |

---

## 🔗 Ressources utiles

- **Region Table** : Vérifier la disponibilité des services par région
- **AWS Infrastructure Map** : Visualisation globale de l'infrastructure

---

## 📝 Points clés à retenir

> ✅ **Région** = groupe d'AZ géographiquement proches  
> ✅ **AZ** = 1+ datacenters isolés mais connectés  
> ✅ Les 4 critères de choix d'une région : Compliance, Latency, Service availability, Pricing  
> ✅ Minimum 3 AZ par région (max 6)  
> ✅ Services globaux (IAM, Route 53) vs services régionaux (EC2, Lambda)  
> ✅ Points of Presence = distribution de contenu avec faible latence
