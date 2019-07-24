---
title: "Part 10 - Sauvegarder et recharger"
date: 2019-03-30T09:34:04-07:00
draft: false
---

Sauvegarder et recharger est une partie essentielle de la majorité des Roguelike
mais cela peut-être pénible si vous l'abordez tardivement. À la fin de ce
chapitre votre jeu pourra sauvegarder et charger un fichier du disque et vous
pourrez facilement étendre à plusieurs fichiers si vous le souhaitez. Mais avant
de s'y plonger concentrons nous sur la boucle principale du jeu.

Le fichier `engine.py` fait environ 250 lignes pour l'instant. Ce n'est pas si
impressionnant en soit (j'ai travaillé sur des fichiers de 10,000 lignes) mais
soyons honnêtes, une grande partie de ce qui y figure n'a rien à y faire.
Qui plus est, la fonction `main` pourrait être découpée en initialisation et
la boucle principale et cela nous le chargement et la sauvegarde bien plus
facile.

Le premier pas est de déplacer autant que possible l'initialisation des
variables en dehors de la boucle principale. Nous allons créer quelques
fonctions qui vont créer le joueur, créer la carte et charger des variables
comme `map_width` et `fov_algorithm`. Nous allons créer un nouveau dossier
appelé `loader_functions` et y ajouter un fichier `initialize_new_game.py`.

Notre première fonction dans ce nouveau fichier va renvoyer les variables qui
sont en haut de la fonction `main`. Cela va ressembler à ça :

{{< highlight py3 >}}
import tcod as libtcod


def get_constants():
    window_title = 'Roguelike Tutorial Revised'

    screen_width = 80
    screen_height = 50

    bar_width = 20
    panel_height = 7
    panel_y = screen_height - panel_height

    message_x = bar_width + 2
    message_width = screen_width - bar_width - 2
    message_height = panel_height - 1

    map_width = 80
    map_height = 43

    room_max_size = 10
    room_min_size = 6
    max_rooms = 30

    fov_algorithm = 0
    fov_light_walls = True
    fov_radius = 10

    max_monsters_per_room = 3
    max_items_per_room = 2

    colors = {
        'dark_wall': libtcod.Color(0, 0, 100),
        'dark_ground': libtcod.Color(50, 50, 150),
        'light_wall': libtcod.Color(130, 110, 50),
        'light_ground': libtcod.Color(200, 180, 50)
    }

    constants = {
        'window_title': window_title,
        'screen_width': screen_width,
        'screen_height': screen_height,
        'bar_width': bar_width,
        'panel_height': panel_height,
        'panel_y': panel_y,
        'message_x': message_x,
        'message_width': message_width,
        'message_height': message_height,
        'map_width': map_width,
        'map_height': map_height,
        'room_max_size': room_max_size,
        'room_min_size': room_min_size,
        'max_rooms': max_rooms,
        'fov_algorithm': fov_algorithm,
        'fov_light_walls': fov_light_walls,
        'fov_radius': fov_radius,
        'max_monsters_per_room': max_monsters_per_room,
        'max_items_per_room': max_items_per_room,
        'colors': colors
    }

    return constants
{{</ highlight >}}

*\*Remarque : `window_title` est nouveau. Avant on se contentait de passer le
titre de la fenêtre comme une chaîne mais on pourrait aussi bien la définir
comme un élément de ce dictionnaire.*

Pourquoi ce nom "constants" ? Python ne dispose pas d'un moyen de déclarer une
variable qui ne change pas (Java a "final", C\# a "readonly" etc.) aussi je
voulais un nom qui signfie le fait que ces variables de *devraient* pas changer.
Le programme *pourrait, en théorie* les modifier durant le cours du jeu mais
pour l'instant, on ne le fera pas. Vous pouvez utiliser un autre nom si vous
préférez comme "game\_variables" ou quelque chose d'autre.

Mettons cette fonction en action dans notre fichier `engine.py`. Importons
d'abord la fonction.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from input_handlers import handle_keys, handle_mouse
+from loader_functions.initialize_new_game import get_constants
from map_objects.game_map import GameMap
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from input_handlers import handle_keys, handle_mouse
<span class="new-text">from loader_functions.initialize_new_game import get_constants</span>
from map_objects.game_map import GameMap
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Alors, appelons la dans la première ligne de `main`. Retirons aussi les
variables similaires.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def main():
+   constants = get_constants()

-   screen_width = 80
-   screen_height = 50

-   bar_width = 20
-   panel_height = 7
-   panel_y = screen_height - panel_height

-   message_x = bar_width + 2
-   message_width = screen_width - bar_width - 2
-   message_height = panel_height - 1

-   map_width = 80
-   map_height = 43

-   room_max_size = 10
-   room_min_size = 6
-   max_rooms = 30

-   fov_algorithm = 0
-   fov_light_walls = True
-   fov_radius = 10

-   max_monsters_per_room = 3
-   max_items_per_room = 2

-   colors = {
-       'dark_wall': (0, 0, 100),
-       'dark_ground': (50, 50, 150),
-       'light_wall': (130, 110, 50),
-       'light_ground': (200, 180, 50)
-   }
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def main():
    <span class="new-text">constants = get_constants()</span>

    <span class="crossed-out-text">screen_width = 80</span>
    <span class="crossed-out-text">screen_height = 50</span>

    <span class="crossed-out-text">bar_width = 20</span>
    <span class="crossed-out-text">panel_height = 7</span>
    <span class="crossed-out-text">panel_y = screen_height - panel_height</span>

    <span class="crossed-out-text">message_x = bar_width + 2</span>
    <span class="crossed-out-text">message_width = screen_width - bar_width - 2</span>
    <span class="crossed-out-text">message_height = panel_height - 1</span>

    <span class="crossed-out-text">map_width = 80</span>
    <span class="crossed-out-text">map_height = 43</span>

    <span class="crossed-out-text">room_max_size = 10</span>
    <span class="crossed-out-text">room_min_size = 6</span>
    <span class="crossed-out-text">max_rooms = 30</span>

    <span class="crossed-out-text">fov_algorithm = 0</span>
    <span class="crossed-out-text">fov_light_walls = True</span>
    <span class="crossed-out-text">fov_radius = 10</span>

    <span class="crossed-out-text">max_monsters_per_room = 3</span>
    <span class="crossed-out-text">max_items_per_room = 2</span>

    <span class="crossed-out-text">colors = {</span>
        <span class="crossed-out-text">'dark_wall': (0, 0, 100),</span>
        <span class="crossed-out-text">'dark_ground': (50, 50, 150),</span>
        <span class="crossed-out-text">'light_wall': (130, 110, 50),</span>
        <span class="crossed-out-text">'light_ground': (200, 180, 50)</span>
    <span class="crossed-out-text">}</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

D'accord, si vous utilisez un IDE (comme PyCharm) alors il devient probablement
fou en ce moment... De toute évidence on ne peut retirer autant de variables et
espérer que tout fonctionne bien. Nous devons modifier toutes les utilisations
de ces "constantes" et les remplacer à un appel au dictionnaire `constants`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    libtcod.console_set_custom_font('arial10x10.png', libtcod.FONT_TYPE_GREYSCALE | libtcod.FONT_LAYOUT_TCOD)

-   libtcod.console_init_root(screen_width, screen_height, 'libtcod tutorial revised', False)
+   libtcod.console_init_root(constants['screen_width'], constants['screen_height'], constants['window_title'], False)

-   con = libtcod.console_new(screen_width, screen_height)
-   panel = libtcod.console_new(screen_width, panel_height)
+   con = libtcod.console_new(constants['screen_width'], constants['screen_height'])
+   panel = libtcod.console_new(constants['screen_width'], constants['panel_height'])

-   game_map = GameMap(map_width, map_height)
-   game_map.make_map(max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities,
-                     max_monsters_per_room, max_items_per_room)
+   game_map = GameMap(constants['map_width'], constants['map_height'])
+   game_map.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],
+                     constants['map_width'], constants['map_height'], player, entities,
+                     constants['max_monsters_per_room'], constants['max_items_per_room'])

    fov_recompute = True

    fov_map = initialize_fov(game_map)

-   message_log = MessageLog(message_x, message_width, message_height)
+   message_log = MessageLog(constants['message_x'], constants['message_width'], constants['message_height'])

    key = libtcod.Key()
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    libtcod.console_set_custom_font('arial10x10.png', libtcod.FONT_TYPE_GREYSCALE | libtcod.FONT_LAYOUT_TCOD)

    <span class="crossed-out-text">libtcod.console_init_root(screen_width, screen_height, 'libtcod tutorial revised', False)</span>
    <span class="new-text">libtcod.console_init_root(constants['screen_width'], constants['screen_height'], constants['window_title'], False)</span>

    <span class="crossed-out-text">con = libtcod.console_new(screen_width, screen_height)</span>
    <span class="crossed-out-text">panel = libtcod.console_new(screen_width, panel_height)</span>
    <span class="new-text">con = libtcod.console_new(constants['screen_width'], constants['screen_height'])
    panel = libtcod.console_new(constants['screen_width'], constants['panel_height'])</span>

    <span class="crossed-out-text">game_map = GameMap(map_width, map_height)</span>
    <span class="crossed-out-text">game_map.make_map(max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities,</span>
                      <span class="crossed-out-text">max_monsters_per_room, max_items_per_room)</span>
    <span class="new-text">game_map = GameMap(constants['map_width'], constants['map_height'])
    game_map.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],
                      constants['map_width'], constants['map_height'], player, entities,
                      constants['max_monsters_per_room'], constants['max_items_per_room'])</span>

    fov_recompute = True

    fov_map = initialize_fov(game_map)

    <span class="crossed-out-text">message_log = MessageLog(message_x, message_width, message_height)</span>
    <span class="new-text">message_log = MessageLog(constants['message_x'], constants['message_width'], constants['message_height'])</span>

    key = libtcod.Key()</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        if fov_recompute:
-           recompute_fov(fov_map, player.x, player.y, fov_radius, fov_light_walls, fov_algorithm)
+           recompute_fov(fov_map, player.x, player.y, constants['fov_radius'], constants['fov_light_walls'],
+                         constants['fov_algorithm'])

-       render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width,
-                  screen_height, bar_width, panel_height, panel_y, mouse, colors, game_state)
+       render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log,
+                  constants['screen_width'], constants['screen_height'], constants['bar_width'],
+                  constants['panel_height'], constants['panel_y'], mouse, constants['colors'], game_state)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        if fov_recompute:
            <span class="crossed-out-text">recompute_fov(fov_map, player.x, player.y, fov_radius, fov_light_walls, fov_algorithm)</span>
            <span class="new-text">recompute_fov(fov_map, player.x, player.y, constants['fov_radius'], constants['fov_light_walls'],
                          constants['fov_algorithm'])</span>

        <span class="crossed-out-text">render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width,</span>
                   <span class="crossed-out-text">screen_height, bar_width, panel_height, panel_y, mouse, colors, game_state)</span>
        <span class="new-text">render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log,
                   constants['screen_width'], constants['screen_height'], constants['bar_width'],
                   constants['panel_height'], constants['panel_y'], mouse, constants['colors'], game_state)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

*\*Note : pourquoi utilisons-nous un crochet plutôt qu'une méthode `get()` ?
Dans la majorité des autres parties, nous avons utilisé la notation 'get' mais
ici je trouve que ça fait plus de sens d'employer un crochet.
Les crochets vont faire planter le jeu si la variable n'est pas trouvée et,
dans ce cas, c'est certainement ce que nous voulons. Le jeu ne peut tourner
sans ces variables aussi il n'y aucune raison de continuer le programme sans
elle.*

Cela représentant beaucoup de changement mais nous avons réussi à retirer les
constantes de la boucle principale \! Remarquez que vous pourriez
*considérablement* raccourcir ces définitions de fonctions en leur passant
directement le dictionnaire de constantes plutôt qu'en donnant seulement celles
dont la fonction a besoin. Cela ne fait pas une grande différence et c'est
surtout une question de gout. Je vais laisser les choses ainsi dans ce tutoriel
parce que modifier les fonctions maintenant serait une tâche considérable.

Et maintenant ? Une autre chose à faire est de modifier l'initialisation du
joueur, de la liste des entités et de la carte de jeu dans un fonction séparée.
Mettez la fonction suivante dans `initialize_new_game.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def get_constants():
    ...

+def get_game_variables(constants):
+   fighter_component = Fighter(hp=30, defense=2, power=5)
+   inventory_component = Inventory(26)
+   player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR,
+                   fighter=fighter_component, inventory=inventory_component)
+   entities = [player]
+
+   game_map = GameMap(constants['map_width'], constants['map_height'])
+   game_map.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],
+                     constants['map_width'], constants['map_height'], player, entities,
+                     constants['max_monsters_per_room'], constants['max_items_per_room'])
+
+   message_log = MessageLog(constants['message_x'], constants['message_width'], constants['message_height'])
+
+   game_state = GameStates.PLAYERS_TURN
+
+   return player, entities, game_map, message_log, game_state
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def get_constants():
    ...

<span class="new-text">def get_game_variables(constants):
    fighter_component = Fighter(hp=30, defense=2, power=5)
    inventory_component = Inventory(26)
    player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR,
                    fighter=fighter_component, inventory=inventory_component)
    entities = [player]

    game_map = GameMap(constants['map_width'], constants['map_height'])
    game_map.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],
                      constants['map_width'], constants['map_height'], player, entities,
                      constants['max_monsters_per_room'], constants['max_items_per_room'])

    message_log = MessageLog(constants['message_x'], constants['message_width'], constants['message_height'])

    game_state = GameStates.PLAYERS_TURN

    return player, entities, game_map, message_log, game_state</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous devons maintenant ajouter quelques imports dans `initialize_new_game.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

+from components.fighter import Fighter
+from components.inventory import Inventory
+
+from entity import Entity
+
+from game_messages import MessageLog
+
+from game_states import GameStates
+
+from map_objects.game_map import GameMap
+
+from render_functions import RenderOrder


def get_constants():
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

<span class="new-text">from components.fighter import Fighter
from components.inventory import Inventory

from entity import Entity

from game_messages import MessageLog

from game_states import GameStates

from map_objects.game_map import GameMap

from render_functions import RenderOrder</span>


def get_constants():
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Rien n'a changé dans la manière d'initialiser ces variables. Tout ce qu'on a
fait est de les ajouter à une fonction, qu'on appellera une fois dans la
boucle principale. Faisons le. Commencez par importer la fonction
`get_game_variables` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from input_handlers import handle_keys, handle_mouse
-from loader_functions.initialize_new_game import get_constants
+from loader_functions.initialize_new_game import get_constants, get_game_variables
from map_objects.game_map import GameMap
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from input_handlers import handle_keys, handle_mouse
from loader_functions.initialize_new_game import get_constants, <span class="new-text">get_game_variables</span>
from map_objects.game_map import GameMap
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Ensuite modifiez ainsi la fonction `main` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
-   fighter_component = Fighter(hp=30, defense=2, power=5)
-   inventory_component = Inventory(26)
-   player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR,
-                   fighter=fighter_component, inventory=inventory_component)
-   entities = [player]

    libtcod.console_set_custom_font('arial10x10.png', libtcod.FONT_TYPE_GREYSCALE | libtcod.FONT_LAYOUT_TCOD)

    libtcod.console_init_root(constants['screen_width'], constants['screen_height'], constants['window_title'], False)

    con = libtcod.console_new(constants['screen_width'], constants['screen_height'])
    panel = libtcod.console_new(constants['screen_width'], constants['panel_height'])

-   game_map = GameMap(constants['map_width'], constants['map_height'])
-   game_map.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],
-                     constants['map_width'], constants['map_height'], player, entities,
-                     constants['max_monsters_per_room'], constants['max_items_per_room'])

+   player, entities, game_map, message_log, game_state = get_game_variables(constants)

    fov_recompute = True

    fov_map = initialize_fov(game_map)

-   message_log = MessageLog(constants['message_x'], constants['message_width'], constants['message_height'])

    key = libtcod.Key()
    mouse = libtcod.Mouse()

-   game_state = GameStates.PLAYERS_TURN
    previous_game_state = game_state

    targeting_item = None
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    <span class="crossed-out-text">fighter_component = Fighter(hp=30, defense=2, power=5)</span>
    <span class="crossed-out-text">inventory_component = Inventory(26)</span>
    <span class="crossed-out-text">player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR,</span>
                    <span class="crossed-out-text">fighter=fighter_component, inventory=inventory_component)</span>
    <span class="crossed-out-text">entities = [player]</span>

    libtcod.console_set_custom_font('arial10x10.png', libtcod.FONT_TYPE_GREYSCALE | libtcod.FONT_LAYOUT_TCOD)

    libtcod.console_init_root(constants['screen_width'], constants['screen_height'], constants['window_title'], False)

    con = libtcod.console_new(constants['screen_width'], constants['screen_height'])
    panel = libtcod.console_new(constants['screen_width'], constants['panel_height'])

    <span class="crossed-out-text">game_map = GameMap(constants['map_width'], constants['map_height'])</span>
    <span class="crossed-out-text">game_map.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],</span>
                      <span class="crossed-out-text">constants['map_width'], constants['map_height'], player, entities,</span>
                      <span class="crossed-out-text">constants['max_monsters_per_room'], constants['max_items_per_room'])</span>

    <span class="new-text">player, entities, game_map, message_log, game_state = get_game_variables(constants)</span>

    fov_recompute = True

    fov_map = initialize_fov(game_map)

    <span class="crossed-out-text">message_log = MessageLog(constants['message_x'], constants['message_width'], constants['message_height'])</span>

    key = libtcod.Key()
    mouse = libtcod.Mouse()

    <span class="crossed-out-text">game_state = GameStates.PLAYERS_TURN</span>
    previous_game_state = game_state

    targeting_item = None
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Une conséquence intéressante du retrait de ces lignes et qu'on n'a plus besoin
d'autant d'imports. Modifiez la section d'imports en haut de `engine.py` pour
la rendre ainsi :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

-from components.fighter import Fighter
-from components.inventory import Inventory
from death_functions import kill_monster, kill_player
-from entity import Entity, get_blocking_entities_at_location
+from entity get_blocking_entities_at_location
from fov_functions import initialize_fov, recompute_fov
-from game_messages import Message, MessageLog
+from game_messages import Message
from game_states import GameStates
from input_handlers import handle_keys, handle_mouse
from loader_functions.initialize_new_game import get_constants, get_game_variables
-from map_objects.game_map import GameMap
-from render_functions import clear_all, render_all, RenderOrder
+from render_functions import clear_all, render_all
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

<span class="crossed-out-text">from components.fighter import Fighter</span>
<span class="crossed-out-text">from components.inventory import Inventory</span>
from death_functions import kill_monster, kill_player
from entity import <span class="crossed-out-text">Entity</span>, get_blocking_entities_at_location
from fov_functions import initialize_fov, recompute_fov
from game_messages import Message, <span class="crossed-out-text">MessageLog</span>
from game_states import GameStates
from input_handlers import handle_keys, handle_mouse
from loader_functions.initialize_new_game import get_constants, get_game_variables
<span class="crossed-out-text">from map_objects.game_map import GameMap</span>
from render_functions import clear_all, render_all<span class="crossed-out-text">, RenderOrder</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Il est temps d'envisager la sauvegarde et le chargement de notre jeu. Pour ce
faire nous devons sauvegarder certaines (pas forcement toutes) les données
vers un espace extérieur. Dans la majorité des applications cela serait
une base de données SQL ou NoSQL mais c'est sûrement trop pour notre petit
projet. Nous utiliserons pluôt une fichier de données.

Qu'avons-nous besoin de sauvegarder ? Les éléments clé sont la liste des entités
(contenant le joueur), la boucle de jeu, le journal de messages et l'état du
jeu. Ce sont les variables renvoyées par la fonction d'initialisation ainsi
nous pourrons démarrer une partie ou en charger une ancienne en remplaçant
simplement la fonction. Plus d'information à ce propos un peu plus tard.

Malheureusement un simple JSON n'est pas assez pour sauvegarder et charger nos
données. Nos objets sont trop complexes pour les enregistrer dans un fichier
JSON. Il y a plusieurs solutions pour résoudre ce problème. La première serait
d'écrire nous même des sérialiseurs pour nos classes et nos objets, ce n'est pas
une mauvaise idée. Mais afin de conserver la simplicité de ce tutoriel nous
utiliserons une librairie. Pour être précis : `shelve` (ranger). Cette librairie
permet de sauvegarder et charger des objets Pythons complexes sans devoir écrire
des serialiseurs particuliers.

Dans les versions récentes,`shelve` est déjà présent.
Si vous utilisez une version ancienne de Python, il faut installer `shelve`
(pip est la meillere manière). Ensuite créez un nouveau fichier dans
`loader_functions` appelé `data_loaders.py`. Nous allons commencer par ecrire
une fonction de sauvegarde.

{{< highlight py3 >}}
import shelve


def save_game(player, entities, game_map, message_log, game_state):
    with shelve.open('savegame.dat', 'n') as data_file:
        data_file['player_index'] = entities.index(player)
        data_file['entities'] = entities
        data_file['game_map'] = game_map
        data_file['message_log'] = message_log
        data_file['game_state'] = game_state
{{</ highlight >}}

Avec `shelve` nous encodons les données dans un dictionnaire qui sera enregistré
plus tard dans le fichier. Remarquez qu'on ne sauvegarde pas le `player`
parce que le joueur est déjà un élément de la liste `entities`. Nous n'avons
besoin que de l'indice dans la liste pour en extraire le joueur plus tard.

Et c'est tout ce qui nous faut pour sauvegarder le jeu \! Sans le module
`shelve` cela nous aurait demandé beaucoup plus d'efforts pour sauvegarder le
jeu. Heureusement, cela simplifie aussi le chargement du jeu. Implémentons
le maintenant. Dans le même fichier (`data_loaders.py`), créons une nouvelle
fonction appelée `load_game`. Vous devrez importer `GameMap` pour faire
fonctionner ces changements.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
+import os

import shelve


def save_game(player, entities, game_map, message_log, game_state):
    ...

+def load_game():
+   if not os.path.isfile('savegame.dat'):
+       raise FileNotFoundError
+
+   with shelve.open('savegame.dat', 'r') as data_file:
+       player_index = data_file['player_index']
+       entities = data_file['entities']
+       game_map = data_file['game_map']
+       message_log = data_file['message_log']
+       game_state = data_file['game_state']
+
+   player = entities[player_index]
+
+   return player, entities, game_map, message_log, game_state
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="new-text">import os</span>

import shelve


def save_game(player, entities, game_map, message_log, game_state):
    ...

<span class="new-text">def load_game():
    if not os.path.isfile('savegame.dat'):
        raise FileNotFoundError

    with shelve.open('savegame.dat', 'r') as data_file:
        player_index = data_file['player_index']
        entities = data_file['entities']
        game_map = data_file['game_map']
        message_log = data_file['message_log']
        game_state = data_file['game_state']

    player = entities[player_index]

    return player, entities, game_map, message_log, game_state</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Ce n'est que le contraire de la fonction d'enregistrement. Nous extrayons
les données du fichier et renvoyons les variables nécessaires au moteur du
jeu.

Les fonctions pour enregistrer et charger le jeu sont faites et nous avons
maintenant besoin d'une manière de les employer. Avant de ce faire, c'est
certainement une bonne idée que de penser à la manière dont notre jeu débute
une partie. Pour l'instant, le jeu démarre directement, lançant le joueur
dans l'action.  Ce n'est pas la manière traditionnelle de débuter un jeu.
Presque tous les jeux proposent un écran de démarrage qui permet au joueur
de débuter une partie, d'en charger une existante, de quitter ou modifier
les réglages. Implémentons quelque chose de similaire pour le notre. Nous
allons permettre au joueur de débuter une partie, de charger une sauvegarde ou
de sortir.

Nous aurons besoin d'une nouvelle fonction pour afficher notre menu principal.
Ouvrez `menus.py` et ajouter les fonctions suivantes :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def inventory_menu(con, header, inventory, inventory_width, screen_width, screen_height):
    ...


+def main_menu(con, background_image, screen_width, screen_height):
+   libtcod.image_blit_2x(background_image, 0, 0, 0)
+
+   libtcod.console_set_default_foreground(0, libtcod.light_yellow)
+   libtcod.console_print_ex(0, int(screen_width / 2), int(screen_height / 2) - 4, libtcod.BKGND_NONE, libtcod.CENTER,
+                            'TOMBS OF THE ANCIENT KINGS')
+   libtcod.console_print_ex(0, int(screen_width / 2), int(screen_height - 2), libtcod.BKGND_NONE, libtcod.CENTER,
+                            'By (Your name here)')
+
+   menu(con, '', ['Play a new game', 'Continue last game', 'Quit'], 24, screen_width, screen_height)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def inventory_menu(con, header, inventory, inventory_width, screen_width, screen_height):
    ...


<span class="new-text">def main_menu(con, background_image, screen_width, screen_height):
    libtcod.image_blit_2x(background_image, 0, 0, 0)

    libtcod.console_set_default_foreground(0, libtcod.light_yellow)
    libtcod.console_print_ex(0, int(screen_width / 2), int(screen_height / 2) - 4, libtcod.BKGND_NONE, libtcod.CENTER,
                             'TOMBS OF THE ANCIENT KINGS')
    libtcod.console_print_ex(0, int(screen_width / 2), int(screen_height - 2), libtcod.BKGND_NONE, libtcod.CENTER,
                             'By (Your name here)')

    menu(con, '', ['Play a new game', 'Continue last game', 'Quit'], 24, screen_width, screen_height)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Notre fonction principale fonctionne en supposant qu'on entre directement dans
le jeu. Il serait plus adapté que la fonction principale lance le menu principal
et, si le joueur choisit de débuter une nouvelle partie ou d'en continuer une
ancienne, le jeu principal débute. Nous pouvons déplacer la logique du jeu
principal dans une fonction séparée qui s'appelera `play_game`. Cette fonction
sera dans notre fichier `engine.py` (ce n'est pas indispensable mais ça n'a
aucune sens de la placer ailleurs pour l'instant).

*\*Remarque: Je ne vais présenter de coloration syntaxique pour illustrer les
changement ici, il y en aurait beaucoup trop.*

{{< highlight py3 >}}
def play_game(player, entities, game_map, message_log, game_state, con, panel, constants):
    fov_recompute = True

    fov_map = initialize_fov(game_map)

    key = libtcod.Key()
    mouse = libtcod.Mouse()

    game_state = GameStates.PLAYERS_TURN
    previous_game_state = game_state

    targeting_item = None

    while not libtcod.console_is_window_closed():
        libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS | libtcod.EVENT_MOUSE, key, mouse)

        if fov_recompute:
            recompute_fov(fov_map, player.x, player.y, constants['fov_radius'], constants['fov_light_walls'],
                          constants['fov_algorithm'])

        render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log,
                   constants['screen_width'], constants['screen_height'], constants['bar_width'],
                   constants['panel_height'], constants['panel_y'], mouse, constants['colors'], game_state)

        fov_recompute = False

        libtcod.console_flush()

        clear_all(con, entities)

        action = handle_keys(key, game_state)
        mouse_action = handle_mouse(mouse)

        move = action.get('move')
        pickup = action.get('pickup')
        show_inventory = action.get('show_inventory')
        drop_inventory = action.get('drop_inventory')
        inventory_index = action.get('inventory_index')
        exit = action.get('exit')
        fullscreen = action.get('fullscreen')

        left_click = mouse_action.get('left_click')
        right_click = mouse_action.get('right_click')

        player_turn_results = []

        if move and game_state == GameStates.PLAYERS_TURN:
            dx, dy = move
            destination_x = player.x + dx
            destination_y = player.y + dy

            if not game_map.is_blocked(destination_x, destination_y):
                target = get_blocking_entities_at_location(entities, destination_x, destination_y)

                if target:
                    attack_results = player.fighter.attack(target)
                    player_turn_results.extend(attack_results)
                else:
                    player.move(dx, dy)

                    fov_recompute = True

                game_state = GameStates.ENEMY_TURN

        elif pickup and game_state == GameStates.PLAYERS_TURN:
            for entity in entities:
                if entity.item and entity.x == player.x and entity.y == player.y:
                    pickup_results = player.inventory.add_item(entity)
                    player_turn_results.extend(pickup_results)

                    break
            else:
                message_log.add_message(Message('There is nothing here to pick up.', libtcod.yellow))

        if show_inventory:
            previous_game_state = game_state
            game_state = GameStates.SHOW_INVENTORY

        if drop_inventory:
            previous_game_state = game_state
            game_state = GameStates.DROP_INVENTORY

        if inventory_index is not None and previous_game_state != GameStates.PLAYER_DEAD and inventory_index < len(
                player.inventory.items):
            item = player.inventory.items[inventory_index]

            if game_state == GameStates.SHOW_INVENTORY:
                player_turn_results.extend(player.inventory.use(item, entities=entities, fov_map=fov_map))
            elif game_state == GameStates.DROP_INVENTORY:
                player_turn_results.extend(player.inventory.drop_item(item))

        if game_state == GameStates.TARGETING:
            if left_click:
                target_x, target_y = left_click

                item_use_results = player.inventory.use(targeting_item, entities=entities, fov_map=fov_map,
                                                        target_x=target_x, target_y=target_y)
                player_turn_results.extend(item_use_results)
            elif right_click:
                player_turn_results.append({'targeting_cancelled': True})

        if exit:
            if game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
                game_state = previous_game_state
            elif game_state == GameStates.TARGETING:
                player_turn_results.append({'targeting_cancelled': True})
            else:
                save_game(player, entities, game_map, message_log, game_state)

                return True

        if fullscreen:
            libtcod.console_set_fullscreen(not libtcod.console_is_fullscreen())

        for player_turn_result in player_turn_results:
            message = player_turn_result.get('message')
            dead_entity = player_turn_result.get('dead')
            item_added = player_turn_result.get('item_added')
            item_consumed = player_turn_result.get('consumed')
            item_dropped = player_turn_result.get('item_dropped')
            targeting = player_turn_result.get('targeting')
            targeting_cancelled = player_turn_result.get('targeting_cancelled')

            if message:
                message_log.add_message(message)

            if dead_entity:
                if dead_entity == player:
                    message, game_state = kill_player(dead_entity)
                else:
                    message = kill_monster(dead_entity)

                message_log.add_message(message)

            if item_added:
                entities.remove(item_added)

                game_state = GameStates.ENEMY_TURN

            if item_consumed:
                game_state = GameStates.ENEMY_TURN

            if item_dropped:
                entities.append(item_dropped)

                game_state = GameStates.ENEMY_TURN

            if targeting:
                previous_game_state = GameStates.PLAYERS_TURN
                game_state = GameStates.TARGETING

                targeting_item = targeting

                message_log.add_message(targeting_item.item.targeting_message)

            if targeting_cancelled:
                game_state = previous_game_state

                message_log.add_message(Message('Targeting cancelled'))

        if game_state == GameStates.ENEMY_TURN:
            for entity in entities:
                if entity.ai:
                    enemy_turn_results = entity.ai.take_turn(player, fov_map, game_map, entities)

                    for enemy_turn_result in enemy_turn_results:
                        message = enemy_turn_result.get('message')
                        dead_entity = enemy_turn_result.get('dead')

                        if message:
                            message_log.add_message(message)

                        if dead_entity:
                            if dead_entity == player:
                                message, game_state = kill_player(dead_entity)
                            else:
                                message = kill_monster(dead_entity)

                            message_log.add_message(message)

                            if game_state == GameStates.PLAYER_DEAD:
                                break

                    if game_state == GameStates.PLAYER_DEAD:
                        break
            else:
                game_state = GameStates.PLAYERS_TURN
{{</ highlight >}}

C'est le même code que ce qui était dans notre jeu jusqu'ici mais dans une
fonction. Nous passons toutes les variables à notre fonction principale.
Si le joueur presse Escape durant le jeu, on retourne à la boucle principale,
qui affiche le menu. La grande différence est qu'on appelle `save_game` avant
de quitter le jeu.

Maintenant modifions la boucle principale. Elle va afficher le menu principal
et, selon le choix du joueur, on va débuter une nouvelle partie ;
charger une existante ou quitter le programme.

{{< highlight py3 >}}
def main():
    constants = get_constants()

    libtcod.console_set_custom_font('arial10x10.png', libtcod.FONT_TYPE_GREYSCALE | libtcod.FONT_LAYOUT_TCOD)

    libtcod.console_init_root(constants['screen_width'], constants['screen_height'], constants['window_title'], False)

    con = libtcod.console_new(constants['screen_width'], constants['screen_height'])
    panel = libtcod.console_new(constants['screen_width'], constants['panel_height'])

    player = None
    entities = []
    game_map = None
    message_log = None
    game_state = None

    show_main_menu = True
    show_load_error_message = False

    main_menu_background_image = libtcod.image_load('menu_background.png')

    key = libtcod.Key()
    mouse = libtcod.Mouse()

    while not libtcod.console_is_window_closed():
        libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS | libtcod.EVENT_MOUSE, key, mouse)

        if show_main_menu:
            main_menu(con, main_menu_background_image, constants['screen_width'],
                      constants['screen_height'])

            if show_load_error_message:
                message_box(con, 'No save game to load', 50, constants['screen_width'], constants['screen_height'])

            libtcod.console_flush()

            action = handle_main_menu(key)

            new_game = action.get('new_game')
            load_saved_game = action.get('load_game')
            exit_game = action.get('exit')

            if show_load_error_message and (new_game or load_saved_game or exit_game):
                show_load_error_message = False
            elif new_game:
                player, entities, game_map, message_log, game_state = get_game_variables(constants)
                game_state = GameStates.PLAYERS_TURN

                show_main_menu = False
            elif load_saved_game:
                try:
                    player, entities, game_map, message_log, game_state = load_game()
                    show_main_menu = False
                except FileNotFoundError:
                    show_load_error_message = True
            elif exit_game:
                break

        else:
            libtcod.console_clear(con)
            play_game(player, entities, game_map, message_log, game_state, con, panel, constants)

            show_main_menu = True
{{</ highlight >}}

On charge l'image de fond avec `image_load` pour afficher notre menu principal.
L'image de fond utilisée dans ce tutoriel est [disponible ici](http://roguecentral.org/doryen/files/menu_background1.png). Téléchargez la et ajoutez la à votre dossier de jeu.

En dehors de ça, la majorité devrait vous paraître familière. On affiche
le menu principal avec trois options et on lit les événements clavier pour
déterminer quelle option choisir. Si le joueur débute une nouvelle partie,
on utilise la fonction `get_game_variables` définie plus tôt. Dans tous les cas
on obtient les mêmes variables. En supposant qu'une de ces options soit choisie
on passe les variables à la fonction `play_game` et le jeu continue comme il l'a
fait jusque là.

On n'a pas encore implémenté les fonctions `message_box` ni `handle_main_menu`
aussi faisons le maintenant. Commençons par `message_box` et on l'ajoutera à
`menus.py` à la fin du fichier :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
+def message_box(con, header, width, screen_width, screen_height):
+   menu(con, header, [], width, screen_width, screen_height)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="new-text">def message_box(con, header, width, screen_width, screen_height):
    menu(con, header, [], width, screen_width, screen_height)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Plutôt direct. La boîte de message n'est qu'un menu vide.

Maintenant `handle_main_menu` qui va dans `input_handlers.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def handle_inventory_keys(key):
    ...

+def handle_main_menu(key):
+   key_char = chr(key.c)
+
+   if key_char == 'a':
+       return {'new_game': True}
+   elif key_char == 'b':
+       return {'load_game': True}
+   elif key_char == 'c' or  key.vk == libtcod.KEY_ESCAPE:
+       return {'exit': True}
+
+   return {}


def handle_mouse(mouse):
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def handle_inventory_keys(key):
    ...

<span class="new-text">def handle_main_menu(key):
    key_char = chr(key.c)

    if key_char == 'a':
        return {'new_game': True}
    elif key_char == 'b':
        return {'load_game': True}
    elif key_char == 'c' or  key.vk == libtcod.KEY_ESCAPE:
        return {'exit': True}

    return {}</span>


def handle_mouse(mouse):
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Rien de très compliqué ici : notre menu principal aura trois options. On renvoie
le résultat de l'option choisie. Remarquez que l'option `Quit` peut être obtenue
avec les touches 'c' ou 'Escape'.

Souvenez vous d'importer ces nouvelles fonctions à `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

from death_functions import kill_monster, kill_player
from entity import get_blocking_entities_at_location
from fov_functions import initialize_fov, recompute_fov
from game_messages import Message
from game_states import GameStates
-from input_handlers import handle_keys, handle_mouse
+from input_handlers import handle_keys, handle_mouse, handle_main_menu
from loader_functions.initialize_new_game import get_constants, get_game_variables
+from loader_functions.data_loaders import load_game, save_game
+from menus import main_menu, message_box
from render_functions import clear_all, render_all
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

from death_functions import kill_monster, kill_player
from entity import get_blocking_entities_at_location
from fov_functions import initialize_fov, recompute_fov
from game_messages import Message
from game_states import GameStates
from input_handlers import handle_keys, handle_mouse<span class="new-text">, handle_main_menu</span>
from loader_functions.initialize_new_game import get_constants, get_game_variables
<span class="new-text">from loader_functions.data_loaders import load_game, save_game</span>
<span class="new-text">from menus import main_menu, message_box</span>
from render_functions import clear_all, render_all
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Ce chapitre est clos. Le gameplay n'a pas changé mais l'enregistrement et le
chargement ne sont pas choses aisées. Soyez fier de vous \!

Si vous voulez voir le code actuel entièrement, [cliquez ici](https://github.com/TStand90/roguelike_tutorial_revised/tree/part-10.

[Cliquez ici pour vous rendre à la partie suivante de ce tutoriel.](/tutorials/tcod/part-11)

<script src="/js/codetabs.js"></script>
