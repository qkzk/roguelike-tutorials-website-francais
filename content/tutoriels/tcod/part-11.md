---
title: "Part 11 - Explorer les profondeurs du donjons"
date: 2019-03-30T09:34:06-07:00
draft: false
---
Notre jeu ne sera pas un jeu d'exploration de donjon tant qu'on n'aura qu'un
niveau à parcourir. Dans ce chapitre, nous permettrons au joueur de descendre
d'un niveau et nous mettrons un simple système de niveau en place afin de
rendre l'exploration plus gratifiante.

Commençons par modifier `GameMap` pour retenir la profondeur actuelle. Cela
nous aidera quand nous écrirons nos escaliers. Ouvrez `game_map` et réalisez
les modifications suivantes :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class GameMap:
-   def __init__(self, width, height):
+   def __init__(self, width, height, dungeon_level=1):
        self.width = width
        self.height = height
        self.tiles = self.initialize_tiles()

+       self.dungeon_level = dungeon_level
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class GameMap:
    def __init__(self, width, height<span class="new-text">, dungeon_level=1</span>):
        self.width = width
        self.height = height
        self.tiles = self.initialize_tiles()

        <span class="new-text">self.dungeon_level = dungeon_level</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Les escaliers en eux-même seront d'autres entités, comme vous pouviez vous y
attendre. Nous allons créer un nouveau composant qui règle cela en dehors des
autres et qu'on appelle `Stairs` (escaliers). Créez un fichier appelé
`stairs.py` et ajoutez-y la classe suivante :

{{< highlight py3 >}}
class Stairs:
    def __init__(self, floor):
        self.floor = floor
{{</ highlight >}}

La variable étage nous indique à quel étage nous allons arriver si on emprunte
les marches. Notre ne permet que de descendre mais vous pouvez aussi l'utiliser
pour monter d'un étage.

Comme notre autres composants, on doit le passer à `Entity`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class Entity:
    def __init__(self, x, y, char, color, name, blocks=False, render_order=RenderOrder.CORPSE, fighter=None, ai=None,
-                item=None, inventory=None):
+                item=None, inventory=None, stairs=None):
        self.x = x
        self.y = y
        self.char = char
        self.color = color
        self.name = name
        self.blocks = blocks
        self.render_order = render_order
        self.fighter = fighter
        self.ai = ai
        self.item = item
        self.inventory = inventory
+       self.stairs = stairs

        if self.fighter:
            self.fighter.owner = self

        if self.ai:
            self.ai.owner = self

        if self.item:
            self.item.owner = self

        if self.inventory:
            self.inventory.owner = self

+       if self.stairs:
+           self.stairs.owner = self
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class Entity:
    def __init__(self, x, y, char, color, name, blocks=False, render_order=RenderOrder.CORPSE, fighter=None, ai=None,
                 item=None, inventory=None<span class="new-text">, stairs=None</span>):
        self.x = x
        self.y = y
        self.char = char
        self.color = color
        self.name = name
        self.blocks = blocks
        self.render_order = render_order
        self.fighter = fighter
        self.ai = ai
        self.item = item
        self.inventory = inventory
        <span class="new-text">self.stairs = stairs</span>

        if self.fighter:
            self.fighter.owner = self

        if self.ai:
            self.ai.owner = self

        if self.item:
            self.item.owner = self

        if self.inventory:
            self.inventory.owner = self

        <span class="new-text">if self.stairs:
            self.stairs.owner = self</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Pour placer nos escaliers, on utilise notre fonction `make_map`. Pour garder
les choses simples on placera toujours les escaliers au milieu de la dernière
pièce crée. Modifiez la fonction ainsi :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities,
                 max_monsters_per_room, max_items_per_room):
        rooms = []
        num_rooms = 0

+       center_of_last_room_x = None
+       center_of_last_room_y = None

        for r in range(max_rooms):
            # random width and height
            w = randint(room_min_size, room_max_size)
            h = randint(room_min_size, room_max_size)
            # random position without going out of the boundaries of the map
            x = randint(0, map_width - w - 1)
            y = randint(0, map_height - h - 1)

            # "Rect" class makes rectangles easier to work with
            new_room = Rect(x, y, w, h)

            # run through the other rooms and see if they intersect with this one
            for other_room in rooms:
                if new_room.intersect(other_room):
                    break
            else:
                # this means there are no intersections, so this room is valid

                # "paint" it to the map's tiles
                self.create_room(new_room)

                # center coordinates of new room, will be useful later
                (new_x, new_y) = new_room.center()

+               center_of_last_room_x = new_x
+               center_of_last_room_y = new_y

                if num_rooms == 0:
                    # this is the first room, where the player starts at
                    player.x = new_x
                    player.y = new_y
                else:
                    # all rooms after the first:
                    # connect it to the previous room with a tunnel

                    # center coordinates of previous room
                    (prev_x, prev_y) = rooms[num_rooms - 1].center()

                    # flip a coin (random number that is either 0 or 1)
                    if randint(0, 1) == 1:
                        # first move horizontally, then vertically
                        self.create_h_tunnel(prev_x, new_x, prev_y)
                        self.create_v_tunnel(prev_y, new_y, new_x)
                    else:
                        # first move vertically, then horizontally
                        self.create_v_tunnel(prev_y, new_y, prev_x)
                        self.create_h_tunnel(prev_x, new_x, new_y)

                self.place_entities(new_room, entities, max_monsters_per_room, max_items_per_room)

                # finally, append the new room to the list
                rooms.append(new_room)
                num_rooms += 1

+       stairs_component = Stairs(self.dungeon_level + 1)
+       down_stairs = Entity(center_of_last_room_x, center_of_last_room_y, '>', libtcod.white, 'Stairs',
+                            render_order=RenderOrder.STAIRS, stairs=stairs_component)
+       entities.append(down_stairs)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities,
                 max_monsters_per_room, max_items_per_room):
        rooms = []
        num_rooms = 0

        <span class="new-text">center_of_last_room_x = None
        center_of_last_room_y = None</span>

        for r in range(max_rooms):
            # random width and height
            w = randint(room_min_size, room_max_size)
            h = randint(room_min_size, room_max_size)
            # random position without going out of the boundaries of the map
            x = randint(0, map_width - w - 1)
            y = randint(0, map_height - h - 1)

            # "Rect" class makes rectangles easier to work with
            new_room = Rect(x, y, w, h)

            # run through the other rooms and see if they intersect with this one
            for other_room in rooms:
                if new_room.intersect(other_room):
                    break
            else:
                # this means there are no intersections, so this room is valid

                # "paint" it to the map's tiles
                self.create_room(new_room)

                # center coordinates of new room, will be useful later
                (new_x, new_y) = new_room.center()

                <span class="new-text">center_of_last_room_x = new_x
                center_of_last_room_y = new_y</span>

                if num_rooms == 0:
                    # this is the first room, where the player starts at
                    player.x = new_x
                    player.y = new_y
                else:
                    # all rooms after the first:
                    # connect it to the previous room with a tunnel

                    # center coordinates of previous room
                    (prev_x, prev_y) = rooms[num_rooms - 1].center()

                    # flip a coin (random number that is either 0 or 1)
                    if randint(0, 1) == 1:
                        # first move horizontally, then vertically
                        self.create_h_tunnel(prev_x, new_x, prev_y)
                        self.create_v_tunnel(prev_y, new_y, new_x)
                    else:
                        # first move vertically, then horizontally
                        self.create_v_tunnel(prev_y, new_y, prev_x)
                        self.create_h_tunnel(prev_x, new_x, new_y)

                self.place_entities(new_room, entities, max_monsters_per_room, max_items_per_room)

                # finally, append the new room to the list
                rooms.append(new_room)
                num_rooms += 1

        <span class="new-text">stairs_component = Stairs(self.dungeon_level + 1)
        down_stairs = Entity(center_of_last_room_x, center_of_last_room_y, '>', libtcod.white, 'Stairs',
                             render_order=RenderOrder.STAIRS, stairs=stairs_component)
        entities.append(down_stairs)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Assurez-vous d'importer `Stairs` en haut :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from components.ai import BasicMonster
from components.fighter import Fighter
from components.item import Item
+from components.stairs import Stairs
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from components.ai import BasicMonster
from components.fighter import Fighter
from components.item import Item
<span class="new-text">from components.stairs import Stairs</span>
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous créons deux nouvelles variables pour conserver la position du centre
de la dernière pièce et nous les utilisons pour placer notre escalier. Les
escaliers en eux même sont simplement dans un tuple qui contient les coordonnées
x et y.

Remarquez que dans le code précédent, nous utilisons une nouvelle valeur dans
l'enum `RenderOrder`. Nous devrons l'ajouter à `RenderOrder`. Les escaliers
doivent apparaître en dessous du reste et ils doivent occuper la première
valeur. Les autres devront être décalé d'un étage.


{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class RenderOrder(Enum):
+   STAIRS = 1
-   CORPSE = 1
+   CORPSE = 2
-   ITEM = 2
+   ITEM = 3
-   ACTOR = 3
+   ACTOR = 4
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class RenderOrder(Enum):
    <span class="new-text">STAIRS = 1</span>
    <span class="crossed-out-text">CORPSE = 1</span>
    <span class="new-text">CORPSE = 2</span>
    <span class="crossed-out-text">ITEM = 2</span>
    <span class="new-text">ITEM = 3</span>
    <span class="crossed-out-text">ACTOR = 3</span>
    <span class="new-text">ACTOR = 4</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Remarquez que si vous utilisez Python 3.6 ou une version plus récente, vous
pouvez vous simplifier la tâche en utilisant la nouvelle fonction `auto()`

{{< highlight py3 >}}
class RenderOrder(Enum):
    STAIRS = auto()
    CORPSE = auto()
    ITEM = auto()
    ACTOR = auto()
{{</ highlight >}}

Une difficulté avec notre implémentation actuelle est qu'on ne peut voir les
escaliers que s'ils sont dans le champ de vision du joueur. Cela peut sembler
cohérent de prime abord mais imaginez que le joueur ait découvert les escaliers
et s'en éloigne. Ils n'apparaitront plus sur la carte \! Il serait mieux qu'une
fois découverts, les escaliers soient toujours dessinés.

Pour rendre cela possible, nous pouvons modifier la fonction `draw_entity` dans
`render_functions`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-def draw_entity(con, entity, fov_map):
+def draw_entity(con, entity, fov_map, game_map):
-   if libtcod.map_is_in_fov(fov_map, entity.x, entity.y):
+   if libtcod.map_is_in_fov(fov_map, entity.x, entity.y) or (entity.stairs and game_map.tiles[entity.x][entity.y].explored):
        libtcod.console_set_default_foreground(con, entity.color)
        libtcod.console_put_char(con, entity.x, entity.y, entity.char, libtcod.BKGND_NONE)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="crossed-out-text">def draw_entity(con, entity, fov_map):</span>
<span class="new-text">def draw_entity(con, entity, fov_map, game_map):</span>
    <span class="crossed-out-text">if libtcod.map_is_in_fov(fov_map, entity.x, entity.y):</span>
    <span class="new-text">if libtcod.map_is_in_fov(fov_map, entity.x, entity.y) or (entity.stairs and game_map.tiles[entity.x][entity.y].explored):</span>
        libtcod.console_set_default_foreground(con, entity.color)
        libtcod.console_put_char(con, entity.x, entity.y, entity.char, libtcod.BKGND_NONE)</pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous vérifions maintenant si une entité à le composant 'escaliers' et si la
carte a été explorée. Si c'est le cas, on dessine l'entité qu'elle soit toujours
dans le champ de vision ou non. Cela fonctionne même si une autre entité est
sur les escaliers.

Remarquez que nous passons l'objet `game_map` à `draw_entity`. Nous devons
aussi mettre à jour notre appel de `draw_entity` dans `render_all`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    for entity in entities_in_render_order:
-       draw_entity(con, entity, fov_map)
+       draw_entity(con, entity, fov_map, game_map)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    for entity in entities_in_render_order:
        draw_entity(con, entity, fov_map<span class="new-text">, game_map</span>)</pre>
{{</ original-tab >}}
{{</ codetab >}}

Lancez le jeu maintenant et vous devriez voir des escaliers (si vous parvenez
à les trouver avant de rencontrer votre créateur...). Maintenant faisons en
sorte qu'ils fassent quelque chose.

D'abord ajoutons un gestionnaire pour descendre les marches dans
`input_handlers.py`. Ajoutez ce qui suit à la fonction
`handle_player_turn_keys`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    elif key_char == 'd':
        return {'drop_inventory': True}

+   elif key.vk == libtcod.KEY_ENTER:
+       return {'take_stairs': True}

    if key.vk == libtcod.KEY_ENTER and key.lalt:
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    elif key_char == 'd':
        return {'drop_inventory': True}

    <span class="new-text">elif key.vk == libtcod.KEY_ENTER:
        return {'take_stairs': True}</span>

    if key.vk == libtcod.KEY_ENTER and key.lalt:
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

*\*Remarquez : j'ai utilisé la touche Enter plutôt que l'habituelle touche '\>'.
C'est parce que le code du tutoriel Roguebasin pour la touche '\>' ne fonctionne
pas.*

Ceci étant fait, nous devons implémenter le code pour déplacer le joueur à
l'étage inférieur. Nous devons générer une nouvelle carte, créer une nouvelle
liste d'entité et incrémenter un entier qui représente le niveau. Ce n'est pas
aussi compliqué que ça en a l'air \! Les choses se compliquent un peu si vous
voulez permettre au joueur de remonter mais pour la simplicité de ce tutoriel
nous allons supposer qu'une fois descendu d'un étage, vous ne pouvez plus
remonter.

Maintenant écrivons la fontion qui va nous descendre d'un étage. Ajoutez la
suite à la fin de `game_map.py` :

{{< highlight py3 >}}
    def next_floor(self, player, message_log, constants):
        self.dungeon_level += 1
        entities = [player]

        self.tiles = self.initialize_tiles()
        self.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],
                      constants['map_width'], constants['map_height'], player, entities,
                      constants['max_monsters_per_room'], constants['max_items_per_room'])

        player.fighter.heal(player.fighter.max_hp // 2)

        message_log.add_message(Message('You take a moment to rest, and recover your strength.', libtcod.light_violet))

        return entities
{{</ highlight >}}

La fonction commencen par incrémenter le niveau du donjon. La liste `entities`
est crée depuis zéro, elle ne contient que le joueur. Ensuite nous appelons
`make_map` pour générer un nouveau niveau comme nous l'avons fait au début du
jeu. Nous rendons aussi au joueur la moitié de ses HP pour le récompenser de
ses efforts et nous ajoutons un message dans ce sens. Ensuite nous renvoyons
la liste `entities` pour qu'elle soit utilisée par `engine.py`

Enfin, modifions `engine.py` pour utiliser cette nouvelle fonction.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        inventory_index = action.get('inventory_index')
+       take_stairs = action.get('take_stairs')
        exit = action.get('exit')
        ...
        if inventory_index is not None and previous_game_state != GameStates.PLAYER_DEAD and inventory_index < len(
            ...

+       if take_stairs and game_state == GameStates.PLAYERS_TURN:
+           for entity in entities:
+               if entity.stairs and entity.x == player.x and entity.y == player.y:
+                   entities = game_map.next_floor(player, message_log, constants)
+                   fov_map = initialize_fov(game_map)
+                   fov_recompute = True
+                   libtcod.console_clear(con)
+
+                   break
+           else:
+               message_log.add_message(Message('There are no stairs here.', libtcod.yellow))

        if game_state == GameStates.TARGETING:
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>
        ...
        inventory_index = action.get('inventory_index')
        <span class="new-text">take_stairs = action.get('take_stairs')</span>
        exit = action.get('exit')
        ...
        if inventory_index is not None and previous_game_state != GameStates.PLAYER_DEAD and inventory_index < len(
            ...

        <span class="new-text">if take_stairs and game_state == GameStates.PLAYERS_TURN:
            for entity in entities:
                if entity.stairs and entity.x == player.x and entity.y == player.y:
                    entities = game_map.next_floor(player, message_log, constants)
                    fov_map = initialize_fov(game_map)
                    fov_recompute = True
                    libtcod.console_clear(con)

                    break
            else:
                message_log.add_message(Message('There are no stairs here.', libtcod.yellow))</span>

        if game_state == GameStates.TARGETING:
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Si le joueur se tient sur les escaliers, nous appelons `next_floor` réglons
la liste `entities` sur de nouvelles valeurs. Nous nettoyons aussi l'écran
de façon à ce que la carte ne soit plus découverte et nous forçons le champ
de vision à être recalculé. S'il n'y a pas d'escaliers, nous l'indiquons au
joueur.

Nous pouvons simplement afficher la profondeur juste en dessous de la barre de
HP en générant la fonction `render_all` comme ceci :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    render_bar(panel, 1, 1, bar_width, 'HP', player.fighter.hp, player.fighter.max_hp,
               libtcod.light_red, libtcod.darker_red)
+   libtcod.console_print_ex(panel, 1, 3, libtcod.BKGND_NONE, libtcod.LEFT,
+                            'Dungeon level: {0}'.format(game_map.dungeon_level))

    libtcod.console_set_default_foreground(panel, libtcod.light_gray)
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>
    ...
    render_bar(panel, 1, 1, bar_width, 'HP', player.fighter.hp, player.fighter.max_hp,
               libtcod.light_red, libtcod.darker_red)
    <span class="new-text">libtcod.console_print_ex(panel, 1, 3, libtcod.BKGND_NONE, libtcod.LEFT,
                             'Dungeon level: {0}'.format(game_map.dungeon_level))</span>

    libtcod.console_set_default_foreground(panel, libtcod.light_gray)
    ...
</pre>
{{</ original-tab >}}
{{</ codetab >}}

Et c'est tout \! Nous pouvons enfin plonger dans la différents niveaux.
Néanmoins, la façon dont le jeu fonctionne ne rend pas cette exploration très
intéressante. De manière à faire ressembler notre jeu aux roguelikes nous
devons faire deux choses : donner au personnage un moyen de progresser (en
changeant de niveau ou en équipant du matériel) et rendre les monstres plus
menaçant au fur et à mesure qu'on descend. Nous allons nous concentrer sur le
premier dans ce chapitre et nous aborderons le second volet dans le chapitre
suivant.

La plupart des roguelikes (et des RPG en général) récompensent le joueur avec
de l'expérience une fois un monstre vaincu. Une fois un certain total
d'expérience atteint, le joueur gagne un niveau et devient plus fort. De manière
à y parvenir, nous devons faire plusieurs choses. Commençons par modifier
le composant `Fighter` pour contenir une nouvelle variable `xp`. Cela
représente les poits d'expérience acquis quand un monstre est tué (mais pas
ceux de l'entité elle-même, nous reviendrons là dessus plus tard).

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class Fighter:
-   def __init__(self, hp, defense, power):
+   def __init__(self, hp, defense, power, xp=0):
        self.max_hp = hp
        self.hp = hp
        self.defense = defense
        self.power = power
+       self.xp = xp
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class Fighter:
    def __init__(self, hp, defense, power<span class="new-text">, xp=0</span>):
        self.max_hp = hp
        self.hp = hp
        self.defense = defense
        self.power = power
        <span class="new-text">self.xp = xp</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous n'avons pas besoin de modifier le composant fighter du joueur mais nous
devons modifier le composant de nos ennemis. Ouvrez `game_map.py` et modifiez la
fonction `place_entities` pour inclure les points d'expérience de chaque
composant.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
                ...
                if randint(0, 100) < 80:
-                   fighter_component = Fighter(hp=10, defense=0, power=3)
+                   fighter_component = Fighter(hp=10, defense=0, power=3, xp=35)
                    ai_component = BasicMonster()

                    monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True,
                                     render_order=RenderOrder.ACTOR, fighter=fighter_component, ai=ai_component)
                else:
-                   fighter_component = Fighter(hp=16, defense=1, power=4)
+                   fighter_component = Fighter(hp=16, defense=1, power=4, xp=100)
                    ai_component = BasicMonster()

                    monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True, fighter=fighter_component,
                                     render_order=RenderOrder.ACTOR, ai=ai_component)
                ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                ...
                if randint(0, 100) < 80:
                    fighter_component = Fighter(hp=10, defense=0, power=3<span class="new-text">, xp=35</span>)
                    ai_component = BasicMonster()

                    monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True,
                                     render_order=RenderOrder.ACTOR, fighter=fighter_component, ai=ai_component)
                else:
                    fighter_component = Fighter(hp=16, defense=1, power=4<span class="new-text">, xp=100</span>)
                    ai_component = BasicMonster()

                    monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True, fighter=fighter_component,
                                     render_order=RenderOrder.ACTOR, ai=ai_component)
                ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

L'expérience du joueur sera légèrement différente parce qu'elle doit tenir
compte d'un cumul total. Nous devons aussi connaître le nombre de points
nécessaire avant d'atteindre le prochain niveau. Créons un nouveau composant
pour conserver ces informations que nous appellerons `Level` (niveau). Créez un
nouveau fichier dans `components` appelé `level.py` et ajoutez le code suivant :

{{< highlight py3 >}}
class Level:
    def __init__(self, current_level=1, current_xp=0, level_up_base=200, level_up_factor=150):
        self.current_level = current_level
        self.current_xp = current_xp
        self.level_up_base = level_up_base
        self.level_up_factor = level_up_factor

    @property
    def experience_to_next_level(self):
        return self.level_up_base + self.current_level * self.level_up_factor

    def add_xp(self, xp):
        self.current_xp += xp

        if self.current_xp > self.experience_to_next_level:
            self.current_xp -= self.experience_to_next_level
            self.current_level += 1

            return True
        else:
            return False
{{</ highlight>}}

J'ai réglé toutes les variables de cette sur les valeurs par défaut qui me
conviennent et je vous invite à choisir les vôtres. Il est certainement plus
judicieux de déplacer ces variables vers notre dictionnaire de constantes mais
afin de gagner du temps je les ai placées là.

Le niveau courant, `current_level`, est le niveau du joueur. Il commence
toujours à un sauf si nous chargeons une partie. `current_xp` est une somme
cumulée des points d'expérience. Elle est remise à zéro quand on gagne un
niveau. `level_up_base` et `level_up_factor` sont employées dans la formule
de gain de niveau.

Quand le joueur gagne des points d'expérience, on vérifie si l'expérience est
supérieure au seuil ajouté au niveau multiplié par le facteur. Cela rend
la progression plus lente au fur et à mesure qu'on gagne des niveaux.
Si l'expérience dépasse ce seuil, on reset `current_xp` et on renvoie `True`
(ce que notre moteur interprétera comme le gain d'un niveau par le joueur).

Le seuil actuel de gain de niveau est géré par la propriété
`experience_to_next_level`. Qu'est ce qu'une propriété ? C'est grosso-modo une
variable en lecture seule à laquelle on peut facilement accéder et qui réside
dans la classe de l'objet créé. À chaque accès `experience_to_next_level` aura
la dernière valeur aussi il suffit de dire
`player.level.experience_to_next_level` pour obtenir la bonne valeur.

Ajoutons ce nouveau composant à `Entity` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class Entity:
    def __init__(self, x, y, char, color, name, blocks=False, render_order=RenderOrder.CORPSE, fighter=None, ai=None,
-                item=None, inventory=None, stairs=None):
+                item=None, inventory=None, stairs=None, level=None):
        self.x = x
        self.y = y
        self.char = char
        self.color = color
        self.name = name
        self.blocks = blocks
        self.render_order = render_order
        self.fighter = fighter
        self.ai = ai
        self.item = item
        self.inventory = inventory
        self.stairs = stairs
+       self.level = level

        if self.fighter:
            self.fighter.owner = self

        if self.ai:
            self.ai.owner = self

        if self.item:
            self.item.owner = self

        if self.inventory:
            self.inventory.owner = self

        if self.stairs:
            self.stairs.owner = self

+       if self.level:
+           self.level.owner = self
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class Entity:
    def __init__(self, x, y, char, color, name, blocks=False, render_order=RenderOrder.CORPSE, fighter=None, ai=None,
                 item=None, inventory=None, stairs=None<span class="new-text">, level=None</span>):
        self.x = x
        self.y = y
        self.char = char
        self.color = color
        self.name = name
        self.blocks = blocks
        self.render_order = render_order
        self.fighter = fighter
        self.ai = ai
        self.item = item
        self.inventory = inventory
        self.stairs = stairs
        <span class="new-text">self.level = level</span>

        if self.fighter:
            self.fighter.owner = self

        if self.ai:
            self.ai.owner = self

        if self.item:
            self.item.owner = self

        if self.inventory:
            self.inventory.owner = self

        if self.stairs:
            self.stairs.owner = self

        <span class="new-text">if self.level:
            self.level.owner = self</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous devons maintenant l'ajouter à l'objet `player`. Ouvrez
`initialize_new_game.py` et réalisez les changements suivants :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    fighter_component = Fighter(hp=30, defense=2, power=5)
    inventory_component = Inventory(26)
+   level_component = Level()
    player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR,
-                   fighter=fighter_component, inventory=inventory_component)
+                   fighter=fighter_component, inventory=inventory_component, level=level_component)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>
    fighter_component = Fighter(hp=30, defense=2, power=5)
    inventory_component = Inventory(26)
    <span class="new-text">level_component = Level()</span>
    player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR,
                    fighter=fighter_component, inventory=inventory_component<span class="new-text">, level=level_component</span>)
</pre>
{{</ original-tab >}}
{{</ codetab >}}

Souvenez vous d'importez `Level` en haut :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
from components.fighter import Fighter
from components.inventory import Inventory
+from components.level import Level
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>
from components.fighter import Fighter
from components.inventory import Inventory
<span class="new-text">from components.level import Level</span>
</pre>
{{</ original-tab >}}
{{</ codetab >}}

Par tradition dans les RPG le gain d'expérience se fait à la mort du monstre.
Nous pouvons renvoyer le montant d'xp avec le résultat de la méthode
`take_damage` du composant `Fighter`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    def take_damage(self, amount):
        results = []

        self.hp -= amount

        if self.hp <= 0:
-           results.append({'dead': self.owner})
+           results.append({'dead': self.owner, 'xp': self.xp})

        return results
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    def take_damage(self, amount):
        results = []

        self.hp -= amount

        if self.hp <= 0:
            results.append({'dead': self.owner<span class="new-text">, 'xp': self.xp</span>})

        return results</pre>
{{</ original-tab >}}
{{</ codetab >}}

Et maintenant intégrons ce résultat dans  `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            ...
            targeting_cancelled = player_turn_result.get('targeting_cancelled')
+           xp = player_turn_result.get('xp')

            if message:
                ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            targeting_cancelled = player_turn_result.get('targeting_cancelled')
            <span class="new-text">xp = player_turn_result.get('xp')</span>

            if message:
                ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            ...
            if targeting_cancelled:
                ...

+           if xp:
+               leveled_up = player.level.add_xp(xp)
+               message_log.add_message(Message('You gain {0} experience points.'.format(xp)))
+
+               if leveled_up:
+                   message_log.add_message(Message(
+                       'Your battle skills grow stronger! You reached level {0}'.format(
+                           player.level.current_level) + '!', libtcod.yellow))
+                   previous_game_state = game_state
+                   game_state = GameStates.LEVEL_UP

        if game_state == GameStates.ENEMY_TURN:
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            if targeting_cancelled:
                ...

            <span class="new-text">if xp:
                leveled_up = player.level.add_xp(xp)
                message_log.add_message(Message('You gain {0} experience points.'.format(xp)))

                if leveled_up:
                    message_log.add_message(Message(
                        'Your battle skills grow stronger! You reached level {0}'.format(
                            player.level.current_level) + '!', libtcod.yellow))
                    previous_game_state = game_state
                    game_state = GameStates.LEVEL_UP</span>

        if game_state == GameStates.ENEMY_TURN:
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

De toute évidence nous devons ajouter l'état du jeu `LEVEL_UP` à notre enum
`GameStates`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class GameStates(Enum):
    PLAYERS_TURN = 1
    ENEMY_TURN = 2
    PLAYER_DEAD = 3
    SHOW_INVENTORY = 4
    DROP_INVENTORY = 5
    TARGETING = 6
+   LEVEL_UP = 7
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class GameStates(Enum):
    PLAYERS_TURN = 1
    ENEMY_TURN = 2
    PLAYER_DEAD = 3
    SHOW_INVENTORY = 4
    DROP_INVENTORY = 5
    TARGETING = 6
    <span class="new-text">LEVEL_UP = 7</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Que se passe-t-il quand le joueur gagne un niveau ? Notre système sera plutôt
simple : le joueur aura le choix entre améliorer ses HP, son attaque ou sa
défense. Un menu va apparaître, proposant à l'utilisateur de choisir parmi
une de ces améliorations et ne se fermera qu'une fois la sélection effectuée.

Créons une nouvelle fonction de menu, intitulée `level_up_menu` qui va afficher
nos options :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def main_menu(con, background_image, screen_width, screen_height):
    ...

+def level_up_menu(con, header, player, menu_width, screen_width, screen_height):
+   options = ['Constitution (+20 HP, from {0})'.format(player.fighter.max_hp),
+              'Strength (+1 attack, from {0})'.format(player.fighter.power),
+              'Agility (+1 defense, from {0})'.format(player.fighter.defense)]
+
+   menu(con, header, options, menu_width, screen_width, screen_height)


def message_box(con, header, width, screen_width, screen_height):
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def main_menu(con, background_image, screen_width, screen_height):
    ...

<span class="new-text">def level_up_menu(con, header, player, menu_width, screen_width, screen_height):
    options = ['Constitution (+20 HP, from {0})'.format(player.fighter.max_hp),
               'Strength (+1 attack, from {0})'.format(player.fighter.power),
               'Agility (+1 defense, from {0})'.format(player.fighter.defense)]

    menu(con, header, options, menu_width, screen_width, screen_height)</span>


def message_box(con, header, width, screen_width, screen_height):
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Modifiez la fonction `render_all` pour afficher ce menu après avoir importé
la fonction `level_up_menu`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

from enum import Enum

from game_states import GameStates

-from menus import inventory_menu
+from menus import inventory_menu, level_up_menu
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

from enum import Enum

from game_states import GameStates

from menus import inventory_menu<span class="new-text">, level_up_menu</span>
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    if game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
        ...

+   elif game_state == GameStates.LEVEL_UP:
+       level_up_menu(con, 'Level up! Choose a stat to raise:', player, 40, screen_width, screen_height)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>
    if game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
        ...

    <span class="new-text">elif game_state == GameStates.LEVEL_UP:
        level_up_menu(con, 'Level up! Choose a stat to raise:', player, 40, screen_width, screen_height)</span>
</pre>
{{</ original-tab >}}
{{</ codetab >}}

Bien sûr, nous devrons gérer la saisie dans ce menu. Ouvrez `input_handlers.py`
et ajoutez la fonction suivante :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def handle_main_menu(key):
    ...

+def handle_level_up_menu(key):
+   if key:
+       key_char = chr(key.c)
+
+       if key_char == 'a':
+           return {'level_up': 'hp'}
+       elif key_char == 'b':
+           return {'level_up': 'str'}
+       elif key_char == 'c':
+           return {'level_up': 'def'}
+
+   return {}


def handle_mouse(mouse):
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def handle_main_menu(key):
    ...

<span class="new-text">def handle_level_up_menu(key):
    if key:
        key_char = chr(key.c)

        if key_char == 'a':
            return {'level_up': 'hp'}
        elif key_char == 'b':
            return {'level_up': 'str'}
        elif key_char == 'c':
            return {'level_up': 'def'}

    return {}</span>


def handle_mouse(mouse):
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Modifiez la fonction `handle_keys` pour employer ce nouveau gestionnaire :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def handle_keys(key, game_state):
    if game_state == GameStates.PLAYERS_TURN:
        return handle_player_turn_keys(key)
    elif game_state == GameStates.PLAYER_DEAD:
        return handle_player_dead_keys(key)
    elif game_state == GameStates.TARGETING:
        return handle_targeting_keys(key)
    elif game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
        return handle_inventory_keys(key)
+   elif game_state == GameStates.LEVEL_UP:
+       return handle_level_up_menu(key)

    return {}
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def handle_keys(key, game_state):
    if game_state == GameStates.PLAYERS_TURN:
        return handle_player_turn_keys(key)
    elif game_state == GameStates.PLAYER_DEAD:
        return handle_player_dead_keys(key)
    elif game_state == GameStates.TARGETING:
        return handle_targeting_keys(key)
    elif game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
        return handle_inventory_keys(key)
    <span class="new-text">elif game_state == GameStates.LEVEL_UP:
        return handle_level_up_menu(key)</span>

    return {}</pre>
{{</ original-tab >}}
{{</ codetab >}}

Notre nouveau gestionnaire étant réalisé, nous devons employer ce résultat
dans `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        take_stairs = action.get('take_stairs')
+       level_up = action.get('level_up')
        exit = action.get('exit')
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        take_stairs = action.get('take_stairs')
        <span class="new-text">level_up = action.get('level_up')</span>
        exit = action.get('exit')
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        if take_stairs and game_state == GameStates.PLAYERS_TURN:
            ...

+       if level_up:
+           if level_up == 'hp':
+               player.fighter.max_hp += 20
+               player.fighter.hp += 20
+           elif level_up == 'str':
+               player.fighter.power += 1
+           elif level_up == 'def':
+               player.fighter.defense += 1
+
+           game_state = previous_game_state

        if game_state == GameStates.TARGETING:
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        if take_stairs and game_state == GameStates.PLAYERS_TURN:
            ...

        <span class="new-text">if level_up:
            if level_up == 'hp':
                player.fighter.max_hp += 20
                player.fighter.hp += 20
            elif level_up == 'str':
                player.fighter.power += 1
            elif level_up == 'def':
                player.fighter.defense += 1

            game_state = previous_game_state</span>

        if game_state == GameStates.TARGETING:
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

De manière à aider le joueur à connaître sa progression, créons et écran
de statistiques du personnage ("character screen") qui affichera les valeurs
courantes. Cela va demander un nouvel état du jeu aussi ajoutons le maintenant.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class GameStates(Enum):
    PLAYERS_TURN = 1
    ENEMY_TURN = 2
    PLAYER_DEAD = 3
    SHOW_INVENTORY = 4
    DROP_INVENTORY = 5
    TARGETING = 6
    LEVEL_UP = 7
+   CHARACTER_SCREEN = 8
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class GameStates(Enum):
    PLAYERS_TURN = 1
    ENEMY_TURN = 2
    PLAYER_DEAD = 3
    SHOW_INVENTORY = 4
    DROP_INVENTORY = 5
    TARGETING = 6
    LEVEL_UP = 7
    <span class="new-text">CHARACTER_SCREEN = 8</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous devrions afficher l'écran quand la touche 'c' est enfoncée. Ajoutons cette
touche à `handle_player_turn_keys` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    elif key.vk == libtcod.KEY_ENTER:
        return {'take_stairs': True}

+   elif key_char == 'c':
+       return {'show_character_screen': True}

    if key.vk == libtcod.KEY_ENTER and key.lalt:
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    elif key.vk == libtcod.KEY_ENTER:
        return {'take_stairs': True}

    <span class="new-text">elif key_char == 'c':
        return {'show_character_screen': True}</span>

    if key.vk == libtcod.KEY_ENTER and key.lalt:
        ...
</pre>
{{</ original-tab >}}
{{</ codetab >}}

Et maintenant intégrons le à `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        level_up = action.get('level_up')
+       show_character_screen = action.get('show_character_screen')
        exit = action.get('exit')
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        level_up = action.get('level_up')
        <span class="new-text">show_character_screen = action.get('show_character_screen')</span>
        exit = action.get('exit')
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        if level_up:
            ...

+       if show_character_screen:
+           previous_game_state = game_state
+           game_state = GameStates.CHARACTER_SCREEN

        if game_state == GameStates.TARGETING:
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        if level_up:
            ...

        <span class="new-text">if show_character_screen:
            previous_game_state = game_state
            game_state = GameStates.CHARACTER_SCREEN</span>

        if game_state == GameStates.TARGETING:
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant écrivons un gestionnaire de saisie pour l'écran de statistiques du
personnage. Tout ce qu'il fait est de gérer la touche Escape étant donné que
notre écran n'est pas du tout interactif.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def handle_level_up_menu(key):
    ...

+def handle_character_screen(key):
+   if key.vk == libtcod.KEY_ESCAPE:
+       return {'exit': True}
+
+   return {}


def handle_mouse(mouse):
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def handle_level_up_menu(key):
    ...

<span class="new-text">def handle_character_screen(key):
    if key.vk == libtcod.KEY_ESCAPE:
        return {'exit': True}

    return {}</span>


def handle_mouse(mouse):
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Modifier `handle_keys` pour appeler cette fonction quand on affiche l'écran
du personnage :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def handle_keys(key, game_state):
    if game_state == GameStates.PLAYERS_TURN:
        return handle_player_turn_keys(key)
    elif game_state == GameStates.PLAYER_DEAD:
        return handle_player_dead_keys(key)
    elif game_state == GameStates.TARGETING:
        return handle_targeting_keys(key)
    elif game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
        return handle_inventory_keys(key)
    elif game_state == GameStates.LEVEL_UP:
        return handle_level_up_menu(key)
+   elif game_state == GameStates.CHARACTER_SCREEN:
+       return handle_character_screen(key)

    return {}
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def handle_keys(key, game_state):
    if game_state == GameStates.PLAYERS_TURN:
        return handle_player_turn_keys(key)
    elif game_state == GameStates.PLAYER_DEAD:
        return handle_player_dead_keys(key)
    elif game_state == GameStates.TARGETING:
        return handle_targeting_keys(key)
    elif game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
        return handle_inventory_keys(key)
    elif game_state == GameStates.LEVEL_UP:
        return handle_level_up_menu(key)
    <span class="new-text">elif game_state == GameStates.CHARACTER_SCREEN:
        return handle_character_screen(key)</span>

    return {}</pre>
{{</ original-tab >}}
{{</ codetab >}}

Si le joueur enfonce la touche Escape, nous retournons simplement à l'état
précédent. Pour cela nous pouvons étendre notre code pour 'exit'.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        if exit:
-           if game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
+           if game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY, GameStates.CHARACTER_SCREEN):
                game_state = previous_game_state
            elif game_state == GameStates.TARGETING:
                player_turn_results.append({'targeting_cancelled': True})
            else:
                save_game(player, entities, game_map, message_log, game_state)

                return True
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        if exit:
            if game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY<span class="new-text">, GameStates.CHARACTER_SCREEN</span>):
                game_state = previous_game_state
            elif game_state == GameStates.TARGETING:
                player_turn_results.append({'targeting_cancelled': True})
            else:
                save_game(player, entities, game_map, message_log, game_state)

                return True
</pre>
{{</ original-tab >}}
{{</ codetab >}}

Cela prend soin des saisies de l'utilisateur. Maintenant pour afficher l'écran
nous devons utiliser une nouvelle fonction. Contrairement aux autres fonctions
de menu, nous n'affichons pas de liste d'options. À la place nous savons
d'emblée ce que nous voulons afficher. Aussi, on peut directement afficher
l'information à l'écran. Ouvrez `menus.py` et ajoutez la fonction suivante.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def level_up_menu(con, header, player, menu_width, screen_width, screen_height):
    ...

+def character_screen(player, character_screen_width, character_screen_height, screen_width, screen_height):
+   window = libtcod.console_new(character_screen_width, character_screen_height)
+
+   libtcod.console_set_default_foreground(window, libtcod.white)
+
+   libtcod.console_print_rect_ex(window, 0, 1, character_screen_width, character_screen_height, libtcod.BKGND_NONE,
+                                 libtcod.LEFT, 'Character Information')
+   libtcod.console_print_rect_ex(window, 0, 2, character_screen_width, character_screen_height, libtcod.BKGND_NONE,
+                                 libtcod.LEFT, 'Level: {0}'.format(player.level.current_level))
+   libtcod.console_print_rect_ex(window, 0, 3, character_screen_width, character_screen_height, libtcod.BKGND_NONE,
+                                 libtcod.LEFT, 'Experience: {0}'.format(player.level.current_xp))
+   libtcod.console_print_rect_ex(window, 0, 4, character_screen_width, character_screen_height, libtcod.BKGND_NONE,
+                                 libtcod.LEFT, 'Experience to Level: {0}'.format(player.level.experience_to_next_level))
+   libtcod.console_print_rect_ex(window, 0, 6, character_screen_width, character_screen_height, libtcod.BKGND_NONE,
+                                 libtcod.LEFT, 'Maximum HP: {0}'.format(player.fighter.max_hp))
+   libtcod.console_print_rect_ex(window, 0, 7, character_screen_width, character_screen_height, libtcod.BKGND_NONE,
+                                 libtcod.LEFT, 'Attack: {0}'.format(player.fighter.power))
+   libtcod.console_print_rect_ex(window, 0, 8, character_screen_width, character_screen_height, libtcod.BKGND_NONE,
+                                 libtcod.LEFT, 'Defense: {0}'.format(player.fighter.defense))
+
+   x = screen_width // 2 - character_screen_width // 2
+   y = screen_height // 2 - character_screen_height // 2
+   libtcod.console_blit(window, 0, 0, character_screen_width, character_screen_height, 0, x, y, 1.0, 0.7)


def message_box(con, header, width, screen_width, screen_height):
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def level_up_menu(con, header, player, menu_width, screen_width, screen_height):
    ...

<span class="new-text">def character_screen(player, character_screen_width, character_screen_height, screen_width, screen_height):
    window = libtcod.console_new(character_screen_width, character_screen_height)

    libtcod.console_set_default_foreground(window, libtcod.white)

    libtcod.console_print_rect_ex(window, 0, 1, character_screen_width, character_screen_height, libtcod.BKGND_NONE,
                                  libtcod.LEFT, 'Character Information')
    libtcod.console_print_rect_ex(window, 0, 2, character_screen_width, character_screen_height, libtcod.BKGND_NONE,
                                  libtcod.LEFT, 'Level: {0}'.format(player.level.current_level))
    libtcod.console_print_rect_ex(window, 0, 3, character_screen_width, character_screen_height, libtcod.BKGND_NONE,
                                  libtcod.LEFT, 'Experience: {0}'.format(player.level.current_xp))
    libtcod.console_print_rect_ex(window, 0, 4, character_screen_width, character_screen_height, libtcod.BKGND_NONE,
                                  libtcod.LEFT, 'Experience to Level: {0}'.format(player.level.experience_to_next_level))
    libtcod.console_print_rect_ex(window, 0, 6, character_screen_width, character_screen_height, libtcod.BKGND_NONE,
                                  libtcod.LEFT, 'Maximum HP: {0}'.format(player.fighter.max_hp))
    libtcod.console_print_rect_ex(window, 0, 7, character_screen_width, character_screen_height, libtcod.BKGND_NONE,
                                  libtcod.LEFT, 'Attack: {0}'.format(player.fighter.power))
    libtcod.console_print_rect_ex(window, 0, 8, character_screen_width, character_screen_height, libtcod.BKGND_NONE,
                                  libtcod.LEFT, 'Defense: {0}'.format(player.fighter.defense))

    x = screen_width // 2 - character_screen_width // 2
    y = screen_height // 2 - character_screen_height // 2
    libtcod.console_blit(window, 0, 0, character_screen_width, character_screen_height, 0, x, y, 1.0, 0.7)</span>


def message_box(con, header, width, screen_width, screen_height):
    ...
</pre>
{{</ original-tab >}}
{{</ codetab >}}

De manière à afficher ce nouveau menu, nous allons modifier `render_all` une
fois encore. Commencez par importer le menu.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

from enum import Enum

from game_states import GameStates

-from menus import inventory_menu, level_up_menu
+from menus import character_screen, inventory_menu, level_up_menu
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

from enum import Enum

from game_states import GameStates

from menus import <span class="new-text">character_screen,</span> inventory_menu, level_up_menu
...
</pre>
{{</ original-tab >}}
{{</ codetab >}}

Now, add the menu to the bottom of `render_all`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    elif game_state == GameStates.LEVEL_UP:
        level_up_menu(con, 'Level up! Choose a stat to raise:', player, 40, screen_width, screen_height)

+   elif game_state == GameStates.CHARACTER_SCREEN:
+       character_screen(player, 30, 10, screen_width, screen_height)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    elif game_state == GameStates.LEVEL_UP:
        level_up_menu(con, 'Level up! Choose a stat to raise:', player, 40, screen_width, screen_height)

    <span class="new-text">elif game_state == GameStates.CHARACTER_SCREEN:
        character_screen(player, 30, 10, screen_width, screen_height)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Dernière étape avant de conclure ce chapitre : bien plus tôt nous avons ajouté
les mouvement diagonaux pour le joueur mais nous avons oublié (okay **j'ai
oublié**) d'inclure une commande pour passer le tour. C'est plutôt simple à
faire mais nous en aurons besoin avant le prochain chapitre où la difficulté va
augmenter. Ouvrez `input_handlers.py` et ajoutez ce qui suit à
`handle_player_turn_keys`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def handle_player_turn_keys(key):
    key_char = chr(key.c)

    if key.vk == libtcod.KEY_UP or key_char == 'k':
        return {'move': (0, -1)}
    elif key.vk == libtcod.KEY_DOWN or key_char == 'j':
        return {'move': (0, 1)}
    elif key.vk == libtcod.KEY_LEFT or key_char == 'h':
        return {'move': (-1, 0)}
    elif key.vk == libtcod.KEY_RIGHT or key_char == 'l':
        return {'move': (1, 0)}
    elif key_char == 'y':
        return {'move': (-1, -1)}
    elif key_char == 'u':
        return {'move': (1, -1)}
    elif key_char == 'b':
        return {'move': (-1, 1)}
    elif key_char == 'n':
        return {'move': (1, 1)}
+   elif key_char == 'z':
+       return {'wait': True}
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def handle_player_turn_keys(key):
    key_char = chr(key.c)

    if key.vk == libtcod.KEY_UP or key_char == 'k':
        return {'move': (0, -1)}
    elif key.vk == libtcod.KEY_DOWN or key_char == 'j':
        return {'move': (0, 1)}
    elif key.vk == libtcod.KEY_LEFT or key_char == 'h':
        return {'move': (-1, 0)}
    elif key.vk == libtcod.KEY_RIGHT or key_char == 'l':
        return {'move': (1, 0)}
    elif key_char == 'y':
        return {'move': (-1, -1)}
    elif key_char == 'u':
        return {'move': (1, -1)}
    elif key_char == 'b':
        return {'move': (-1, 1)}
    elif key_char == 'n':
        return {'move': (1, 1)}
    <span class="new-text">elif key_char == 'z':
        return {'wait': True}</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Then, in `engine.py`:

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        move = action.get('move')
+       wait = action.get('wait')
        pickup = action.get('pickup')
        ...

        if move and game_state == GameStates.PLAYERS_TURN:
            ...

+       elif wait:
+           game_state = GameStates.ENEMY_TURN

        elif pickup and game_state == GameStates.PLAYERS_TURN:
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        move = action.get('move')
        <span class="new-text">wait = action.get('wait')</span>
        pickup = action.get('pickup')
        ...

        if move and game_state == GameStates.PLAYERS_TURN:
            ...

        <span class="new-text">elif wait:
            game_state = GameStates.ENEMY_TURN</span>

        elif pickup and game_state == GameStates.PLAYERS_TURN:
            ...
</pre>
{{</ original-tab >}}
{{</ codetab >}}

Ainsi tout ce qu'on fait est de "passer" le tour du joueur. Facile \! Vous
pourriez faire beaucoup de chose, comme ajouter 1 HP au joueur pour avoir
attendu mais je suis mauvais, et je ne pardonne pas aussi je ne le ferai pas.

C'est tout pour ce chapitre. Nous avons donné au joueur beaucoup d'avantages
(À vrai dire, ajouter un point en défense nous rend invulnérable devant les
Orcs...) mais cela va bientôt changer. Au prochain chapitre nous allons
améliorer les montres et *affaiblir* le joueur. C'est un roguelike après tout,
ce n'est pas supposé être facile.

Si vous voulez voir le code actuel entièrement, [cliquez ici](https://github.com/TStand90/roguelike_tutorial_revised/tree/part-11.

[Cliquez ici pour vous rendre à la partie suivante de ce tutoriel.](/tutorials/tcod/part-12)

<script src="/js/codetabs.js"></script>
