---
title: "Part 13 - S'équiper"
date: 2019-03-30T09:34:10-07:00
draft: false
---
Pour la dernière partie de notre tutoriel nous allons ajouter de l'équipement.
L'équipement est un genre d'objet qui le joueur peut porter pour améliorer ses
statistiques. De toute évidence cela peut-être plus complexe, selon le jeu, mais
je vous laisse le soin de l'implémenter selon vos besoins. Pour ce tutoriel,
équiper une arme améliore la puissance d'attaque et équiper un bouclier
améliore la défense.

Vous l'avez peut-être deviné, nous avons besoin d'un nouveau composant qui
nous indique quels objets sont équipables et quels effets ils procurent.
Ce composant s'appelera `Equippable` et nous l'incluerons à un fichier
appelé `equippable.py` qui ira dans le dossier `components`.

{{< highlight py3 >}}
class Equippable:
    def __init__(self, slot, power_bonus=0, defense_bonus=0, max_hp_bonus=0):
        self.slot = slot
        self.power_bonus = power_bonus
        self.defense_bonus = defense_bonus
        self.max_hp_bonus = max_hp_bonus
{{</ highlight >}}

`power_bonus`, `defense_bonus` et `max_hp_bonus` sont les gains que le joueur
obtient en portant certains objets. Une arme donne un bonus d'attaque et un
boucle donne un bonus de défense. Nous n'inclurons aucun objet donnant un
de points de vie mais vous pouvez ajouter une armure ou une bague qui améliore
la santé.

Qu'est-ce que `slot` ? Cela décrit l'endroit où sera porté l'objet. Le joueur
aura deux emplacements différents disponibles : la main principale pour l'arme
et la main secondaire pour le bouclier. Nous implémenterons ça à l'aide d'un
`enum`. Créez un fichier appelé `equipment_slots.py` dans le dossier de base
et ajoutez-y :

{{< highlight py3 >}}
from enum import Enum


class EquipmentSlots(Enum):
    MAIN_HAND = 1
    OFF_HAND = 2
{{</ highlight >}}

Vous pouvez étendre ça autant que vous le souhaitez en donnant au joueur
des emplacements comme la tête, le corps, les jambes et les doigts.

On a maintenant ce qu'il nous faut pour rendre les objets équipables. Mais
que va-t-on porter ? Pour ça il nous faut un autre composant appelé `Equipment`.
Ajoutez la suite à un nouveau fichier du dossier `components` appelé
`equipment.py` :

{{< highlight py3 >}}
from equipment_slots import EquipmentSlots


class Equipment:
    def __init__(self, main_hand=None, off_hand=None):
        self.main_hand = main_hand
        self.off_hand = off_hand

    @property
    def max_hp_bonus(self):
        bonus = 0

        if self.main_hand and self.main_hand.equippable:
            bonus += self.main_hand.equippable.max_hp_bonus

        if self.off_hand and self.off_hand.equippable:
            bonus += self.off_hand.equippable.max_hp_bonus

        return bonus

    @property
    def power_bonus(self):
        bonus = 0

        if self.main_hand and self.main_hand.equippable:
            bonus += self.main_hand.equippable.power_bonus

        if self.off_hand and self.off_hand.equippable:
            bonus += self.off_hand.equippable.power_bonus

        return bonus

    @property
    def defense_bonus(self):
        bonus = 0

        if self.main_hand and self.main_hand.equippable:
            bonus += self.main_hand.equippable.defense_bonus

        if self.off_hand and self.off_hand.equippable:
            bonus += self.off_hand.equippable.defense_bonus

        return bonus

    def toggle_equip(self, equippable_entity):
        results = []

        slot = equippable_entity.equippable.slot

        if slot == EquipmentSlots.MAIN_HAND:
            if self.main_hand == equippable_entity:
                self.main_hand = None
                results.append({'dequipped': equippable_entity})
            else:
                if self.main_hand:
                    results.append({'dequipped': self.main_hand})

                self.main_hand = equippable_entity
                results.append({'equipped': equippable_entity})
        elif slot == EquipmentSlots.OFF_HAND:
            if self.off_hand == equippable_entity:
                self.off_hand = None
                results.append({'dequipped': equippable_entity})
            else:
                if self.off_hand:
                    results.append({'dequipped': self.off_hand})

                self.off_hand = equippable_entity
                results.append({'equipped': equippable_entity})

        return results
{{</ highlight >}}

Cela fait beaucoup de code d'un coup aussi nous allons le découper un peu.

Les deux variables `main_hand` et `off_hand` vont contenir les entités qui
sont équipées. Si elles pointes vers `None`, cela signifie que rien n'est
porté dans cet emplacement.

Les trois propriétés font toutes sensiblement la même chose : elles ajoutent
les "bonus" de la main principale et de la main secondaire et renvoie la valeur.
Comme nous employons des propriétés on peut y accéder comme à une variable
ce qui sera rapidement commode. Si le joueur porte un objet en main principale
et en main secondaire qui améliore l'attaque, par exemple, on récupère le bonus
de la même manière.

`toggle_equip` (basculer l'équipement) est ce qu'on appelle quand on met
ou qu'on enlève un objet dans l'emplacement. Si l'objet n'était pas déjà
porté, on l'équipe et s'il était déjà équipé on l'enlève. On renvoie le
résultat de cette opération comme précédemment et c'est `engine` qui va
se charger de la suite du procédé.

Comme pour les autres composants crées, nous aurons besoin d'ajouter ces
nouveaux à la classe `Entity`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

import math

+from components.item import Item

from render_functions import RenderOrder


class Entity:
    def __init__(self, x, y, char, color, name, blocks=False, render_order=RenderOrder.CORPSE, fighter=None, ai=None,
-                item=None, inventory=None, stairs=None, level=None):
+                item=None, inventory=None, stairs=None, level=None, equipment=None, equippable=None):
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
        self.level = level
+       self.equipment = equipment
+       self.equippable = equippable

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

        if self.level:
            self.level.owner = self

+       if self.equipment:
+           self.equipment.owner = self
+
+       if self.equippable:
+           self.equippable.owner = self
+
+           if not self.item:
+               item = Item()
+               self.item = item
+               self.item.owner = self

    def move(self, dx, dy):
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

import math

<span class="new-text">from components.item import Item</span>

from render_functions import RenderOrder


class Entity:
    def __init__(self, x, y, char, color, name, blocks=False, render_order=RenderOrder.CORPSE, fighter=None, ai=None,
                 item=None, inventory=None, stairs=None, level=None<span class="new-text">, equipment=None, equippable=None</span>):
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
        self.level = level
        <span class="new-text">self.equipment = equipment
        self.equippable = equippable</span>

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

        if self.level:
            self.level.owner = self

        <span class="new-text">if self.equipment:
            self.equipment.owner = self

        if self.equippable:
            self.equippable.owner = self

            if not self.item:
                item = Item()
                self.item = item
                self.item.owner = self</span>

    def move(self, dx, dy):
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Remarquez que si l'entité n'a pas de composant `Item` on lui en ajoute un.
Cela s'explique car chaque pièce d'équipement est aussi un objet qu'on ajoute
à l'inventaire, ramasse et dépose.

Ajoutons un nouveau composant `Equipment` au joueur dans
`initialize_new_game.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    level_component = Level()
+   equipment_component = Equipment()
-   player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR,
-                   fighter=fighter_component, inventory=inventory_component, level=level_component)
+   player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR,
+                   fighter=fighter_component, inventory=inventory_component, level=level_component,
+                   equipment=equipment_component)
    entities = [player]
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    level_component = Level()
    <span class="new-text">equipment_component = Equipment()</span>
    <span class="crossed-out-text">player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR,</span>
                    <span class="crossed-out-text">fighter=fighter_component, inventory=inventory_component, level=level_component)</span>
    <span class="new-text">player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR,
                    fighter=fighter_component, inventory=inventory_component, level=level_component,
                    equipment=equipment_component)</span>
    entities = [player]
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Assurez-vous d'importer le composant dans le fichier :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

+from components.equipment import Equipment
from components.fighter import Fighter
from components.inventory import Inventory
from components.level import Level
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

<span class="new-text">from components.equipment import Equipment</span>
from components.fighter import Fighter
from components.inventory import Inventory
from components.level import Level
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Aussi, comment le joueur procède-t-il pour porter un objet ? L'équipement sera
sivible depuis l'écran d'inventaire comme n'importe quel objet utilisable aussi
pourquoi ne pas étendre cette fonctionnalité ? Nous pouvons modifier la méthode
`use` de `Inventory` pour équiper un objet si c'est possible :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        if item_component.use_function is None:
-           results.append({'message': Message('The {0} cannot be used'.format(item_entity.name), libtcod.yellow)})
+           equippable_component = item_entity.equippable
+
+           if equippable_component:
+               results.append({'equip': item_entity})
+           else:
+               results.append({'message': Message('The {0} cannot be used'.format(item_entity.name), libtcod.yellow)})
        else:
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        if item_component.use_function is None:
            <span class="crossed-out-text">results.append({'message': Message('The {0} cannot be used'.format(item_entity.name), libtcod.yellow)})</span>
            <span class="new-text">equippable_component = item_entity.equippable

            if equippable_component:
                results.append({'equip': item_entity})
            else:
                results.append({'message': Message('The {0} cannot be used'.format(item_entity.name), libtcod.yellow)})</span>
        else:
            ...
</pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant la méthode vérifie si l'objet est équipable et, si c'est le cas,
nous renvoyons le résultat. Sinon, nous affichons un avertissement nous
indiquant que c'est impossible, comme d'habitude.

Mettons la méthode `toggle_equip` en action. Dans `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            ...
            item_dropped = player_turn_result.get('item_dropped')
+           equip = player_turn_result.get('equip')
            targeting = player_turn_result.get('targeting')
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            item_dropped = player_turn_result.get('item_dropped')
            <span class="new-text">equip = player_turn_result.get('equip')</span>
            targeting = player_turn_result.get('targeting')
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            ...
            if item_dropped:
                entities.append(item_dropped)

                game_state = GameStates.ENEMY_TURN

+           if equip:
+               equip_results = player.equipment.toggle_equip(equip)
+
+               for equip_result in equip_results:
+                   equipped = equip_result.get('equipped')
+                   dequipped = equip_result.get('dequipped')
+
+                   if equipped:
+                       message_log.add_message(Message('You equipped the {0}'.format(equipped.name)))
+
+                   if dequipped:
+                       message_log.add_message(Message('You dequipped the {0}'.format(dequipped.name)))
+
+               game_state = GameStates.ENEMY_TURN

            if targeting:
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            if item_dropped:
                entities.append(item_dropped)

                game_state = GameStates.ENEMY_TURN

            <span class="new-text">if equip:
                equip_results = player.equipment.toggle_equip(equip)

                for equip_result in equip_results:
                    equipped = equip_result.get('equipped')
                    dequipped = equip_result.get('dequipped')

                    if equipped:
                        message_log.add_message(Message('You equipped the {0}'.format(equipped.name)))

                    if dequipped:
                        message_log.add_message(Message('You dequipped the {0}'.format(dequipped.name)))

                game_state = GameStates.ENEMY_TURN</span>

            if targeting:
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Il y a un petit bug dans notre implémentation. Le joueur peut déposer un objet
depuis l'inventaire mais continuer à le porter sur lui \! Ce n'est pas correct
aussi nous allons résoudre ce problème dans la méthode `drop_item` de
`Inventory` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
   ...
    def drop_item(self, item):
        results = []

+       if self.owner.equipment.main_hand == item or self.owner.equipment.off_hand == item:
+           self.owner.equipment.toggle_equip(item)

        item.x = self.owner.x
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>   ...
    def drop_item(self, item):
        results = []

        <span class="new-text">if self.owner.equipment.main_hand == item or self.owner.equipment.off_hand == item:
            self.owner.equipment.toggle_equip(item)</span>

        item.x = self.owner.x
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Mais qu'est ce qu'équiper un objet *fait* réellement ? Cela *devrait* donner un
bonus au joueur mais cela ne fait rien pour l'instant. Pourquoi ? Parce que
notre composant `FIghter` ne tient pas compte du bonus d'équipement pour
l'instant. Réglons ça :

Nous devons ajuster la manière d'obtenir les valeurs de `Fighter`. Il serait
plus judicieux que `max_hp`, `power` et `defense` soient des propriétés
afin qu'on puisse les calculer comme une base ajouté à un bonus.
Changeons la fonction d'initialisation pour régler les bases de chacune de
ces valeurs et nous ajouterons les propriétés à chacun pour remplacer nos
anciennes variables.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class Fighter:
    def __init__(self, hp, defense, power, xp=0):
-       self.max_hp = hp
+       self.base_max_hp = hp
        self.hp = hp
-       self.defense = defense
+       self.base_defense = defense
-       self.power = power
+       self.base_power = power
        self.xp = xp

+   @property
+   def max_hp(self):
+       if self.owner and self.owner.equipment:
+           bonus = self.owner.equipment.max_hp_bonus
+       else:
+           bonus = 0
+
+       return self.base_max_hp + bonus
+
+   @property
+   def power(self):
+       if self.owner and self.owner.equipment:
+           bonus = self.owner.equipment.power_bonus
+       else:
+           bonus = 0
+
+       return self.base_power + bonus
+
+   @property
+   def defense(self):
+       if self.owner and self.owner.equipment:
+           bonus = self.owner.equipment.defense_bonus
+       else:
+           bonus = 0
+
+       return self.base_defense + bonus

    def take_damage(self, amount):
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class Fighter:
    def __init__(self, hp, defense, power, xp=0):
        <span class="crossed-out-text">self.max_hp = hp</span>
        <span class="new-text">self.base_max_hp = hp</span>
        self.hp = hp
        <span class="crossed-out-text">self.defense = defense</span>
        <span class="new-text">self.base_defense = defense</span>
        <span class="crossed-out-text">self.power = power</span>
        <span class="new-text">self.base_power = power</span>
        self.xp = xp

    <span class="new-text">@property
    def max_hp(self):
        if self.owner and self.owner.equipment:
            bonus = self.owner.equipment.max_hp_bonus
        else:
            bonus = 0

        return self.base_max_hp + bonus

    @property
    def power(self):
        if self.owner and self.owner.equipment:
            bonus = self.owner.equipment.power_bonus
        else:
            bonus = 0

        return self.base_power + bonus

    @property
    def defense(self):
        if self.owner and self.owner.equipment:
            bonus = self.owner.equipment.defense_bonus
        else:
            bonus = 0

        return self.base_defense + bonus</span>

    def take_damage(self, amount):
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Ainsi maintenant quand nous nous relevons la valeur de `power`, par exemple,
nous prendrons en compte l'équipement du joueur.

Dans la plupart des cas cela suffit. La seule chose qui pose encore problème
dans le code précédent est le gain d'un niveau parce qu'on nous augmentons
directement les valeurs de `max_hp`, `defense` et `power` alors qu'on devrait
augmenter leurs valeurs de base. C'est plutôt simple à résoudre, ouvrez
`engine.py` et apportez les modifications suivantes :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        if level_up:
            if level_up == 'hp':
-               player.fighter.max_hp += 20
+               player.fighter.base_max_hp += 20
                player.fighter.hp += 20
            elif level_up == 'str':
-               player.fighter.power += 1
+               player.fighter.base_power += 1
            elif level_up == 'def':
-               player.fighter.defense += 1
+               player.fighter.base_defense += 1
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        if level_up:
            if level_up == 'hp':
                <span class="crossed-out-text">player.fighter.max_hp += 20</span>
                <span class="new-text">player.fighter.base_max_hp += 20</span>
                player.fighter.hp += 20
            elif level_up == 'str':
                <span class="crossed-out-text">player.fighter.power += 1</span>
                <span class="new-text">player.fighter.base_power += 1</span>
            elif level_up == 'def':
                <span class="crossed-out-text">player.fighter.defense += 1</span>
                <span class="new-text">player.fighter.base_defense += 1</span>
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Ceci étant en place, mettons de l'équipement sur la carte \! Ouvrez
`game_map.py` et modifiez la fonction `place_entities` pour ajouter de
l'équipement dans la donjon. Souvenvez-vous d'importer les composants
nécessaire en haut.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from components.ai import BasicMonster
+from components.equipment import EquipmentSlots
+from components.equippable import Equippable
from components.fighter import Fighter
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from components.ai import BasicMonster
<span class="new-text">from components.equipment import EquipmentSlots
from components.equippable import Equippable</span>
from components.fighter import Fighter
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
       item_chances = {
            'healing_potion': 35,
+           'sword': from_dungeon_level([[5, 4]], self.dungeon_level),
+           'shield': from_dungeon_level([[15, 8]], self.dungeon_level),
            'lightning_scroll': from_dungeon_level([[25, 4]], self.dungeon_level),
            'fireball_scroll': from_dungeon_level([[25, 6]], self.dungeon_level),
            'confusion_scroll': from_dungeon_level([[10, 2]], self.dungeon_level)
        }
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>       item_chances = {
            'healing_potion': 35,
            <span class="new-text">'sword': from_dungeon_level([[5, 4]], self.dungeon_level),
            'shield': from_dungeon_level([[15, 8]], self.dungeon_level),</span>
            'lightning_scroll': from_dungeon_level([[25, 4]], self.dungeon_level),
            'fireball_scroll': from_dungeon_level([[25, 6]], self.dungeon_level),
            'confusion_scroll': from_dungeon_level([[10, 2]], self.dungeon_level)
        }</pre>
{{</ original-tab >}}
{{</ codetab >}}

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
               ...
                if item_choice == 'healing_potion':
                    item_component = Item(use_function=heal, amount=40)
                    item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
                                  item=item_component)
+               elif item_choice == 'sword':
+                   equippable_component = Equippable(EquipmentSlots.MAIN_HAND, power_bonus=3)
+                   item = Entity(x, y, '/', libtcod.sky, 'Sword', equippable=equippable_component)
+               elif item_choice == 'shield':
+                   equippable_component = Equippable(EquipmentSlots.OFF_HAND, defense_bonus=1)
+                   item = Entity(x, y, '[', libtcod.darker_orange, 'Shield', equippable=equippable_component)
                elif item_choice == 'fireball_scroll':
                    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>               ...
                if item_choice == 'healing_potion':
                    item_component = Item(use_function=heal, amount=40)
                    item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
                                  item=item_component)
                <span class="new-text">elif item_choice == 'sword':
                    equippable_component = Equippable(EquipmentSlots.MAIN_HAND, power_bonus=3)
                    item = Entity(x, y, '/', libtcod.sky, 'Sword', equippable=equippable_component)
                elif item_choice == 'shield':
                    equippable_component = Equippable(EquipmentSlots.OFF_HAND, defense_bonus=1)
                    item = Entity(x, y, '[', libtcod.darker_orange, 'Shield', equippable=equippable_component)</span>
                elif item_choice == 'fireball_scroll':
                    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Une chose qu'on peut faire pour rendre le jeu plus intéressant est de donner
une arme de base au joueur. Rien de trop puissant, bien sûr, c'est un
roguelike après tout. Modifions la fonction `get_game_variables` dans
`initialize_new_game.py` pour donne au joueur une dague au début de partie.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

from components.equipment import Equipment
+from components.equippable import Equippable
from components.fighter import Fighter
from components.inventory import Inventory
from components.level import Level

from entity import Entity

+from equipment_slots import EquipmentSlots

from game_messages import MessageLog
...

def get_game_variables(constants):
-   fighter_component = Fighter(hp=100, defense=1, power=4)
+   fighter_component = Fighter(hp=100, defense=1, power=2)
    inventory_component = Inventory(26)
    level_component = Level()
    equipment_component = Equipment()
    player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR,
                    fighter=fighter_component, inventory=inventory_component, level=level_component,
                    equipment=equipment_component)
    entities = [player]

+   equippable_component = Equippable(EquipmentSlots.MAIN_HAND, power_bonus=2)
+   dagger = Entity(0, 0, '-', libtcod.sky, 'Dagger', equippable=equippable_component)
+   player.inventory.add_item(dagger)
+   player.equipment.toggle_equip(dagger)

    game_map = GameMap(constants['map_width'], constants['map_height'])
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

from components.equipment import Equipment
<span class="new-text">from components.equippable import Equippable</span>
from components.fighter import Fighter
from components.inventory import Inventory
from components.level import Level

from entity import Entity

<span class="new-text">from equipment_slots import EquipmentSlots</span>

from game_messages import MessageLog
...

def get_game_variables(constants):
    <span class="crossed-out-text">fighter_component = Fighter(hp=100, defense=1, power=4)</span>
    <span class="new-text">fighter_component = Fighter(hp=100, defense=1, power=2)</span>
    inventory_component = Inventory(26)
    level_component = Level()
    equipment_component = Equipment()
    player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR,
                    fighter=fighter_component, inventory=inventory_component, level=level_component,
                    equipment=equipment_component)
    entities = [player]

    <span class="new-text">equippable_component = Equippable(EquipmentSlots.MAIN_HAND, power_bonus=2)
    dagger = Entity(0, 0, '-', libtcod.sky, 'Dagger', equippable=equippable_component)
    player.inventory.add_item(dagger)
    player.equipment.toggle_equip(dagger)</span>

    game_map = GameMap(constants['map_width'], constants['map_height'])
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Remarquez qu'on modifie aussi la puissance de départ. Nous ne voulons pas que
le joueur soit trop fort au départ \!

Une dernière amélioration à ajouter : affichons dans l'inventaire l'équipement
porté par le joueur. Nous pouvons le faire en modifiant la fonction
`inventory_menu` dans `menus.py` pour vérifier si chaque objet est équipé ou
non. Nous devrons faire quelques chanchements dans les arguments de la fonction,
nous devons passer `player` et non plus seulement l'inventaire. Modifiez la
fonction ainsi :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-def inventory_menu(con, header, inventory, inventory_width, screen_width, screen_height):
+def inventory_menu(con, header, player, inventory_width, screen_width, screen_height):
-   if len(inventory.items) == 0:
+   if len(player.inventory.items) == 0:
        options = ['Inventory is empty.']
    else:
-       options = [item.name for item in inventory.items]
+       options = []
+
+       for item in player.inventory.items:
+           if player.equipment.main_hand == item:
+               options.append('{0} (on main hand)'.format(item.name))
+           elif player.equipment.off_hand == item:
+               options.append('{0} (on off hand)'.format(item.name))
+           else:
+               options.append(item.name)

    menu(con, header, options, inventory_width, screen_width, screen_height)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="crossed-out-text">def inventory_menu(con, header, inventory, inventory_width, screen_width, screen_height):</span>
<span class="new-text">def inventory_menu(con, header, player, inventory_width, screen_width, screen_height):</span>
    <span class="crossed-out-text">if len(inventory.items) == 0:</span>
    <span class="new-text">if len(player.inventory.items) == 0:</span>
        options = ['Inventory is empty.']
    else:
        <span class="crossed-out-text">options = [item.name for item in inventory.items]</span>
        <span class="new-text">options = []

        for item in player.inventory.items:
            if player.equipment.main_hand == item:
                options.append('{0} (on main hand)'.format(item.name))
            elif player.equipment.off_hand == item:
                options.append('{0} (on off hand)'.format(item.name))
            else:
                options.append(item.name)</span>

    menu(con, header, options, inventory_width, screen_width, screen_height)</pre>
{{</ original-tab >}}
{{</ codetab >}}

Parce que nous avons changé les arguments de cette fonction nous devons
adapter son appel dans `render_all`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
            inventory_title = 'Press the key next to an item to drop it, or Esc to cancel.\n'

-       inventory_menu(con, inventory_title, player.inventory, 50, screen_width, screen_height)
+       inventory_menu(con, inventory_title, player, 50, screen_width, screen_height)

    elif game_state == GameStates.LEVEL_UP:
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>
        ...
            inventory_title = 'Press the key next to an item to drop it, or Esc to cancel.\n'

        <span class="crossed-out-text">inventory_menu(con, inventory_title, player.inventory, 50, screen_width, screen_height)</span>
        <span class="new-text">inventory_menu(con, inventory_title, player, 50, screen_width, screen_height)</span>

    elif game_state == GameStates.LEVEL_UP:
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

With that, we have a functional equipment system\! This concludes the
main tutorial. If you're wanting more, feel free to check out the extras
section, which I'll try to update now and then with some new content.
Now go forth, and create the roguelike of your dreams\!

Avec ça, nous avons un système d'équiment qui fonctionne \! Cela conclut
le tutoriel principal. Si vous en voulez davantage, n'hésitez pas à vérifier
les extras que j'essayerai de mettre à jour de temps en temps.
Maintenant continuez sur votre lancée et créez le roguelike de vos rêves.

Si vous voulez voir le code actuel entièrement, [cliquez ici](https://github.com/TStand90/roguelike_tutorial_revised/tree/part-13.

<script src="/js/codetabs.js"></script>
