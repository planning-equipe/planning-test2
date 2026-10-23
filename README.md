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

## À faire de ton côté (15 minutes)

Tous ces réglages se font sur [supabase.com/dashboard](https://supabase.com/dashboard), dans le projet **PlanningEquipe**.

### 1. Mettre la page sur GitHub

Remplace l'ancien `index.html` de ton dépôt par le nouveau, puis note l'adresse de ta page GitHub Pages, par exemple `https://ton-compte.github.io/planning/`.

### 2. Indiquer cette adresse à Supabase

Va dans **Authentication › URL Configuration**.

- **Site URL** : colle l'adresse de ta page.
- **Redirect URLs** : ajoute la même adresse.

Sans ce réglage, le lien reçu par e-mail renvoie vers une page qui n'existe pas. Le code à 6 chiffres, lui, fonctionne quand même.

### 3. Mettre le code dans l'e-mail de connexion

Par défaut, l'e-mail de Supabase ne contient qu'un lien. Or l'antivirus de messagerie des entreprises (par exemple Office 365) ouvre souvent ce lien avant toi, et il ne marche alors plus. Le code à 6 chiffres évite ce problème.

Va dans **Authentication › Emails › Templates** et modifie **deux** modèles : **Magic Link** et **Confirm signup**. Le second sert à la toute première connexion de chaque personne. Pour chacun :

- **Sujet** : `Ton code de connexion au planning`
- **Contenu** :

```html
<h2>Planning équipe</h2>
<p>Ton code de connexion : <strong style="font-size:22px;letter-spacing:4px">{{ .Token }}</strong></p>
<p>Tu peux aussi cliquer sur ce lien : <a href="{{ .ConfirmationURL }}">se connecter</a></p>
<p>Ce code est valable une heure.</p>
```

### 4. Tester avec l'adresse de ton compte Supabase

Ouvre la page, saisis l'adresse e-mail de ton compte Supabase, puis le code reçu. Ça doit marcher tout de suite, même avant l'étape 5.

### 5. Brancher un service d'envoi d'e-mails pour tes collègues (obligatoire)

Sans réglage, Supabase n'envoie les e-mails de connexion qu'aux membres de ton compte Supabase, c'est-à-dire l'adresse de ton compte Supabase, et au maximum quelques-uns par heure. Tes collègues ne recevraient rien : la page leur afficherait « Le service d'envoi d'e-mails n'est pas encore configuré ».

La solution la plus simple est **Brevo**, un service français gratuit jusqu'à 300 e-mails par jour.

1. Crée un compte sur brevo.com.
2. Dans **SMTP & API**, récupère :
   - le serveur (`smtp-relay.brevo.com`) ;
   - le port (`587`) ;
   - l'identifiant ;
   - la clé SMTP.
3. Fais vérifier l'adresse d'expédition, par exemple ton adresse pro.
4. Dans Supabase, va dans **Authentication › Emails › SMTP Settings**. Active **Enable custom SMTP**, puis remplis :
   - les quatre informations de Brevo ;
   - l'adresse d'expédition ;
   - le nom d'expéditeur « Planning équipe ».

Ton service informatique peut aussi te fournir un serveur d'envoi de l'entreprise : les réglages sont les mêmes.

### 6. Donner l'accès à l'équipe

Dans le planning, va dans **Réglages › Accès au planning**. Ajoute l'adresse e-mail de chaque collègue et choisis son rôle :

| Rôle | Ce qu'il peut faire |
|---|---|
| Éditeur | Remplit le planning et les réglages |
| Lecteur | Consulte seulement |
| Administrateur | Comme un éditeur, et gère cette liste |

Chacun se connecte ensuite une seule fois sur chaque appareil. La connexion reste active, il n'y a pas besoin de recommencer chaque jour.

## Bon à savoir

- **La clé dans la page n'est pas un secret.** La clé `sb_publishable_…` écrite dans `index.html` est faite pour être publique. La protection vient de la base : sans adresse autorisée, on ne voit rien.
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
