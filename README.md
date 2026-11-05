# Module TizenBrew « hakuren-tele »

Un module TizenBrew (2.x) qui ouvre, sur la télé Samsung, la page de Hakuren
servie par le PC : `http://<PC>:8770/tele?jeton=…`. Le module n'embarque
**rien** de Hakuren — ni l'avatar, ni la voix, ni le code de la page : tout
vient du PC, à chaque ouverture. Il ne contient qu'un **lanceur**
(`app/index.html`) et une adresse par défaut (`app/config.js`).

## Le format, vérifié dans le code de TizenBrew (05/11/2026)

Source : `docs/MODULES.md` et
`tizenbrew-app/TizenBrew/service-nextgen/service/utils/moduleLoader.js` du
dépôt [reisxd/TizenBrew](https://github.com/reisxd/TizenBrew).

- Un module est un paquet npm ordinaire dont le `package.json` porte, **au
  premier niveau** (pas de champ `"tizenbrew"`) :
  - `packageType` : `"app"` (une page à soi) ou `"mods"` (un script injecté
    dans un site existant, `websiteURL` + `main`). Il n'y a pas de type
    `"service"` : un service Node s'ajoute à l'un ou l'autre par `serviceFile`.
  - `appName` : le nom affiché ;
  - `appPath` (type `app`) : le HTML d'entrée, ex. `app/index.html` ;
  - `keys` : les touches de la télécommande à réserver (`TVInputDevice`),
    vide ici — Retour n'a pas besoin d'être réservée ;
  - `evaluateScriptOnDocumentStart` (facultatif).
- L'identifiant tapé sur la télé est `<type>/<reste>`, et TizenBrew lit
  `https://cdn.jsdelivr.net/<identifiant>/package.json`. Donc :
  - `gh/<utilisateur>/<depot>@main` → dépôt GitHub **public** (jsDelivr ne sert
    que ça) ; `@main` pour que les mises à jour suivent, sinon jsDelivr garde
    la première version indéfiniment ;
  - `npm/<nom>` → paquet **publié sur npm** (public aussi).
- Une page `app` n'est pas ouverte depuis jsDelivr mais depuis le service local
  de TizenBrew : `http://127.0.0.1:8081/module/<identifiant>/<appPath>`. C'est
  une origine **http**, donc notre lanceur peut naviguer vers le PC en `http://`
  sur le LAN sans blocage de contenu mixte.

**Conséquence : il faut un dépôt GitHub public (ou une publication npm)** pour
installer ce module. C'est pourquoi le jeton n'est PAS dans le dépôt : il se
tape une fois sur la télé et reste dans son `localStorage`.

Un module `mods` avec `websiteURL = http://192.168.1.24:8770/tele?jeton=…`
aurait marché aussi, en une ligne — mais le jeton aurait vécu dans un
`package.json` public. Refusé.

## Installer

1. Mettre le contenu de ce dossier (`package.json`, `app/`) à la racine d'un
   dépôt GitHub **public**, par exemple `hakkanodev/hakuren-tele`, branche
   `main`. Rien d'autre n'est nécessaire (pas de build).
2. Dans `app/config.js`, vérifier l'adresse du PC (`http://192.168.1.24:8770`).
   Si l'IP change (DHCP), ce fichier change aussi — ou mieux : réserver l'IP du
   PC dans la Bbox.
3. Sur la télé : TizenBrew → bouton **vert** de la télécommande (gestionnaire de
   modules) → **Ajouter** → type **GitHub** → taper `<utilisateur>/<depot>@main`
   (TizenBrew ajoute `gh/` devant). La saisie est envoyée quand le champ perd le
   focus (pas sur Entrée).
4. Lancer « Hakuren » dans la liste. Au premier lancement, le lanceur demande
   l'adresse (pré-remplie) et le **jeton** (`mobile.token` du PC, le même que
   l'appli iPhone). Ensuite, il ouvre la page directement.

Côté PC, trois conditions (voir `../README.md`) : la page construite
(`tele.generee.html`), les routes câblées dans `mobile.py`, et le serveur
**joignable depuis le LAN** (aujourd'hui il n'écoute que `127.0.0.1`).

## Sécurité, à lire avant de publier

- Le jeton est celui de l'API mobile : il permet de **piloter le PC** (envoyer
  des ordres à Hakuren), pas seulement de voir l'avatar. Sur la télé, il vit
  dans le `localStorage` de l'origine `127.0.0.1:8081`, **partagée par tous les
  modules TizenBrew** : un module malveillant installé sur la même télé pourrait
  le lire. Aujourd'hui seul TizenTube y est installé. Si on veut mieux : un
  second jeton, propre à la télé, limité aux routes de lecture (`/tele`,
  `/api/tele/etat`, `/api/avatar…`, `/api/voix`) — à faire dans `mobile.py`.
- Le trafic est en **http** sur le Wi-Fi (chiffré par WPA2 au niveau radio, en
  clair pour qui est sur le réseau). Le jeton voyage dans l'URL et l'en-tête.
- Le dépôt du module est public : ne jamais y mettre le jeton, ni le nom du
  Wi-Fi, ni autre chose que l'adresse LAN du PC.

## Sans rien installer : Samsung Internet

Sur la télé, ouvrir le navigateur **Internet** et taper
`http://192.168.1.24:8770/tele?jeton=<le jeton>` (long à la télécommande : un
clavier USB sur la télé aide, puis mettre la page en favori). Limites, non
vérifiées d'ici : WebGL est en principe disponible dans le navigateur des Tizen
récents, mais la barre d'adresse reste visible tant qu'on ne passe pas le
navigateur en plein écran (menu du navigateur), le son attend un appui sur la
télécommande (politique de lecture automatique — la page le dit en haut à
droite), et Retour ferme la page au lieu de quitter proprement.
