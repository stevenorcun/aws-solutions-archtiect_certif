# Qu'est-ce qu'une Carte Réseau ? (Bases Informatiques)

## Définition

Une **carte réseau** (ou **NIC** = Network Interface Card) est un composant matériel qui permet à un ordinateur de **communiquer avec d'autres appareils** via un réseau.

---

## Analogie simple

Imaginez votre ordinateur comme une **maison** :

- Les **programmes** (navigateur, jeux, etc.) sont les **pièces** de la maison
- La **carte réseau** est la **porte d'entrée** de la maison
- Le **câble réseau ou WiFi** est le **chemin** vers les autres maisons
- **Internet** est le **quartier/ville** où toutes les maisons communiquent

**Sans carte réseau = pas de porte = vous êtes enfermé, isolé du monde.**

---

## À quoi ça ressemble physiquement ?

### Sur un ordinateur de bureau

```
┌─────────────────────────────────────┐
│  Carte Mère (motherboard)           │
│                                     │
│  [CPU] [RAM] [Disque dur]           │
│                                     │
│  ┌───────────────────────┐         │
│  │  CARTE RÉSEAU         │         │
│  │                       │         │
│  │  Puce électronique    │         │
│  │  qui gère le réseau   │         │
│  └───────────┬───────────┘         │
│              │                      │
└──────────────┼──────────────────────┘
               │
          [Port RJ45] ← Ici vous branchez le câble Ethernet
               │
               └─→ Vers votre box Internet / routeur
```

### Sur un laptop

C'est la même chose, mais intégré directement dans la carte mère + une antenne WiFi.

---

## Rôle de la carte réseau

### 1. Envoyer et recevoir des données

Quand vous tapez `www.google.com` dans votre navigateur :

```
[Votre navigateur]
     ↓ "Je veux aller sur Google"
[Système d'exploitation]
     ↓ "Prépare la requête réseau"
[CARTE RÉSEAU] ← C'est elle qui fait le travail !
     ↓ Convertit les données en signaux électriques
[Câble Ethernet ou WiFi]
     ↓ Voyage sur le réseau
[Routeur/Box Internet]
     ↓
[Internet]
     ↓
[Serveurs de Google]
```

### 2. Identifier votre ordinateur sur le réseau

Chaque carte réseau possède une **adresse MAC** (Media Access Control) unique au monde, comme une plaque d'immatriculation.

```
Exemple de MAC address : 00:1A:2B:3C:4D:5E
```

C'est comme si chaque maison avait un numéro unique dans le quartier.

### 3. Gérer l'adresse IP

La carte réseau reçoit aussi une **adresse IP** (donnée par votre routeur ou serveur DHCP) :

```
Adresse MAC  : 00:1A:2B:3C:4D:5E  ← Identifiant physique permanent
Adresse IP   : 192.168.1.25       ← Identifiant logique temporaire
```

---

## Exemple concret dans votre vie quotidienne

### Vous regardez une vidéo YouTube

```
1. Vous cliquez sur "Play"
   ↓
2. Votre navigateur demande : "Donne-moi la vidéo !"
   ↓
3. La CARTE RÉSEAU envoie la demande via WiFi
   ↓
4. La requête voyage jusqu'aux serveurs YouTube
   ↓
5. YouTube envoie les données vidéo
   ↓
6. La CARTE RÉSEAU reçoit les paquets de données
   ↓
7. Le navigateur affiche la vidéo
```

**Sans carte réseau :** Votre ordinateur ne pourrait jamais envoyer la demande ni recevoir la vidéo. Vous seriez complètement hors ligne.

---

## Types de cartes réseau

| Type                 | Connexion          | Vitesse typique    |
| -------------------- | ------------------ | ------------------ |
| **Ethernet (câble)** | Port RJ45          | 1 Gbps (1000 Mbps) |
| **WiFi**             | Sans fil (antenne) | 300-1200 Mbps      |
| **Fibre optique**    | Port SFP           | 10 Gbps et plus    |

---

## Et dans AWS (ENI) ?

Dans le cloud, il n'y a **pas de carte réseau physique** dans votre instance EC2.

AWS simule une carte réseau **virtuellement** avec un ENI :

```
MONDE PHYSIQUE                    MONDE CLOUD AWS
─────────────────                 ───────────────

[Votre PC]                        [Instance EC2 virtuelle]
    ↓                                  ↓
[Carte réseau physique]           [ENI = Carte réseau VIRTUELLE]
    ↓                                  ↓
[Câble vers routeur]              [Réseau virtuel AWS (VPC)]
    ↓                                  ↓
[Internet]                        [Internet]
```

AWS **émule** le comportement d'une vraie carte réseau, mais en logiciel. C'est pour ça qu'on peut la détacher, la déplacer, en créer plusieurs... chose impossible avec une vraie carte physique soudée !

---

## Résumé

| Question                 | Réponse                                               |
| ------------------------ | ----------------------------------------------------- |
| **C'est quoi ?**         | La porte d'entrée/sortie réseau de votre ordinateur   |
| **Physiquement ?**       | Puce électronique + port (RJ45, WiFi, etc.)           |
| **Rôle ?**               | Envoyer/recevoir des données sur le réseau            |
| **Identifiant unique ?** | Adresse MAC (permanent) + IP (temporaire)             |
| **Dans AWS ?**           | ENI = version virtuelle/logicielle de la carte réseau |
