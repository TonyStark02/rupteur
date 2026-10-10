# Enregistrer des boucles pour Rupteur

## Ce qu'il faut

Pour chaque moteur, deux jeux de boucles à régime stabilisé :

- **on** : moteur en charge (pied au plancher, sur banc à rouleaux ou en montée).
- **off** : moteur pied levé (frein moteur, en descente en prise).

Régimes conseillés : 1 000, 1 500, 2 000, puis tous les 1 000 tr/min jusqu'au rupteur. Plus il y a de points, moins chaque boucle est étirée en hauteur, et plus le son reste naturel.

## Prise de son

- Micro près de la sortie d'échappement, à l'abri du vent (bonnette).
- 48 kHz, 24 bits, sans compression ni normalisation automatique.
- 3 à 5 secondes par régime, régime aussi stable que possible.

## Découpe

Une boucle doit pouvoir se répéter sans clic :

1. Garde une portion d'environ 1 à 2 secondes au régime le plus stable.
2. Coupe sur un **nombre entier de cycles moteur**. Un cycle dure `120 / régime` secondes, soit 40 ms à 3 000 tr/min.
3. Place les points de coupe sur des passages par zéro.
4. Exporte en mono WAV, nommé `on_3000.wav`, `off_3000.wav`, etc.

## Effets

- `pschitt.wav` et `flutter.wav` : un seul déclenchement, du début du souffle jusqu'au silence.
- `pop1.wav`, `pop2.wav`… : une détonation par fichier, très courte (50 à 300 ms), sans silence autour. Plusieurs variantes rendent le crépitement moins répétitif.

## Droits

N'utilise que des enregistrements que tu as faits toi-même ou dont la licence autorise la réutilisation. Ne commite pas dans un dépôt public des sons sous licence restrictive.
