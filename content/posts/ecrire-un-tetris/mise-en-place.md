---
date: '2026-10-24T13:02:37+02:00'
draft: false
title: 'Tetris: Introduction et mise en place'
---

# Introduction

Tetris est un célèbre jeu de des années 1980, simple mais néanmoins amusant.
C'est un bon premier projet qui permet d'apprendre à gerer les erreurs, dessiner a l'écran, interagir avec l'utilisateur en temps réel, interagir avec du code C.

L'exercice consiste à implémentater de Tetris en [C++23](https://cppreference.com/cpp/23) en utilisant la [SDL3](https://wiki.libsdl.org/SDL3/FrontPage).

Ce guide risque d'utilser bon nombres de nouvelles fonctions et fonctionnalités plus avancées de C++. Il est normal que ces notions soit nouvelles, l'interêt d'un tel projet est de les recontrer "en pratique".
Le bon reflexe a avoir est de consulter la documentation pour comprendre quel leur utilité. Devenir un developpeur autonome c'est avant tout savoir utiliser la documentation par soit même.

La SDL étant écrite en C, un certe travail de "glue" est nécessaire pour que les choses "se passent bien". Il ne faut pas s'étonner si certaines fonctions sont un peu rude à utiliser. 

# Mise en place

Placer dans votre dossier habituel pour vous projets.
entrez les commandes suivantes : 
```
xmake create tetris
zed ./tetris
```

Il faut maintenant éditer le ``xmake.lua`` pour ajouter la SDL a notre project Tetris.
On en profite aussi pour activer C++23.
```lua
add_rules("mode.debug", "mode.release")

add_requires("libsdl3") --on depend de la SDL3

target("tetris") -- on definit un projet tetris
    set_kind("binary") --ce projet est un executable
    set_languages("cxx23") -- on utilise C++23
    add_packages("libsdl3") -- on ajoute la SDL3 a ce projet
    add_files("src/*.cpp") -- tous nos fichiers sources se trouvent dans le dossier "src"
```
Il faut maintenant générer les fichiers de configurations de Clangd dans Zed, cela va aussi contraintre xmake a installer la SDL3 sur notre système.

```
xmake project -k compile_commands.json
```

# Présentation des callbacks fourni par la SDL.

La SDL propose [different moyen de démarrer un programme](https://wiki.libsdl.org/SDL3/README-main-functions).
Il y a deux façon de démarrer un programme:
- utiliser une fonctionne ``main`` comme jusqu'à présent, et récuperer soit même les évènements.
- laisser la SDL enrigistrer des "callbacks" pour nous. Cette méthode est préférable car la SDL peut mieux s'adapter aux différences entre les différentes platforme.

Dans les deux cas, la SDL est conçu pour remplacer notre fonction ``main`` usuelle
Les points d'entrées par défaut ressemble alors à
```cpp
#define SDL_MAIN_USE_CALLBACKS
#include <SDL3/SDL_init.h>
#include "SDL3/SDL_main.h"

SDL_AppResult SDL_AppInit(void** appstate, int argc, char **argv) {
  return SDL_APP_CONTINUE;
}

SDL_AppResult SDL_AppIterate(void* appstate) { return SDL_APP_CONTINUE; }

SDL_AppResult SDL_AppEvent(void* appstate, SDL_Event *event) {
  return SDL_APP_SUCCESS;
}

void SDL_AppQuit(void* appstate, SDL_AppResult result) {}
```

Il ne faut pas hésiter à utiliser l'intégration de clangd pour obtenir des informations sur ces fonctions. 

- ``SDL_MAIN_USE_CALLBACKS`` active le système de callbacks.
- ``SDL_AppInit`` est la fonction qui sert à initialiser notre programme. Le double pointeur qu'elle attend. ce ``appstate`` peut être une ressource quelconque disponible tout au long de l'application.
- ``SDL_AppIterate`` est appelé en boucle tout au long de l'application. C'est dans cette fonction qu'on pourra effectuer le rendu.
- ``SDL_AppEvent`` est appelé a chaque fois qu'un évènement a lieu, c'est-à-dire, si la souris est déplacée, si une touche est pressée, si le chargeur est branché sur l'ordinateur, si l'utilisateur demande qu'une fenêtre soit fermée, etc.

- ``SDL_AppQuit`` est appelé à la fin de la vie de notre programme, c'est a cette endroit qu'on libera les ressources de notre application.

# Abstraire sous programme et un première fenêtre.

Puisque la SDL est en C, écrivons un objet en "bon c++" qui va représenter notre application.
```cpp
#include <SDL3/SDL_init.h>
#include <memory>
#define SDL_MAIN_USE_CALLBACKS
#include "SDL3/SDL_main.h"

class App {
public:
  App() {}
  SDL_AppResult on_iterate() { return SDL_APP_CONTINUE; };
  SDL_AppResult on_event(SDL_Event &event) { return SDL_APP_CONTINUE; }
};

SDL_AppResult SDL_AppInit(void **appstate, int argc, char **argv) {
  auto app = std::make_unique<App>();
  *appstate = app.release();
  return SDL_APP_CONTINUE;
}

SDL_AppResult SDL_AppIterate(void *appstate) {
  // on doit convertir le convertir le type du pointeur
  auto app = static_cast<App *>(appstate);
  return app->on_iterate();
  return SDL_APP_CONTINUE;
}

SDL_AppResult SDL_AppEvent(void *appstate, SDL_Event *event) {
  auto app = static_cast<App *>(appstate);
  return app->on_event(*event);
}

void SDL_AppQuit(void *appstate, SDL_AppResult result) {
  // reconstruire un unique pointer va invoker son destructeur a la fin de la portée, et donc libérer la mémoire allouee pour notre application
  std::unique_ptr<App> app{static_cast<App *>(appstate)};
}
```
Ce programme fait certes rien, mais il ne fait rien jolimenent, on peut maintenant stocker tous les objets dont on a besoin dans notre objet application.

# Ouvrir une fenêtre et attendre que l'utilisateur la ferme.

La [documentation de la SDL](https://wiki.libsdl.org/SDL3/SDL_CreateWindow) nous montre les étapes que l'on doit faire pour ouvrir une fenêtre, il faut:
- initialiser le module vidéo et event de la SDL
- tenter d'ouvrir la fenetre
- gérer l'erreur si l'operation échoue
- ne pas oublier de fermer la fenêtre à la fin du programme.

On commence donc par initiliser la SDL : 
```cpp

SDL_AppResult SDL_AppInit(void **appstate, int argc, char **argv) {
  if (!SDL_Init(SDL_INIT_EVENTS | SDL_INIT_VIDEO))
    return SDL_APP_FAILURE;

  auto app = std::make_unique<App>();
  *appstate = app.release();
  return SDL_APP_CONTINUE;
}
```

En C, ces étapes sont un peu verbeuse, on peut automatiser certaines étapes (comme la liberation de la mémoire, ce qui nous évitera de l'oublier) avec le type [unique_pointer](https://en.cppreference.com/cpp/memory/unique_ptr) qui peut être utilisé avec la bonne fonction de destruction. On peut ajouter au début de notre programme, l'alias du type suivant (que l'on placera dans son propre namespace)

```cpp
#include <SDL3/SDL_video.h>
namespace sdl {
using Window = std::unique_ptr<SDL_Window, decltype(&SDL_DestroyWindow)>;
} // namespace sdl
```

On peut donc ajouter une fenêtre à notre application.
```cpp
class App {
  sdl::Window window;

public:
  App() : window(nullptr, &SDL_DestroyWindow) {
    this->window.reset(SDL_CreateWindow("tetris", 600, 400, 0));
    if (this->window ==
        nullptr) // SDL_CreateWindow renvoit nullptr en cas d'echec
      throw std::runtime_error(
          "unable to open a Window:"s +
          SDL_GetError()); // on fait une jolie erreur bien lisible
  }
  SDL_AppResult on_iterate() { return SDL_APP_CONTINUE; };
  SDL_AppResult on_event(SDL_Event &event) { return SDL_APP_CONTINUE; }
};
```
le ``s`` derrière la chaine de caractère permet créer directement un ``std::string`` qui supporte la concaténation, ce qui n'est pas le cas des ``char*``. Il faut ajouter la ligne suivante pour l'activer cependant: 
```cpp
using namespace std::string_literals;
```

On peut lever une exception dans certains cas, il serait donc préférable de la gérer dans le cycle de vie de notre application, on modifie donc la fonction d'initialisation: 
```cpp
SDL_AppResult SDL_AppInit(void **appstate, int argc, char **argv) {
  try {
    if (!SDL_Init(SDL_INIT_EVENTS | SDL_INIT_VIDEO))
      throw std::runtime_error(std::string{"failed to init SDL: "} +
                               SDL_GetError());

    auto app = std::make_unique<App>();
    *appstate = app.release();
  } catch (const std::exception &error) {
    std::println(std::cerr, "an error occured while starting: {}", error.what());
    return SDL_APP_FAILURE;
  }
  return SDL_APP_CONTINUE;
}
```
Si vous ne les avez pas déjà inclus, la gestion d'erreur utilise ces entêtes:
```cpp
#include <iostream>
#include <print>
#include <stdexcept>
```

Il ne reste plus qu'a termine l'application lorsque l'utilisateur demande a fermer la fenêtre. Pour cela on écoute les évènements dans ``AppEvent``

```cpp
SDL_AppResult on_event(SDL_Event &event) {
    switch (event.type) { // on utilise un switch car on pourra être amene a
                          // ecouter d'autres events.
    case SDL_EventType::SDL_EVENT_WINDOW_CLOSE_REQUESTED:
      return SDL_APP_SUCCESS;
    default:
      break;
    }
    return SDL_APP_CONTINUE;
  }
};
```

# Le code entier

```cpp
#include <SDL3/SDL_init.h>
#include <memory>
#define SDL_MAIN_USE_CALLBACKS
#include <SDL3/SDL_main.h>
#include <SDL3/SDL_video.h>

#include <iostream>
#include <print>
#include <stdexcept>
using namespace std::string_literals;

namespace sdl {
using Window = std::unique_ptr<SDL_Window, decltype(&SDL_DestroyWindow)>;
} // namespace sdl

class App {
  sdl::Window window;

public:
  App() : window(nullptr, &SDL_DestroyWindow) {
    this->window.reset(SDL_CreateWindow("tetris", 600, 400, 0));
    if (this->window ==
        nullptr) // SDL_CreateWindow renvoit nullptr en cas d'echec
      throw std::runtime_error(
          "unable to open a Window:"s +
          SDL_GetError()); // on fait une jolie erreur bien lisible
  }
  SDL_AppResult on_iterate() { return SDL_APP_CONTINUE; };
  SDL_AppResult on_event(SDL_Event &event) {
    switch (event.type) { // on utilise un switch car on pourra être amene a
                          // ecouter d'autres events.
    case SDL_EventType::SDL_EVENT_WINDOW_CLOSE_REQUESTED:
      return SDL_APP_SUCCESS;
    default:
      break;
    }
    return SDL_APP_CONTINUE;
  }
};

SDL_AppResult SDL_AppInit(void **appstate, int argc, char **argv) {
  try {
    if (!SDL_Init(SDL_INIT_EVENTS | SDL_INIT_VIDEO))
      throw std::runtime_error(std::string{"failed to init SDL: "} +
                               SDL_GetError());

    auto app = std::make_unique<App>();
    *appstate = app.release();

  } catch (const std::exception &error) {
    std::println(std::cerr, "an error occured while starting: {}", error.what());
    return SDL_APP_FAILURE;
  }
  return SDL_APP_CONTINUE;
}

SDL_AppResult SDL_AppIterate(void *appstate) {
  // on doit convertir le convertir le type du pointeur
  auto app = static_cast<App *>(appstate);
  return app->on_iterate();
  return SDL_APP_CONTINUE;
}

SDL_AppResult SDL_AppEvent(void *appstate, SDL_Event *event) {
  auto app = static_cast<App *>(appstate);
  return app->on_event(*event);
}

void SDL_AppQuit(void *appstate, SDL_AppResult result) {
  // reconstruire un unique pointer va invoker son destructeur a la fin de la
  // portée, et donc libérer la mémoire allouee pour notre application
  std::unique_ptr<App> app{static_cast<App *>(appstate)};
}
```