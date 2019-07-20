---
title: "Part 8 - Les items et l'inventaire"
date: 2019-03-30T09:33:55-07:00
draft: false
---

Pour l'instant, notre jeu implémente les mouvements, l'exploration des donjons,
le combat et l'AI (okay, on abuse un peu du terme "intelligence" dans
intelligence artificielle mais vous voyez ce que je veux dire). Il est temps
de passer à une étape fondamentale des roguelike : les items \! Pourquoi
explorer les donjons si ce n'est pour récupérer quelques butins, après tout ?

Commençons par un type d'item, les potions de soin en l'occurrence et nous
passerons à l'implémentation de l'inventaire. Dans le chapitre suivant nous
ajouterons d'autres types d'items mais pour l'instant, les potions de soin
feront l'affaire.

Faites les changements suivants à `place_entities` pour placer quelques entités
potions de soin. Elles ne font rien pour l'instant mais elles apparaîtront déjà
sur la carte.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-   def place_entities(self, room, entities, max_monsters_per_room):
+   def place_entities(self, room, entities, max_monsters_per_room, max_items_per_room):
        number_of_monsters = randint(0, max_monsters_per_room)
+       number_of_items = randint(0, max_items_per_room)

        for i in range(number_of_monsters):
            ...

+       for i in range(number_of_items):
+           x = randint(room.x1 + 1, room.x2 - 1)
+           y = randint(room.y1 + 1, room.y2 - 1)
+
+           if not any([entity for entity in entities if entity.x == x and entity.y == y]):
+               item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM)
+
+               entities.append(item)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    def place_entities(self, room, entities, max_monsters_per_room<span class="new-text">, max_items_per_room</span>):
        number_of_monsters = randint(0, max_monsters_per_room)
        <span class="new-text">number_of_items = randint(0, max_items_per_room)</span>

        for i in range(number_of_monsters):
            ...

        <span class="new-text">for i in range(number_of_items):
            x = randint(room.x1 + 1, room.x2 - 1)
            y = randint(room.y1 + 1, room.y2 - 1)

            if not any([entity for entity in entities if entity.x == x and entity.y == y]):
                item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM)

                entities.append(item)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Mettez à jour l'appel à `place_entities` dans `make_map`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-               self.place_entities(new_room, entities, max_monsters_per_room)
+               self.place_entities(new_room, entities, max_monsters_per_room, max_items_per_room)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                self.place_entities(new_room, entities, max_monsters_per_room<span class="new-text">, max_items_per_room</span>)</pre>
{{</ original-tab >}}
{{</ codetab >}}

Mettez la définition de `make_map` à jour pour inclure la nouvelle variable
`max_items_per_room`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities,
-                max_monsters_per_room):
+                max_monsters_per_room, max_items_per_room):
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities,
                 max_monsters_per_room<span class="new-text">, max_items_per_room</span>):</pre>
{{</ original-tab >}}
{{</ codetab >}}

Et enfin, l'appel dans `engine.py`. Nous définirons aussi la variable ici.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    max_monsters_per_room = 3
+   max_items_per_room = 2
    ...
    game_map = GameMap(map_width, map_height)
    game_map.make_map(max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities,
-                     max_monsters_per_room)
+                     max_monsters_per_room, max_items_per_room)

    fov_recompute = True
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    max_monsters_per_room = 3
    <span class="new-text">max_items_per_room = 2</span>
    ...
    game_map = GameMap(map_width, map_height)
    game_map.make_map(max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities,
                      max_monsters_per_room, <span class="new-text">max_items_per_room</span>)

    fov_recompute = True
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Vous devriez maintenant voir quelques potions de soin deci delà dans le donjon.
Il est pour l'instant impossible de les ramasser. De toute évidence, on peut
commencer par donner au joueur un inventaire. Créons un nouveau composant,
appelé Inventory, qui contiendra une liste d'objets ainsi qu'une capacité
maximale pour l'inventaire. Créez un nouveau fichier dans le dosser
`components`, appelez le `inventory.py` et ajoutez-y le code suivant :

{{< highlight py3 >}}
class Inventory:
    def __init__(self, capacity):
        self.capacity = capacity
        self.items = []
{{</ highlight >}}

La liste `items` contiendra la liste des entités. "Capacity" indique combien
d'objets nous pouvons ramasser en tout.

Mais quel type d'entités pouvons nous ramassez ? Nous n'avons pas forcement
envie que le joueur puisse ramasser *n'importe quelle* entité sur lequel il
marche (les cadavres, par exemple...) aussi comment les distinguer ?

Créons un nouveau composant, appelé `Item` que nous ajouterons aux entités
quand nous voudrons qu'elle puisse être ramassées. Créez un fichier appelé
`item.py` dans `components` et ajoutez-y la classe suivante :

{{< highlight py3 >}}
class Item:
    def __init__(self):
        pass
{{</ highlight >}}

Quoi ? Une classe vide ? Ne vous inquiétez pas, nous y ajouterons des choses
plus intéressantes une fois qu'on abordera le ramassage des items. Mais pour
l'instant, si une entité a ce composant, on peut la ramasser et sinon on ne
le peut. Aussi, tout ce dont on a besoin pour l'instant est d'une classe vide.

Maintenant, modifions la classe `Entity` pour accepter les composants `Item` et
`Inventaire`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class Entity:
-   def __init__(self, x, y, char, color, name, blocks=False, render_order=RenderOrder.CORPSE, fighter=None, ai=None):
+   def __init__(self, x, y, char, color, name, blocks=False, render_order=RenderOrder.CORPSE, fighter=None, ai=None,
+                item=None, inventory=None):
        self.x = x
        self.y = y
        self.char = char
        self.color = color
        self.name = name
        self.blocks = blocks
        self.render_order = render_order
        self.fighter = fighter
        self.ai = ai
+       self.item = item
+       self.inventory = inventory

        if self.fighter:
            self.fighter.owner = self

        if self.ai:
            self.ai.owner = self

+       if self.item:
+           self.item.owner = self
+
+       if self.inventory:
+           self.inventory.owner = self
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class Entity:
    def __init__(self, x, y, char, color, name, blocks=False, render_order=RenderOrder.CORPSE, fighter=None, ai=None<span class="new-text">,
                 item=None, inventory=None</span>):
        self.x = x
        self.y = y
        self.char = char
        self.color = color
        self.name = name
        self.blocks = blocks
        self.render_order = render_order
        self.fighter = fighter
        self.ai = ai
        <span class="new-text">self.item = item
        self.inventory = inventory</span>

        if self.fighter:
            self.fighter.owner = self

        if self.ai:
            self.ai.owner = self

        <span class="new-text">if self.item:
            self.item.owner = self

        if self.inventory:
            self.inventory.owner = self</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Notre classe `Entity` étant mise à jour, nous devons modifier le joueur et la
potion de soin pour avoir respectivement les composants `Inventory` et `Item`.
Ce tutoriel ne donnera pas aux monstres un inventaire, mais vous pourriez
attacher le composant d'inventaire aux monstres pour un effet similaire.

Dans `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    fighter_component = Fighter(hp=30, defense=2, power=5)
+   inventory_component = Inventory(26)
-   player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR, fighter=fighter_component)
+   player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR,
+                   fighter=fighter_component, inventory=inventory_component)
    entities = [player]
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    fighter_component = Fighter(hp=30, defense=2, power=5)
    <span class="new-text">inventory_component = Inventory(26)</span>
    <span class="crossed-out-text">player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR, fighter=fighter_component)</span>
    <span class="new-text">player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR,
                    fighter=fighter_component, inventory=inventory_component)</span>
    entities = [player]
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Bien-sûr assurez vous d'importer `Inventory` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
from components.fighter import Fighter
+from components.inventory import Inventory
from death_functions import kill_monster, kill_player
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>from components.fighter import Fighter
<span class="new-text">from components.inventory import Inventory</span>
from death_functions import kill_monster, kill_player
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Ensuite, modifiez la partie relative aux potions de soin de la fonction
`place_entities`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
+               item_component = Item()
-               item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM)
+               item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
+                             item=item_component)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                <span class="new-text">item_component = Item()</span>
                <span class="crossed-out-text">item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM)</span>
                <span class="new-text">item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
                              item=item_component)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

... Et pensez à l'import :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from components.ai import BasicMonster
from components.fighter import Fighter
+from components.item import Item

from entity import Entity
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from components.ai import BasicMonster
from components.fighter import Fighter
<span class="new-text">from components.item import Item</span>

from entity import Entity
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Comment notre joeur va-t-il s'y prendre pour ramasser les objets ? Les
roguelikes permettent généralement de ramasser un item sur lequel on se trouve
et le notre ne sera pas différent. Nombreux sont ceux qui utilisent la touche
'g' (certainement pour 'grab', ramasser ou 'get', obtenir) pour ramasser et
nous ferons de même. Modifiez `handle_keys` dans `input_handlers.py` pour
utiliser la touche 'g'.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    elif key_char == 'n':
        return {'move': (1, 1)}

+   if key_char == 'g':
+       return {'pickup': True}

    if key.vk == libtcod.KEY_ENTER and key.lalt:
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    elif key_char == 'n':
        return {'move': (1, 1)}

    <span class="new-text">if key_char == 'g':
        return {'pickup': True}</span>

    if key.vk == libtcod.KEY_ENTER and key.lalt:
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant nous devons vérifier l'action de ramasser dans notre moteur.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        move = action.get('move')
+       pickup = action.get('pickup')
        exit = action.get('exit')
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        move = action.get('move')
        <span class="new-text">pickup = action.get('pickup')</span>
        exit = action.get('exit')
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant nous devons *faire* quelque chose quand cette variable 'pickup'
(ramasser) est vraie. D'abord, nous allons créer une méthode de la classe
`Inventory` pour ajouter un item à l'inventaire. Nous utiliserons le même
concept de de "results" que la dernière fois. Si l'ajout d'un objet à
l'inventaire est réussi nous renvoyons un résultat disant qu'on a ramassé
l'objet. Sinon on envoie un résultat qui indique un échec. Le moteur devra
déterminer ce qu'il faut faire avec l'entité "item".

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
+import tcod as libtcod

+from game_messages import Message


class Inventory:
    def __init__(self, capacity):
        self.capacity = capacity
        self.items = []

+   def add_item(self, item):
+       results = []
+
+       if len(self.items) >= self.capacity:
+           results.append({
+               'item_added': None,
+               'message': Message('You cannot carry any more, your inventory is full', libtcod.yellow)
+           })
+       else:
+           results.append({
+               'item_added': item,
+               'message': Message('You pick up the {0}!'.format(item.name), libtcod.blue)
+           })
+
+           self.items.append(item)
+
+       return results
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="new-text">import tcod as libtcod

from game_messages import Message</span>


class Inventory:
    def __init__(self, capacity):
        self.capacity = capacity
        self.items = []

    <span class="new-text">def add_item(self, item):
        results = []

        if len(self.items) >= self.capacity:
            results.append({
                'item_added': None,
                'message': Message('You cannot carry any more, your inventory is full', libtcod.yellow)
            })
        else:
            results.append({
                'item_added': item,
                'message': Message('You pick up the {0}!'.format(item.name), libtcod.blue)
            })

            self.items.append(item)

        return results</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Ajoutons du code à `engine.py` pour exécuter le résultat de l'ajout d'un objet
à l'inventaire.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        if move and game_state == GameStates.PLAYERS_TURN:
            ...

+       elif pickup and game_state == GameStates.PLAYERS_TURN:
+           for entity in entities:
+               if entity.item and entity.x == player.x and entity.y == player.y:
+                   pickup_results = player.inventory.add_item(entity)
+                   player_turn_results.extend(pickup_results)
+
+                   break
+           else:
+               message_log.add_message(Message('There is nothing here to pick up.', libtcod.yellow))

        if exit:
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        if move and game_state == GameStates.PLAYERS_TURN:
            ...

        <span class="new-text">elif pickup and game_state == GameStates.PLAYERS_TURN:
            for entity in entities:
                if entity.item and entity.x == player.x and entity.y == player.y:
                    pickup_results = player.inventory.add_item(entity)
                    player_turn_results.extend(pickup_results)

                    break
            else:
                message_log.add_message(Message('There is nothing here to pick up.', libtcod.yellow))</span>

        if exit:
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous devons importer `Message` pour que cela fonctionne :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-from game_messages import MessageLog
+from game_messages import Message, MessageLog
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>from game_messages import <span class="new-text">Message,</span> MessageLog</pre>
{{</ original-tab >}}
{{</ codetab >}}

Pour faire simple, nous bouclons sur chaque entité de la carte, vérifiant si
c'est un objet et si elle est disposée à la même place que le joueur. Si c'est
le cas on l'ajoute à l'inventaire et on ajoute le résultat à
`player_turn_results` et sinon, on crée un message pour informer le joueur que
rien ne peut être ramassé. L'expression 'break' nous assure qu'on ne peut
ramasser qu'un objet à la fois (mais peut-être que dans votre jeu, vous
permettrez au joueur d'en ramasser plusieurs d'un coup).

Maintenant, passons à l'exécution du "ramassage" dans notre boucle qui s'occupe
du résultat du tour du joueur.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        for player_turn_result in player_turn_results:
            message = player_turn_result.get('message')
            dead_entity = player_turn_result.get('dead')
+           item_added = player_turn_result.get('item_added')

            if message:
                message_log.add_message(message)

            if dead_entity:
                if dead_entity == player:
                    message, game_state = kill_player(dead_entity)
                else:
                    message = kill_monster(dead_entity)

                message_log.add_message(message)

+           if item_added:
+               entities.remove(item_added)
+
+               game_state = GameStates.ENEMY_TURN

        if game_state == GameStates.ENEMY_TURN:
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        for player_turn_result in player_turn_results:
            message = player_turn_result.get('message')
            dead_entity = player_turn_result.get('dead')
            <span class="new-text">item_added = player_turn_result.get('item_added')</span>

            if message:
                message_log.add_message(message)

            if dead_entity:
                if dead_entity == player:
                    message, game_state = kill_player(dead_entity)
                else:
                    message = kill_monster(dead_entity)

                message_log.add_message(message)

            <span class="new-text">if item_added:
                entities.remove(item_added)

                game_state = GameStates.ENEMY_TURN</span>

        if game_state == GameStates.ENEMY_TURN:
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Cette partie s'occupe de retirer l'entité de la liste des entités
affichées maintenant que cela est dans l'inventaire du joueur et non plus
sur la carte et passe le tour à l'ennemi. Plutôt simple.

Lancez le projet et vous devriez être capable de ramasser les potions de soin
du sol. Bien sûr, cela ne fait aucun bien à notre vagabond pour l'instant,
on ne peut rien en faire. Ce n'est donc qu'un gachis de tour.

Avant d'en passer à l'utilisation de l'objet, nous devons avoir un moyen
d'examiner et de choisir les objets à utiliser. Nous créons une interface pour
l'inventaire que le joueur peut ouvrir et dans lequel il peut choisir un objet.
Créons un nouveau fichier, appelé `menus.py` où nous stockerons nos fonctions de
menus pour l'inventaire et tous les autres menus dont on aura besoin dans ce
tutoriel. Ajoutez-y le code suivant :

{{< highlight py3 >}}
import tcod as libtcod


def menu(con, header, options, width, screen_width, screen_height):
    if len(options) > 26: raise ValueError('Cannot have a menu with more than 26 options.')

    # calculate total height for the header (after auto-wrap) and one line per option
    header_height = libtcod.console_get_height_rect(con, 0, 0, width, screen_height, header)
    height = len(options) + header_height

    # create an off-screen console that represents the menu's window
    window = libtcod.console_new(width, height)

    # print the header, with auto-wrap
    libtcod.console_set_default_foreground(window, libtcod.white)
    libtcod.console_print_rect_ex(window, 0, 0, width, height, libtcod.BKGND_NONE, libtcod.LEFT, header)

    # print all the options
    y = header_height
    letter_index = ord('a')
    for option_text in options:
        text = '(' + chr(letter_index) + ') ' + option_text
        libtcod.console_print_ex(window, 0, y, libtcod.BKGND_NONE, libtcod.LEFT, text)
        y += 1
        letter_index += 1

    # blit the contents of "window" to the root console
    x = int(screen_width / 2 - width / 2)
    y = int(screen_height / 2 - height / 2)
    libtcod.console_blit(window, 0, 0, width, height, 0, x, y, 1.0, 0.7)
{{</ highlight >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def menu(con, header, options, width, screen_width, screen_height):
    ...

+def inventory_menu(con, header, inventory, inventory_width, screen_width, screen_height):
+   # show a menu with each item of the inventory as an option
+   if len(inventory.items) == 0:
+       options = ['Inventory is empty.']
+   else:
+       options = [item.name for item in inventory.items]
+
+   menu(con, header, options, inventory_width, screen_width, screen_height)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def menu(con, header, options, width, screen_width, screen_height):
    ...

<span class="new-text">def inventory_menu(con, header, inventory, inventory_width, screen_width, screen_height):
    # show a menu with each item of the inventory as an option
    if len(inventory.items) == 0:
        options = ['Inventory is empty.']
    else:
        options = [item.name for item in inventory.items]

    menu(con, header, options, inventory_width, screen_width, screen_height)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Comment afficher ce menu ? Une manière est de changer l'état du jeu et, quand
l'état du jeu est sur `menu d'inventaire`, on affiche le menu et on accepte
les saisies clavier. Aussi ajoutons l'option à `GameStates` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
from enum import Enum


class GameStates(Enum):
    PLAYERS_TURN = 1
    ENEMY_TURN = 2
    PLAYER_DEAD = 3
+   SHOW_INVENTORY = 4
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>from enum import Enum


class GameStates(Enum):
    PLAYERS_TURN = 1
    ENEMY_TURN = 2
    PLAYER_DEAD = 3
    <span class="new-text">SHOW_INVENTORY = 4</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Quand bascule-t-on vers ce nouvel état ? Ajoutons une nouvelle touche pour
passer en mode "inventaire".  Ainsi que vous l'aurez deviné on utilisera la
touche "i" pour ce faire.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    if key_char == 'g':
        return {'pickup': True}

+   elif key_char == 'i':
+       return {'show_inventory': True}

    if key.vk == libtcod.KEY_ENTER and key.lalt:
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    if key_char == 'g':
        return {'pickup': True}

    <span class="new-text">elif key_char == 'i':
        return {'show_inventory': True}</span>

    if key.vk == libtcod.KEY_ENTER and key.lalt:
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Non seulement voulons nous basculer l'état du jeu vers `SHOW_INVENTORY`
(afficher l'inventaire) mais aussi voulons nous revenir vers le précédent état
du jeu si nous quittons le menu sans rien faire. Cela permet aussi de ne pas
gaspiller un tour si on se contente d'ouvrir l'inventaire ainsi que d'ouvrir
l'inventaire après la mort (la rendant encore plus douloureuse). Aussi nous
avons besoin d'une variable qui retienne l'état précédent.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    game_state = GameStates.PLAYERS_TURN
+   previous_game_state = game_state

    while not libtcod.console_is_window_closed():
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    game_state = GameStates.PLAYERS_TURN
    <span class="new-text">previous_game_state = game_state</span>

    while not libtcod.console_is_window_closed():
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        pickup = action.get('pickup')
+       show_inventory = action.get('show_inventory')
        exit = action.get('exit')
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        pickup = action.get('pickup')
        <span class="new-text">show_inventory = action.get('show_inventory')</span>
        exit = action.get('exit')
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        elif pickup and game_state == GameStates.PLAYERS_TURN:
            ...

+       if show_inventory:
+           previous_game_state = game_state
+           game_state = GameStates.SHOW_INVENTORY

        if exit:
-           return True
+           if game_state == GameStates.SHOW_INVENTORY:
+               game_state = previous_game_state
+           else:
+               return True
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>
        elif pickup and game_state == GameStates.PLAYERS_TURN:
            ...

        <span class="new-text">if show_inventory:
            previous_game_state = game_state
            game_state = GameStates.SHOW_INVENTORY</span>

        if exit:
            <span class="new-text">if game_state == GameStates.SHOW_INVENTORY:
                game_state = previous_game_state
            else:</span>
                <span style="color: blue">return True</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous modifions notre précédente fonction de sortie pour revenir à l'état
précédent si on ouvre l'inventaire. Ainsi, la touche "escape" ferme seulement
le menu sans quitter le jeu.

Maintenant nous basculons l'état du jeu mais nous devons toujours afficher le
menu. Cela va sans dire, nous devons modifier `render_all`. L'inventaire n'étant
affiché que lorsque l'état est sur `SHOW_INVENTORY`, la fonction `render_all`
doit connaître cet état. Modifiez l'appel dans `engine.py`.


{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width,
-                  screen_height, bar_width, panel_height, panel_y, mouse, colors)
+render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width,
+                  screen_height, bar_width, panel_height, panel_y, mouse, colors, game_state)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width,
                   screen_height, bar_width, panel_height, panel_y, mouse, colors, <span class="new-text">game_state</span>)</pre>
{{</ original-tab >}}
{{</ codetab >}}

Et maintenant la définition :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width, screen_height,
-              bar_width, panel_height, panel_y, mouse, colors):
+              bar_width, panel_height, panel_y, mouse, colors, game_state):
    ...
    ...
    libtcod.console_blit(panel, 0, 0, screen_width, panel_height, 0, 0, panel_y)

+   if game_state == GameStates.SHOW_INVENTORY:
+       inventory_menu(con, 'Press the key next to an item to use it, or Esc to cancel.\n',
+                      player.inventory, 50, screen_width, screen_height)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width, screen_height,
               bar_width, panel_height, panel_y, mouse, colors<span class="new-text">, game_state</span>):
    ...
    ...
    libtcod.console_blit(panel, 0, 0, screen_width, panel_height, 0, 0, panel_y)

    <span class="new-text">if game_state == GameStates.SHOW_INVENTORY:
        inventory_menu(con, 'Press the key next to an item to use it, or Esc to cancel.\n',
                       player.inventory, 50, screen_width, screen_height)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous devons importer `GameStates` et `inventory_menu` pour que cela fontionne.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

from enum import Enum

+from game_states import GameStates
+
+from menus import inventory_menu


class RenderOrder(Enum):
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

from enum import Enum

<span class="new-text">from game_states import GameStates

from menus import inventory_menu</span>


class RenderOrder(Enum):
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Lancez le projet maintenant. Vous devriez pouvoir ouvrir l'inventaire et voir
tous les objets ramassés jusque là. Nous ne pouvons toujours rien en faire
mais nous y sommes presque \!

De manière à choisir un objet de l'inventaire, nous devons modifier
`handle_keys`. Pourquoi ? Parce que nous allons choisir les objets avec les
touches du clavier ce qui est très bien mais certaines sont déjà utilisées
pour le déplacement et d'autres choses. Il serait agréable que notre fonction
réagisse différemment selon l'état du jeu.

Voici ce qu'on va faire : nous allons séparer `handle_keys` en différentes
fonctions, chacune renvoyant un résultat différent selon l'état du jeu. Renommez
la fonction `handle_keys` en `handle_player_turn_keys` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-def handle_keys(key):
+def handle_player_turn_keys(key):
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="crossed-out-text">def handle_keys(key):</span>
<span class="new-text">def handle_player_turn_keys(key):</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Ensuite, créez une nouvelle fonction `handle_keys` qui appelle
`handle_player_turn_keys`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

+from game_states import GameStates
+
+
+def handle_keys(key, game_state):
+   if game_state == GameStates.PLAYERS_TURN:
+       return handle_player_turn_keys(key)
+
+   return {}


def handle_player_turn_keys(key):
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

<span class="new-text">from game_states import GameStates


def handle_keys(key, game_state):
    if game_state == GameStates.PLAYERS_TURN:
        return handle_player_turn_keys(key)

    return {}</span>


def handle_player_turn_keys(key):
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

N'oubliez pas de modifier l'appel à `handle_keys` dans `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-action = handle_keys(key)
+action = handle_keys(key, game_state)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>action = handle_keys(key<span class="new-text">, game_state</span>)</pre>
{{</ original-tab >}}
{{</ codetab >}}

Avant de passer à la partie sur l'inventaire, assurons nos arrières et ajoutons
un gestionnaire de touches pour la mort du joueur.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
def handle_keys(key, game_state):
    if game_state == GameStates.PLAYERS_TURN:
        return handle_player_turn_keys(key)
+   elif game_state == GameStates.PLAYER_DEAD:
+       return handle_player_dead_keys(key)

    return {}


def handle_player_turn_keys(key):
    ...


+def handle_player_dead_keys(key):
+   key_char = chr(key.c)
+
+   if key_char == 'i':
+       return {'show_inventory': True}
+
+   if key.vk == libtcod.KEY_ENTER and key.lalt:
+       # Alt+Enter: toggle full screen
+       return {'fullscreen': True}
+   elif key.vk == libtcod.KEY_ESCAPE:
+       # Exit the menu
+       return {'exit': True}
+
+   return {}
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
def handle_keys(key, game_state):
    if game_state == GameStates.PLAYERS_TURN:
        return handle_player_turn_keys(key)
    <span class="new-text">elif game_state == GameStates.PLAYER_DEAD:
        return handle_player_dead_keys(key)</span>

    return {}


def handle_player_turn_keys(key):
    ...


<span class="new-text">def handle_player_dead_keys(key):
    key_char = chr(key.c)

    if key_char == 'i':
        return {'show_inventory': True}

    if key.vk == libtcod.KEY_ENTER and key.lalt:
        # Alt+Enter: toggle full screen
        return {'fullscreen': True}
    elif key.vk == libtcod.KEY_ESCAPE:
        # Exit the menu
        return {'exit': True}

    return {}</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant crééons une nouvelle fonction, appelée `handle_inventory_keys` qui
va gérer nos saisies quand le menu d'inventaire est ouvert.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
def handle_keys(key, game_state):
    if game_state == GameStates.PLAYERS_TURN:
        return handle_player_turn_keys(key)
    elif game_state == GameStates.PLAYER_DEAD:
        return handle_player_dead_keys(key)
+   elif game_state == GameStates.SHOW_INVENTORY:
+       return handle_inventory_keys(key)

    return {}
...

+def handle_inventory_keys(key):
+   index = key.c - ord('a')
+
+   if index >= 0:
+       return {'inventory_index': index}
+
+   if key.vk == libtcod.KEY_ENTER and key.lalt:
+       # Alt+Enter: toggle full screen
+       return {'fullscreen': True}
+   elif key.vk == libtcod.KEY_ESCAPE:
+       # Exit the menu
+       return {'exit': True}
+
+   return {}
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
def handle_keys(key, game_state):
    if game_state == GameStates.PLAYERS_TURN:
        return handle_player_turn_keys(key)
    elif game_state == GameStates.PLAYER_DEAD:
        return handle_player_dead_keys(key)
    <span class="new-text">elif game_state == GameStates.SHOW_INVENTORY:
        return handle_inventory_keys(key)</span>

    return {}
...

<span class="new-text">def handle_inventory_keys(key):
    index = key.c - ord('a')

    if index >= 0:
        return {'inventory_index': index}

    if key.vk == libtcod.KEY_ENTER and key.lalt:
        # Alt+Enter: toggle full screen
        return {'fullscreen': True}
    elif key.vk == libtcod.KEY_ESCAPE:
        # Exit the menu
        return {'exit': True}

    return {}</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Qu'est-ce que cette histoire avec la fonction `ord` ? Disons le simplement,
on converti la touche pressée en un indice. 'a' sera 0, 'b' sera 1 et ainsi de
suite. Cela nous permettra de choisir un item de l'inventaire dans `engine.py`.
Tout ce que le gestionnaire de saisie doit faire est renvoyer l'indice de ce
qui est ramassé. Il n'a pas besoin de savoir quoi que ce soit de l'objet
ni ce qui doit en être fait.

Ajoutons cet indice à `engin.py` et utilisons le \!

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        show_inventory = action.get('show_inventory')
+       inventory_index = action.get('inventory_index')
        exit = action.get('exit')
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        show_inventory = action.get('show_inventory')
        <span class="new-text">inventory_index = action.get('inventory_index')</span>
        exit = action.get('exit')
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        if show_inventory:
            ...

+       if inventory_index is not None and previous_game_state != GameStates.PLAYER_DEAD and inventory_index < len(
+               player.inventory.items):
+           item = player.inventory.items[inventory_index]
+           print(item)

        if exit:
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        if show_inventory:
            ...

        <span class="new-text">if inventory_index is not None and previous_game_state != GameStates.PLAYER_DEAD and inventory_index < len(
                player.inventory.items):
            item = player.inventory.items[inventory_index]
            print(item)</span>

        if exit:
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

*\*Remarque : l'expression print n'est là que pour l'exemple. Nous avons presque
tout ce qu'il faut pour utiliser l'objet, je vous le promet \!*

Nous prenons l'indice qui a été choisi et "utilisons" (seulement un print pour
l'instant) l'objet en question. Lancez le projet et vérifiez que cela
fonctionne. Maintenant qu'on peut ouvrir le menu et choisir un objet, on peut
enfin passer à son utilisation.

Aussi, comment utiliser cet objet ? Le composant `Item` semble être un lieu
évident pour faire quelque chose. Mais chaque objet devrait faire quelque chose
de différent, n'est ce pas ? Aussi la classe `Item` ne va pas contenir les
fonctoins pour soigner le joueur ou faire des dégâts à un ennemi. À la place
elle contiendra seulement les appels aux fonctions de soin et de dégâts ainsi
que les paramètres dont elles auront besoin. Ensuite nous enverrons les
résultats des ces fonctions (comme d'habitude) et les exécuterons.

Modifiez `Item` ainsi :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class Item:
-   def __init__(self):
+   def __init__(self, use_function=None, **kwargs):
-       pass
+       self.use_function = use_function
+       self.function_kwargs = kwargs
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class Item:
    def __init__(self<span class="new-text">, use_function=None, **kwargs</span>):
        <span class="crossed-out-text">pass</span>
        <span class="new-text">self.use_function = use_function
        self.function_kwargs = kwargs</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Et que dire des fonctions elles mêmes ? Définissons les séparément, cela
nous permettra (dans le chapitre suivant) d'attribuer librement des fonctions
au paramètre `use_function`. Ainsi nous adapterons le comportement en fonction
de l'item selon nos besoins.

Créez un fichier, appelé `item_functions.py` et ajoutez-y la fonction suivante :

{{< highlight py3 >}}
import tcod as libtcod

from game_messages import Message


def heal(*args, **kwargs):
    entity = args[0]
    amount = kwargs.get('amount')

    results = []

    if entity.fighter.hp == entity.fighter.max_hp:
        results.append({'consumed': False, 'message': Message('You are already at full health', libtcod.yellow)})
    else:
        entity.fighter.heal(amount)
        results.append({'consumed': True, 'message': Message('Your wounds start to feel better!', libtcod.green)})

    return results
{{</ highlight >}}

Nous prenons l'entité qui utilise l'item comme premier argument (toutes nos
fonctions le feront, même si elles n'en ont pas besoin). Nous extrayons aussi
le nombre ("amount") des "kwargs" ce qui sera fourni par le `function_kwargs`
du composant `Item`.

Nous devons ajouter la méthode `heal` au composant `Fighter` pour que cela
fonctionne (remarque : les deux fonctions sont appelées "heal" mais elles
sont différentes).

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    def take_damage(self, amount):
        ...

+   def heal(self, amount):
+       self.hp += amount
+
+       if self.hp > self.max_hp:
+           self.hp = self.max_hp

    def attack(self, target):
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    def take_damage(self, amount):
        ...

    <span class="new-text">def heal(self, amount):
        self.hp += amount

        if self.hp > self.max_hp:
            self.hp = self.max_hp</span>

    def attack(self, target):
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Cela prendra plus de sens une fois qu'on aura passé la fonction `heal` aux
potions de soin. Faisons le maintenant : dans la fonction `place_entities` de
`game_map` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            ...
            if not any([entity for entity in entities if entity.x == x and entity.y == y]):
-               item_component = Item()
+               item_component = Item(use_function=heal, amount=4)
                item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
                              item=item_component)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            if not any([entity for entity in entities if entity.x == x and entity.y == y]):
                <span class="crossed-out-text">item_component = Item()</span>
                <span class="new-text">item_component = Item(use_function=heal, amount=4)</span>
                item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
                              item=item_component)</pre>
{{</ original-tab >}}
{{</ codetab >}}

Vous devez importer `heal` pour cela.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from entity import Entity

+from item_functions import heal

from map_objects.rectangle import Rect
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from entity import Entity

<span class="new-text">from item_functions import heal</span>

from map_objects.rectangle import Rect
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant notre item dispose d'une fonction à exécuter quand il est employé.
Mais *où* cela doit-il être appelé ? Pourquoi notre inventaire n'appellerait-il
pas la fonction ? Ajoutez la fonction suivante à `Inventory` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
+   def use(self, item_entity, **kwargs):
+       results = []
+
+       item_component = item_entity.item
+
+       if item_component.use_function is None:
+           results.append({'message': Message('The {0} cannot be used'.format(item_entity.name), libtcod.yellow)})
+       else:
+           kwargs = {**item_component.function_kwargs, **kwargs}
+           item_use_results = item_component.use_function(self.owner, **kwargs)
+
+           for item_use_result in item_use_results:
+               if item_use_result.get('consumed'):
+                   self.remove_item(item_entity)
+
+           results.extend(item_use_results)
+
+       return results
+
+   def remove_item(self, item):
+       self.items.remove(item)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    <span class="new-text">def use(self, item_entity, **kwargs):
        results = []

        item_component = item_entity.item

        if item_component.use_function is None:
            results.append({'message': Message('The {0} cannot be used'.format(item_entity.name), libtcod.yellow)})
        else:
            kwargs = {**item_component.function_kwargs, **kwargs}
            item_use_results = item_component.use_function(self.owner, **kwargs)

            for item_use_result in item_use_results:
                if item_use_result.get('consumed'):
                    self.remove_item(item_entity)

            results.extend(item_use_results)

        return results

    def remove_item(self, item):
        self.items.remove(item)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        if inventory_index is not None and previous_game_state != GameStates.PLAYER_DEAD and inventory_index < len(
                player.inventory.items):
            item = player.inventory.items[inventory_index]
-           print(item)
+           player_turn_results.extend(player.inventory.use(item))
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        if inventory_index is not None and previous_game_state != GameStates.PLAYER_DEAD and inventory_index < len(
                player.inventory.items):
            item = player.inventory.items[inventory_index]
            <span class="crossed-out-text">print(item)</span>
            <span class="new-text">player_turn_results.extend(player.inventory.use(item))</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Finally, let's handle the results of the item use function in
`engine.py`:

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        for player_turn_result in player_turn_results:
            message = player_turn_result.get('message')
            dead_entity = player_turn_result.get('dead')
            item_added = player_turn_result.get('item_added')
+           item_consumed = player_turn_result.get('consumed')
            ...
            if item_added:
                entities.remove(item_added)

                game_state = GameStates.ENEMY_TURN

+           if item_consumed:
+               game_state = GameStates.ENEMY_TURN
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        for player_turn_result in player_turn_results:
            message = player_turn_result.get('message')
            dead_entity = player_turn_result.get('dead')
            item_added = player_turn_result.get('item_added')
            <span class="new-text">item_consumed = player_turn_result.get('consumed')</span>
            ...
            if item_added:
                entities.remove(item_added)

                game_state = GameStates.ENEMY_TURN

            <span class="new-text">if item_consumed:
                game_state = GameStates.ENEMY_TURN</span>
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Lancez le projet maintenant. Vous pouvez consommer les potions de soin et cela
va utiliser un tour. Les potions ne seront pas dépensées si vous êtes déjà
au maximum de santé.

Une dernière étape avant de clore ce chapitre : abandonner (drop) un objet. Cela
peut semble inutile mais plus tard, quand nous aurons de nombreux niveaux dans
le donjon, le joueur devra prendre des décisions concernant les objets à
conserver et abandonner.

D'abord ajoutons un nouvel état au jeu :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class GameStates(Enum):
    PLAYERS_TURN = 1
    ENEMY_TURN = 2
    PLAYER_DEAD = 3
    SHOW_INVENTORY = 4
+   DROP_INVENTORY = 5
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class GameStates(Enum):
    PLAYERS_TURN = 1
    ENEMY_TURN = 2
    PLAYER_DEAD = 3
    SHOW_INVENTORY = 4
    <span class="new-text">DROP_INVENTORY = 5</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Ensuite modifier `handle_player_turn_keys` pour réagir à la touche 'd' :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    elif key_char == 'i':
        return {'show_inventory': True}

+   elif key_char == 'd':
+       return {'drop_inventory': True}

    if key.vk == libtcod.KEY_ENTER and key.lalt:
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    elif key_char == 'i':
        return {'show_inventory': True}

    <span class="new-text">elif key_char == 'd':
        return {'drop_inventory': True}</span>

    if key.vk == libtcod.KEY_ENTER and key.lalt:
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Mettez à jour le gestionnaire d'action pour en tenir compte :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        show_inventory = action.get('show_inventory')
+       drop_inventory = action.get('drop_inventory')
        inventory_index = action.get('inventory_index')
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        show_inventory = action.get('show_inventory')
        <span class="new-text">drop_inventory = action.get('drop_inventory')</span>
        inventory_index = action.get('inventory_index')
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous devrons changer l'état du jeu quand un joueur presse cette touche :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        if show_inventory:
            previous_game_state = game_state
            game_state = GameStates.SHOW_INVENTORY

+       if drop_inventory:
+           previous_game_state = game_state
+           game_state = GameStates.DROP_INVENTORY

        if inventory_index is not None and previous_game_state != GameStates.PLAYER_DEAD and inventory_index < len(
                player.inventory.items):
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        if show_inventory:
            previous_game_state = game_state
            game_state = GameStates.SHOW_INVENTORY

        <span class="new-text">if drop_inventory:
            previous_game_state = game_state
            game_state = GameStates.DROP_INVENTORY</span>

        if inventory_index is not None and previous_game_state != GameStates.PLAYER_DEAD and inventory_index < len(
                player.inventory.items):
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Vous pouvez croire qu'on doive ajouter une autre fonction pour gérer cette
touche mais ce n'est pas le cas. Nous allons simplement utiliser le code de
`SHOW_INVENTORY` dans ce but. Modifiez simplement le bloc "if" dans
`handle_keys` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def handle_keys(key, game_state):
    if game_state == GameStates.PLAYERS_TURN:
        return handle_player_turn_keys(key)
    elif game_state == GameStates.PLAYER_DEAD:
        return handle_player_dead_keys(key)
-   elif game_state == GameStates.SHOW_INVENTORY:
+   elif game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
        return handle_inventory_keys(key)

    return {}
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def handle_keys(key, game_state):
    if game_state == GameStates.PLAYERS_TURN:
        return handle_player_turn_keys(key)
    elif game_state == GameStates.PLAYER_DEAD:
        return handle_player_dead_keys(key)
    <span class="crossed-out-text">elif game_state == GameStates.SHOW_INVENTORY:</span>
    <span class="new-text">elif game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):</span>
        return handle_inventory_keys(key)

    return {}</pre>
{{</ original-tab >}}
{{</ codetab >}}

Aussi, changez la partie `exit` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        if exit:
-           if game_state in GameStates.SHOW_INVENTORY:
+           if game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
                game_state = previous_game_state
            else:
                return True
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        if exit:
            <span class="crossed-out-text">if game_state in GameStates.SHOW_INVENTORY:</span>
            <span class="new-text">if game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):</span>
                game_state = previous_game_state
            else:
                return True
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Et maintenant l'affichage du menu "drop" (déposer). Cela ne change pas vraiment
du menu d'inventaire aussi nous pouvons employer la même fonction et lui
donner un autre titre.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
-   if game_state == GameStates.SHOW_INVENTORY:
-       inventory_menu(con, 'Press the key next to an item to use it, or Esc to cancel.\n',
-                      player.inventory, 50, screen_width, screen_height)

+   if game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
+       if game_state == GameStates.SHOW_INVENTORY:
+           inventory_title = 'Press the key next to an item to use it, or Esc to cancel.\n'
+       else:
+           inventory_title = 'Press the key next to an item to drop it, or Esc to cancel.\n'
+
+       inventory_menu(con, inventory_title, player.inventory, 50, screen_width, screen_height)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    <span class="crossed-out-text">if game_state == GameStates.SHOW_INVENTORY:</span>
        <span class="crossed-out-text">inventory_menu(con, 'Press the key next to an item to use it, or Esc to cancel.\n',</span>
                       <span class="crossed-out-text">player.inventory, 50, screen_width, screen_height)</span>

    <span class="new-text">if game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
        if game_state == GameStates.SHOW_INVENTORY:
            inventory_title = 'Press the key next to an item to use it, or Esc to cancel.\n'
        else:
            inventory_title = 'Press the key next to an item to drop it, or Esc to cancel.\n'

        inventory_menu(con, inventory_title, player.inventory, 50, screen_width, screen_height)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Que se passe-t-il quand le joueur presse une touche de ce menu ? Modifions la
partie dans `engine.py` qui gère la partie `inventory_index` pour tenir compte
de l'état du jeu. Si l'état est sur `SHOW_INVENTORY`, alors on utilise l'objet,
et s'il est sur `DROP_INVENTORY` alors on appelle une fonction qui lâche
l'objet.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
if inventory_index is not None and previous_game_state != GameStates.PLAYER_DEAD and inventory_index < len(
                player.inventory.items):
            item = player.inventory.items[inventory_index]

-           player_turn_results.extend(player.inventory.use(item))

+           if game_state == GameStates.SHOW_INVENTORY:
                player_turn_results.extend(player.inventory.use(item))
+           elif game_state == GameStates.DROP_INVENTORY:
+               player_turn_results.extend(player.inventory.drop_item(item))
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>if inventory_index is not None and previous_game_state != GameStates.PLAYER_DEAD and inventory_index < len(
                player.inventory.items):
            item = player.inventory.items[inventory_index]

            <span class="new-text">if game_state == GameStates.SHOW_INVENTORY:</span>
                <span style="color: blue">player_turn_results.extend(player.inventory.use(item))</span>
            <span class="new-text">elif game_state == GameStates.DROP_INVENTORY:
                player_turn_results.extend(player.inventory.drop_item(item))</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous n'avons pas défini la méthode `drop_item` de l'inventaire pour l'instant
aussi faisons le maintenant. Cela va retirer l'objet de l'inventaire, établir
les coordonnées de l'objet sur celle du joueur (car l'objet est déposé aux
pieds du joueur) et renvoyer le résultat.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    def remove_item(self, item):
        self.items.remove(item)

+   def drop_item(self, item):
+       results = []
+
+       item.x = self.owner.x
+       item.y = self.owner.y
+
+       self.remove_item(item)
+       results.append({'item_dropped': item, 'message': Message('You dropped the {0}'.format(item.name),
+                                                                libtcod.yellow)})
+
+       return results
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    def remove_item(self, item):
        self.items.remove(item)

    <span class="new-text">def drop_item(self, item):
        results = []

        item.x = self.owner.x
        item.y = self.owner.y

        self.remove_item(item)
        results.append({'item_dropped': item, 'message': Message('You dropped the {0}'.format(item.name),
                                                                 libtcod.yellow)})

        return results</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Enfin, gérer les résultats dans `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        for player_turn_result in player_turn_results:
            message = player_turn_result.get('message')
            dead_entity = player_turn_result.get('dead')
            item_added = player_turn_result.get('item_added')
            item_consumed = player_turn_result.get('consumed')
+           item_dropped = player_turn_result.get('item_dropped')
            ...
            if item_consumed:
                game_state = GameStates.ENEMY_TURN

+           if item_dropped:
+               entities.append(item_dropped)
+
+               game_state = GameStates.ENEMY_TURN
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        for player_turn_result in player_turn_results:
            message = player_turn_result.get('message')
            dead_entity = player_turn_result.get('dead')
            item_added = player_turn_result.get('item_added')
            item_consumed = player_turn_result.get('consumed')
            <span class="new-text">item_dropped = player_turn_result.get('item_dropped')</span>
            ...
            if item_consumed:
                game_state = GameStates.ENEMY_TURN

            <span class="new-text">if item_dropped:
                entities.append(item_dropped)

                game_state = GameStates.ENEMY_TURN</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Quel chapitre ! Cela a demandé beaucoup de réglages pour utiliser les objets
mais nous disposons d'un cadre sur lequel bâtir. C'est ce que nous ferons
dans le chapitre suivant en ajoutant divers parchemins pour lancer des sorts
à nos ennemis.

Si vous voulez voir le code actuel entièrement, [cliquez ici](https://github.com/TStand90/roguelike_tutorial_revised/tree/part8).

[Cliquez ici pour vous rendre à la partie suivante de ce tutoriel.](/tutorials/tcod/part-9)

<script src="/js/codetabs.js"></script>
