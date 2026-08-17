---
date: '2026-08-17T13:02:37+02:00'
draft: false
title: 'Présentation de git'
---

# Kesko

Git est un logiciel de gestion de version décentralisé. Il permet d'envoyer et de récuperer son code source sur un serveur distant.
Il est possible d'héberger son propre serveur git soit même, mais la plupart des utilisateurs utilise git pour partager leur créations.
Il est donc fortement conseillé d'utiliser un service d'hébergement, comme [github](https://github.com/) ou [gitlab](https://about.gitlab.com/fr-fr/).

# installer git sous window

```cmd
winget install git
```

# Concept généraux de Git

## initialiser un repos.

Git permet de cloner l'arboresence d'un dossier (un projet) et de la synchroniser avec celle d'un serveur.
Pour utiliser git dans son projet, il faut généralement "l'initialiser" en éxecutant la commande suivante dans le dossier correspondant.
```cmd
git init
```

## ignorer des fichiers

Il est possible de créer, si ce n'est pas déjà présent, un fichier nommé ``.gitingore`` qui contient une liste des fichiers ou dossier que git doit ignorer.
Il contient typiquement tous les dossiers et fichiers de autogénérés localements, ce qui comprends: 
- le dossier ``.xmake``
- le dossier ``build``
- le fichier ``compile_commands.json``

ce qui donne par exemple le fichier suivant : 
```
# Xmake cache
.xmake/
build/

# clangd configuration
compile_commands.json

# MacOS Cache
.DS_Store
```
Les ``#`` sont des commentaires.

# Envoyer son code sur Github

Git est assez verbreux, il y a trois phases pour envoyer sont code: 
- selectionner les fichiers concernés, une façon de faire est d'utiliser la commande ``git add``, ``git add .`` permet d'ajouter tous les fichiers de manière récursive, ``git status`` permet de voir les fichiers trackés, ils sont souvent affichés en vert/rouge/bleu dans l'éditeur de texte en fonction de leur status.
- ajouter ses fichiers a un **commit**, c'est-à-dire un ensemble de modification qui partagent un même nom. Généralement, on essaye de décrire la fonctionnalité que l'on implémente dans le nom. On utilise généralement la commande ``git commit -m "nom du commit"``
- envoyer tous les commits sur un serveur avec ``git push``. Il faut configurer au préalable le serveur distant. Sur github, il suffit de créer un repos et de suivre la procédure indiqué (la seconde, si l'on dispose déjà d'un repos).

