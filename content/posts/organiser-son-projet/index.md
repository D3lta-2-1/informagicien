---
date: '2026-08-11T13:02:37+02:00'
draft: false
title: 'Organiser un projet logiciel'
cover: 
    image: cover.jpg
---

# Introduction

Structurer un projet quel qu'il soit, qu'il s'agisse d'un jeu, une application mobile, de calculs scientifiques, n'est pas une mince affaire.
Quels outils utiliser ? Comment éviter qu'ils deviennent illisible ? Comment se partager son contenu, que faut-il inclure et exclure ? Comment
permettre à autrui de le lancer ?

# Un projet, qu'est ce que c'est ?

De façon indépendamment du langage, un projet est l'ensemble du **code source** et des **ressources** ou **assets** (images, vidéos, sons, etc.)
utilisés. 
En pratique cela se présente sous la forme d'une arborescence de dossiers et fichiers, organisée d'une manière plus ou moins cohérente.

Le plus petit projet possible est composé d'un unique fichier, mais certains projets comme Linux possèdent plusieurs millions de lignes (27,8
millions) séparés en plusieurs fichiers.

# Comment est transformé le code source en application ?

Avant de pouvoir exécuter (lancer sur l'ordinateur) un projet, un certain nombre de tâches doivent être réaliser :
- récupérer les dépendances du projet, c'est-à-dire du code extérieur que notre projet utilise
- transformer le code source en binaires, c'est-à-dire des suites d'instructions logiques qui ne sont plus lisibles par l'homme, et plus facile a
traiter. (ce processus est généralement appelé **compilation**), généralement plusieurs binaires vont être produits, pour différentes raisons, la
première étant le temps de compilation. Cette tâche étant assez couteuse, séparer en différentes unités permet de ne pas tout recommencer au
moindre changement du code source.
- réunir chacun de ces binaires dans un seul exécutable, qui peut enfin être lancé par le système, cette phase est appelée résolution des symboles
ou **linking** en anglais.

Les langages pour lesquels les deux dernières tâches sont réalisés à l'avance, comme le C, C++, Rust, Go sont dits **AOT** (**A**head **O**f
**T**ime). Avoir le binaire final est donc suffisant pour démarrer le programme sur n'importe quel ordinateur.

Une autre approche consiste à réaliser les deux dernières tâches au dernier instant. C'est par exemple l'approche de python. Il est donnécessaire
d'avoir Python d'installé sur son ordinateur pour exetc.uer un programme en Python. Ces langages sont dits **Interprétés** ou **JIT** (**J**ust
**I**n **T**ime).

Il est tout a fait possible de faire chacune de ses étapes à la main et d'appeler un par un les outils correspondant, mais c'est assez
fastidieux. 

# Automatisation

En pratique, on délègue toutes ces tâches à un programme qui va s'en occuper, le **build system**:
On écrit un fichier de configuration qui décrit comment le code source est découpé, quelles sont ses dépendances et cet outil se débrouille pour
installer ce qu'il faut, puis effetc.uer les tâches nécessaires pour obtenir un exetc.uable. Le second avantage est qu'ils sont généralement plus
futés que nous pour réutiliser les résultats des compilations précédentes.

Utiliser ces outils présente un deuxième avantage, ils simplifient considérablement le partage de projet: ils sont en charge d'installer tout le nécessaire pour que l'exécutable soit corretc.ement produit, et évitent donc à un autre utilisateur de devoir installer les dépendances un à un.

| Langage | Build system            |
| :------ |:-----------------------:|
| C/C++   |   Xmake/Cmake/Menson    |
| Rust    |     Cargo               |
| Java    |     Gradle/Maven        |
| Python  |     Uv                  |
| Ocaml   |     Dune                |

La plupart de ces outils s'utilisent uniquement dans le Terminal mais sont particulièrement bien documentés et disposent de patrons pour commencer plus facilement.

# Exemples 

## En C++

![image](projet-cpp.png)

* le fichier ``xmake.lua`` sert à configurer la construction du projet.
* les sources du programme sont réunies dans un dossier appelé ``src`` (source)
* d'autres outils, qui ne sont pas diretc.ement nécessaire à la construction du projet peuvent parfois disposer eux aussi d'un fichier de configuration à la **racine du projet** (le dossier qui contient l'entièreté du projet), par exemple ce projet dispose d'un fichier ``.gitignore``, qui permet de configurer ``git`` un logiciel de partage de code source
* certains dossiers vont être créé par les outils de développement, par exemple le dossier ``.xmake``, qui contient des informations internes au fonctionnement de xmake, ou ``build``, qui contient l'exécutable du logiciel. Ces dossiers peuvent être supprimés sans problème car ils ne servent qu'à accelerer différentes tâches. Ils seront donc reconstruits à la prochaine exécution du code source.
* il est aussi possible de trouver des données "personnelles", comme la configuration de l'éditeur de texte (ici Zed) que l'on utilise sur ce projet. 

## En Rust

![image](projet-rust.png)

* ici, ce sont les fichiers ``Cargo.toml`` et ``Cargo.lock`` qui ont la même fonction que le ``xmake.lua`` du projet précédent.
* On retrouve toujours un dossier ``src``, qui contient les sources du programme.
* ici le dossier qui contient les binaires générés lors de la compilation s'appelle ``target`` à la place de ``build`` mais assure la même fonction.