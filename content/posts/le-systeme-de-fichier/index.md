---
date: '2026-08-11T13:02:37+02:00'
draft: false
title: 'Le Système de Fichier'
cover: 
    image: arborescence.svg
---

# Introduction

Le systeme de fichier est un élément essentiel d'un ordinateur, il est chargé d'offrir un modèle qui permet d'acceder, ranger et trier ses données, qui peuvent aussi bien être des images, du texte, ou des programmes (binaires).

Ecrire un programme, et plus généralement produire quelque chose avec un ordinateur implique généralement utiliser d'une façon ou d'une autre
le systeme de fichier. Savoir l'utiliser est une compétence souvent présupposé par beaucoup de ressources. 

Faisons donc un petit point pour être certains de ne rien oublier.

# L'arborescence

Tout système de fichiers expose deux types d'objets :
- **Les fichiers**, qui contiennent les données, le système va essayer de distinguer les fichiers en façon de ce qu'ils représentent (image, texte, ect). En pratique, un fichier n'est qu'un tas d'octets et rien d'autre, si un programme décider de créer une image, il ne demande pas au système de créer un fichier image spécifiquement, il ne fait que demander un endroit où écrire quelque chose. 

- **Les dossiers**, qui contient d'autre dossiers et fichiers, les données forment donc un arbre dans lequel les noeux internes sont les dossiers, et chaque feuille est un fichier.

Pour retrouver un fichier, on lui associe souvent une addresse, qui est la suite des dossiers qu'il faut emprumpter pour atteindre le fichier ainsi que le nom du fichier.
Ainsi, une photo de cookie déposée sur mon bureau aura pour addresse: 
```
C:/user/Etienne/desktop/cookie.png
```
Car oui, le bureau est un dossier. Il est parfaitement possible de configurer le système pour utiliser un autre dossier comme bureau.

# Comment interagir avec le système de fichier ?

## 1 / Via explorer.exe

``explorer.exe`` ou plus simplement "l'explorateur de fichier" est la manière la plus simple et la plus fréquente d'interagir avec le système de fichier. Car son utilisation reste relativement intuitive, je ne vais pas le présenter plus que nécessaire.


Cependant le système de fichier se veut accessible, et cache certains détails par défaut aux utilisateurs.
- certains dossiers sont cachés par défaut
- les extensions de fichier (par exemple ``.png``) sont cachées par défaut. Ces extensions ne sont une convention de nommage qui permet d'intenfier quel genre de données le fichier contient. Elles sont généralement l'acronyme du format de données utilisé, par exemple ``png`` est l'acronyme de **P**ortable **N**etwork **G**raphics

Il est fortement conseillé de le configurer pour affichier les dossiers cachés ainsi que les extensions des fichiers: 
![image](config_explorer.png)

## 2 / Via un terminal

Il en existe deux sous Windows, ``cmd`` et ``powershell``, ``powershell`` est essentiellement une version modernisée de ``cmd``.
Il y a deux façons d'ouvrir un terminal: 
- via le raccourci ``windows + r`` puis en entrant ``powershell``, ce qui ouvre le terminal dans le dossier utilisateur: ![image](ouvrir_terminal1.png)
- en pressant clic doit dans l'explorer de fichier, ce qui ouvre le terminal dans le dossier courant: ![image](ouvrir_terminal2.png)

une fois lancé, le terminal se présente sous la forme suivante
```cmd
PS C:\Users\demo\Desktop>
```
Pour l'utiliser, il suffit d'écrire une commande dedans.
Le terminal dispose d'une commande ``help`` qui explique les autres commandes.
Il est généralement possible d'utiliser ``help`` ou ``--help`` derrière la plupart des commandes dans le terminal pour apprendre leur utilisation.
Le terminal peut effectuer la plupart des operations fréquente, cela comprends: 
- ``cd``, qui permet de se déplacer de dossier en dossier, en entrant le chemin relatif (depuis le dossier actif) ou absolu (le chemin entier)
- ``mkdir``, pour "make directory", qui permet de créer un dossier
- ``rmdir``, qui permet de supprimer un dossier
- ``ls``, qui permet d'obtenir une listes des dossiers et fichiers dans le reperatoire actif.

![image](demo_cmd.png)

J'ai crée un dossier dans le bureau, ce dossier est bien visible depuis le terminal et l'explorateur de fichier. Je me déplace dedans pour vérifier le contenu du dossier crée, et sans surprise celui-ci est vide. Je retourne dans le burean en utilisant le chemin relatif ``..`` qui
est un alias pour le chemin du dossier parent.

# Quel est l'interêt du terminal ?

## Démarrer des programmes:

Le terminal peut aussi démarrer des programmes, la commande suivante, démarre firefox: 
```
PS C:\Program Files\Mozilla Firefox> ./firefox.exe
```
La syntaxe ``./`` indique qu'il s'agit d'un chemin relatif
Il est tout à fait possible d'uitiser le chemin absolu, par exemple si le terminal est dans un autre répertoire:
```
PS C:\Users\demo> C:/"Program Files"/"Mozilla Firefox"/firefox.exe
```
Le raccourci firefox présent sur le bureau (l'icon) execute enfaite cette commande, ce qui permet de lancer firefox.

## Passer des arguments:

Certains programmes, lorsqu'on les lance, peuvent prendre des **arguments**. Cette liste d'argument est vide lorsqu'on ouvre un programme depuis l'explorateur de fichier, on est alors contraint d'utiliser un terminal pour pouvoir les utiliser.

Par exemple, firefox peut prendre [tout ces arguments](https://wiki.mozilla.org/Firefox/CommandLineOptions#Windows).
Ce qui permet par exemple d'ouvrir firefox sur un onglet prédéfini:

```
C:/"Program Files"/"Mozilla Firefox"/firefox.exe -new-tab "https://www.youtube.com/watch?v=dQw4w9WgXcQ&list=RDdQw4w9WgXcQ&start_radio=1"
```
Il suffit de copier la commande dans un terminal, dans n'importe quel répertoire pour voir son effet.

Certains programmes qui n'ont pas d'interfaces, s'utilisent uniquement de cette manière, c'est le cas de python par exemple!

# C'est quoi le "Path", ou chemin d'accès ?

Il vous arrivera peut être après avoir installé un outil, de recontrer cette erreur en essayer de l'appeler: 

![image](erreur_path.png)

Le message d'erreur parle de "Verifier si un chemin d'accès existe". Qu'est ce que cela peut bien être ?

Plus étonnant, si l'on doit spécifier une chemin lorsqu'on démare un programme, comment des programmes comme python peuvent-ils être lancés ?
Pour rappel python s'utilise de la manière suivante : 
```cmd
python [nom du fichier a démarer]
```

Lorsqu'on ne spécifie pas de chemin lors de l'appel d'un programme, Window le cherche dans une liste spéciale, communément appelé "Path".
Si le nom entré est accessible, depuis cette liste, alors le programme est appelé. 

Pour modifier cette liste, cherchez ``variable d'environnement système`` ou simplement ``path`` dans le menu démarrer:
![image](var_sys.png)

Ce qui va ouvrir la fenêtre de gauche, il suffit de cliquer sur ``variables d'environnement...`` pour obtenir la fenêtre de droite.

![image](path.png)

Il deux variables qui s'appellent "Path", une est générale, et affecte tous les utilisateurs de l'ordinateur,
la seconde n'est que "locale" et affecte l'utilisateur qui fait le changement (l'utilisateur de cet ordinateur s'appel démo).

Précisement, le path n'est pas une liste des fichiers qui représentes les programmes qui peuvent être appelés n'importe où, mais une liste de
répertoires (de dossiers).

Il est fréquent, lorsqu'on installe un outil de développement, que son installateur propose de "l'ajouter au path". C'est-à-dire d'ajouter le répertoire du logiciel installé pour le rendre disponbile partout ("dans n'importe quel terminal").

Il est parfois necessaire d'aller ajouter soit même un répertoire dans le Path, par exemple si on a refusé que ce soit fait automatiquement lors de l'installation
 d'un outil. Généralement, il n'est pas nécessaire de s'en occuper, mais il est pratique de savoir comment cela fonctionne en cas
de problèmes/erreurs.

# Comment est installé un logiciel sur mon ordinateur ?

Comment firefox est-il installé sur l'ordinateur, si l'icon sur le bureau n'est qu'une ligne de commande en costard ?
A quoi servent les "autres" dossiers présent sur le disque (dans ``C:/``) si toutes les données utilisateurs sont stocké dans ``C:/User`` ?
Comment un logiciel peut il être installé sur un même ordinateur mais avoir des sauvegardes différentes pour tous les utilisateurs ?

La plupart des logiciels séparent leurs fichiers en deux parties: 
- Les fichiers qui contiennent les binaires de firefox (le programme en lui même), qui se trouvent dans ``C:/Program Files/``, ce dossier contient l'intégralité des programmes installé sur l'ordinateur, (si l'installation est correcte). C'est généralement des répertoires fils de ce dossier qui vont être ajouté au path !
- Les sauvegardes sont quand elles beaucoup plus dur a trouvé, elles se trouvent dans un dossier caché accessible avec l'alias ``%LOCALAPPDATA%``, qui est résolu vers le chemin ``C:/Users/[Username]/Appdata/Local/``, dossier est donc différent pour chaque utilisateur.

Le dossier ``C:/Windows/`` contient les fichiers interne de Windows, nécessaire au fonctionnement du système, et ne dois généralement pas être modifié.