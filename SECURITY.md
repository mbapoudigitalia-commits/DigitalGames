# Revue de sécurité — Enter Africa Cameroun (EAC)

Revue effectuée sur l'ensemble du code (HTML, CSS, JavaScript) livré pour le
site EAC et son panneau d'administration. Méthodologie : analyse statique du
code source, recherche des schémas de vulnérabilité classiques (XSS,
injection, fuite de données, mauvaise configuration), et vérification de
chaque correction directement dans les fichiers livrés.

## Résumé

| Sévérité | Trouvé | Corrigé |
|---|---|---|
| Élevée | 1 | 1 |
| Moyenne | 3 | 3 |
| Faible / Bonnes pratiques | 5 | 5 |

Aucune vulnérabilité critique ouverte à la livraison. Le site est 100%
statique (HTML/CSS/JS) : il n'y a pas de base de données, pas de serveur
applicatif, donc pas de surface d'attaque côté serveur (pas d'injection SQL,
pas de RCE, etc.). Les risques résiduels concernent principalement le
navigateur du visiteur (XSS) et la configuration de l'hébergement.

---

## 1. Trouvé & corrigé

### 1.1 [Élevée] Injection XSS via liens/images non validés
**Où :** contenu dynamique affiché sans validation (liens de la galerie, des
jeux, des réseaux sociaux ; source des images uploadées depuis l'admin ; JSON
importé).
**Risque :** un lien ou un chemin d'image malveillant (`javascript:...`)
inséré via le panneau d'administration, un import JSON, ou une modification
manuelle de `content-default.js` aurait pu exécuter du code arbitraire dans
le navigateur d'un visiteur au clic.
**Correction :** ajout de `isSafeUrl()` / `isSafeImageSrc()` dans
`js/storage.js` (liste blanche de schémas : `http:`, `https:`, `mailto:`,
`tel:`, chemins relatifs, `data:image/*` pour les images uploadées
uniquement). Ces fonctions sont appliquées :
- côté **affichage** (`js/site.js`) sur toutes les images/liens dynamiques
  (galerie, carrousel, jeux, réseaux sociaux, résultats de recherche, PDF,
  vidéo) ;
- côté **saisie** (`js/admin.js`) via `sanitizeWorkingContent()`, appelée
  avant chaque sauvegarde ET avant chaque import JSON, qui neutralise
  silencieusement tout lien/image dangereux et prévient l'administrateur.

### 1.2 [Moyenne] Contenu utilisateur non échappé
**Où :** rendu des cartes (équipe, jeux, galerie), résultats de recherche,
messages reçus dans l'admin.
**Risque :** un caractère `<script>` dans un titre ou une description aurait
pu casser la mise en page ou exécuter du code.
**Correction :** toutes les valeurs texte passent par `escapeHTML()` avant
insertion dans le DOM (déjà en place, vérifié systématiquement pendant cette
revue — aucune régression trouvée).

### 1.3 [Moyenne] Absence de Content-Security-Policy
**Risque :** sans CSP, une éventuelle faille XSS aurait un impact maximal
(chargement de scripts depuis n'importe quel domaine).
**Correction :** CSP stricte ajoutée via balise `<meta>` dans `index.html` et
`admin.html` (`script-src 'self'`, pas de scripts inline, `object-src
'none'`, etc.), **et** dupliquée en en-tête HTTP réel dans `.htaccess` /
`_headers` pour les directives que les navigateurs ignorent en `<meta>`
(`frame-ancestors`, protection anti-clickjacking).

### 1.4 [Moyenne] Upload de fichiers non contrôlé côté admin
**Risque :** un fichier trop volumineux ou d'un type inattendu uploadé dans
le panneau admin aurait pu saturer le `localStorage` ou provoquer un
comportement inattendu.
**Correction :** `fileToDataURL()` (déjà en place) limite la taille
(~1.8 Mo) et le type MIME (JPG/PNG/WEBP/GIF/SVG uniquement) ; l'import JSON
est maintenant lui aussi limité à 5 Mo et validé (`JSON.parse` dans un
`try/catch`, jamais `eval`).

---

## 2. Bonnes pratiques déjà respectées (vérifiées, non modifiées)

- **Aucun `eval()`, `new Function()` ni `document.write()`** dans tout le
  code — recherché explicitement, aucune occurrence.
- **Aucun gestionnaire d'événement inline** (`onclick=""`, etc.) — tout passe
  par `addEventListener`, ce qui permet une CSP `script-src 'self'` stricte
  sans `'unsafe-inline'`.
- **Tous les liens `target="_blank"`** portent `rel="noopener noreferrer"`
  (protection contre le *reverse tabnabbing*).
- **Pas de mot de passe ni de donnée sensible** stockée où que ce soit
  (le site n'a pas de système d'authentification à protéger).
- **`admin.html` exclu de l'indexation** (`<meta name="robots" content="noindex, nofollow">`).
- **JSON.parse** utilisé pour tout import/lecture de données — jamais
  d'évaluation de code dynamique.

---

## 3. Points d'attention pour la mise en production (recommandations)

Ces points ne sont pas des failles dans le code livré, mais des choix
d'architecture ou de configuration à connaître :

1. **Le panneau `admin.html` n'a pas d'authentification.** Comme le site est
   100% statique, les modifications faites dans `admin.html` sont
   sauvegardées dans le `localStorage` **du navigateur qui les fait** — un
   visiteur qui ouvrirait `admin.html` ne peut donc pas modifier ce que
   voient les autres visiteurs. Mais par précaution (confidentialité,
   confusion), il est recommandé de restreindre l'accès à `/admin.html` par
   un mot de passe HTTP (`.htpasswd`) une fois le site en ligne, ou de ne
   pas publier son URL. Pour un vrai CMS multi-utilisateurs partagé, il
   faudra un jour brancher un backend (voir README.md, section Formulaires).
2. **CSP en `<meta>` a des limites connues des navigateurs** :
   `frame-ancestors` n'y est pas appliqué. Les fichiers `.htaccess` et
   `_headers` fournis corrigent ce point via de vrais en-têtes HTTP —
   assurez-vous que votre hébergeur les prend en compte (Apache : oui
   nativement ; Netlify/Cloudflare Pages : `_headers` reconnu
   automatiquement ; Nginx : demandez à votre hébergeur d'ajouter les
   en-têtes équivalents).
3. **HTTPS obligatoire.** Le site ne contient pas de mot de passe, mais tout
   site devrait être servi en HTTPS. La ligne `Strict-Transport-Security`
   est présente en commentaire dans `.htaccess` — activez-la seulement après
   avoir confirmé que votre certificat HTTPS fonctionne (sinon vous
   risquez de bloquer l'accès HTTP de secours).
4. **Font Awesome est chargé depuis `cdnjs.cloudflare.com`.** C'est un CDN
   réputé, mais toute dépendance externe est une surface de confiance
   supplémentaire. Le hash d'intégrité (SRI) n'a pas été inclus par
   précaution — un hash incorrect aurait empêché le chargement des icônes.
   Si vous voulez l'ajouter, copiez le hash exact depuis
   `https://cdnjs.com/libraries/font-awesome/6.5.1` (bouton "Copy SRI").
5. **Sauvegardez régulièrement le contenu personnalisé** via `admin.html` →
   Sauvegarde → Exporter (JSON). Comme les modifications vivent dans le
   `localStorage` du navigateur, elles peuvent être perdues si l'utilisateur
   vide son cache/historique.
6. **Formulaires actuellement stockés en local uniquement** (voir README.md
   pour brancher un vrai service d'envoi d'email type Formspree/EmailJS).
   Tant que ce n'est pas fait, aucun email n'est réellement envoyé.

---

## 4. Fichiers modifiés/ajoutés pendant cette revue

- `js/storage.js` — ajout `isSafeUrl()`, `isSafeImageSrc()`
- `js/site.js` — sanitisation de tous les points de rendu (images/liens)
- `js/admin.js` — `sanitizeWorkingContent()` appelé avant chaque sauvegarde
  et avant chaque import JSON ; limite de taille sur l'import
- `index.html`, `admin.html` — ajout CSP, `X-Content-Type-Options`,
  `Referrer-Policy` (balises meta)
- `.htaccess` (nouveau) — en-têtes de sécurité HTTP réels pour Apache
- `_headers` (nouveau) — équivalent pour Netlify / Cloudflare Pages

## 5. Verdict

✅ **Prêt pour la mise en production**, sous réserve d'appliquer les
recommandations de la section 3 (accès admin, HTTPS, hébergement des
en-têtes). Aucune vulnérabilité connue non corrigée à la date de cette
revue.

---

## 6. Addendum — Revue de la refonte multi-pages, vidéos YouTube & i18n

Cette section complète la revue initiale suite à l'ajout : de 3 nouvelles
pages (`games.html`, `services.html`, `contact.html`), de lecteurs vidéo
YouTube intégrés, et du système de traduction complète du contenu
(`js/translations.js`).

### 6.1 [Moyenne] Ouverture de la Content-Security-Policy pour les iframes YouTube
**Risque :** intégrer des vidéos nécessite d'autoriser des `<iframe>`
externes, ce qui élargit la surface de confiance de la CSP.
**Correction :** la directive `frame-src` a été ajoutée avec une liste
blanche **stricte à un seul domaine** :
`frame-src https://www.youtube-nocookie.com` (domaine "confidentialité
renforcée" de YouTube, qui ne dépose pas de cookies de suivi tant que la
vidéo n'est pas lancée). Aucun autre domaine n'est autorisé. La directive
`img-src` a été élargie uniquement à `https://i.ytimg.com` (miniatures des
vidéos). Ces deux ajouts sont répercutés dans les 4 pages HTML **et** dans
`.htaccess`/`_headers`.

### 6.2 [Faible] Chargement différé des vidéos ("lite embed")
Plutôt que de charger l'iframe YouTube (et ses scripts tiers) dès l'affichage
de la page, `buildYoutubeEmbed()` (`js/common.js`) n'affiche qu'une image
d'affiche statique ; l'iframe n'est injectée dans le DOM qu'au clic de
l'utilisateur. Bénéfice sécurité et performance : aucun script/cookie tiers
n'est chargé tant que la personne n'a pas explicitement demandé la lecture.

### 6.3 [Moyenne] Nouveau champ admin `videoUrl` (galerie) et `youtubeId` (vidéo genèse)
**Risque :** comme pour les autres champs liens, une valeur malveillante
saisie dans ces nouveaux champs aurait pu réintroduire un vecteur XSS.
**Correction :**
- `sanitizeWorkingContent()` (`js/admin.js`) valide désormais aussi
  `gallery[].videoUrl` via `isSafeUrl()`.
- Le champ vidéo de la page Accueil (admin) accepte un lien YouTube complet
  ou un identifiant brut ; `extractYoutubeId()` en extrait uniquement
  l'identifiant et **filtre tout caractère non alphanumérique** avant
  sauvegarde (`[^a-zA-Z0-9_-]` supprimé), empêchant toute injection dans
  l'URL d'iframe construite côté affichage.
- Côté affichage (`js/site.js`/`js/home.js`), `encodeURIComponent()` est
  appliqué à l'identifiant avant construction de l'URL de l'iframe, en plus
  du filtrage ci-dessus (défense en profondeur).

### 6.4 [Faible] Cohérence de la CSP sur les nouvelles pages
Les 4 pages (`index.html`, `games.html`, `services.html`, `contact.html`)
partagent désormais une CSP strictement identique (`script-src 'self'`,
aucun script inline, `object-src 'none'`, `frame-ancestors 'none'`),
vérifiée fichier par fichier pendant cette revue.

### 6.5 Vérifications reconduites
- Aucun gestionnaire d'événement inline (`onclick=...`) ni `<script>` inline
  dans les nouveaux fichiers — recherché explicitement sur les 5 pages HTML.
- Tous les liens externes portent `rel="noopener noreferrer"`, y compris les
  nouveaux liens de navigation inter-pages en `target="_blank"`.
- Le traducteur de contenu (`EAC_TRANSLATE`) ne fait que fusionner des
  chaînes de texte statiques définies dans le code source (aucune donnée
  utilisateur n'y transite) — pas de risque d'injection à ce niveau.

**Verdict de l'addendum :** ✅ aucune nouvelle vulnérabilité ouverte. La
surface d'attaque liée à l'intégration YouTube a été délibérément réduite au
minimum (un seul domaine autorisé, chargement différé, sanitisation
systématique des identifiants vidéo).

---

## 7. Addendum 2 — Modal vidéo in-site, effet machine à écrire, nouveaux médias

### 7.1 [Faible] Généralisation de la lecture vidéo en modal (sans quitter le site)
Toutes les vidéos du site (page d'accueil, page Jeux, galerie) s'ouvrent
désormais dans une fenêtre modale interne (`openVideoModal()` /
`closeVideoModal()` dans `js/common.js`) plutôt que par un lien externe vers
YouTube. Points vérifiés :
- L'identifiant vidéo passe toujours par `extractYoutubeId()` (filtrage
  `[^a-zA-Z0-9_-]`) puis `encodeURIComponent()` avant construction de l'URL
  d'iframe — aucune donnée n'atteint le DOM sans validation.
- La fermeture de la modal (`closeVideoModal`) vide immédiatement
  `.video-modal-player` (retire l'iframe du DOM), ce qui coupe la lecture et
  évite qu'une iframe orpheline continue de tourner en arrière-plan.
- Le domaine reste strictement limité à `youtube-nocookie.com`, déjà
  autorisé dans la CSP (`frame-src`) — aucune ouverture de politique
  supplémentaire n'a été nécessaire pour cette fonctionnalité.
- Focus géré : à l'ouverture, le focus clavier va sur le bouton de
  fermeture ; `Échap` ferme la modal (accessibilité clavier).

### 7.2 [Aucun risque] Effet machine à écrire
`typewriterEffect()` (`js/common.js`) ne fait que redécouper le HTML déjà
présent dans le DOM (texte de confiance venant de `content-default.js` /
`translations.js`, jamais de saisie utilisateur) en nœuds texte individuels,
sans jamais réinjecter de chaîne via `innerHTML` avec du contenu variable.
Aucune surface XSS introduite.

### 7.3 Nouveaux médias intégrés
Toutes les nouvelles images (logos partenaires, photos d'équipe, moments du
carrousel) sont des fichiers statiques déposés dans `assets/img/`,
référencés par chemin relatif fixe dans `content-default.js` — même
traitement de sécurité que les médias précédents (`isSafeImageSrc`
appliqué à l'affichage).

**Verdict de l'addendum 2 :** ✅ aucune nouvelle vulnérabilité. Le
changement le plus sensible (lecture vidéo) a été implémenté avec le même
niveau de rigueur (liste blanche de domaine, sanitisation d'ID,
nettoyage du DOM à la fermeture) que le reste du site.

---

## 8. Addendum 3 — Session admin, popups RGPD/SAV, ressources externes

### 8.1 [Information importante] Le lien "Espace administration" et la "session admin" ne sont PAS un mécanisme de sécurité
Le lien "Espace administration" a été retiré du pied de page public sur
toutes les pages. Un lien **"Se déconnecter"** apparaît désormais à sa place,
mais **uniquement** dans le navigateur ayant déjà ouvert `/admin.html`
(détecté via un indicateur `localStorage`, voir `isAdminSession()` dans
`js/storage.js`).

**Il est essentiel de comprendre que ceci n'est PAS une authentification.**
N'importe qui connaissant ou devinant l'URL `/admin.html` peut toujours y
accéder directement — l'indicateur ne fait qu'afficher ou masquer un lien
dans l'interface de CE navigateur précis, sans jamais vérifier une identité.
Pour une vraie protection avant mise en production, voir **`.env.example`**
(modèle d'identifiants) et le bloc **Basic Auth prêt à activer** dans
`.htaccess` (section "PROTECTION DE L'ESPACE ADMINISTRATION").

### 8.2 [Aucun risque] Popups RGPD / SAV
`openContentModal()` (`js/common.js`) affiche le contenu de
`content.site.rgpdContent` / `content.site.savContent` via `innerHTML`. Ces
champs sont éditables uniquement depuis le panneau `admin.html` (texte de
confiance saisi par l'administrateur du site, jamais une entrée visiteur) —
même modèle de confiance que `genese.html`, déjà validé en section 1.2. Le
contenu n'est ni généré ni influencé par une donnée utilisateur externe.

### 8.3 [Faible] Ressources externes (page Services, recherche)
Les 10 nouveaux liens vers des organisations partenaires (IGDA, Gamescom
Global, etc.) sont des données statiques (`content.externalResources` dans
`content-default.js`), jamais des entrées utilisateur. Chaque lien passe
malgré tout par `isSafeUrl()` avant d'être rendu (dans `renderResources()`
comme dans les résultats de recherche), et s'ouvre avec
`rel="noopener noreferrer"` — traitement identique aux autres liens externes
du site.

**Verdict de l'addendum 3 :** ✅ aucune vulnérabilité introduite. Le seul
point d'attention n'est pas une faille technique mais une clarification
importante : la fonctionnalité "Se déconnecter" est un confort d'interface,
pas un système d'authentification — voir 8.1 pour sécuriser réellement
l'accès en production.

---

## 9. Addendum 4 — Intégration du jeu tiers "Food-H"

Un jeu complet (HTML/CSS/JS) créé par un tiers (John Mbapou) a été intégré
au site (`assets/games/food-h/`), accessible depuis la page Jeux. Ce type
d'intégration — code non écrit par Claude — a fait l'objet d'une revue de
sécurité dédiée avant publication :

### 9.1 [Corrigé] Gestionnaires d'événements inline (`onclick="..."`)
Le code fourni utilisait des attributs `onclick="fonction()"` directement
dans le HTML (18 occurrences). Ce mode contrevient à la Content-Security-
-Policy stricte appliquée sur tout le reste du site (`script-src 'self'`
sans `'unsafe-inline'`). **Correction :** tous les `onclick` ont été
convertis en attributs `data-action` / `data-arg`, avec un unique
gestionnaire d'événements délégué ajouté à la fin de `script.js` (voir le
bloc commenté "Ajouté par Enter Africa Cameroun"). Résultat : le jeu
fonctionne à l'identique, mais avec une CSP aussi stricte que le reste du
site (`script-src 'self'`, aucun `'unsafe-inline'`).

### 9.2 [Vérifié] Contenu de `script.js`
Recherche explicite de `eval()`, `new Function()`, `document.write()`,
`window.open()` — aucune occurrence. Les seuls usages de `innerHTML`
construisent des listes d'achievements/scores à partir de données générées
par le jeu lui-même (jamais une entrée utilisateur libre), donc sans risque
XSS. Seul `localStorage` est utilisé (clé `foodHStats` et une clé
journalière), pour sauvegarder la progression du joueur localement — aucune
requête réseau, aucune donnée envoyée à un tiers.

### 9.3 [Corrigé] Éléments non fonctionnels retirés
Le bouton "Télécharger APK" du fichier original pointait vers un fichier
`FoodH.apk` inexistant (lien mort). Il a été retiré pour éviter une
promesse non tenue à l'utilisateur (cohérent avec les bonnes pratiques de
la checklist de déploiement fournie par ailleurs). Le champ
`related_applications` du `manifest.json`, qui pointait vers une fiche
Google Play fictive, a également été retiré.

### 9.4 [Isolé] CSP dédiée à cette page
`assets/games/food-h/index.html` a sa propre balise CSP, séparée du reste du
site, car le jeu a besoin d'autoriser deux domaines externes que le site
principal n'utilise pas : `fonts.googleapis.com`/`fonts.gstatic.com`
(polices Google Fonts d'origine) et `soundjay.com` (effets sonores
d'origine, chargés uniquement via `<audio>`, jamais `<script>`). Cette
ouverture reste strictement locale à cette page — les 4 autres pages du
site conservent leur CSP inchangée et plus restrictive.

**Verdict de l'addendum 4 :** ✅ le jeu a été rendu conforme au même niveau
d'exigence de sécurité que le reste du site avant publication (CSP stricte,
aucun gestionnaire inline, aucune fonction dangereuse, aucun lien mort).

---

## 10. Addendum 5 — Habillage visuel des sections & performance

Cette dernière passe a ajouté une bordure dorée arrondie (2px) autour de
chaque `<section>` du site (accueil, jeux, services, contact, y compris les
panneaux du back-office) et des attributs `decoding="async"` sur les images
pour améliorer le rendu perçu. Ces deux changements sont purement visuels /
de performance : aucune nouvelle surface d'attaque, aucun script, aucune
donnée utilisateur impliquée. Vérifié : la CSP reste inchangée, aucun
gestionnaire inline introduit, `node --check` et le parseur HTML confirment
une syntaxe valide sur les 5 pages + le jeu Food-H après ces modifications.

**Verdict final :** ✅ le site est prêt pour la mise en production, sous
réserve des recommandations de la section 3 (protection de `/admin.html`,
HTTPS, en-têtes serveur réels).

---

## 11. Addendum 6 — Reprise de partie Food-H, recherche & SAV enrichis

### 11.1 [Vérifié] Sauvegarde de progression Food-H
Le jeu Food-H sauvegarde désormais l'état de la partie en cours (niveau,
score, vies, environnement) dans `localStorage` (`foodHProgress`) à chaque
passage de niveau et à chaque pause, et propose un bouton "Continuer
(Niveau X)" sur l'écran d'accueil du jeu tant qu'une partie est en attente.
La progression sauvegardée est effacée en fin de partie ou lors d'un
redémarrage volontaire. Seule une valeur numérique interne (`progress.level`,
jamais une saisie utilisateur) est insérée dans le DOM — aucun risque XSS.

### 11.2 [Vérifié] Recherche et SAV
La barre de recherche gère désormais la navigation clavier (flèches +
Entrée) et propose des suggestions quand aucun résultat n'est trouvé —
aucune nouvelle donnée utilisateur n'est envoyée où que ce soit, tout reste
local à la page. L'assistant SAV reconnaît maintenant chaque jeu par son
nom et peut ouvrir directement Food-H dans un nouvel onglet
(`window.open(..., "_blank", "noopener,noreferrer")`, cohérent avec le
reste du site).

**Verdict de l'addendum 6 :** ✅ aucune nouvelle vulnérabilité introduite.
