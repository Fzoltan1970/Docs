# LaserBase -- Traitement d'image niveau atelier

Ce document décrit le modèle de traitement d'image de LaserBase. L'objectif
est de comprendre son fonctionnement : quelles représentations existent,
dans quel ordre la pipeline s'exécute et comment le programme passe de
l'image RAW au G-code.

Ce n'est pas une spécification développeur. L'accent est mis sur un
modèle mental utilisable et techniquement exact.

------------------------------------------------------------------------

## 1. La Vue d'Ensemble du Système

LaserBase ne repose pas uniquement sur le dither binaire.

Le système prend en charge plusieurs stratégies de traitement :

- raster en niveaux de gris
- raster de dither binaire
- modèle tonal hybride
- traitement depth à deux branches

Pour cette raison, la question centrale n'est pas seulement de choisir un
algorithme de dither, mais de savoir quelle représentation d'image le
mode choisi produit et quel type de G-code en découle.

------------------------------------------------------------------------

## 2. Représentations

### 2.1 RAW

RAW est l'image source chargée.

Il est lié à :

- la rotation réelle de 90°
- le crop
- l'orientation source

Le crop n'est pas défini sur l'image déjà traitée, mais dans l'espace RAW.

### 2.2 Raster Traité

Il s'agit de l'image déjà :

- liée à la taille physique cible
- alignée sur le pas machine
- porteuse des transformations d'image
- prête pour l'export

### 2.3 BASE

BASE dépend du mode.

Il n'est pas correct de l'enseigner comme toujours binaire.

- En **Grayscale**, BASE est en niveaux de gris
- En **Hybrid**, BASE est en niveaux de gris avec motif
- Dans les modes de dither binaire, BASE est binaire

### 2.4 État Depth

En mode depth, le système conserve deux rasters traités :

- une branche grayscale
- une branche dither

La preview peut basculer entre les deux, et l'export est construit en
deux passes.

------------------------------------------------------------------------

## 3. Géométrie, DPI et Alignement Machine

### 3.1 Raster Demandé et Raster Réel

L'utilisateur fournit :

- la taille physique en millimètres
- le DPI demandé
- l'axe de scan
- les données machine

Le programme ne dessine pas directement un raster à partir de cela. Il
calcule un raster réel aligné sur ce que la machine peut pas à pas
produire.

Si les lignes sur l'axe de pas ne peuvent être placées qu'à des
intervalles précis, le pitch réel et le DPI effectif sont ajustés à cette
géométrie autorisée.

### 3.2 DPI Effectif

Le DPI effectif est :

    DPI effectif = 25.4 / real_pitch_mm

Il peut différer du DPI demandé.

### 3.3 Étape de Décision : BASE ou REPAIR

Avant le traitement, le système décide :

- si l'image source suffit pour le raster réel requis
- ou si un conditionnement d'image / rééchantillonnage est d'abord nécessaire

Si le nombre requis de lignes ou de colonnes dépasse ce que l'image
source peut fournir directement, la décision devient **REPAIR**. Sinon,
elle est **BASE**.

------------------------------------------------------------------------

## 4. Espace RAW et Orientation

### 4.1 Rotate 90

`rotate_90` est une transformation géométrique réelle.

Ce n'est pas une astuce de preview. Cela change l'orientation de l'image
source. À cause de cela, les étapes suivantes travaillent déjà sur
l'image RAW tournée :

- crop
- évaluation géométrique
- rééchantillonnage si nécessaire
- construction du raster final

### 4.2 Preview Rotate Back

Preview rotate back est une couche d'affichage séparée.

Elle transforme uniquement la vue. Elle ne modifie pas :

- l'état du noyau
- la définition du crop
- l'image BASE
- le G-code

Les deux rotations restent donc distinctes :

- `rotate_90` = géométrie de traitement
- preview rotate back = confort visuel

------------------------------------------------------------------------

## 5. Modèle de Crop

### 5.1 Pourquoi un Crop en Espace RAW

Le crop est défini sur l'image source afin que la découpe se fasse le
plus tôt possible.

Cela signifie :

- le traitement suivant démarre déjà à partir de la zone recadrée
- les décisions géométriques peuvent utiliser la taille source effective recadrée
- le crop peut être enregistré et restauré avec précision

### 5.2 Formes de Crop

- rectangle / carré
- cercle

Avec un crop circulaire, la géométrie doit rester carrée. Cette règle est
aussi imposée au niveau de l'interface.

------------------------------------------------------------------------

## 6. Modes de Traitement

Le mode de traitement est le choix central du système.

Il définit quelle représentation prend BASE, ce que signifie la preview
et comment le G-code traduit les données raster en puissance et en
mouvement.

### 6.1 Grayscale

Dans ce mode, le raster reste en niveaux de gris. Le générateur G-code
mappe la tonalité du pixel vers des valeurs PWM.

Il ne s'agit pas d'une gravure on/off, mais d'un profil de puissance plus
continu.

### 6.2 Dither Binaire

Les modes par diffusion d'erreur et ordonnés appartiennent à cette
catégorie :

- Floyd--Steinberg
- Atkinson
- JJN
- Stucki
- Bayer
- Halftone clustered

Ici, le résultat final est binaire. Au niveau G-code, un pixel reçoit
soit une brûlure complète, soit aucune.

### 6.3 Hybrid

Hybrid n'est pas un mode de dither classique.

Il part d'une image en niveaux de gris et ajoute une correction de motif
dépendante du ton. Elle est plus forte dans les tons moyens et plus
faible aux extrêmes.

Son rôle est :

- pas du grayscale pur
- pas du dither binaire pur
- un transfert plus texturé des champs tonals

En pratique, Hybrid comble l'écart entre un grayscale trop lisse et un
dither binaire trop dur. Il est utile quand la conservation des tons est
importante, mais qu'un grayscale entièrement lisse ne fournit pas assez
de structure.

### 6.4 Depth

Depth maintient deux branches séparées :

- branche grayscale
- branche dither

Chaque branche peut stocker son propre état de contrôle. La preview passe
de l'une à l'autre et l'export produit deux passes.

Depth est utile lorsque le ton et la structure de points ne doivent pas
être portés par le même raster. Par rapport à un grayscale simple, il
offre un espace de travail séparé pour ces deux rôles.

------------------------------------------------------------------------

## 7. Le Rôle Réel des Contrôles d'Image

### 7.1 Negative

Dans la pipeline, `negative` intervient tôt.

Cela signifie que les étapes ultérieures pour brightness, contrast,
gamma et sharpen travaillent déjà dans l'espace tonal inversé.

### 7.2 Brightness et Contrast

Les deux agissent sur l'image de travail en niveaux de gris.

- brightness : décalage linéaire
- contrast : mise à l'échelle autour du point médian

### 7.3 Gamma

Gamma est une correction non linéaire dans l'espace tonal. Le système
limite la valeur saisie à une plage autorisée et l'applique via une LUT.

### 7.4 Radius et Amount

Le sharpen est basé sur unsharp mask.

### 7.5 Mirror X / Mirror Y

Les opérations miroir font partie des transformations d'image et
s'exécutent après les étapes de modification des tons.

### 7.6 Threshold

Threshold n'est pas un curseur universel.

Il n'affecte que le seuil utilisé par les modes binaires à diffusion
d'erreur. En Bayer, halftone clustered, grayscale et hybrid, son rôle est
différent ou matériellement sans effet.

### 7.7 Serpentine Scan

Ce n'est pas un algorithme de dither séparé, mais une alternance de la
direction de propagation de l'erreur d'une ligne à l'autre.

Son rôle :

- il n'affecte que les modes à diffusion d'erreur
- il n'a pas le même effet en Bayer ou dans les modes non binaires

### 7.8 One Pixel Off

Il s'agit d'une étape de nettoyage appliquée à une sortie raster binaire.

Ce n'est pas un filtre d'image général. Son objectif est de supprimer des
pixels isolés qui ressortent de leur voisinage.

------------------------------------------------------------------------

## 8. L'Ordre Réel de la Pipeline

L'ordre logique est :

1.  charger l'image RAW
2.  `rotate_90` optionnel
3.  crop dans l'espace RAW
4.  conditionnement géométrique / rééchantillonnage si nécessaire
5.  `negative`
6.  `contrast` + `brightness`
7.  `gamma`
8.  sharpen (`radius` + `amount`)
9.  `mirror_x` / `mirror_y`
10. redimensionnement final à la taille raster
11. opération de dither / hybrid selon le mode
12. `one_pixel_off` optionnel

Les conséquences principales sont :

- le crop est une étape précoce côté RAW
- `rotate_90` est appliqué avant le crop
- `negative` n'intervient pas après le dithering
- mirror s'exécute après les contrôles tonals

Cet ordre est important, car le comportement des contrôles en découle. Le
même réglage produit des résultats différents selon l'endroit où il entre
dans la pipeline.

------------------------------------------------------------------------

## 9. Modèle de Preview

### 9.1 La Preview N'a Pas un Sens Unique

La preview de droite est construite à partir de `_right_view_mode` et de
la branche traitée active.

Cette preview n'est pas toujours une image directe 1:1 de la gravure
finale. En dither binaire et en depth, elle montre davantage la structure
de traitement et le caractère du raster que la réponse finale du
matériau.

En pratique, cela signifie :

- il peut ne pas y avoir de preview
- il peut y avoir une vue BASE traitée
- en depth, la preview active peut être grayscale ou dither

### 9.2 Nearest Preview

Cela modifie uniquement l'interpolation d'affichage. L'image traitée
n'est pas modifiée.

### 9.3 Fullscreen

Le fullscreen utilise la même branche preview active que la vue normale à
droite, mais à une échelle plus grande et plus facile à inspecter.

------------------------------------------------------------------------

## 10. Recommandation Auto

### 10.1 Ce Qu'elle Utilise

Les entrées sont :

- clé matériau
- clé technique
- puissance du module utilisateur

Le système de recommandation agrège des enregistrements issus de la base.

### 10.2 Pas une Formule Linéaire Unique

Auto n'est pas seulement une formule de type "puissance proportionnelle à
la vitesse".

La logique combine deux types de recommandation :

1.  recommandation de module basée sur les médianes des enregistrements
2.  recommandation basée sur un proxy d'énergie

Le résultat final peut combiner les deux.

### 10.3 Fallback

Le fallback n'est pas une remontée hiérarchique arbitraire.

Le système :

- cherche d'abord une correspondance exacte de matériau
- peut sinon revenir à des préfixes safe-stop spécifiques
- n'utilise pas de fallback général de famille de niveau supérieur

Cela définit les limites du comportement de fallback.

### 10.4 Comportement UI

En mode Auto, les valeurs recommandées de Speed et Max power sont liées à
une base. Si l'utilisateur modifie l'une, l'autre varie autour de cette
base.

La recommandation est la plus solide lorsqu'elle provient d'une
correspondance exacte. En cas de fallback safe-stop, elle reste un point
de départ plus protégé, mais moins directement adapté au matériau donné.

Le système de recommandation s'appuie sur les enregistrements stockés dans la base.
Ces enregistrements ne se limitent pas à des données prédéfinies : l'utilisateur peut ajouter ses propres entrées.

Les données enregistrées peuvent être exportées et rechargées dans un autre environnement.
Les fichiers de clés importés mettent à jour l'ensemble de données préparé centralement.

------------------------------------------------------------------------

## 11. Overscan

La formule d'overscan est :

    overscan ≈ 1.15 × v² / (2a)

où :

- `v` est la vitesse de scan en mm/s
- `a` est l'accélération de l'axe actif en mm/s²

Le facteur 1.15 agit comme une marge de sécurité.

Du point de vue de l'export, il existe trois états :

- off
- auto
- manual

En mode manual, la valeur saisie remplace le calcul automatique.

------------------------------------------------------------------------

## 12. Stratégie G-code

### 12.1 Grayscale et Hybrid

Ici, le générateur G-code mappe les valeurs de pixel vers une puissance
continue :

    power = s_min + (s_max - s_min) × tone

Pour cette raison, `min_power` agit comme borne basse du mappage tonal.

### 12.2 Dither Binaire

Dans les modes binaires, les pixels noirs reçoivent le niveau de
puissance supérieur et les pixels blancs reçoivent zéro.

Dans ce cas, `min_power` n'est pas l'élément principal de contrôle de
l'évaluation binaire.

### 12.3 Export Depth

En mode depth, le programme :

1.  construit la première passe à partir de la branche dither
2.  construit la deuxième passe à partir de la branche grayscale
3.  concatène les deux blocs G-code

La deuxième passe hérite aussi de certaines valeurs d'état d'export de la
première, il ne s'agit donc pas de deux exports totalement indépendants.

------------------------------------------------------------------------

## 13. Frame

L'export frame est plus qu'une simple case à cocher.

Quand frame est actif :

- le programme ouvre une boîte de dialogue de paramètres séparée
- des valeurs distinctes de speed, power et pass count peuvent être définies
- un fichier frame séparé est généré à côté du G-code enregistré

Avec un crop circulaire, la trajectoire frame est circulaire ; sinon elle
est rectangulaire.

------------------------------------------------------------------------

## 14. Enregistrement et Rechargement

### 14.1 Save Image

Enregistre l'image traitée actuellement active.

En mode depth, cela signifie que la sortie enregistrée n'est pas un
"BASE" abstrait, mais la branche active à ce moment-là.

### 14.2 Save DB

L'action d'enregistrement stocke également l'état du workspace basé sur
des sidecars.

Contenu typique :

- référence image
- mode machine et profil machine
- champs de géométrie
- sélection matériau / technique
- base control
- gcode control
- auto control
- état du crop
- état depth et contrôles des branches

### 14.3 Reload

Reload reconstruit le workspace puis relance le traitement.

Pour cette raison, save/reload a une logique de session, pas seulement de
fiche enregistrée.

------------------------------------------------------------------------

## 15. Sender en Bref, Vue Atelier

Sender n'est pas seulement un flux brut.

Fonctions pertinentes :

- gestion du port et de la connexion
- line / byte / auto stream modes
- pause / resume / stop / e-stop
- alarm unlock
- jog
- laser marqueur
- exécution de frame séparée
- gestion des positions machine-start et work-start
- gestion du point de départ tenant compte du homing

Au niveau atelier, cela compte parce que la gestion du frame et le
positionnement sont eux aussi des processus à plusieurs états dans Sender.

------------------------------------------------------------------------

## 16. Modèle Mental Correct en Résumé

En bref :

- RAW et preview ne sont pas la même chose
- BASE n'est pas toujours binaire
- la preview n'est pas toujours une image directe unique de la gravure finale
- `rotate_90` est une opération géométrique réelle
- preview rotate back ne concerne que l'affichage
- le crop vit dans l'espace RAW
- la recommandation Auto est basée sur la base de données et limitée par safe-stop
- la stratégie G-code dépend du mode
- le mode depth comporte deux branches et deux passes
- l'enregistrement transporte l'état du workspace

Cela donne une image correcte et exploitable du fonctionnement de
LaserBase.
