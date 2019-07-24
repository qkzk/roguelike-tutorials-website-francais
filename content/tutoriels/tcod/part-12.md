---
title: "Part 12 - Augmenter la difficulté"
date: 2019-03-30T09:34:08-07:00
draft: false
---
Notre jeu ne ressemble plus vraiment à un Roguelike : il est bien trop facile \!
Qui plus est la difficulté n'augmente quand l'on progresse. Nous allons
y remédier dans ce chapitre en rendant les ennemis plus fort et en répartissant
les ennemis et les objets à travers les niveaux de profondeur.

Avant de s'attaquer à ça, nous allons résoudre un problème de design qui nous
causera des soucis plus tard : notre fonction de choix aléatoire pour les
monstres et les objets. Pour l'instant on tire un entier entre 1 et 100 et
on vérifie si ce nombre est inférieur à une certaine "chance" pour un certain
objet. Cela convient pour un nombre limité d'options mais vous voudrez
certainement ajouter plus de deux ennemis et plus de quatre objets.

Il est plus judicieux de tirer au sort en attribuant à chaque objet un "poids"
qui indique sa probabilité d'apparaître. Nous pourrions les options et leurs
poids à une fonction qui en choisirait un aléatoirement et renverrait la
réponse.

Créez un fichier appelé `random_utils.py` et ajoutez-y :

{{< highlight py3 >}}
from random import randint


def random_choice_index(chances):
    random_chance = randint(1, sum(chances))

    running_sum = 0
    choice = 0
    for w in chances:
        running_sum += w

        if random_chance <= running_sum:
            return choice
        choice += 1


def random_choice_from_dict(choice_dict):
    choices = list(choice_dict.keys())
    chances = list(choice_dict.values())

    return choices[random_choice_index(chances)]
{{</ highlight >}}

Ajoutons mettons cette nouvelle fonction en action. Ouvrez `game_map.py` et 
modifiez ainsi :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from map_objects.tile import Tile

+from random_utils import random_choice_from_dict

from render_functions import RenderOrder
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from map_objects.tile import Tile

<span class="new-text">from random_utils import random_choice_from_dict</span>

from render_functions import RenderOrder
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    def place_entities(self, room, entities, max_monsters_per_room, max_items_per_room):
        number_of_monsters = randint(0, max_monsters_per_room)
        number_of_items = randint(0, max_items_per_room)

+       monster_chances = {'orc': 80, 'troll': 20}
+       item_chances = {'healing_potion': 70, 'lightning_scroll': 10, 'fireball_scroll': 10, 'confusion_scroll': 10}

        for i in range(number_of_monsters):
            x = randint(room.x1 + 1, room.x2 - 1)
            y = randint(room.y1 + 1, room.y2 - 1)

            if not any([entity for entity in entities if entity.x == x and entity.y == y]):
+               monster_choice = random_choice_from_dict(monster_chances)

-               if randint(0, 100) < 80:
+               if monster_choice == 'orc':
                    fighter_component = Fighter(hp=10, defense=0, power=3, xp=35)
                    ai_component = BasicMonster()

                    monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True,
                                     render_order=RenderOrder.ACTOR, fighter=fighter_component, ai=ai_component)
                else:
                    fighter_component = Fighter(hp=16, defense=1, power=4, xp=100)
                    ai_component = BasicMonster()

                    monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True, fighter=fighter_component,
                                     render_order=RenderOrder.ACTOR, ai=ai_component)

                entities.append(monster)

        for i in range(number_of_items):
            x = randint(room.x1 + 1, room.x2 - 1)
            y = randint(room.y1 + 1, room.y2 - 1)

            if not any([entity for entity in entities if entity.x == x and entity.y == y]):
-               item_chance = randint(0, 100)
+               item_choice = random_choice_from_dict(item_chances)

-               if item_chance < 70:
+               if item_choice == 'healing_potion':
                    item_component = Item(use_function=heal, amount=4)
                    item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
                                  item=item_component)
-               elif item_chance < 80:
+               elif item_choice == 'fireball_scroll':
                    item_component = Item(use_function=cast_fireball, targeting=True, targeting_message=Message(
                        'Left-click a target tile for the fireball, or right-click to cancel.', libtcod.light_cyan),
                                          damage=12, radius=3)
                    item = Entity(x, y, '#', libtcod.red, 'Fireball Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)
-               elif item_chance < 90:
+               elif item_choice == 'confusion_scroll':
                    item_component = Item(use_function=cast_confuse, targeting=True, targeting_message=Message(
                        'Left-click an enemy to confuse it, or right-click to cancel.', libtcod.light_cyan))
                    item = Entity(x, y, '#', libtcod.light_pink, 'Confusion Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)
                else:
                    item_component = Item(use_function=cast_lightning, damage=20, maximum_range=5)
                    item = Entity(x, y, '#', libtcod.yellow, 'Lightning Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)

                entities.append(item)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    def place_entities(self, room, entities, max_monsters_per_room, max_items_per_room):
        number_of_monsters = randint(0, max_monsters_per_room)
        number_of_items = randint(0, max_items_per_room)

        <span class="new-text">monster_chances = {'orc': 80, 'troll': 20}
        item_chances = {'healing_potion': 70, 'lightning_scroll': 10, 'fireball_scroll': 10, 'confusion_scroll': 10}</span>

        for i in range(number_of_monsters):
            x = randint(room.x1 + 1, room.x2 - 1)
            y = randint(room.y1 + 1, room.y2 - 1)

            if not any([entity for entity in entities if entity.x == x and entity.y == y]):
                <span class="new-text">monster_choice = random_choice_from_dict(monster_chances)</span>

                <span class="crossed-out-text">if randint(0, 100) < 80:</span>
                <span class="new-text">if monster_choice == 'orc':</span>
                    fighter_component = Fighter(hp=10, defense=0, power=3, xp=35)
                    ai_component = BasicMonster()

                    monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True,
                                     render_order=RenderOrder.ACTOR, fighter=fighter_component, ai=ai_component)
                else:
                    fighter_component = Fighter(hp=16, defense=1, power=4, xp=100)
                    ai_component = BasicMonster()

                    monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True, fighter=fighter_component,
                                     render_order=RenderOrder.ACTOR, ai=ai_component)

                entities.append(monster)

        for i in range(number_of_items):
            x = randint(room.x1 + 1, room.x2 - 1)
            y = randint(room.y1 + 1, room.y2 - 1)

            if not any([entity for entity in entities if entity.x == x and entity.y == y]):
                <span class="crossed-out-text">item_chance = randint(0, 100)</span>
                <span class="new-text">item_choice = random_choice_from_dict(item_chances)</span>

                <span class="crossed-out-text">if item_chance < 70:</span>
                <span class="new-text">if item_choice == 'healing_potion':</span>
                    item_component = Item(use_function=heal, amount=4)
                    item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
                                  item=item_component)
                <span class="crossed-out-text">elif item_chance < 80:</span>
                <span class="new-text">elif item_choice == 'fireball_scroll':</span>
                    item_component = Item(use_function=cast_fireball, targeting=True, targeting_message=Message(
                        'Left-click a target tile for the fireball, or right-click to cancel.', libtcod.light_cyan),
                                          damage=12, radius=3)
                    item = Entity(x, y, '#', libtcod.red, 'Fireball Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)
                <span class="crossed-out-text">elif item_chance < 90:</span>
                <span class="new-text">elif item_choice == 'confusion_scroll':</span>
                    item_component = Item(use_function=cast_confuse, targeting=True, targeting_message=Message(
                        'Left-click an enemy to confuse it, or right-click to cancel.', libtcod.light_cyan))
                    item = Entity(x, y, '#', libtcod.light_pink, 'Confusion Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)
                else:
                    item_component = Item(use_function=cast_lightning, damage=20, maximum_range=5)
                    item = Entity(x, y, '#', libtcod.yellow, 'Lightning Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)

                entities.append(item)</pre>
{{</ original-tab >}}
{{</ codetab >}}

On attribue à chaque ennemi ou objet un poids et la sélection est effectuée par
notre nouvelle fonction. Remarquez que le cumul des poids est 100 mais cela
n'est pas indispensable.

C'est beaucoup plus extensible que notre solution précédente mais nous n'avons
pas encore atteint notre objectif. Nous aimerions avoir des poids dépendant
du niveau de profondeur. Ainsi, plus on descend et plus on a de chances de
rencontrer des Trolls plutôt que des Orcs et plus on a de chance de trouver
des objets utiles plutôt que seulement des potions de soin.

Créons une nouvelle fonction qui prendra un intervalle de valeurs indiquant
quand une chose apparaît dans le donjon et avec quel poids.
La fonction renvoie le poids correspondant au niveau du donjon. Elle va
dans `random_utils.py` et ressemble à ça :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
from random import randint


+def from_dungeon_level(table, dungeon_level):
+   for (value, level) in reversed(table):
+       if dungeon_level >= level:
+           return value
+
+   return 0


def random_choice_index(chances):
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>from random import randint


<span class="new-text">def from_dungeon_level(table, dungeon_level):
    for (value, level) in reversed(table):
        if dungeon_level >= level:
            return value

    return 0</span>


def random_choice_index(chances):
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Cela n'a peut-être pas beaucoup de sens maintenant mais mettons la en action.
On peut retirer les variables `max_monsters_per_room` et `max_items_per_room`
qui sont maintenant déterminées par le niveau de profondeur et non plus passées
directement à la fonction.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from map_objects.tile import Tile

-from random_utils import random_choice_from_dict
+from random_utils import from_dungeon_level, random_choice_from_dict

from render_functions import RenderOrder
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from map_objects.tile import Tile

from random_utils import <span class="new-text">from_dungeon_level,</span> random_choice_from_dict

from render_functions import RenderOrder</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-   def place_entities(self, room, entities, max_monsters_per_room, max_items_per_room):
+   def place_entities(self, room, entities):
+       max_monsters_per_room = from_dungeon_level([[2, 1], [3, 4], [5, 6]], self.dungeon_level)
+       max_items_per_room = from_dungeon_level([[1, 1], [2, 4]], self.dungeon_level)

        number_of_monsters = randint(0, max_monsters_per_room)
        number_of_items = randint(0, max_items_per_room)

-       monster_chances = {'orc': 80, 'troll': 20}
-       item_chances = {'healing_potion': 70, 'lightning_scroll': 10, 'fireball_scroll': 10, 'confusion_scroll': 10}

+       monster_chances = {
+           'orc': 80,
+           'troll': from_dungeon_level([[15, 3], [30, 5], [60, 7]], self.dungeon_level)
+       }

+       item_chances = {
+           'healing_potion': 35,
+           'lightning_scroll': from_dungeon_level([[25, 4]], self.dungeon_level),
+           'fireball_scroll': from_dungeon_level([[25, 6]], self.dungeon_level),
+           'confusion_scroll': from_dungeon_level([[10, 2]], self.dungeon_level)
+       }
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    <span class="crossed-out-text">def place_entities(self, room, entities, max_monsters_per_room, max_items_per_room):</span>
    <span class="new-text">def place_entities(self, room, entities):</span>
        <span class="new-text">max_monsters_per_room = from_dungeon_level([[2, 1], [3, 4], [5, 6]], self.dungeon_level)
        max_items_per_room = from_dungeon_level([[1, 1], [2, 4]], self.dungeon_level)</span>

        number_of_monsters = randint(0, max_monsters_per_room)
        number_of_items = randint(0, max_items_per_room)

        <span class="crossed-out-text">monster_chances = {'orc': 80, 'troll': 20}</span>
        <span class="crossed-out-text">item_chances = {'healing_potion': 70, 'lightning_scroll': 10, 'fireball_scroll': 10, 'confusion_scroll': 10}</span>

        <span class="new-text">monster_chances = {
            'orc': 80,
            'troll': from_dungeon_level([[15, 3], [30, 5], [60, 7]], self.dungeon_level)
        }</span>

        <span class="new-text">item_chances = {
            'healing_potion': 35,
            'lightning_scroll': from_dungeon_level([[25, 4]], self.dungeon_level),
            'fireball_scroll': from_dungeon_level([[25, 6]], self.dungeon_level),
            'confusion_scroll': from_dungeon_level([[10, 2]], self.dungeon_level)
        }</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous aurons aussi besoin de mettre à jour notre appel à `place_entities` depuis
`make_map`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
                        ...
                        self.create_h_tunnel(prev_x, new_x, new_y)

-               self.place_entities(new_room, entities, max_monsters_per_room, max_items_per_room)
+               self.place_entities(new_room, entities)

                rooms.append(new_room)
                ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                        ...
                        self.create_h_tunnel(prev_x, new_x, new_y)

                <span class="crossed-out-text">self.place_entities(new_room, entities, max_monsters_per_room, max_items_per_room)</span>
                <span class="new-text">self.place_entities(new_room, entities)</span>

                rooms.append(new_room)
                ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Parce que nous avons retiré `max_monsters_per_room` et `max_items_per_room` de
l'appel de la fonction, nous pouvons aussi les retirer de la définition de
`make_map`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-   def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities,
-                max_monsters_per_room, max_items_per_room):
+   def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities):
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>
    <span class="crossed-out-text">def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities,</span>
                 <span class="crossed-out-text">max_monsters_per_room, max_items_per_room):</span>
    <span class="new-text">def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities):</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous devrions aussi retirer ces variables des appels de `make_map` dans
`initialize_new_game.py`...

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-   game_map.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],
-                     constants['map_width'], constants['map_height'], player, entities,
-                     constants['max_monsters_per_room'], constants['max_items_per_room'])
+   game_map.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],
+                     constants['map_width'], constants['map_height'], player, entities)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    <span class="crossed-out-text">game_map.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],</span>
                      <span class="crossed-out-text">constants['map_width'], constants['map_height'], player, entities,</span>
                      <span class="crossed-out-text">constants['max_monsters_per_room'], constants['max_items_per_room'])</span>
    <span class="new-text">game_map.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],
                      constants['map_width'], constants['map_height'], player, entities)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

... and in `next_floor` (in
`game_map.py`):

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-       self.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],
-                     constants['map_width'], constants['map_height'], player, entities,
-                     constants['max_monsters_per_room'], constants['max_items_per_room'])
+       self.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],
+                     constants['map_width'], constants['map_height'], player, entities)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        <span class="crossed-out-text">self.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],</span>
                      <span class="crossed-out-text">constants['map_width'], constants['map_height'], player, entities,</span>
                      <span class="crossed-out-text">constants['max_monsters_per_room'], constants['max_items_per_room'])</span>
        <span class="new-text">self.make_map(constants['max_rooms'], constants['room_min_size'], constants['room_max_size'],
                      constants['map_width'], constants['map_height'], player, entities)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Notre nouveau donjon devient plus dangereux au fur et à mesure de notre
descente \! Le jeu reste très facile néanmoins aussi changeons ça en modifiant
les valeurs par défaut pour chaque composant de l'entité `Fighter` notamment
le joueur lui même. Nous allons aussi modifier les valeurs des potions
de soin, du parchemin boule de feu et du parchemin d'éclair.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def get_game_variables(constants):
-   fighter_component = Fighter(hp=30, defense=2, power=5)
+   fighter_component = Fighter(hp=100, defense=1, power=4)
    inventory_component = Inventory(26)
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>
def get_game_variables(constants):
    <span class="crossed-out-text">fighter_component = Fighter(hp=30, defense=2, power=5)</span>
    <span class="new-text">fighter_component = Fighter(hp=100, defense=1, power=4)</span>
    inventory_component = Inventory(26)
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
                ...
                if monster_choice == 'orc':
-                   fighter_component = Fighter(hp=10, defense=0, power=3, xp=35)
+                   fighter_component = Fighter(hp=20, defense=0, power=4, xp=35)
                    ai_component = BasicMonster()

                    monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True,
                                     render_order=RenderOrder.ACTOR, fighter=fighter_component, ai=ai_component)
                else:
-                   fighter_component = Fighter(hp=16, defense=1, power=4, xp=100)
+                   fighter_component = Fighter(hp=30, defense=2, power=8, xp=100)
                    ai_component = BasicMonster()

                    monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True, fighter=fighter_component,
                                     render_order=RenderOrder.ACTOR, ai=ai_component)
                ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                ...
                if monster_choice == 'orc':
                    <span class="crossed-out-text">fighter_component = Fighter(hp=10, defense=0, power=3, xp=35)</span>
                    <span class="new-text">fighter_component = Fighter(hp=20, defense=0, power=4, xp=35)</span>
                    ai_component = BasicMonster()

                    monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True,
                                     render_order=RenderOrder.ACTOR, fighter=fighter_component, ai=ai_component)
                else:
                    <span class="crossed-out-text">fighter_component = Fighter(hp=16, defense=1, power=4, xp=100)</span>
                    <span class="new-text">fighter_component = Fighter(hp=30, defense=2, power=8, xp=100)</span>
                    ai_component = BasicMonster()

                    monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True, fighter=fighter_component,
                                     render_order=RenderOrder.ACTOR, ai=ai_component)
                ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            ...
            if not any([entity for entity in entities if entity.x == x and entity.y == y]):
                item_choice = random_choice_from_dict(item_chances)

                if item_choice == 'healing_potion':
-                   item_component = Item(use_function=heal, amount=4)
+                   item_component = Item(use_function=heal, amount=40)
                    item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
                                  item=item_component)
                elif item_choice == 'fireball_scroll':
-                   item_component = Item(use_function=cast_fireball, targeting=True, targeting_message=Message(
-                       'Left-click a target tile for the fireball, or right-click to cancel.', libtcod.light_cyan),
-                                         damage=12, radius=3)
+                   item_component = Item(use_function=cast_fireball, targeting=True, targeting_message=Message(
+                       'Left-click a target tile for the fireball, or right-click to cancel.', libtcod.light_cyan),
+                                         damage=25, radius=3)
                    item = Entity(x, y, '#', libtcod.red, 'Fireball Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)
                elif item_choice == 'confusion_scroll':
                    item_component = Item(use_function=cast_confuse, targeting=True, targeting_message=Message(
                        'Left-click an enemy to confuse it, or right-click to cancel.', libtcod.light_cyan))
                    item = Entity(x, y, '#', libtcod.light_pink, 'Confusion Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)
                else:
-                   item_component = Item(use_function=cast_lightning, damage=20, maximum_range=5)
+                   item_component = Item(use_function=cast_lightning, damage=40, maximum_range=5)
                    item = Entity(x, y, '#', libtcod.yellow, 'Lightning Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)
                ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            if not any([entity for entity in entities if entity.x == x and entity.y == y]):
                item_choice = random_choice_from_dict(item_chances)

                if item_choice == 'healing_potion':
                    <span class="crossed-out-text">item_component = Item(use_function=heal, amount=4)</span>
                    <span class="new-text">item_component = Item(use_function=heal, amount=40)</span>
                    item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
                                  item=item_component)
                elif item_choice == 'fireball_scroll':
                    <span class="crossed-out-text">item_component = Item(use_function=cast_fireball, targeting=True, targeting_message=Message(</span>
                        <span class="crossed-out-text">'Left-click a target tile for the fireball, or right-click to cancel.', libtcod.light_cyan),</span>
                                          <span class="crossed-out-text">damage=12, radius=3)</span>
                    <span class="new-text">item_component = Item(use_function=cast_fireball, targeting=True, targeting_message=Message(
                        'Left-click a target tile for the fireball, or right-click to cancel.', libtcod.light_cyan),
                                          damage=25, radius=3)</span>
                    item = Entity(x, y, '#', libtcod.red, 'Fireball Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)
                elif item_choice == 'confusion_scroll':
                    item_component = Item(use_function=cast_confuse, targeting=True, targeting_message=Message(
                        'Left-click an enemy to confuse it, or right-click to cancel.', libtcod.light_cyan))
                    item = Entity(x, y, '#', libtcod.light_pink, 'Confusion Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)
                else:
                    <span class="crossed-out-text">item_component = Item(use_function=cast_lightning, damage=20, maximum_range=5)</span>
                    <span class="new-text">item_component = Item(use_function=cast_lightning, damage=40, maximum_range=5)</span>
                    item = Entity(x, y, '#', libtcod.yellow, 'Lightning Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)
                ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Et voilà pour aujourd'hui. La prochaine partie sera la dernière, nous y sommes
presque.

Si vous voulez voir le code actuel entièrement, [cliquez ici](https://github.com/TStand90/roguelike_tutorial_revised/tree/part-12.

[Cliquez ici pour vous rendre à la partie suivante de ce tutoriel.](/tutorials/tcod/part-13)

<script src="/js/codetabs.js"></script>
