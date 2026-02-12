# AWS Console Simultaneous Sign-In

## 🔄 Multi-Session Support (Support multi-sessions)

**Fonctionnalité** : Permet de se connecter à **plusieurs comptes AWS simultanément** dans le **même navigateur**

---

## 📌 Comment ça marche ?

### Avant cette fonctionnalité

- ❌ 1 navigateur = 1 seul compte AWS connecté à la fois
- ❌ Besoin de navigateurs différents ou mode incognito pour plusieurs comptes
- ❌ Déconnexion/reconnexion fréquente

### Avec Multi-Session Support

- ✅ Plusieurs comptes AWS dans des **onglets/fenêtres différents du même navigateur**
- ✅ Chaque session = compte AWS indépendant
- ✅ Pas besoin de se déconnecter/reconnecter

---

## 🎯 Activation et utilisation

### Étapes

1. Cliquer sur l'option **"Multi-session support"** dans la console
2. **Activer** la fonctionnalité
3. Cliquer sur **"Add a session"** (Ajouter une session)
4. Se connecter avec un autre **Account ID** ou **Root user**
5. Chaque fenêtre affiche son propre **Account ID** distinct

---

## 💡 Exemple pratique (dans la vidéo)

### Démonstration avec 2 comptes AWS

#### Fenêtre 1 (Compte A)

- Création d'un volume EBS de 1 GB
- Volume visible dans EC2 → EBS → Volumes

#### Fenêtre 2 (Compte B)

- Navigation vers EC2 → EBS → Volumes
- **Aucun volume visible** (car compte différent)

### Conclusion

Les deux comptes sont totalement **isolés** et **indépendants**, même dans le même navigateur

---

## 🚀 Avantages

✅ **Gain de temps** : Plus besoin de se déconnecter/reconnecter  
✅ **Productivité** : Gérer plusieurs comptes clients/projets facilement  
✅ **Flexibilité** : Comparaison rapide entre environnements (dev/prod)  
✅ **Organisation** : Idéal pour consultants ou admins gérant plusieurs comptes

---

## 📝 Points clés à retenir

> **Multi-session support** = nouvelle fonctionnalité AWS  
> Permet **plusieurs comptes AWS dans un même navigateur**  
> Chaque session reste **complètement isolée**  
> Très utile pour les utilisateurs gérant **plusieurs comptes AWS**  
> Fonctionnalité révolutionnaire pour les utilisateurs AWS expérimentés
