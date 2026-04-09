# LaserBase -- Manuel d'utilisation

Ce manuel décrit le fonctionnement de LaserBase. L'objectif n'est pas de
développer chaque détail interne, mais de rendre le programme utilisable
correctement et d'expliquer clairement ce qui se passe entre l'image et
le G-code.

Le texte suit le flux de travail réel.

------------------------------------------------------------------------

# 1. Ce Qu'est LaserBase

LaserBase est une suite logicielle conçue pour préparer et exécuter des
travaux de gravure laser.

Ses parties principales sont :

- **Fenêtre principale** -- base de paramètres et gestion des enregistrements
- **Image Workspace** -- traitement d'image, preview, export G-code
- **Sender** -- envoi du G-code vers la machine
- **Sketch** -- surface simple de dessin et d'esquisse

LaserBase n'est pas un éditeur d'image généraliste. Son rôle est de
transformer une image en raster adapté à la machine et en fichier de
commande exploitable.

------------------------------------------------------------------------

# 2. La Base de la Gravure

Le résultat de gravure est déterminé ensemble par trois facteurs :

1.  la puissance laser
2.  la vitesse de déplacement
3.  la densité de points ou de lignes

Pour cette raison, LaserBase ne traite pas seulement une image. Le
programme relie aussi l'image à une taille physique, à un DPI, à la
géométrie machine et aux paramètres G-code.

------------------------------------------------------------------------

# 3. Le Flux de Travail Complet

Un travail typique se déroule ainsi :

1.  charger l'image
2.  définir la taille, le DPI et les données machine
3.  définir le crop et la géométrie
4.  choisir le mode de traitement
5.  vérifier la preview
6.  régler vitesse et puissance, ou utiliser la recommandation automatique
7.  enregistrer le G-code ou l'image traitée
8.  enregistrer un enregistrement et le recharger plus tard si nécessaire

------------------------------------------------------------------------

# 4. Image Workspace

L'Image Workspace est la partie centrale du programme.

L'image source apparaît à gauche, la vue traitée à droite. Cette vue de
droite n'a pas toujours le même sens : selon le mode sélectionné, elle
peut montrer des niveaux de gris, un dither binaire, ou dans le mode
depth l'une des branches traitées.

Pour cette raison, la preview doit toujours être interprétée avec le mode
de traitement sélectionné.

------------------------------------------------------------------------

# 5. Chargement d'une Image

Utiliser **Load image** pour charger l'image.

Formats pris en charge :

- PNG
- JPG / JPEG
- BMP

Après le chargement, le programme stocke l'image source et l'utilise comme
point de départ de tout le traitement.

------------------------------------------------------------------------

# 6. Image RAW, Image Traitée et BASE

Le programme utilise plusieurs niveaux d'image.

**Image RAW**

Il s'agit de l'image source chargée. Le crop et la rotation réelle se
rapportent à cette image.

**Image traitée**

Il s'agit du raster déjà aligné sur la taille physique choisie, le DPI et
la géométrie machine, avec les transformations d'image déjà appliquées.

**BASE**

BASE n'est pas toujours binaire.

- En mode **Grayscale**, BASE est un raster en niveaux de gris
- En mode **Hybrid**, BASE est une image tonale en niveaux de gris avec motif
- Dans les modes de dither binaire, BASE est une image de points noir et blanc

Le G-code est toujours généré à partir du raster traité actif.

------------------------------------------------------------------------

# 7. Taille et DPI

La taille de gravure est donnée en millimètres.

Le DPI définit la densité des lignes ou des points :

    espacement des lignes (mm) = 25.4 / DPI

Le DPI demandé n'est pas toujours identique au DPI physiquement utilisé
par la machine. LaserBase construit un raster réel aligné sur le système
de pas de la machine, de sorte que la taille d'image traitée et le DPI
effectif peuvent différer des valeurs demandées.

------------------------------------------------------------------------

# 8. Profil Machine

Le profil machine contient les données physiques de la machine :

- steps/mm par axe
- max rate
- acceleration
- module laser
- données de profil G-code

Ces valeurs ne servent pas uniquement à l'export. Le programme les
utilise aussi pour l'alignement du raster et le calcul automatique de
l'overscan.

En mode fiber, le workspace utilise un profil machine virtuel ; en mode
diode, les données machine font partie du traitement lui-même.

------------------------------------------------------------------------

# 9. Crop

Le crop est défini dans l'espace RAW. C'est important.

Cela signifie que la découpe n'est pas appliquée à l'image déjà traitée,
mais à l'image source avant le traitement suivant.

Formes disponibles :

- carré / rectangle
- cercle

Pour un crop circulaire, la largeur et la hauteur doivent être identiques.
Si le crop est actif mais invalide, le bouton Process ne s'exécute pas.

------------------------------------------------------------------------

# 10. Rotation Réelle et Preview Rotate Back

Les deux rotations ne sont pas la même chose.

**Rotate 90**

Il s'agit d'une transformation géométrique réelle. Elle change
l'orientation de l'image source, et le crop, la géométrie et le
traitement suivent cette orientation.

**Preview rotate back**

Il s'agit uniquement d'une couche d'affichage. Le traitement et le
G-code ne sont pas modifiés. Seules les vues preview de gauche et de
droite sont remises dans l'autre orientation.

Si l'image doit être tournée pour la machine, **Rotate 90** est le bon
contrôle. Si le but est seulement d'observer l'image plus confortablement,
preview rotate back est l'outil adapté.

------------------------------------------------------------------------

# 11. Modes de Traitement

Le mode de traitement est la décision centrale dans le workspace.

Il ne définit pas seulement quel algorithme s'exécute. Il définit aussi
quel type de BASE est produit, ce que signifie la preview et comment le
G-code est généré.

LaserBase n'est pas limité au dither binaire.

**Grayscale**

Le raster traité reste en niveaux de gris. Le G-code attribue une valeur
PWM à chaque tonalité de pixel.

**Hybrid**

Il s'agit d'un mode basé sur les niveaux de gris qui traite les tons de
manière plus structurée. Ce n'est ni un dither binaire classique, ni un
grayscale pur.

Hybrid est utile lorsque le grayscale pur paraît trop plat et le dither
binaire trop dur. Il se situe entre le champ tonal et la structure de
points.

**Modes de dither binaire**

- Floyd--Steinberg
- Atkinson
- JJN
- Stucki
- Bayer
- Halftone clustered

Dans ces modes, le raster final est binaire : un point brûle ou ne brûle
pas.

**Depth**

Il ne s'agit pas d'un algorithme de dither séparé, mais d'un modèle de
traitement à deux branches.

En mode depth, le programme conserve une branche grayscale et une branche
dither, et la preview peut basculer entre elles. À l'export, le G-code
est construit en deux passes.

Depth est utile lorsque la restitution des tons et la structure de points
ne doivent pas être portées par un seul raster. Par rapport à un
grayscale simple, il fournit deux branches traitées distinctes avec des
rôles différents.

------------------------------------------------------------------------

# 12. Contrôles d'Image

**Contrast, Brightness, Gamma, Radius, Amount**

Ces contrôles façonnent l'entrée de l'étape de traitement.

**Negative**

Inversion tonale.

**Mirror X / Mirror Y**

Miroir géométrique de l'image.

**Threshold**

Ce n'est pas un curseur d'image général. Il agit sur le seuil des modes
binaires par diffusion d'erreur.

**Serpentine scan**

Dans les modes de dither par diffusion d'erreur, il inverse le sens de
traitement d'une ligne sur deux. Ce n'est pas un mode de dither
indépendant, mais un commutateur complémentaire.

**1 pixel off**

Dans les modes binaires, c'est une étape de nettoyage qui peut supprimer
des pixels isolés.

------------------------------------------------------------------------

# 13. Recommandation Auto

**Auto** n'est pas un simple calculateur de rapport.

La recommandation est construite à partir de la base de données en
utilisant :

- la technique
- le matériau
- le module laser

Le système essaie d'en déduire des valeurs de Speed / Max power / Min power.

S'il n'existe pas de correspondance exacte, le programme utilise une
logique de fallback. Il ne remonte pas arbitrairement tous les niveaux ;
il ne peut revenir que vers des groupes safe-stop pris en charge. Quand
cela se produit, le programme l'indique et peut demander une
confirmation.

En mode Auto, Speed et Max power restent liés : si l'un est modifié
manuellement, l'autre est ajusté autour de la base de recommandation
actuelle.

Auto est le plus fiable lorsqu'il repose sur une correspondance exacte
matériau-technique. En cas de fallback, il faut surtout le considérer
comme un point de départ sûr, pas comme une valeur finale.

Le système de recommandation s'appuie sur les enregistrements stockés dans la base.
Ces enregistrements ne se limitent pas à des données prédéfinies : l'utilisateur peut ajouter ses propres entrées.

Les données enregistrées peuvent être exportées et rechargées dans un autre environnement.
Les fichiers de clés importés mettent à jour l'ensemble de données préparé centralement.

------------------------------------------------------------------------

# 14. Overscan

L'overscan est la distance d'entrée et de sortie en dehors des limites de
l'image.

En mode automatique, LaserBase le calcule à partir de la vitesse et de
l'accélération d'axe. Une valeur manuelle peut aussi être saisie.

Si l'overscan est trop faible, les bords de l'image peuvent être déformés
parce que la machine n'a pas encore atteint une vitesse constante.

------------------------------------------------------------------------

# 15. Traitement et Étape de Décision

**Process** ne se contente pas d'exécuter un filtre.

Avant le traitement, le programme évalue :

- la taille demandée
- le DPI demandé
- la géométrie de pas de la machine
- la résolution réelle de l'image source

Le résultat est :

- **BASE** -- le traitement peut être exécuté directement
- **REPAIR** -- un conditionnement d'image / rééchantillonnage est nécessaire pour le raster cible

Le raster traité réel est construit après cette décision.

------------------------------------------------------------------------

# 16. Preview

La preview de droite montre la branche traitée sélectionnée.

Cette preview n'est pas dans tous les modes une image directe 1:1 de la
gravure finale. C'est particulièrement vrai pour le dither binaire et le
mode depth, où elle montre davantage la logique du raster que la réponse
du matériau.

En mode depth, deux previews sont disponibles :

- Grayscale
- Dither

**Nearest preview** change la manière d'afficher l'image, pas la manière
de la traiter.

En mode fullscreen, il est utile de vérifier :

- les transitions tonales
- les stries
- le sur-accentuage
- la densité de points binaires
- la différence entre les deux branches en mode depth

------------------------------------------------------------------------

# 17. Génération du G-code

La stratégie G-code dépend du mode.

**Grayscale / Hybrid**

Le programme dérive les valeurs PWM à partir de la tonalité du pixel, de
sorte que la puissance laser peut varier au sein d'une ligne.

**Dither binaire**

Les points noirs brûlent avec la puissance maximale configurée, les points
blancs ne brûlent pas.

**Depth**

Deux passes successives sont produites : la branche dither et la branche
grayscale reçoivent chacune leur propre bloc G-code.

------------------------------------------------------------------------

# 18. Save Image, G-code et Frame

**Save image**

Enregistre l'image traitée actuellement active. Cela peut être la branche
grayscale, hybrid ou dither.

**G-code**

Enregistre le fichier de commande de gravure.

**Frame**

S'il est activé, le programme crée aussi un fichier de frame séparé. Ses
paramètres sont saisis dans une boîte de dialogue dédiée. Avec un crop
circulaire, le frame est circulaire ; sinon il est rectangulaire.

------------------------------------------------------------------------

# 19. Enregistrement dans la Base

La fenêtre principale fonctionne comme base de paramètres, mais
l'enregistrement du workspace contient davantage.

Lors de l'enregistrement, le programme stocke :

- la référence à l'image source
- l'état machine et géométrie
- l'état du crop
- les contrôles BASE et G-code
- l'état Auto
- les réglages des deux branches en mode depth

Pour cette raison, le reload n'est pas seulement un remplissage
d'enregistrement, mais une restauration de l'état du workspace.

------------------------------------------------------------------------

# 20. Rechargement

Un enregistrement de workspace sauvegardé peut être rechargé depuis la
fenêtre principale.

Dans ce cas, le programme :

1.  recharge l'image source
2.  restaure taille, DPI et données machine
3.  restaure le crop et les contrôles de traitement
4.  relance le traitement

L'état restauré est donc de type session, et non une simple liste de
paramètres.

------------------------------------------------------------------------

# 21. Sender

Sender fonctionne dans une fenêtre séparée.

Fonctions de base :

1.  sélectionner un port
2.  connecter
3.  charger le G-code
4.  lancer l'envoi

Pendant l'utilisation :

- Pause / Resume
- STOP
- E-STOP
- $X Unlock en état d'alarme
- déplacement jog manuel
- laser marqueur
- commandes terminal

Sender prend aussi en charge un frame séparé, la mémorisation des
positions machine-start et work-start, ainsi que le positionnement par
offset.

------------------------------------------------------------------------

# 22. Fenêtre Principale, New Entry, Calculator, Sketch

**Fenêtre principale**

Stocke et gère les enregistrements.

**New Entry**

Crée un nouvel enregistrement.

**Calculator**

Aide à calculer de nouveaux paramètres proportionnels à partir
d'enregistrements existants.

**Sketch**

Surface simple de dessin et d'esquisse dans une fenêtre séparée.

------------------------------------------------------------------------

# 23. Problèmes Courants

- **Pas d'image** -- charger d'abord une image source
- **Pas de profil machine en mode diode** -- en saisir un ou en charger un
- **Crop invalide** -- le corriger ou le désactiver
- **Crop circulaire avec dimensions inégales** -- largeur et hauteur doivent être identiques
- **Pas de recommandation auto** -- il n'existe pas de données adaptées à cette combinaison matériau / technique / module
- **Le G-code ne peut pas être enregistré** -- le traitement doit d'abord se terminer correctement

------------------------------------------------------------------------

# 24. Pratique Utile

- Sur un nouveau matériau, vérifier d'abord la preview grayscale et la preview dither binaire.
- Si l'image doit être tournée pour la machine, utiliser le vrai Rotate 90.
- Si l'objectif est seulement une orientation d'affichage différente, utiliser preview rotate back.
- En mode Auto, vérifier si la recommandation est exacte ou basée sur un fallback.
- Utiliser depth lorsque les deux rôles raster doivent être gérés séparément.
- Utiliser l'export Frame avant de positionner le travail réel.
- Considérer l'enregistrement du workspace comme un enregistrement d'état de travail, pas seulement comme un enregistrement de fiche.

------------------------------------------------------------------------

# 25. Référence Rapide

    espacement des lignes (mm) = 25.4 / DPI
    DPI effectif = 25.4 / actual pitch_mm

Modes :

- Grayscale -> raster PWM basé sur la tonalité
- Hybrid -> raster en niveaux de gris avec motif
- Dither -> raster binaire à points
- Depth -> traitement à deux branches et export en deux passes

Rotation :

- Rotate 90 -> changement géométrique réel
- Preview rotate back -> affichage uniquement

Enregistrement :

- Save image -> branche traitée active
- Save db -> état du workspace + enregistrement
- G-code -> fichier de commande, avec éventuellement un fichier de frame
