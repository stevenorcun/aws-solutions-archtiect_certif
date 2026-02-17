# AWS – EC2 Instance Connect (Connexion via Navigateur)

## Qu'est-ce que c'est ?

EC2 Instance Connect est une alternative à SSH qui fonctionne **directement dans le navigateur**, sans gestion de clé `.pem`.

---

## Comment se connecter

1. Sélectionnez votre instance EC2
2. Cliquez sur **Connect** (en haut)
3. Choisissez l'onglet **EC2 Instance Connect**
4. Le nom d'utilisateur `ec2-user` est pré-rempli automatiquement (ne pas modifier)
5. Cliquez sur **Connect**

> AWS génère automatiquement une clé SSH temporaire — aucune gestion de clé de votre côté.

---

## Prérequis indispensable

Le port **22 (SSH) doit être ouvert** dans le security group, car EC2 Instance Connect utilise SSH en arrière-plan.

Si la connexion échoue, vérifiez les règles inbound :

| Type | Port | Source           |
| ---- | ---- | ---------------- |
| SSH  | 22   | 0.0.0.0/0 (IPv4) |
| SSH  | 22   | ::/0 (IPv6)      |

> Selon votre configuration réseau, il peut être nécessaire d'ajouter les **deux entrées** (IPv4 et IPv6).

---

## Avantages

- Aucune installation requise
- Aucune gestion de clé `.pem`
- Fonctionne sur **Mac, Linux, Windows** (tous)
- Session directement dans le navigateur

## Limitation

- Fonctionne uniquement avec les instances **Amazon Linux 2**

---

> **Rappel :** Dans la suite du cours, quand SSH est mentionné, vous pouvez toujours utiliser EC2 Instance Connect à la place.
