---
title: "Part 9 - Lancer des sorts"
date: 2019-03-30T09:34:01-07:00
draft: false
---

Ajouter des potions de soin fut une grande avancée mais nous ne nous arrêterons
pas là. Continuons maintenant avec quelques objets offensifs. Nous allons
ajouter quelques parchemins qui donneront au joueur une attaque à distance à
usage unique. Cela permet plus d'options tactiques ce qui est une direction
que vous devez chercher à améliorer dans un jeu.

Commençons simplement par un sort qui frappe l'ennemi le plus proche. Nous
allons créer un simple parchemin d'éclair (scoll of lightning) qui va viser
automatiquement l'adversaire le plus proche. Commencez par ajouter la fonction
à `item_functions.py`

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def heal(*args, **kwargs):
    ...

+def cast_lightning(*args, **kwargs):
+   caster = args[0]
+   entities = kwargs.get('entities')
+   fov_map = kwargs.get('fov_map')
+   damage = kwargs.get('damage')
+   maximum_range = kwargs.get('maximum_range')
+
+   results = []
+
+   target = None
+   closest_distance = maximum_range + 1
+
+   for entity in entities:
+       if entity.fighter and entity != caster and libtcod.map_is_in_fov(fov_map, entity.x, entity.y):
+           distance = caster.distance_to(entity)
+
+           if distance < closest_distance:
+               target = entity
+               closest_distance = distance
+
+   if target:
+       results.append({'consumed': True, 'target': target, 'message': Message('A lighting bolt strikes the {0} with a loud thunder! The damage is {1}'.format(target.name, damage))})
+       results.extend(target.fighter.take_damage(damage))
+   else:
+       results.append({'consumed': False, 'target': None, 'message': Message('No enemy is close enough to strike.', libtcod.red)})
+
+   return results
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def heal(*args, **kwargs):
    ...

<span class="new-text">def cast_lightning(*args, **kwargs):
    caster = args[0]
    entities = kwargs.get('entities')
    fov_map = kwargs.get('fov_map')
    damage = kwargs.get('damage')
    maximum_range = kwargs.get('maximum_range')

    results = []

    target = None
    closest_distance = maximum_range + 1

    for entity in entities:
        if entity.fighter and entity != caster and libtcod.map_is_in_fov(fov_map, entity.x, entity.y):
            distance = caster.distance_to(entity)

            if distance < closest_distance:
                target = entity
                closest_distance = distance

    if target:
        results.append({'consumed': True, 'target': target, 'message': Message('A lighting bolt strikes the {0} with a loud thunder! The damage is {1}'.format(target.name, damage))})
        results.extend(target.fighter.take_damage(damage))
    else:
        results.append({'consumed': False, 'target': None, 'message': Message('No enemy is close enough to strike.', libtcod.red)})

    return results</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant nous devons en déposer quelqu'uns uns sur la carte. La plupart des
items seront des potions de soin mais nous allons ajouter quelques parchemins
d'éclair. Dans `game_map.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            ...
            if not any([entity for entity in entities if entity.x == x and entity.y == y]):
+               item_chance = randint(0, 100)
-               item_component = Item(use_function=heal, amount=4)
-               item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
-                              item=item_component)
+
+               if item_chance < 70:
+                   item_component = Item(use_function=heal, amount=4)
+                   item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
+                                 item=item_component)
+               else:
+                   item_component = Item(use_function=cast_lightning, damage=20, maximum_range=5)
+                   item = Entity(x, y, '#', libtcod.yellow, 'Lightning Scroll', render_order=RenderOrder.ITEM,
+                                 item=item_component)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            if not any([entity for entity in entities if entity.x == x and entity.y == y]):
                <span class="new-text">item_chance = randint(0, 100)

                if item_chance < 70:</span>
                    <span style="color: blue">item_component = Item(use_function=heal, amount=4)
                    item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
                                  item=item_component)</span>
                <span class="new-text">else:
                    item_component = Item(use_function=cast_lightning, damage=20, maximum_range=5)
                    item = Entity(x, y, '#', libtcod.yellow, 'Lightning Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Assurez-vous d'importer `cast_lightning` en haut du fichier.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from entity import Entity

-from item_functions import heal
+from item_functions import cast_lightning, heal

from map_objects.rectangle import Rect
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from entity import Entity

from item_functions import <span class="new-text">cast_lightning,</span> heal

from map_objects.rectangle import Rect
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Enfin, nous allons ajuster notre appel à "utiliser" dans `engine.py`, notre
parchemin d'éclair ayant besoin de davantage d'arguments qu'on n'en passe
actuellement.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            ...
            if game_state == GameStates.SHOW_INVENTORY:
-               player_turn_results.extend(player.inventory.use(item))
+               player_turn_results.extend(player.inventory.use(item, entities=entities, fov_map=fov_map))
            elif game_state == GameStates.DROP_INVENTORY:
                player_turn_results.extend(player.inventory.drop_item(item))
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            if game_state == GameStates.SHOW_INVENTORY:
                <span class="crossed-out-text">player_turn_results.extend(player.inventory.use(item))</span>
                <span class="new-text">player_turn_results.extend(player.inventory.use(item, entities=entities, fov_map=fov_map))</span>
            elif game_state == GameStates.DROP_INVENTORY:
                player_turn_results.extend(player.inventory.drop_item(item))</pre>
{{</ original-tab >}}
{{</ codetab >}}

Lancez le projet et vous devriez avoir un parchemin d'éclair fonctionnel.
C'était plutôt simple \!

*\*Conseil : pour tester vous pouvez augmenter le nombre maximal d'items par
pièce.*

Inutile de le dire, le sort serait bien plus pratique si on pouvait choisir sa
cible. Nous n'allons pas changer le sort d'éclair mais plutôt ajouter un autre
type de sort qui permette de viser. Concentrons nous sur une boule de feu qui
permette non seulement de viser mais aussi de toucher plusieurs ennemis dans
un rayon donné.

Nous allons travailler dans l'autre sens cette fois, partant d'un sort de boule
de feu "fireball" nous allons modifier tout le reste pour le faire fonctionner.
Voici le sort de boule de feu qui va dans `item_functions.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
def cast_lightning(*args, **kwargs):
    ...

+def cast_fireball(*args, **kwargs):
+   entities = kwargs.get('entities')
+   fov_map = kwargs.get('fov_map')
+   damage = kwargs.get('damage')
+   radius = kwargs.get('radius')
+   target_x = kwargs.get('target_x')
+   target_y = kwargs.get('target_y')
+
+   results = []
+
+   if not libtcod.map_is_in_fov(fov_map, target_x, target_y):
+       results.append({'consumed': False, 'message': Message('You cannot target a tile outside your field of view.', libtcod.yellow)})
+       return results
+
+   results.append({'consumed': True, 'message': Message('The fireball explodes, burning everything within {0} tiles!'.format(radius), libtcod.orange)})
+
+   for entity in entities:
+       if entity.distance(target_x, target_y) <= radius and entity.fighter:
+           results.append({'message': Message('The {0} gets burned for {1} hit points.'.format(entity.name, damage), libtcod.orange)})
+           results.extend(entity.fighter.take_damage(damage))
+
+   return results
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
def cast_lightning(*args, **kwargs):
    ...

<span class="new-text">def cast_fireball(*args, **kwargs):
    entities = kwargs.get('entities')
    fov_map = kwargs.get('fov_map')
    damage = kwargs.get('damage')
    radius = kwargs.get('radius')
    target_x = kwargs.get('target_x')
    target_y = kwargs.get('target_y')

    results = []

    if not libtcod.map_is_in_fov(fov_map, target_x, target_y):
        results.append({'consumed': False, 'message': Message('You cannot target a tile outside your field of view.', libtcod.yellow)})
        return results

    results.append({'consumed': True, 'message': Message('The fireball explodes, burning everything within {0} tiles!'.format(radius), libtcod.orange)})

    for entity in entities:
        if entity.distance(target_x, target_y) <= radius and entity.fighter:
            results.append({'message': Message('The {0} gets burned for {1} hit points.'.format(entity.name, damage), libtcod.orange)})
            results.extend(entity.fighter.take_damage(damage))

    return results</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Que devons-nous changer pour faire marcher cette fonction ? La manière évidente
est de passer les dégâts, le rayon et la position de la cible. Dégâts et rayon
sont simples, on peut le faire quand on crée l'objet dans `place_entities`. La
visée est plus délicate, on n'en connaît rien tant que le joueur n'a pas choisi
une tuile après avoir utilisé l'objet.

Nous allons avoir besoin d'un autre état du jeu pour la visée. Quand le joueur
choisit un certain type d'objet, le jeu va lui demander de choisir une position
avant de continuer. Le joueur peut alors cliquer sur une position et faire un
clic droit pour annuler. Nous aurons donc besoin d'un nouveau gestionnaire de
saisie.

Commençez par la partie facile : ajouter un nouvel état ) `GameStates` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class GameStates(Enum):
    PLAYERS_TURN = 1
    ENEMY_TURN = 2
    PLAYER_DEAD = 3
    SHOW_INVENTORY = 4
    DROP_INVENTORY = 5
+   TARGETING = 6
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class GameStates(Enum):
    PLAYERS_TURN = 1
    ENEMY_TURN = 2
    PLAYER_DEAD = 3
    SHOW_INVENTORY = 4
    DROP_INVENTORY = 5
    <span class="new-text">TARGETING = 6</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Maitenant modifiez le gestionnaire de saisie. Nous allons ajouter une fonction
pour les touches quand on vise ainsi qu'un gestionnaire de souris générique
pour savoir où vise le joueur.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def handle_keys(key, game_state):
    if game_state == GameStates.PLAYERS_TURN:
        return handle_player_turn_keys(key)
    elif game_state == GameStates.PLAYER_DEAD:
        return handle_player_dead_keys(key)
+   elif game_state == GameStates.TARGETING:
+       return handle_targeting_keys(key)
    elif game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
        return handle_inventory_keys(key)
    ...


+def handle_targeting_keys(key):
+   if key.vk == libtcod.KEY_ESCAPE:
+       return {'exit': True}
+
+   return {}

def handle_player_dead_keys(key):
    ...


+def handle_mouse(mouse):
+   (x, y) = (mouse.cx, mouse.cy)
+
+   if mouse.lbutton_pressed:
+       return {'left_click': (x, y)}
+   elif mouse.rbutton_pressed:
+       return {'right_click': (x, y)}
+
+   return {}
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def handle_keys(key, game_state):
    if game_state == GameStates.PLAYERS_TURN:
        return handle_player_turn_keys(key)
    elif game_state == GameStates.PLAYER_DEAD:
        return handle_player_dead_keys(key)
    <span class="new-text">elif game_state == GameStates.TARGETING:
        return handle_targeting_keys(key)</span>
    elif game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
        return handle_inventory_keys(key)
    ...


<span class="new-text">def handle_targeting_keys(key):
    if key.vk == libtcod.KEY_ESCAPE:
        return {'exit': True}

    return {}</span>

def handle_player_dead_keys(key):
    ...


<span class="new-text">def handle_mouse(mouse):
    (x, y) = (mouse.cx, mouse.cy)

    if mouse.lbutton_pressed:
        return {'left_click': (x, y)}
    elif mouse.rbutton_pressed:
        return {'right_click': (x, y)}

    return {}</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Si le joueur est en mode visée, la seule touche acceptée est Escape, ce qui
annule la visée. Le gestionnaire de souris ne tient pas compte de l'état du
jeu, il ne fait que dire au moteur si le bouton gauche ou droit a été cliqué.
Le moteur devra décider ce qu'il doit en faire.
Modifiez `engine.py` pour recevoir les événements souris.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        action = handle_keys(key, game_state)
+       mouse_action = handle_mouse(mouse)

        move = action.get('move')
        pickup = action.get('pickup')
        show_inventory = action.get('show_inventory')
        inventory_index = action.get('inventory_index')
        exit = action.get('exit')
        fullscreen = action.get('fullscreen')

+       left_click = mouse_action.get('left_click')
+       right_click = mouse_action.get('right_click')

        player_turn_results = []
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        action = handle_keys(key, game_state)
        <span class="new-text">mouse_action = handle_mouse(mouse)</span>

        move = action.get('move')
        pickup = action.get('pickup')
        show_inventory = action.get('show_inventory')
        inventory_index = action.get('inventory_index')
        exit = action.get('exit')
        fullscreen = action.get('fullscreen')

        <span class="new-text">left_click = mouse_action.get('left_click')
        right_click = mouse_action.get('right_click')</span>

        player_turn_results = []</pre>
{{</ original-tab >}}
{{</ codetab >}}

Bien sûr, nous devons importer `handle_mouse` dans `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from game_states import GameStates
-from input_handlers import handle_keys
+from input_handlers import handle_keys, handle_mouse
from map_objects.game_map import GameMap
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from game_states import GameStates
from input_handlers import handle_keys<span class="new-text">, handle_mouse</span>
from map_objects.game_map import GameMap
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Mais comment savons-nous qu'un certain item a besoin d'une visée ? Nous pouvons
ajouter un attribut au composant `Item` qui nous l'indiquera. Nous devrions
aussi ajouter un message, affiché quand l'utilisateur active l'objet, pour
l'informer qu'une cible doit être choisie. Modifiez la fonction `__init__`
depuis `Item` comme ceci :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class Item:
-   def __init__(self, use_function=None, **kwargs):
+   def __init__(self, use_function=None, targeting=False, targeting_message=None, **kwargs):
        self.use_function = use_function
+       self.targeting = targeting
+       self.targeting_message = targeting_message
        self.function_kwargs = kwargs
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class Item:
    def __init__(self, use_function=None, <span class="new-text">targeting=False, targeting_message=None,</span> **kwargs):
        self.use_function = use_function
        <span class="new-text">self.targeting = targeting
        self.targeting_message = targeting_message</span>
        self.function_kwargs = kwargs</pre>
{{</ original-tab >}}
{{</ codetab >}}

Parce que les valeurs de `targeting` et `targeting_message` sont None par
défaut, nous n'avons pas à nous soucier de modifier les objets que nous avons
déjà crée.

Nous devrons modifier notre fonction `use` depuis `Inventory` pour tenir compte
de la variable de visée. Si l'objet a besoin d'une cible, nous devrions renvoyer
un résultat qui l'indique au moteur plutôt que de consommer l'objet. Sinon, on
continue comme avant. Ajoutez une nouvelle expression "if" à `use` et entourez
le code précédent dans un bloc "else" ainsi :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    def use(self, item_entity, **kwargs):
        results = []

        item_component = item_entity.item

        if item_component.use_function is None:
            results.append({'message': Message('The {0} cannot be used'.format(item_entity.name), libtcod.yellow)})
        else:
-           kwargs = {**item_component.function_kwargs, **kwargs}
-           item_use_results = item_component.use_function(self.owner, **kwargs)

-           for item_use_result in item_use_results:
-               if item_use_result.get('consumed'):
-                   self.remove_item(item_entity)
-
-           results.extend(item_use_results)
+           if item_component.targeting and not (kwargs.get('target_x') or kwargs.get('target_y')):
+               results.append({'targeting': item_entity})
+           else:
+               kwargs = {**item_component.function_kwargs, **kwargs}
+               item_use_results = item_component.use_function(self.owner, **kwargs)
+
+               for item_use_result in item_use_results:
+                   if item_use_result.get('consumed'):
+                       self.remove_item(item_entity)
+
+                results.extend(item_use_results)

        return results
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    def use(self, item_entity, **kwargs):
        results = []

        item_component = item_entity.item

        if item_component.use_function is None:
            results.append({'message': Message('The {0} cannot be used'.format(item_entity.name), libtcod.yellow)})
        else:
            <span class="new-text">if item_component.targeting and not (kwargs.get('target_x') or kwargs.get('target_y')):
                results.append({'targeting': item_entity})
            else:</span>
                <span style="color: blue">kwargs = {**item_component.function_kwargs, **kwargs}
                item_use_results = item_component.use_function(self.owner, **kwargs)

                for item_use_result in item_use_results:
                    if item_use_result.get('consumed'):
                        self.remove_item(item_entity)

                results.extend(item_use_results)</span>

        return results</pre>
{{</ original-tab >}}
{{</ codetab >}}

Simplement, on vérifie si l'objet a "targetting" (visée) sur True et, si c'est
le cas, si nous avons reçu les variables `target_x` et `target_y`. Sinon, on
peut supposer que la cible n'a pas été choisie et l'état du jeu doit passer sur
visée. Si les variables sont reçues, on peut utiliser le sort normalement.

Maintenant, il faut modifier le moteur pour gérer ce nouveau type de sort.
Remarquez que le résultat renvoie une entité objet au moteur. C'est parce que
le moteur aura besoin de se souvenir de l'objet choisi. Aussi, nous aurons
besoin d'une nouvelle variable juste avant la boucle principale pour conserver
l'objet choisi s'il a une visée.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    game_state = GameStates.PLAYERS_TURN
    previous_game_state = game_state

+   targeting_item = None

    while not libtcod.console_is_window_closed():
        ...
            message = player_turn_result.get('message')
            dead_entity = player_turn_result.get('dead')
            item_added = player_turn_result.get('item_added')
            item_consumed = player_turn_result.get('consumed')
            item_dropped = player_turn_result.get('item_dropped')
+           targeting = player_turn_result.get('targeting')
            ...

            if item_consumed:
                game_state = GameStates.ENEMY_TURN

+           if targeting:
+               previous_game_state = GameStates.PLAYERS_TURN
+               game_state = GameStates.TARGETING
+
+               targeting_item = targeting
+
+               message_log.add_message(targeting_item.item.targeting_message)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    game_state = GameStates.PLAYERS_TURN
    previous_game_state = game_state

    <span class="new-text">targeting_item = None</span>

    while not libtcod.console_is_window_closed():
        ...
            message = player_turn_result.get('message')
            dead_entity = player_turn_result.get('dead')
            item_added = player_turn_result.get('item_added')
            item_consumed = player_turn_result.get('consumed')
            item_dropped = player_turn_result.get('item_dropped')
            <span class="new-text">targeting = player_turn_result.get('targeting')</span>
            ...

            if item_consumed:
                game_state = GameStates.ENEMY_TURN

            <span class="new-text">if targeting:
                previous_game_state = GameStates.PLAYERS_TURN
                game_state = GameStates.TARGETING

                targeting_item = targeting

                message_log.add_message(targeting_item.item.targeting_message)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant notre état de jeu va basculer sur visée quand on choisit un objet
de l'inventaire qui en nécessite une. Remarquez qu'on fait quelque chose
d'étrange avec le précédent état du jeu : on le règle sur le tour du joueur
plutôt que le précédent état du jeu. Cela évitera de réouvrir l'inventaire
quand on annule la visée.

Maintenant il faut faire quelque chose des clics gauche et droit ajoutés plus
tôt. Si le joueur fait un clic gauche, nous lançons la fonction "use" à nouveau
cette fois avec les variables de la cible. Si le joueur fait un clic droit, nous
annulons la visée. Nous pouvons aussi ajouter l'annulation de la visée à Escape.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        if inventory_index is not None and previous_game_state != GameStates.PLAYER_DEAD and inventory_index < len(
                player.inventory.items):
            ...

+       if game_state == GameStates.TARGETING:
+           if left_click:
+               target_x, target_y = left_click
+
+               item_use_results = player.inventory.use(targeting_item, entities=entities, fov_map=fov_map,
+                                                       target_x=target_x, target_y=target_y)
+               player_turn_results.extend(item_use_results)
+           elif right_click:
+               player_turn_results.append({'targeting_cancelled': True})

        if exit:
            if game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
                game_state = previous_game_state
+           elif game_state == GameStates.TARGETING:
+               player_turn_results.append({'targeting_cancelled': True})
            else:
                return True

        if fullscreen:
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        if inventory_index is not None and previous_game_state != GameStates.PLAYER_DEAD and inventory_index < len(
                player.inventory.items):
            ...

        <span class="new-text">if game_state == GameStates.TARGETING:
            if left_click:
                target_x, target_y = left_click

                item_use_results = player.inventory.use(targeting_item, entities=entities, fov_map=fov_map,
                                                        target_x=target_x, target_y=target_y)
                player_turn_results.extend(item_use_results)
            elif right_click:
                player_turn_results.append({'targeting_cancelled': True})</span>

        if exit:
            if game_state in (GameStates.SHOW_INVENTORY, GameStates.DROP_INVENTORY):
                game_state = previous_game_state
            <span class="new-text">elif game_state == GameStates.TARGETING:
                player_turn_results.append({'targeting_cancelled': True})</span>
            else:
                return True

        if fullscreen:
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Add the following to make the target cancellation revert the game state:

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            targeting = player_turn_result.get('targeting')
+           targeting_cancelled = player_turn_result.get('targeting_cancelled')

            if message:
                ...

+           if targeting_cancelled:
+               game_state = previous_game_state
+
+               message_log.add_message(Message('Targeting cancelled'))
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            targeting = player_turn_result.get('targeting')
            <span class="new-text">targeting_cancelled = player_turn_result.get('targeting_cancelled')</span>

            if message:
                ...

            <span class="new-text">if targeting_cancelled:
                game_state = previous_game_state

                message_log.add_message(Message('Targeting cancelled'))</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Enfin, ajoutons le parchemin de boule de feu sur la carte. Modifiez
`place_entities` comme ceci :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
                ...
                item_chance = randint(0, 100)

                if item_chance < 70:
                    item_component = Item(use_function=heal, amount=4)
                    item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
                                  item=item_component)
+               elif item_chance < 85:
+                   item_component = Item(use_function=cast_fireball, targeting=True, targeting_message=Message(
+                       'Left-click a target tile for the fireball, or right-click to cancel.', libtcod.light_cyan),
+                                         damage=12, radius=3)
+                   item = Entity(x, y, '#', libtcod.red, 'Fireball Scroll', render_order=RenderOrder.ITEM,
+                                 item=item_component)
                else:
                    item_component = Item(use_function=cast_lightning, damage=20, maximum_range=5)
                    item = Entity(x, y, '#', libtcod.yellow, 'Lightning Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                ...
                item_chance = randint(0, 100)

                if item_chance < 70:
                    item_component = Item(use_function=heal, amount=4)
                    item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
                                  item=item_component)
                <span class="new-text">elif item_chance < 85:
                    item_component = Item(use_function=cast_fireball, targeting=True, targeting_message=Message(
                        'Left-click a target tile for the fireball, or right-click to cancel.', libtcod.light_cyan),
                                          damage=12, radius=3)
                    item = Entity(x, y, '#', libtcod.red, 'Fireball Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)</span>
                else:
                    item_component = Item(use_function=cast_lightning, damage=20, maximum_range=5)
                    item = Entity(x, y, '#', libtcod.yellow, 'Lightning Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)</pre>
{{</ original-tab >}}
{{</ codetab >}}

Vous devez importer `cast_fireball` et `Message` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from entity import Entity

+from game_messages import Message

-from item_functions import cast_lightning, heal
+from item_functions import cast_fireball, cast_lightning, heal

from map_objects.rectangle import Rect
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from entity import Entity

<span class="new-text">from game_messages import Message</span>

from item_functions import <span class="new-text">cast_fireball,</span> cast_lightning, heal

from map_objects.rectangle import Rect
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Il faut encore un changement pour que `cast_fireball` puisse fonctionner :
nous avons besoin d'une fonction `distance` dans `Entity` pour obtenir la
distance entre une entité et un point arbitraire.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    def move_towards(self, target_x, target_y, game_map, entities):
        ...

+   def distance(self, x, y):
+       return math.sqrt((x - self.x) ** 2 + (y - self.y) ** 2)

    def distance_to(self, other):
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>
    def move_towards(self, target_x, target_y, game_map, entities):
        ...

    <span class="new-text">def distance(self, x, y):
        return math.sqrt((x - self.x) ** 2 + (y - self.y) ** 2)</span>

    def distance_to(self, other):
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Lancez le jeu et vous devriez avoir un sort de boule de feu qui fonctionne.
Soyez prudent, le joueur prend des dégâts si vous lancez le sort trop proche
de lui.

Ajoutons un dernier sort pour s'amuser : confusion. Cela va demander de changer
la visée de l'AI pour quelques tours et la rétablir à la normale une fois que
le sort est terminé.

Commençons par ajouter un état "confus" à l'AI dans `ai.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

+from random import randint
+
+from game_messages import Message


class BasicMonster:
    ...


+class ConfusedMonster:
+   def __init__(self, previous_ai, number_of_turns=10):
+       self.previous_ai = previous_ai
+       self.number_of_turns = number_of_turns
+
+   def take_turn(self, target, fov_map, game_map, entities):
+       results = []
+
+       if self.number_of_turns > 0:
+           random_x = self.owner.x + randint(0, 2) - 1
+           random_y = self.owner.y + randint(0, 2) - 1
+
+           if random_x != self.owner.x and random_y != self.owner.y:
+               self.owner.move_towards(random_x, random_y, game_map, entities)
+
+           self.number_of_turns -= 1
+       else:
+           self.owner.ai = self.previous_ai
+           results.append({'message': Message('The {0} is no longer confused!'.format(self.owner.name), libtcod.red)})
+
+       return results
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

<span class="new-text">from random import randint

from game_messages import Message</span>


class BasicMonster:
    ...


<span class="new-text">class ConfusedMonster:
    def __init__(self, previous_ai, number_of_turns=10):
        self.previous_ai = previous_ai
        self.number_of_turns = number_of_turns

    def take_turn(self, target, fov_map, game_map, entities):
        results = []

        if self.number_of_turns > 0:
            random_x = self.owner.x + randint(0, 2) - 1
            random_y = self.owner.y + randint(0, 2) - 1

            if random_x != self.owner.x and random_y != self.owner.y:
                self.owner.move_towards(random_x, random_y, game_map, entities)

            self.number_of_turns -= 1
        else:
            self.owner.ai = self.previous_ai
            results.append({'message': Message('The {0} is no longer confused!'.format(self.owner.name), libtcod.red)})

        return results</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

La classe est initialisée avec un nombre de tours durant lesquels l'entité est
confuse. On garde en mémoire l'état précédent de l'AI pour y revenir une fois
que la confusion est terminée. Pour la méthode `take_turn`, l'entité se déplace
de manière aléatoire (ou reste sur place) et le compteur diminue. Une fois que
le compteur atteint 0, l'entité n'est plus confuse et revient à son AI
précédente.

Maintenant le sort de confusion. Ajouter à `item_functions.py` les éléments
suivants :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def cast_fireball(*args, **kwargs):
    ...

+def cast_confuse(*args, **kwargs):
+   entities = kwargs.get('entities')
+   fov_map = kwargs.get('fov_map')
+   target_x = kwargs.get('target_x')
+   target_y = kwargs.get('target_y')
+
+   results = []
+
+   if not libtcod.map_is_in_fov(fov_map, target_x, target_y):
+       results.append({'consumed': False, 'message': Message('You cannot target a tile outside your field of view.', libtcod.yellow)})
+       return results
+
+   for entity in entities:
+       if entity.x == target_x and entity.y == target_y and entity.ai:
+           confused_ai = ConfusedMonster(entity.ai, 10)
+
+           confused_ai.owner = entity
+           entity.ai = confused_ai
+
+           results.append({'consumed': True, 'message': Message('The eyes of the {0} look vacant, as he starts to stumble around!'.format(entity.name), libtcod.light_green)})
+
+           break
+   else:
+       results.append({'consumed': False, 'message': Message('There is no targetable enemy at that location.', libtcod.yellow)})
+
+   return results
+
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def cast_fireball(*args, **kwargs):
    ...

<span class="new-text">def cast_confuse(*args, **kwargs):
    entities = kwargs.get('entities')
    fov_map = kwargs.get('fov_map')
    target_x = kwargs.get('target_x')
    target_y = kwargs.get('target_y')

    results = []

    if not libtcod.map_is_in_fov(fov_map, target_x, target_y):
        results.append({'consumed': False, 'message': Message('You cannot target a tile outside your field of view.', libtcod.yellow)})
        return results

    for entity in entities:
        if entity.x == target_x and entity.y == target_y and entity.ai:
            confused_ai = ConfusedMonster(entity.ai, 10)

            confused_ai.owner = entity
            entity.ai = confused_ai

            results.append({'consumed': True, 'message': Message('The eyes of the {0} look vacant, as he starts to stumble around!'.format(entity.name), libtcod.light_green)})

            break
    else:
        results.append({'consumed': False, 'message': Message('There is no targetable enemy at that location.', libtcod.yellow)})

    return results
</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous devons importer la classe `ConfusedMonster` en haut du fichier :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

+from components.ai import ConfusedMonster

from game_messages import Message
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

<span class="new-text">from components.ai import ConfusedMonster</span>

from game_messages import Message
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Finally, we'll put the scroll on the map. First, import the
`cast_confuse` function:

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from game_messages import Message

-from item_functions import cast_fireball, cast_lightning, heal
+from item_functions import cast_confuse, cast_fireball, cast_lightning, heal

from map_objects.rectangle import Rect
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from game_messages import Message

from item_functions import <span class="new-text">cast_confuse,</span> cast_fireball, cast_lightning, heal

from map_objects.rectangle import Rect
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous devons aussi modifier les chances d'apparition de nos parchemins, de façon
à ce qu'ils aient tous 10% de chance d'apparaître.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
                if item_chance < 70:
                    item_component = Item(use_function=heal, amount=4)
                    item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
                                  item=item_component)
-               elif item_chance < 85:
+               elif item_chance < 80:
                    item_component = Item(use_function=cast_fireball, targeting=True, targeting_message=Message(
                        'Left-click a target tile for the fireball, or right-click to cancel.', libtcod.light_cyan),
                                          damage=12, radius=3)
                    item = Entity(x, y, '#', libtcod.red, 'Fireball Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)
+               elif item_chance < 90:
+                   item_component = Item(use_function=cast_confuse, targeting=True, targeting_message=Message(
+                       'Left-click an enemy to confuse it, or right-click to cancel.', libtcod.light_cyan))
+                   item = Entity(x, y, '#', libtcod.light_pink, 'Confusion Scroll', render_order=RenderOrder.ITEM,
+                                 item=item_component)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                if item_chance < 70:
                    item_component = Item(use_function=heal, amount=4)
                    item = Entity(x, y, '!', libtcod.violet, 'Healing Potion', render_order=RenderOrder.ITEM,
                                  item=item_component)
                <span class="crossed-out-text">elif item_chance < 85:</span>
                <span class="new-text">elif item_chance < 80:</span>
                    item_component = Item(use_function=cast_fireball, targeting=True, targeting_message=Message(
                        'Left-click a target tile for the fireball, or right-click to cancel.', libtcod.light_cyan),
                                          damage=12, radius=3)
                    item = Entity(x, y, '#', libtcod.red, 'Fireball Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)
                <span class="new-text">elif item_chance < 90:
                    item_component = Item(use_function=cast_confuse, targeting=True, targeting_message=Message(
                        'Left-click an enemy to confuse it, or right-click to cancel.', libtcod.light_cyan))
                    item = Entity(x, y, '#', libtcod.light_pink, 'Confusion Scroll', render_order=RenderOrder.ITEM,
                                  item=item_component)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Lancez le projet maintenant et vous devriez pouvoir lancer confusion sur les
ennemis. Les ennemis confus vont gacher leur tours en se déplaçant aléatoirement
ou en restant sur place.

C'est tout pour aujourd'hui. Nous avons maintenant trois parchemins différents
que le joueur peut employer contre les ennemis. N'hésitez pas à en ajouter
d'autres !

Si vous voulez voir le code actuel entièrement, [cliquez ici](https://github.com/TStand90/roguelike_tutorial_revised/tree/part9).

[Cliquez ici pour vous rendre à la partie suivante de ce tutoriel.](/tutorials/tcod/part-10)

<script src="/js/codetabs.js"></script>
