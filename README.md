# Planning équipe – version Supabase

Le planning garde son interface, mais les données sont maintenant dans une base Supabase :
projet **PlanningEquipe**, hébergé à Paris (eu-west-3).

## Ce qui est déjà fait

- **La base est créée**, avec 3 tables :
  - `cases` : une ligne par case du planning ;
  - `reglages` : l'équipe, les véhicules, le matériel, les couleurs et les ponts ;
  - `membres` : les adresses autorisées à ouvrir le planning.
- **Tes réglages actuels sont importés** : les 10 personnes, les 5 véhicules, les 7 matériels et les couleurs. Le planning ne contenait encore aucune saisie.
- **Le premier administrateur est déjà déclaré** dans la table `membres`.
- **La sécurité est en place** :
  - une personne qui n'est pas dans la liste ne voit rien, même connectée ;
  - un lecteur peut consulter mais pas modifier ;
  - le contrôle se fait dans la base elle-même, pas seulement dans la page.
- **Les saisies simultanées sont gérées** :
  - si deux personnes modifient la même case, la deuxième reçoit un message et voit la nouvelle valeur ;
  - un véhicule ne peut pas être réservé deux fois le même jour ;
  - les saisies des autres apparaissent en direct, sans recharger la page.

## Connexion : adresse e-mail + mot de passe

Le planning n'envoie aucun e-mail : pas besoin de service d'envoi (SMTP). L'administrateur crée lui-même le compte de chaque personne.

### Ajouter un collègue (2 minutes)

1. **Créer son compte** dans Supabase : **Authentication › Users › Add user › Create new user**.
   - Saisis son adresse e-mail et un mot de passe provisoire.
   - Coche **Auto Confirm User**. Sans cette case, la connexion est refusée.
2. **Lui donner l'accès** dans le planning : **Réglages › Accès au planning**, avec son adresse et son rôle.
3. **Lui transmettre** l'adresse de la page, son e-mail et son mot de passe provisoire. Il peut ensuite choisir son propre mot de passe dans **Réglages › Mon compte**.

| Rôle | Ce qu'il peut faire |
|---|---|
| Éditeur | Remplit le planning et les réglages |
| Lecteur | Consulte seulement |
| Administrateur | Comme un éditeur, et gère la liste des accès |

Chacun se connecte une seule fois par appareil : la connexion reste active.

### Mot de passe oublié

Supabase ne permet pas de modifier directement le mot de passe d'un compte dans le tableau de bord. Dans **Authentication › Users**, supprime le compte de la personne, puis recrée-le avec un nouveau mot de passe provisoire (étape 1 ci-dessus). Son accès dans le planning et ses saisies sont conservés, car ils sont liés à son adresse e-mail.

### Réglage de sécurité conseillé

Dans **Authentication › Sign In / Providers**, désactive **Allow new users to sign up**. Ainsi, seuls les comptes que tu crées existent. Les comptes créés depuis le tableau de bord continuent de fonctionner.

## Bon à savoir

- **La clé dans la page n'est pas un secret.** La clé `sb_publishable_…` écrite dans `index.html` est faite pour être publique. La protection vient de la base : sans compte et sans adresse autorisée, on ne voit rien.
- **Mise en veille du projet gratuit.** Supabase met en veille un projet gratuit après 7 jours sans aucune activité, par exemple pendant la fermeture d'août. Si ça arrive :
  - le planning affiche une erreur de connexion ;
  - il suffit de cliquer sur **Restore project** dans le tableau de bord ;
  - les données ne sont pas perdues.
- **Sauvegardes.** Le plan gratuit n'offre pas de sauvegarde téléchargeable. Pour garder une copie, va dans **Table Editor › cases › Export to CSV** de temps en temps.
- **Connexion avec les comptes Microsoft (plus tard).**
  1. Ton service informatique doit créer une « inscription d'application » dans Azure / Entra ID.
  2. Il faut ensuite activer le fournisseur **Azure** dans **Authentication › Sign In / Providers**.
  3. Enfin, mets `CONNEXION_MICROSOFT = true` en haut du script de `index.html` : un bouton « Se connecter avec Microsoft » apparaît sur l'écran de connexion.
- **Réinstaller la base ailleurs.** Le fichier `installation.sql` recrée toute la structure de la base. Pour l'utiliser, colle-le dans **SQL Editor** puis clique sur **Run**. Il ne contient pas les données.
