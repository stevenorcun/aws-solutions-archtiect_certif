# AWS – Se Connecter à une Instance EC2 (Introduction)

## Les méthodes disponibles

| Méthode                  | OS compatibles             | Prérequis                 |
| ------------------------ | -------------------------- | ------------------------- |
| **SSH**                  | Mac, Linux, Windows ≥ 10   | Terminal                  |
| **PuTTY**                | Toutes versions Windows    | Installer PuTTY           |
| **EC2 Instance Connect** | Mac, Linux, Windows (tous) | Navigateur web uniquement |

---

## EC2 Instance Connect (recommandé)

- Fonctionne directement depuis le **navigateur web**
- Aucune installation requise
- Compatible avec tous les OS
- ⚠️ Fonctionne uniquement avec les instances **Amazon Linux 2**

> C'est la méthode utilisée dans la suite du cours.

---

## Quelle méthode choisir ?

- **Mac / Linux** → SSH (voir leçon dédiée)
- **Windows < 10** → PuTTY (voir leçon dédiée)
- **Windows ≥ 10** → SSH ou PuTTY (voir leçons dédiées)
- **Tout le monde** → EC2 Instance Connect (le plus simple)

---

## En cas de problème avec SSH

SSH est historiquement la partie qui pose le plus de difficultés. Si ça ne fonctionne pas :

1. Revérifier la **règle du security group** (port 22 ouvert ?)
2. Revérifier la **commande SSH** (typo ?)
3. Consulter le **guide de dépannage** fourni après les leçons
4. Essayer **EC2 Instance Connect** — règle souvent les problèmes

> **Important :** Il suffit qu'**une seule méthode fonctionne**. Si EC2 Instance Connect fonctionne, vous pouvez continuer le cours sans SSH. Et si aucune méthode ne fonctionne, ce n'est pas bloquant — SSH est peu utilisé dans ce cours d'introduction.
