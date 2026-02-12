- Sur AWS quand on arrive sur l'interface IAM on peut se rendre compte que c'est un service GLOBAL c'est à dire que les utilisateurs, groupes et politiques IAM ne sont pas liés à une région spécifique. Ils sont disponibles dans toutes les régions AWS.

- Lors de la création d'un nouveau user, on pourra les attribuer un groupe , par exemple le groupe "admin" avbec les droit AdministratorAccess et il va donc hériter des droits.

- On pourra également le donner des tags qui vont permettre de faire du filtrage et de l'organisation dans la console AWS.

- On pourra également lui donnes un alias qui est un nom d'utilisateur plus facile à retenir que l'ID utilisateur généré automatiquement par AWS.

- Une fois fais tout cela, sur l'interface graphique AWS lors de la connexion on choisira "IAM user" et on rentrera l'alias ou l'ID utilisateur ainsi que le mot de passe pour se connecter à la console AWS.
