# CMB — Vérificateur de Bons de Commande (Application PWA)

Application web installable permettant de vérifier l'authenticité des bons de commande CMB par **scan du QR code**, avec **fonctionnement hors ligne** une fois installée.

## Contenu du paquet

```
index.html                  → l'application
manifest.webmanifest        → déclaration PWA (nom, icônes, couleurs)
sw.js                       → service worker (cache hors ligne)
lib/html5-qrcode.min.js     → scan QR via caméra (embarqué, pas de CDN)
lib/nacl.min.js             → vérification de signature Ed25519 (embarqué)
icons/                      → icônes de l'application (CMB)
```

Aucune dépendance externe, aucune donnée transmise : la clé publique CMB est intégrée dans `index.html` et toute la vérification se fait sur l'appareil.

## Pourquoi un hébergement HTTPS est nécessaire

Les navigateurs n'autorisent l'accès à la **caméra** et l'installation d'une **PWA** que sur une origine sécurisée (HTTPS) ou sur `localhost`. Un simple double-clic sur `index.html` (mode `file://`) **désactive la caméra** ; dans ce cas l'application bascule automatiquement sur la saisie manuelle.

## Déploiement (3 options)

### Option A — GitHub Pages (gratuit, HTTPS automatique)
1. Créer un dépôt (ex. `cmb-verif`), y déposer tout le contenu de ce dossier.
2. Settings → Pages → Source : branche `main`, dossier `/root`.
3. L'URL fournie (`https://<compte>.github.io/cmb-verif/`) est prête.

### Option B — Hébergeur CMB existant
Copier le dossier dans un répertoire servi en HTTPS (ex. `https://verif.cmb.ci`). Vérifier que le type MIME de `.webmanifest` est `application/manifest+json` (la plupart des serveurs le gèrent par défaut).

### Option C — Netlify / Cloudflare Pages
Glisser-déposer le dossier sur l'interface ; l'HTTPS et l'URL sont générés automatiquement.

## Installation sur smartphone (par l'agent de contrôle)

**Android (Chrome)**
1. Ouvrir l'URL de l'application.
2. Menu ⋮ → « Ajouter à l'écran d'accueil » (ou la bannière proposée).
3. Une icône CMB apparaît ; l'app s'ouvre en plein écran.

**iPhone (Safari)**
1. Ouvrir l'URL dans Safari.
2. Bouton Partager → « Sur l'écran d'accueil ».
3. L'icône CMB est ajoutée.

Au **premier lancement en ligne**, le service worker met l'application en cache. Ensuite, **elle fonctionne sans connexion** (y compris la caméra et la vérification).

## Utilisation

1. Ouvrir l'app, appuyer sur **« Scanner le QR code »**, autoriser la caméra.
2. Viser le QR du bon de commande.
3. Résultat immédiat :
   - **Vert / AUTHENTIQUE** : signature valide, détails du BC affichés.
   - **Rouge / NON VALIDE** : document falsifié ou non émis par CMB.

En l'absence de caméra, utiliser **« Saisir / coller le code manuellement »**.

## Mise à jour des clés

Si la paire de clés est régénérée au siège, remplacer la valeur `CMB_PUB_B64` en haut du `<script>` de `index.html` par le nouveau contenu de `CMB_cle_publique.b64`, puis incrémenter `const CACHE = "cmb-verif-v1"` → `v2` dans `sw.js` pour forcer la mise à jour chez les agents.

## Sécurité

- La clé **publique** embarquée ne présente aucun risque : elle ne permet que de vérifier, jamais de signer.
- La clé **privée** reste exclusivement au siège ; elle ne figure pas dans ce paquet.
