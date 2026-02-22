# AWS – EBS Volumes : Démonstration Pratique

## Observer les volumes EBS d'une instance

### Depuis la console EC2

1. Sélectionner votre instance
2. Onglet **Storage**
3. Section **Block devices**

**Résultat** :

```
Root device: /dev/xvda
  └─ Volume ID: vol-0abc123...
  └─ Size: 8 GB
  └─ Status: attached
```

### Accéder à la console Volumes

**Menu gauche** → **Elastic Block Store** → **Volumes**

Vous verrez tous vos volumes EBS :

- Volume ID
- Size
- Type (gp2, gp3, io1...)
- State (in-use, available)
- Attached instance

---

## Créer un nouveau volume EBS

### Étape 1 : Créer le volume

1. **Volumes** → **Create volume**
2. Paramètres :
   - **Volume Type** : gp2 (General Purpose SSD)
   - **Size** : 2 GB
   - **Availability Zone** : ⚠️ **MÊME AZ que votre instance EC2**

**Comment trouver l'AZ de votre instance ?**

```
Instance EC2 → Onglet Networking
  └─ Availability Zone: eu-west-1b
```

3. **Create volume**

---

### Étape 2 : Attacher le volume à l'instance

```
AVANT
─────
[Instance EC2 en eu-west-1b]
  └─ Volume root: 8 GB (attaché)

[Nouveau volume: 2 GB] (disponible, non attaché)
```

**Actions** :

1. Sélectionner le nouveau volume (2 GB)
2. **Actions** → **Attach volume**
3. Sélectionner votre instance
4. **Attach volume**

```
APRÈS
─────
[Instance EC2 en eu-west-1b]
  ├─ Volume root: 8 GB (/dev/xvda)
  └─ Volume data: 2 GB (/dev/xvdb) ✅ Nouveau !
```

---

### Vérification

Retourner sur l'instance → Onglet **Storage** → **Block devices**

```
┌─────────────┬──────────┬────────┬────────────────────┐
│ Device      │ Volume   │ Size   │ Delete on Term.    │
├─────────────┼──────────┼────────┼────────────────────┤
│ /dev/xvda   │ vol-abc  │ 8 GB   │ ✅ Yes             │
│ /dev/xvdb   │ vol-def  │ 2 GB   │ ❌ No              │
└─────────────┴──────────┴────────┴────────────────────┘
```

---

## Utiliser le nouveau volume (hors scope du cours)

Pour réellement utiliser ce volume dans Linux, il faut :

1. Le formater
2. Le monter dans le système de fichiers

**Documentation AWS** : "Make an Amazon EBS volume available for use on Linux"

> Ce n'est **pas requis pour l'examen**, juste bon à savoir.

---

## Démonstration : EBS lié à une AZ

### Test : Créer un volume dans une AZ différente

**Configuration** :

- Instance EC2 : **eu-west-1b**
- Nouveau volume : **eu-west-1a** (AZ différente)
- Size : 2 GB, Type: gp2

**Résultat** :

```
[Volume en eu-west-1a]
  ↓
Actions → Attach volume
  ↓
❌ Aucune instance disponible !
```

**Pourquoi ?** L'instance est dans **eu-west-1b**, le volume dans **eu-west-1a** → **incompatible**.

---

### Nettoyage du volume incompatible

```
Sélectionner le volume (eu-west-1a)
  ↓
Actions → Delete volume
  ↓
Volume supprimé en quelques secondes ✅
```

**Avantage du cloud** : Création/suppression instantanée de ressources.

---

## Delete on Termination : Démonstration

### Configuration actuelle

```
[Instance EC2]
  ├─ Root (8 GB) : Delete on Termination = ✅ Yes
  └─ Data (2 GB) : Delete on Termination = ❌ No
```

**Comment vérifier ?**

1. Instance → **Storage** → **Block devices**
2. Colonne **Delete on Termination**

---

### D'où vient cette configuration ?

Lors de la création de l'instance EC2 :

**Section Configure Storage** → **Advanced**

```
┌─────────────┬──────┬──────────────────────────┐
│ Device      │ Size │ Delete on Termination    │
├─────────────┼──────┼──────────────────────────┤
│ Root        │ 8 GB │ ✅ (par défaut)          │
│ New volume  │ 2 GB │ ❌ (par défaut)          │
└─────────────┴──────┴──────────────────────────┘
```

Vous pouvez **modifier** ces valeurs avant de lancer l'instance.

---

### Test : Terminer l'instance

**Prédiction** :

- Root volume (8 GB) → ❌ Sera supprimé
- Data volume (2 GB) → ✅ Sera conservé

**Actions** :

1. **Instance State** → **Terminate instance**
2. Confirmer
3. Aller dans **Volumes** et rafraîchir

---

### Résultat après terminaison

```
AVANT
─────
Volumes:
  ├─ vol-abc (8 GB, root) - in-use
  └─ vol-def (2 GB, data) - in-use


APRÈS TERMINAISON
─────────────────
Volumes:
  └─ vol-def (2 GB, data) - available ✅

vol-abc (8 GB, root) → ❌ SUPPRIMÉ automatiquement
```

**Le volume de 2 GB est maintenant disponible** et peut être attaché à une nouvelle instance !

---

## Cas d'usage : Conserver le root volume

### Scénario

Vous voulez **garder les données du root** même après terminaison (logs système, config personnalisée).

**Solution** :

1. Lors de la création → **Configure Storage** → **Advanced**
2. Décocher **Delete on Termination** pour le root volume
3. Lancer l'instance

**Résultat** : Même après terminaison, le root volume reste disponible.

---

## Schéma récapitulatif

```
CRÉATION D'INSTANCE
───────────────────
[Instance EC2]
  ├─ Root (8 GB) [Delete on Term: Yes]
  └─ Data (2 GB) [Delete on Term: No]


AJOUT MANUEL DE VOLUME
──────────────────────
Créer volume 2 GB dans MÊME AZ
  ↓
Attacher à l'instance
  ↓
[Instance EC2]
  ├─ Root (8 GB) [Delete on Term: Yes]
  └─ Data (2 GB) [Delete on Term: No] ✅


TENTATIVE AZ DIFFÉRENTE
───────────────────────
Créer volume 2 GB dans AZ DIFFÉRENTE
  ↓
Essayer d'attacher
  ↓
❌ Impossible (volumes liés à leur AZ)


TERMINAISON
───────────
Terminate instance
  ↓
Root (Delete on Term: Yes) → ❌ Supprimé
Data (Delete on Term: No) → ✅ Conservé
```

---

## Points clés de la démo

✅ **Volumes visibles** dans l'onglet Storage de l'instance  
✅ **Créer un volume** : choisir la MÊME AZ que l'instance  
✅ **Attacher/détacher** des volumes à la volée  
✅ **Volumes liés à l'AZ** : impossible d'attacher cross-AZ  
✅ **Delete on Termination** : Root=Yes, Data=No par défaut  
✅ **Suppression instantanée** de volumes non attachés  
✅ **Volumes orphelins** restent disponibles après terminaison

---

## Nettoyage final

Pour éviter les coûts :

1. **Terminer l'instance** (déjà fait)
2. **Supprimer les volumes orphelins** :
   - Sélectionner le volume 2 GB
   - **Actions** → **Delete volume**

> ⚠️ Les volumes EBS sont facturés même s'ils ne sont pas attachés !
