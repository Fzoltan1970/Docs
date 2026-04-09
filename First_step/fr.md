# Premiers pas

Le travail réel commence dans la fenêtre **Image Workspace**.
Le traitement commence généralement par le chargement d'une image existante.
Il est aussi possible de créer un dessin dans la fenêtre **Sketch** et de l'envoyer vers le traitement d'image.
Sketch est accessible depuis l'EXE de lancement, on peut y dessiner, puis envoyer le dessin terminé vers le traitement d'image.
La fenêtre principale est utile, mais ce n'est pas la première étape obligatoire.

## 1. Chargement de l'Image

Ouvrez l'Image Workspace et chargez l'image.
Toutes les étapes suivantes partent d'ici.
Sans image, il n'y a pas de traitement.

Ensuite, réglez l'environnement machine de base.

## 2. Machine Mode et Profil

Commencez par choisir le **machine mode**.
En mode **Diode**, vous travaillez avec votre propre profil machine.
En mode **Fiber**, ce n'est pas le même type d'étape bloquante.

En mode Diode, entrez ou chargez un **machine profile**.
Le profil décrit le comportement de la machine.
Le traitement correct et le G-code correct reposent dessus.
Si vous le sautez, le traitement en mode Diode ne sera pas valide.

Vous n'avez pas besoin de connaître les valeurs du firmware par cœur.
Le programme peut communiquer avec la machine via USB et lire les données machine de départ.
Si cela réussit, la base du profil est remplie automatiquement.

La **valeur du module** fait aussi partie du profil.
Si vous avez plusieurs modules laser, il vaut mieux enregistrer un profil distinct pour chacun.
Le module n'est pas qu'une valeur enregistrée : le système en tient aussi compte ensuite.

Enregistrez le profil dès qu'il est utilisable.
Ainsi, vous n'aurez pas à tout ressaisir lors des travaux suivants.
Le profil enregistré peut être rechargé plus tard.

Quand le machine mode et le profil sont prêts, passez à la géométrie.

## 3. Géométrie

Saisissez ensemble les éléments suivants :
- **width**
- **height**
- **DPI**
- **scan axis**

Ensemble, ils définissent le raster et la taille réelle de gravure.
Si l'un d'eux manque ou est incorrect, le résultat ne sera pas correct non plus.

Ensuite, préparez l'image elle-même.

## 4. Préparation de l'Image

Si vous n'utilisez pas toute l'image, définissez un **crop**.
C'est optionnel.

Si l'image doit réellement être tournée, utilisez **Rotate 90**.
Cela agit aussi sur le traitement.

Si vous voulez seulement voir l'image dans une autre orientation, utilisez la rotation de preview.
Ce n'est qu'une couche d'affichage.

Vous arrivez maintenant à la décision la plus importante.

## 5. Mode de Traitement

Choisissez un **mode de traitement**.
Il détermine le résultat final ainsi que le comportement du G-code.

**Grayscale** convient si vous voulez un rendu tonal plus continu.
**Dither** convient si vous voulez travailler avec une image binaire à points.
**Hybrid** est utile si le grayscale est trop lisse et le dither trop dur.
**Depth** convient si vous voulez travailler avec deux branches traitées séparées.

Ensuite, vérifiez la preview.

## 6. Preview et Process

La preview sert au contrôle.
Dans certains modes, elle n'est pas une image directe 1:1 de la gravure finale.

Si les réglages sont corrects, lancez **Process**.
C'est là que l'image traitée est créée.
Cette image devient la base de l'export.

Ensuite, enregistrez le fichier de commande.

## 7. Export

Enregistrez le **G-code**.
Cela ne lance pas encore l'usinage.

L'**overscan** est un réglage côté export.
Il contrôle la façon dont la machine dépasse les bords de l'image.

Le décalage de ligne / line shift / bidirectional compensation est une correction liée au profil.
Ce n'est pas une étape de débutant, ni une décision séparée de traitement d'image.

Quand le G-code est prêt, passez à l'exécution.

## 8. Exécution

Le G-code enregistré s'exécute dans la fenêtre **Sender**.
C'est là que l'exécution réelle a lieu.

Ouvrez Sender.
Sélectionnez le port, puis connectez-vous à la machine.
Si le port n'apparaît pas, vérifiez le câblage et que la machine est bien connectée.

Chargez le G-code enregistré précédemment.
Réglez la position.

Si nécessaire, lancez un **frame** séparé.
C'est optionnel, et utile avant le positionnement.

Ensuite, démarrez le travail.
Si nécessaire, vous pouvez utiliser Pause, Resume et Stop.

## Remarque

Il est utile de distinguer trois types d'enregistrement.

L'**enregistrement du profil** sert à réutiliser la machine.
Il permet de recharger rapidement plus tard la même machine ou le même module.

L'**enregistrement du workspace** sauvegarde l'état complet courant.
Cela inclut l'image réelle et les réglages de traitement.
Utilisez-le si vous voulez revenir plus tard au même travail.

L'**enregistrement DB** est plus limité.
Il s'agit plutôt d'un enregistrement au niveau du record et des paramètres.
Ce n'est pas la même chose que conserver l'état complet du workspace.

Vous pouvez recharger un travail précédent depuis la fenêtre principale.
Le fonctionnement détaillé est décrit dans la section Connaissances.
