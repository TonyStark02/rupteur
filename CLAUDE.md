# Rupteur

Page web qui joue un son de moteur thermique dans une voiture électrique (navigateur Tesla, ou téléphone relié en Bluetooth). Le son suit la vitesse GPS. Même principe que dribe.app / teslasound.io, **sans paiement ni scènes d'ambiance** : uniquement le moteur.

Anciennement « Banc Moteur ». État d'avancement et prochaines étapes : `state.json`.

## Fichiers
- `index.html` : toute l'app (HTML, CSS, JS, aucune dépendance, aucun build). Icônes intégrées en data URI.
- `assets/` : logo source (`logo.svg`) et rendus PNG 512 / 180.
- `samples/README.md` : guide d'enregistrement et de découpe des boucles.
- `.github/workflows/pages.yml` : publication GitHub Pages (Source : GitHub Actions).

## Architecture (`index.html`)
- **Profils** (`PROFILES`) : un objet par moteur, groupé par origine (`group`). Champs : `cyl`, `amp` (exactement `cyl` valeurs), `res`, `decay`, `noise`, `drive`, `lp`, `power`, `crk`, `idle`/`redline`/`limiter`, `final`, `gears`, `turbo`/`spool`/`dv`, `sc` (compresseur), `cam` (VTEC), `layout` (libellé spécial). `applyProfile` copie les paramètres dans `CAR` et calcule `RPMS`.
- **Simulation** (`step`, appelée à chaque image) :
  - vitesse : mode Test (accélération intégrée, frein moteur dépendant du régime et du rapport) ou GPS (`watchPosition`, vitesse lissée, accélération dérivée → `S.throttle`) ;
  - régime : à vide, montée et descente à taux limité (`freeRev`) ; en prise, il suit la roue, avec patinage d'embrayage en 1re et sursaut `S.kick` sur coup de pédale ;
  - rupteur, boîte auto (seuils fonction de l'agressivité et de la zone rouge), manuelle avec Q/E ;
  - turbo : `S.boost` monte avec charge et régime, reste piégé pied levé jusqu'à la soupape ;
  - dump valve : déclenchée quand la pédale chute de plus de 35 % avec pression (`S.thrRef`, `S.dvCool`), et aux passages montants ;
  - pétarades : crépitement aléatoire en frein moteur, plus dense juste après le lever (`S.liftT`), plus une possible au lever (`S.armed`), au passage à fond et au rupteur.
- **Audio** (`initAudio`, `mixVoices`) :
  - banque par moteur (`banks[id]`, `bank.on` / `bank.off` par régime) ; une source en boucle par sample, poids en fondu à puissance constante entre les deux régimes voisins, `playbackRate = rpm / rpmSample`, mélange on/off selon `S.load` ;
  - chaîne moteur : `lp` (s'ouvre avec la charge) → `pre` (attaque de saturation) → `shaper` → `master` → compresseur ; rugissement d'admission (`roarG`) injecté dans `pre` ;
  - effets sur `fxBus`, hors compresseur moteur avec un limiteur dédié : `pop`, `dumpValve` (pschitt ou flutter synthétisés), sifflement turbo / compresseur ;
  - boucles par défaut générées par `synthLoop` : longueur = nombre entier de cycles moteur, pour un bouclage sans clic.
- **Import** : `on_<rpm>.wav` / `off_<rpm>.wav` vont dans la banque du moteur courant ; `pschitt|psh|bov*`, `flutter*`, `pop*` vont dans `fxS` et remplacent la synthèse pour tous les moteurs. Rien n'est conservé au rechargement.
- **Interface** : choix du moteur dans un panneau à onglets (`openSheet`, `renderEngines`). Turbo et Dump valve sont des sélecteurs segmentés (`.seg` + input caché) réinitialisés sur la config d'origine à chaque changement de moteur (`applyEngineDefaults`, point ambre = d'origine). Seuls le moteur et la boîte sont mémorisés en localStorage.
- **Écran large** (≥ 900 px, ordinateur ou écran de voiture en paysage) : deux colonnes, `.drive` (compteur, pédales) à gauche, `.side` (moteur, réglages, banque) à droite ; le panneau moteur devient une fenêtre centrée. Clavier : W/S ou ↑/↓ pour les pédales, Q/E ou ←/→ pour les rapports ; raccourcis affichés sur les pédales seulement avec une souris.
- **iOS** : `unlockIOS` force la session audio en lecture pour passer outre le bouton silencieux.

## Règles
- Pas de paiement, pas de scènes ou d'ambiances : uniquement le moteur.
- Pas de marques constructeurs dans les noms, les visuels ou le dépôt.
- Toute l'app reste dans un seul `index.html` sans dépendance externe hors Google Fonts.
- Le GPS exige HTTPS : tester en mode Test en local, en mode GPS seulement une fois publié.
- Interface en français.

## Vérifier une modification
- Syntaxe : extraire le `<script>` et lancer `node --check`.
- Simulation sans navigateur : évaluer le script jusqu'à `/* ---------- GPS` avec un faux `document.getElementById` qui renvoie `{value}`, puis appeler `step(1/60)` en boucle.
