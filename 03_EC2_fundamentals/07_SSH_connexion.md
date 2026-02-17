# AWS – SSH sur Windows 10 (PowerShell)

## Vérifier que SSH est disponible

Dans **PowerShell** ou **Command Prompt**, tapez :

```bash
ssh
```

Si une aide s'affiche → SSH est disponible, vous pouvez continuer.
Si la commande est introuvable → utilisez **PuTTY** à la place.

---

## Se connecter à l'instance EC2

### Prérequis

- Fichier `.pem` téléchargé sur votre machine
- Port **22** ouvert dans le security group de l'instance

### Commande

```bash
# 1. Se placer dans le dossier contenant le .pem
cd .\Desktop

# 2. Lancer la connexion SSH
ssh -i EC2Tutorial.pem ec2-user@<IP_PUBLIQUE>
```

> Remplacez `<IP_PUBLIQUE>` par l'adresse IPv4 publique de votre instance EC2.

À la question _"The authenticity of the host cannot be trusted"_, répondez **yes**.

---

## Résoudre les erreurs de permissions

Si vous obtenez une erreur de permission sur le fichier `.pem` :

1. Clic droit sur le fichier `.pem` → **Properties** → onglet **Security**
2. Cliquez sur **Advanced**
3. Vérifiez que le **propriétaire** est bien votre compte utilisateur (sinon cliquez **Change**)
4. Cliquez sur **Disable inheritance** → **Remove all inherited permissions**
5. Ajoutez-vous en tant que principal avec **Full Control**
6. Validez avec **OK**

Après cette manipulation, vous ne serez plus jamais bloqué par des erreurs de permissions sur ce fichier.

---

## Quitter la session SSH

```bash
exit
# ou
Ctrl + D
```
