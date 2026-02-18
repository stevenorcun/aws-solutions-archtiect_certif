# AWS – Elastic Network Interface (ENI)

## Qu'est-ce qu'un ENI ?

Un **ENI** (Elastic Network Interface) est une **carte réseau virtuelle** qui donne à une instance EC2 (ou d'autres services AWS) un accès au réseau.

C'est un composant logique dans un VPC qui peut être créé, attaché, détaché et déplacé entre instances.

---

## Schéma de base

```
╔════════════════════════════════════════════╗
║  AVAILABILITY ZONE : us-east-1a            ║
║                                            ║
║  ┌──────────────────────────────────┐     ║
║  │  Instance EC2                    │     ║
║  │                                  │     ║
║  │  ┌────────────────────────┐     │     ║
║  │  │  eth0 (ENI primaire)   │     │     ║
║  │  │                        │     │     ║
║  │  │  • IP privée: 10.0.1.5 │     │     ║
║  │  │  • IP publique (opt.)  │     │     ║
║  │  │  • Security Groups     │     │     ║
║  │  │  • MAC address         │     │     ║
║  │  └────────────────────────┘     │     ║
║  │                                  │     ║
║  └──────────────────────────────────┘     ║
║                                            ║
╚════════════════════════════════════════════╝
```

---

## Attributs d'un ENI

| Attribut                 | Description                            |
| ------------------------ | -------------------------------------- |
| **Primary private IPv4** | IP privée principale (obligatoire)     |
| **Secondary IPv4**       | IP privées supplémentaires (optionnel) |
| **Elastic IP**           | IP publique fixe (1 par IP privée)     |
| **Public IPv4**          | IP publique standard (optionnel)       |
| **Security Groups**      | 1 ou plusieurs groupes de sécurité     |
| **MAC Address**          | Adresse physique unique                |

---

## ENI multiples sur une instance

Une instance EC2 peut avoir **plusieurs ENI** attachés.

```
╔════════════════════════════════════════════════════════╗
║  Instance EC2                                          ║
║                                                        ║
║  ┌─────────────────────┐    ┌─────────────────────┐  ║
║  │  eth0 (primaire)    │    │  eth1 (secondaire)  │  ║
║  │                     │    │                     │  ║
║  │  • 10.0.1.5         │    │  • 10.0.1.8         │  ║
║  │  • SG-web           │    │  • SG-admin         │  ║
║  └─────────────────────┘    └─────────────────────┘  ║
║                                                        ║
╚════════════════════════════════════════════════════════╝
```

**Cas d'usage :**

- Séparer le trafic (web public sur eth0, admin sur eth1)
- Plusieurs IPs privées pour héberger plusieurs sites
- Licensing basé sur MAC address

---

## Déplacement d'ENI pour failover

Les ENI peuvent être **détachés et rattachés** à d'autres instances à la volée, ce qui permet des stratégies de haute disponibilité.

### Scénario : Failover automatique

```
AVANT (Instance principale active)
╔═══════════════════════════╗    ╔═══════════════════════════╗
║  Instance EC2 #1 (active) ║    ║  Instance EC2 #2 (backup) ║
║                           ║    ║                           ║
║  eth0: 10.0.1.5           ║    ║  eth0: 10.0.1.9           ║
║  eth1: 10.0.1.20 ✅       ║    ║                           ║
╚═══════════════════════════╝    ╚═══════════════════════════╝
      ↑
Application accède à 10.0.1.20


APRÈS (Instance #1 tombe, failover vers #2)
╔═══════════════════════════╗    ╔═══════════════════════════╗
║  Instance EC2 #1 ❌ DOWN  ║    ║  Instance EC2 #2 (active) ║
║                           ║    ║                           ║
║  eth0: 10.0.1.5           ║    ║  eth0: 10.0.1.9           ║
║                           ║    ║  eth1: 10.0.1.20 ✅       ║
╚═══════════════════════════╝    ╚═══════════════════════════╝
                                       ↑
                    Application accède toujours à 10.0.1.20
```

**Avantages :**

- L'IP privée **10.0.1.20** reste la même
- Pas besoin de reconfigurer les clients
- Failover transparent et rapide

---

## ⚠️ Contrainte importante

Les ENI sont **liés à une Availability Zone**.

```
✅ POSSIBLE
AZ us-east-1a
├─ Instance A ← ENI créé dans us-east-1a
└─ Instance B ← Peut recevoir ce même ENI

❌ IMPOSSIBLE
AZ us-east-1a
└─ Instance A ← ENI créé dans us-east-1a

AZ us-east-1b
└─ Instance C ← Ne peut PAS recevoir cet ENI
```

Un ENI créé dans **us-east-1a** ne peut être attaché qu'à des instances dans **us-east-1a**.

---

## Création indépendante

Les ENI peuvent être créés **indépendamment** des instances EC2 :

1. Créer un ENI via la console EC2
2. Le conserver sans l'attacher
3. L'attacher plus tard à une instance (à la volée)
4. Le détacher et le réutiliser ailleurs

**Cas d'usage :**

- Préparer une configuration réseau avant de lancer l'instance
- Garder une IP fixe même si l'instance est terminée
- Simplifier les migrations et les tests

---

## Récapitulatif

| Caractéristique | Détail                                              |
| --------------- | --------------------------------------------------- |
| **Nature**      | Carte réseau virtuelle                              |
| **Attachement** | Peut être créé indépendamment et attaché à la volée |
| **Mobilité**    | Déplaçable entre instances (dans la même AZ)        |
| **IPs**         | 1+ privées, 0-1 publique/élastique par IP privée    |
| **Scope**       | Limité à une AZ                                     |
| **Usage**       | Failover, multi-IP, séparation réseau               |
