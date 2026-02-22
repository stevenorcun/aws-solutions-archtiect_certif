# EC2 Instance Storage – EBS Volumes (Introduction)

## Qu'est-ce qu'un EBS Volume ?

**EBS** = **Elastic Block Store**

C'est un **disque réseau** (network drive) que vous pouvez attacher à vos instances EC2 pendant qu'elles tournent.

> 💡 Vous les avez déjà utilisés sans le savoir ! Chaque instance EC2 a au moins un volume EBS root par défaut.

---

## Caractéristiques principales

### 1. Persistance des données

- Les données **survivent** à l'arrêt de l'instance
- Les données **survivent** à la terminaison de l'instance (si configuré)
- Vous pouvez recréer une instance et rattacher le même volume EBS → **vos données sont toujours là**

### 2. Attachement à une instance

- Au niveau CCP (Certified Cloud Practitioner), un volume EBS peut être attaché à **1 seule instance à la fois**
- Peut être **détaché et réattaché** rapidement à une autre instance

### 3. Lié à une Availability Zone (AZ)

- Un volume créé dans **us-east-1a** ne peut être attaché qu'à une instance dans **us-east-1a**
- ❌ Impossible d'attacher directement un volume de us-east-1a vers us-east-1b
- ✅ Mais possible via **snapshot** (on verra ça plus tard)

---

## Analogie : Le "USB stick réseau"

Pensez à un volume EBS comme une **clé USB réseau** :

- Vous pouvez la débrancher d'un ordinateur et la rebrancher sur un autre
- Mais elle n'est **pas physique** → elle passe par le réseau
- Comme c'est par le réseau, il peut y avoir une **petite latence**

---

## EBS = Network Drive (Disque réseau)

### Ce que ça implique

```
[Instance EC2]  ←─ réseau ─→  [EBS Volume]
   (compute)                    (storage)
```

✅ **Avantages** :

- Détachable/réattachable rapidement
- Pas besoin d'éteindre l'instance pour changer de volume

⚠️ **Inconvénient** :

- Légère latence (communication réseau)

---

## Provisionnement de capacité

Vous devez définir **à l'avance** :

| Paramètre  | Description                             |
| ---------- | --------------------------------------- |
| **Taille** | Ex: 10 GB, 100 GB, 1 TB                 |
| **IOPS**   | I/O Operations Per Second (performance) |

💰 **Facturation** : Vous payez pour la capacité provisionnée, **même si vous ne l'utilisez pas entièrement**.

✅ Vous pouvez **augmenter** la capacité et les IOPS plus tard si besoin.

---

## Schéma : EBS Volumes et Instances

### Dans une seule AZ (us-east-1a)

```
╔═══════════════════════════════════════════════════════╗
║  AVAILABILITY ZONE : us-east-1a                       ║
║                                                        ║
║  [Instance EC2 #1]                                    ║
║       ↕ (réseau)                                      ║
║  [EBS Volume 10 GB] ← Attaché                         ║
║                                                        ║
║                                                        ║
║  [Instance EC2 #2]                                    ║
║       ↕ (réseau)                                      ║
║  [EBS Volume 20 GB] ← Attaché                         ║
║                                                        ║
║                                                        ║
║  [EBS Volume 30 GB] ← Non attaché (disponible)        ║
║                                                        ║
╚═══════════════════════════════════════════════════════╝
```

### Règles importantes

✅ **Possible** :

- 1 instance → 2+ volumes EBS (comme 2 clés USB sur le même PC)
- 1 volume EBS → non attaché (reste disponible pour plus tard)

❌ **Impossible** (au niveau CCP) :

- 1 volume EBS → 2 instances simultanément

---

## Multi-AZ : Les volumes sont liés à leur AZ

```
╔══════════════════════════╗    ╔══════════════════════════╗
║  AZ : us-east-1a         ║    ║  AZ : us-east-1b         ║
║                          ║    ║                          ║
║  [Instance EC2 #1]       ║    ║  [Instance EC2 #3]       ║
║       ↕                  ║    ║       ↕                  ║
║  [EBS Volume A]          ║    ║  [EBS Volume C]          ║
║                          ║    ║                          ║
║  [Instance EC2 #2]       ║    ║  [EBS Volume D]          ║
║       ↕                  ║    ║  (non attaché)           ║
║  [EBS Volume B]          ║    ║                          ║
║                          ║    ║                          ║
╚══════════════════════════╝    ╚══════════════════════════╝

❌ Volume A ne peut PAS être attaché à Instance #3
✅ Volume A peut être attaché à Instance #1 ou #2 uniquement
```

---

## Delete on Termination (Important pour l'examen)

Quand vous créez une instance EC2, chaque volume EBS a un attribut **Delete on Termination** :

| Volume                      | Delete on Termination par défaut            |
| --------------------------- | ------------------------------------------- |
| **Root volume** (OS)        | ✅ Activé (volume supprimé avec l'instance) |
| **Volumes supplémentaires** | ❌ Désactivé (volumes conservés)            |

### Schéma du comportement

```
CONFIGURATION PAR DÉFAUT
────────────────────────
[Instance EC2]
  ├─ /dev/xvda (root) [Delete on Termination: ✅ Oui]
  └─ /dev/xvdb (data) [Delete on Termination: ❌ Non]

Quand vous terminez l'instance :
  ├─ Root volume → ❌ SUPPRIMÉ
  └─ Data volume → ✅ CONSERVÉ
```

### Cas d'usage

**Scénario 1 : Sauvegarder les données root**

```
Vous voulez garder le root volume même après terminaison
  ↓
Désactiver "Delete on Termination" pour le root
  ↓
Terminer l'instance → le root volume reste disponible ✅
```

**Scénario 2 : Nettoyer automatiquement**

```
Volume de données temporaires (logs, cache)
  ↓
Activer "Delete on Termination"
  ↓
Terminer l'instance → tout est nettoyé automatiquement ✅
```

---

## Configuration lors de la création

Dans la console EC2, section **Configure Storage** :

```
┌─────────────┬──────┬──────┬─────────────────────────┐
│ Device      │ Size │ Type │ Delete on Termination   │
├─────────────┼──────┼──────┼─────────────────────────┤
│ /dev/xvda   │ 8 GB │ gp3  │ ✅ (root - par défaut)  │
│ /dev/xvdb   │ 20GB │ gp3  │ ❌ (data - par défaut)  │
└─────────────┴──────┴──────┴─────────────────────────┘
```

Vous pouvez **cocher/décocher** cette option selon vos besoins.

---

## Récapitulatif

| Caractéristique           | Détail                                        |
| ------------------------- | --------------------------------------------- |
| **Type**                  | Disque réseau (network drive)                 |
| **Persistance**           | Les données survivent aux arrêts/redémarrages |
| **Attachement**           | 1 volume → 1 instance (niveau CCP)            |
| **Scope**                 | Lié à une AZ spécifique                       |
| **Capacité**              | Provisionnée à l'avance (taille + IOPS)       |
| **Facturation**           | Basée sur la capacité provisionnée            |
| **Détachable**            | Oui, rapidement                               |
| **Delete on Termination** | Root: Oui par défaut / Data: Non par défaut   |

---

## Points clés pour l'examen

✅ **EBS = disque réseau**, pas physique  
✅ **Persistance** des données après arrêt/terminaison  
✅ **1 volume → 1 instance** (au niveau CCP)  
✅ **Lié à une AZ** (mais snapshot permet de déplacer)  
✅ **Delete on Termination** : root=Oui, data=Non par défaut  
✅ Capacité **provisionnée à l'avance** et facturable  
✅ Peut être **non attaché** et attaché plus tard
