# AWS – EC2 Spot Instances : Deep Dive

## Fonctionnement

Vous définissez un **prix maximum** que vous êtes prêt à payer. Tant que le prix spot du marché est inférieur à votre max → l'instance tourne. Si le prix dépasse votre max → vous avez **2 minutes** pour réagir.

### Options en cas de dépassement du prix

| Option        | Comportement             | Reprise possible                |
| ------------- | ------------------------ | ------------------------------- |
| **Stop**      | L'instance est arrêtée   | ✅ Oui, quand le prix redescend |
| **Terminate** | L'instance est supprimée | 🔄 Oui, mais repart de zéro     |

---

## Spot Block (SECTION OBSELÈTE => AWS NE FAIT PLUS DE SPOT BLOCKS)

- Bloque une instance spot pour une durée fixe : **1 à 6 heures**
- Aucune interruption pendant ce bloc (cas très rares en pratique)
- Utile si vous avez besoin d'une durée garantie

---

## Spot Request : passer une commande

Une **Spot Request** est un bon de commande envoyé à AWS :

> _"Je veux X instances, je suis prêt à payer max Y€/heure, avec cette configuration."_

### One-time vs Persistent

Imaginez que vous commandez des pizzas :

**One-time** → vous commandez une pizza. Elle arrive, la commande est terminée. Si on vous la reprend, elle est partie, c'est fini.

**Persistent** → vous avez un abonnement. Si on vous reprend votre pizza (prix trop élevé), votre abonnement la recommande automatiquement dès que le prix redevient acceptable.

### ⚠️ Comment terminer correctement des Spot Instances

> **L'ordre est crucial — ceci peut tomber à l'examen.**

C'est pourquoi si vous voulez vraiment arrêter, il faut **d'abord annuler l'abonnement** avant de rendre la pizza, sinon une nouvelle sera automatiquement commandée.

**❌ Mauvais ordre**

1. Terminer les instances → la requête persistante en relance automatiquement de nouvelles

**✅ Bon ordre**

1. **Annuler la Spot Request** (doit être en état : open, active ou disabled)
2. **Terminer les instances** associées

---

## Spot Fleet : un capitaine qui optimise pour vous

Avec une **Spot Request classique**, vous choisissez vous-même le type d'instance et l'AZ. Vous êtes responsable de trouver le meilleur prix.

Avec un **Spot Fleet**, vous déléguez à AWS :

> _"Voici une liste de configurations acceptables (différents types d'instances, différentes AZ). Débrouille-toi pour m'en trouver au meilleur prix."_

C'est comme utiliser Skyscanner : au lieu de chercher vous-même chaque vol, vous donnez vos critères et l'outil trouve la meilleure option.

AWS choisit automatiquement parmi vos **launch pools** selon la stratégie choisie, et s'arrête quand la capacité cible ou le budget est atteint.

### Stratégies d'allocation

| Stratégie                    | Comportement                                                        | Idéal pour                                      |
| ---------------------------- | ------------------------------------------------------------------- | ----------------------------------------------- |
| **Lowest price**             | Lance depuis le pool au prix le plus bas                            | Workloads courts, économies maximales           |
| **Diversified**              | Répartit sur tous les pools définis                                 | Disponibilité, workloads longs                  |
| **Capacity optimized**       | Choisit le pool avec le plus de capacité disponible                 | Besoins en capacité garantie                    |
| **Price capacity optimized** | Choisit d'abord la capacité la plus haute, puis le prix le plus bas | ✅ Meilleur choix pour la plupart des workloads |

---

## Résumé

|              | Spot Instance simple            | Spot Fleet                  |
| ------------ | ------------------------------- | --------------------------- |
| Contrôle     | Vous choisissez le type et l'AZ | AWS choisit parmi vos pools |
| Optimisation | Manuelle                        | Automatique                 |
| Économies    | Jusqu'à 90%                     | Encore plus optimisé        |
