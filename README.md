# Enter Africa Cameroun (EAC) — Site web

Site en **HTML / CSS / JavaScript pur**, mobile-first, responsive, multi-pages,
traduisible en un clic (FR/EN/ES/DE), et pensé pour être mis à jour par
l'équipe EAC **sans écrire de code** via un panneau d'administration.

## 🗂️ Structure du site

Le site compte **4 pages indépendantes** :

| Page | Rôle | Navigation |
|---|---|---|
| `index.html` | Accueil (genèse, galerie, moments clés, vision 2030, équipe) | Page de départ — liens internes |
| `games.html` | Nos jeux (vidéo de lancement, activités, catalogue, vidéos) | Ouvre dans un **nouvel onglet** |
| `services.html` | Services, tarifs, équipe, FAQ | Ouvre dans un **nouvel onglet** |
| `contact.html` | Formulaire et coordonnées | Ouvre dans un **nouvel onglet** |

Seul le logo et le lien « Accueil » ramènent vers `index.html` dans le même
onglet ; les liens Services / Jeux / Contact ouvrent toujours un nouvel
onglet, comme demandé.

```
eac/
├── index.html / games.html / services.html / contact.html / admin.html
├── css/
│   ├── style.css             ← Palette officielle + tous les composants
│   └── admin.css
├── js/
│   ├── content-default.js    ← ⭐ Tout le contenu (textes FR, chemins d'images/vidéos, liens)
│   ├── translations.js       ← Traductions EN/ES/DE du contenu dynamique
│   ├── i18n.js                ← Traductions des textes fixes de l'interface
│   ├── storage.js            ← Sauvegarde des modifications admin (localStorage) + sécurité
│   ├── common.js             ← Logique partagée par toutes les pages (menu, thème, langue, recherche, chatbot, footer)
│   ├── home.js / games.js / services.js / contact.js  ← Rendu spécifique à chaque page
│   └── admin.js
├── assets/
│   ├── img/{team,games,slider,partners,bonus}/  ← Photos réelles + quelques visuels générés
│   ├── docs/                  ← PDF du lead magnet
│   └── logo.png                ← Logo officiel EAC
├── SECURITY.md                ← Rapport de revue de sécurité
├── .htaccess / _headers       ← En-têtes de sécurité HTTP réels
├── .env.example                ← Modèle d'identifiants pour protéger /admin.html
└── README.md
```

## 🎬 Vidéos

Toutes les vidéos du site s'ouvrent dans une **fenêtre modale interne**
(`js/common.js` → `openVideoModal()`) : clic sur une affiche vidéo → lecteur
YouTube en superposition, lecture jusqu'à l'arrêt ou la fin, **sans jamais
quitter le site**. Fermeture par la croix, un clic hors du lecteur, ou la
touche Échap.
- Page d'accueil : vidéo de lancement d'Ongola Land dans la section Genèse.
- Page Jeux : même vidéo en grand format + grille de 8 vidéos EAC.
- Galerie de l'accueil : les légendes marquées d'une icône ▶ (Reliving
  EnterAfrica, ESPOTO Tutorial, etc.) ouvrent directement la vidéo dans la
  modal.

Pour remplacer une vidéo : ouvrez `js/content-default.js`, repérez le champ
`youtubeId` (dans `genese.video` ou le tableau `videos`) ou `videoUrl` (dans
`gallery`), et remplacez-le par votre nouvelle vidéo. Depuis `admin.html` →
Accueil, vous pouvez aussi coller directement un lien YouTube complet.

## ✍️ Effet machine à écrire

Le titre principal et le texte de bienvenue de la page d'accueil s'animent
en effet "machine à écrire" au chargement de la page (`js/common.js` →
`typewriterEffect()`), en conservant les mots mis en couleur.

## 🌍 Traduction en un clic

Le sélecteur de langue (FR/EN/ES/DE) en haut de chaque page traduit :
- Les textes fixes de l'interface (menus, titres de section, formulaires) —
  définis dans `js/i18n.js` ;
- **Le contenu dynamique** (genèse, FAQ, services, tarifs, jeux, vision,
  carrousel) — définis dans `js/translations.js`.

Les traductions ont été générées automatiquement puis relues ; il est
recommandé de les faire valider par un locuteur natif avant une mise en
production définitive. Les noms propres (personnes, organisations, titres de
jeux) ne sont volontairement pas traduits.

Pour corriger ou compléter une traduction : ouvrez `js/translations.js`,
retrouvez la langue et le champ concerné (même structure que
`content-default.js`), et modifiez le texte.

## 🎨 Palette de couleurs

| | Principale | Secondaire (fond) | Tertiaire |
|---|---|---|---|
| **Mode sombre** | `#4B95D9` | `#0C131A` (fond encore assombri vers le noir pur) | `#B413B9` |
| **Mode clair** | `#F7E1D7` | `#FFFFFF` | `#A39E9E` |

Deux accents supplémentaires, appliqués de façon mesurée dans toute
l'interface : un **reflet or** (`--color-gold`) sur les bordures des
boutons, et un **vert fluo** (`--color-neon-green`) en fine bordure sur les
images de la galerie. Ces couleurs sont centralisées dans `css/style.css`.
Note d'accessibilité : en mode clair, `--color-primary-text` (une teinte
plus soutenue que `#F7E1D7`) est utilisée uniquement pour le texte et les
liens, afin de rester lisible sur fond blanc — la couleur `#F7E1D7`
d'origine reste utilisée telle quelle pour les fonds et dégradés, désormais
très atténuée pour ne pas nuire à la lisibilité.

## 🖼️ Galerie — nouveautés

- **Padding augmenté** et cartes en `object-fit: contain` : chaque photo
  s'affiche en entier (plus de recadrage agressif).
- **Fine bordure vert fluo** autour de chaque image, avec léger halo au
  survol.
- **Zoom au survol** de l'image dans son cadre.
- **Légende cliquable** : si une carte a une vidéo (`videoUrl`), sa légende
  ouvre la modal vidéo interne (▶). Si elle a un lien classique (`link`),
  elle ouvre ce lien (nouvel onglet si externe ou vers une autre page du
  site). Sinon, elle reste un simple texte.

## 🚀 Mettre le site en ligne

Site 100% statique — déposez tous les fichiers sur n'importe quel
hébergement (FTP, GitHub Pages, Netlify, Vercel...). Les fichiers
`.htaccess` (Apache) et `_headers` (Netlify/Cloudflare Pages) appliquent
automatiquement les en-têtes de sécurité recommandés.

## 🖼️ Remplacer ou ajouter des images, vidéos et documents

### Option A — panneau d'administration (`admin.html`)
Onglets : Général, Accueil, Équipe & Galerie, Jeux, Services & Tarifs, FAQ,
Messages reçus, Sauvegarde. Les modifications sont stockées dans le
navigateur (localStorage) — utilisez **Sauvegarde → Exporter (JSON)** pour
les garder en sécurité ou les partager avec la personne qui gère le
déploiement du site.

### Option B — remplacement direct des fichiers
1. Déposez votre fichier dans le bon dossier (`assets/img/team/`,
   `assets/img/games/`, `assets/img/slider/`, `assets/img/partners/`,
   `assets/docs/`).
2. Modifiez le chemin correspondant dans `js/content-default.js`.
3. Enregistrez.

## 📬 Formulaires

Stockés localement (visibles dans `admin.html` → Messages reçus) tant
qu'aucun service d'envoi d'e-mail n'est branché. Voir les options
Formspree/Web3Forms/EmailJS pour recevoir de vrais e-mails (section dédiée
plus bas dans ce document, inchangée par rapport à la version précédente).

Le PDF envoyé automatiquement après inscription sur la page Jeux est
`assets/docs/checklist-avant-publication-site-web.pdf` — remplaçable dans
`content-default.js` → `capture.pdfFile`.

## 🔐 Accès à l'espace administration

Le lien "Espace administration" n'apparaît plus dans le pied de page public.
`/admin.html` reste accessible par son URL directe (à garder confidentielle
ou à protéger — voir `.env.example` et le bloc Basic Auth prêt à activer
dans `.htaccess`). Une fois que vous avez ouvert le panneau une première
fois, un lien **"Se déconnecter"** apparaît dans le pied de page de ce
navigateur — ce n'est qu'un confort d'interface, pas une authentification
réelle (voir `SECURITY.md`, section 8.1, pour sécuriser correctement l'accès
avant mise en production).

## 📜 Popups RGPD & SAV

Les liens "RGPD" et "SAV" du pied de page ouvrent désormais une fenêtre avec
le contenu correspondant, sans quitter le site. Ce contenu est éditable
depuis `admin.html` → Général → "Contenu de la popup RGPD/SAV".

## 🌍 Réseau & Ressources (page Services)

Une nouvelle section liste les organisations partenaires du secteur du jeu
vidéo africain (IGDA, Gamescom Global, Games Week Africa...). Ces
ressources sont aussi indexées dans la barre de recherche du site.

## 🤖 Assistant SAV

Le chatbot propose désormais des suggestions rapides cliquables (Nos jeux,
Nos tarifs, Nous contacter, RGPD/SAV) et peut ouvrir directement les popups
RGPD/SAV depuis la conversation. Ses réponses s'affichent toujours sur fond
blanc et texte noir, dans les deux thèmes, pour une lisibilité maximale.

## 🎮 Jeu Food-H

Un nouveau jeu, **Food-H** (sensibilisation au gaspillage alimentaire), créé
par **John Mbapou**, est jouable directement depuis la page Jeux (bouton
"▶ Jouer"). Il est hébergé dans `assets/games/food-h/` comme une mini-page
autonome avec sa propre Content-Security-Policy (voir `SECURITY.md`,
section 9). Pour l'éditer : modifiez directement les fichiers dans ce
dossier (`index.html`, `style.css`, `script.js`).

## 💳 Nos offres

Les 3 formules de la page Services ont été pensées comme des **packs
numériques concrets et accessibles** pour un gamer (accès aux jeux EAC,
places en Discord, sessions de mentorat, réductions ateliers...) plutôt que
des forfaits de formation génériques. Les boutons affichent "À venir" en
attendant la mise en place du paiement en ligne.

## 🔒 Sécurité

Voir **`SECURITY.md`** pour le rapport complet. Résumé : validation
anti-XSS de tous les liens/images (y compris nouveaux champs vidéo),
Content-Security-Policy stricte avec autorisation ciblée des iframes
YouTube (`frame-src https://www.youtube-nocookie.com`), en-têtes HTTP réels
via `.htaccess`/`_headers`, fichiers d'upload limités en taille/type.

## ♿ Accessibilité & performance

Navigation clavier complète, lien d'évitement, respect de
`prefers-reduced-motion`, images en `loading="lazy"`, vidéos en chargement
différé (lecteur "lite"), design mobile-first.
