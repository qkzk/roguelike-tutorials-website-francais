---
title: "Part 6 - Faire mal (et prendre des coups)"
date: 2019-03-30T09:33:50-07:00
draft: false
---

La dernière partie du tutorial a préparé les combats, il est temps de les
implémenter.

De manière à créer des entités "tuables", plutôt que d'ajouter des "points de
vie" à chaque entité, nous allons créer un **composant**, appelé `Fighter` qui
va contenir l'information relative au combat : HP, max HP, attaque et défense.
Si une entité peut combattre ce composant lui sera attaché et sinon, il n'en
aura pas. Cette manière de procéder est appelé **composition** et c'est une
alternative à la programmation par héritage habituelle.

Créer un paquet Python (un dossier avec un fichier vide \_\_init\_\_.py) appelé
`components`. Ajoutez-y un fichier `fighter.py` contenant le code suivant :

{{< highlight py3 >}}
class Fighter:
    def __init__(self, hp, defense, power):
        self.max_hp = hp
        self.hp = hp
        self.defense = defense
        self.power = power
{{</ highlight >}}

Ces variables devraient sembler familières à quiconque a joué à un RPG. HP
désigne la santé de l'entité, defense attenue les dégats et power est la force
d'attaque de l'entité. Peut-être que le jeu que vous envisagez dispose d'un
système de combat plus complexe mais nous resterons simple.

Un autre composant dont nous aurons besoin défini l'AI des ennemis. Certaines
entités (les ennemis) aurons une AI, d'autres (le joueur, les objets) n'en
auront pas. Nous reglerons notre boucle de jeu pour donner un tour à chaque
entité qui a une AI de prendre un tour et les autres n'auront pas de tour.

Créer un fichier dans `components` appelé `ai.py` et ajouter la classe suivante
dedans :

{{< highlight py3 >}}
class BasicMonster:
    def take_turn(self):
        print('The ' + self.owner.name + ' wonders when it will get to move.')
{{</ highlight >}}

Nous avons défini une méthode de base appelée `take_turn` qui sera appelée
dans notre boucle de jeu dans un instant. C'est juste un exemple pour l'instant
mais, dès la fin du chapitre, la fonction `take_turn` va réellement déplacer
l'entité.

Notre classe étant en place, nous allons porter notre attention sur la classe
`Entity` une fois encore. Nous devons lui passer les composants via le
constructeur comme nous avons fait à chaque fois. Modifiez la fonction `__init__`
dans `Entity` pour qu'elle ressemble à :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class Entity:
-   def __init__(self, x, y, char, color, name, blocks=False):
+   def __init__(self, x, y, char, color, name, blocks=False, fighter=None, ai=None):
        self.x = x
        self.y = y
        self.char = char
        self.color = color
        self.name = name
        self.blocks = blocks
+       self.fighter = fighter
+       self.ai = ai
+
+       if self.fighter:
+           self.fighter.owner = self
+
+       if self.ai:
+           self.ai.owner = self
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class Entity:
    <span class="crossed-out-text">def __init__(self, x, y, char, color, name, blocks=False):</span>
    <span class="new-text">def __init__(self, x, y, char, color, name, blocks=False, fighter=None, ai=None):</span>
        self.x = x
        self.y = y
        self.char = char
        self.color = color
        self.name = name
        self.blocks = blocks
        <span class="new-text">self.fighter = fighter
        self.ai = ai

        if self.fighter:
            self.fighter.owner = self

        if self.ai:
            self.ai.owner = self</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Ainsi les composants `fighter` et `ai` sont optionnels et les entités qui n'en
ont pas pas besoin n'en dépendront pas pour faire quoi que ce soit.

Pourquoi devoir régler le propriétaire du composant sur `self` ? Parce que nous
aurons besoin d'accéder à l'entité depuis le composant. Dans notre extrait de
code précédent pour le `BasicMonster`, nous avons pu accéder au nom ("name") de
l'entité en référençant le propriétaire ("owner"). Nous devons simplement nous
assurer de régler le propriétaire à l'initialisation de l'entité.

Maintenant nous allons devoir ajouter notre nouveau composant à chaque entité
que nous avons crée jusque là. Commençons par la plus facile : le joueur. Le
joueur n'a pas besoin d'une AI (parce que nous contrôlons directement l'objet
joueur) mais il lui faut un composant `Fighter`.

En premier, importer le composant `Fighter` dans `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

+from components.fighter import Fighter
from entity import Entity, get_blocking_entities_at_location
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

<span class="new-text">from components.fighter import Fighter</span>
from entity import Entity, get_blocking_entities_at_location</pre>
{{</ original-tab >}}
{{</ codetab >}}

Ensuite, créons le composant et ajoutons le à l'entité du joueur.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
+   fighter_component = Fighter(hp=30, defense=2, power=5)
-   player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True)
+   player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, fighter=fighter_component)
    entities = [player]
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
        <pre>
    <span class="new-text">fighter_component = Fighter(hp=30, defense=2, power=5)</span>
    <span class="crossed-out-text">player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True)</span>
    <span class="new-text">player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, fighter=fighter_component)</span>
    entities = [player]
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Et maintenant pour les monstres. Nous aurons besoin des composants Fighter
et BasicMonster pour ceux là.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
                if randint(0, 100) < 80:
+                   fighter_component = Fighter(hp=10, defense=0, power=3)
+                   ai_component = BasicMonster()

-                   monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True)
+                   monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True,
+                                    fighter=fighter_component, ai=ai_component)
                else:
+                   fighter_component = Fighter(hp=16, defense=1, power=4)
+                   ai_component = BasicMonster()

-                   monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True)
+                   monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True, fighter=fighter_component,
+                                    ai=ai_component)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>
                if randint(0, 100) < 80:
                    <span class="new-text">fighter_component = Fighter(hp=10, defense=0, power=3)
                    ai_component = BasicMonster()</span>

                    <span class="crossed-out-text">monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True)</span>
                    <span class="new-text">monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True,
                                     fighter=fighter_component, ai=ai_component)</span>
                else:
                    <span class="new-text">fighter_component = Fighter(hp=16, defense=1, power=4)
                    ai_component = BasicMonster()</span>

                    <span class="crossed-out-text">monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True)</span>
                    <span class="new-text">monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True, fighter=fighter_component,
                                     ai=ai_component)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Souvenez-vous d'importer les classes nécessaires en haut.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod
from random import randint

+from components.ai import BasicMonster
+from components.fighter import Fighter

from entity import Entity

from map_objects.rectangle import Rect
from map_objects.tile import Tile
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod
from random import randint

<span class="new-text">from components.ai import BasicMonster
from components.fighter import Fighter</span>

from entity import Entity

from map_objects.rectangle import Rect
from map_objects.tile import Tile</pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant nous pouvons modifier la boucle qui parcourt les tours des monstres
pour utiliser la fonction `take_turn`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        if game_state == GameStates.ENEMY_TURN:
            for entity in entities:
-               if entity != player:
+               if entity.ai:
-                   print('The ' + entity.name + ' ponders the meaning of its existence.')
+                   entity.ai.take_turn()

            game_state = GameStates.PLAYERS_TURN
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        if game_state == GameStates.ENEMY_TURN:
            for entity in entities:
                <span class="crossed-out-text">if entity != player:</span>
                <span class="new-text">if entity.ai:</span>
                    <span class="crossed-out-text">print('The ' + entity.name + ' ponders the meaning of its existence.')</span>
                    <span class="new-text">entity.ai.take_turn()</span>

            game_state = GameStates.PLAYERS_TURN
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous n'avons pas changé grand chose (on affiche toujours quelque chose
plutôt de donner vraiment un tour aux monstres) mais on avance. Remarquez que
plutôt que de vérifier si l'entité n'est pas le joueur, nous vérifions si elle
a un composant AI. Le joueur n'en a pas donc la boucle le passe. Ce sera aussi
le cas des objets que nous implémenterons plus tard, ils n'auront pas de "tour".

Maintenant, implémentons l'AI. Notre AI sera très simple (et même stupide). Si
l'ennemi peut "voir" le joueur, elle va déplacer se déplacer vers le joueur et
si elle est proche du joueur elle va l'attaquer. Nous n'implémenterons pas le
FOV de l'ennemi dans ce tutoriel. À la place, nous supposons simplement que si
vous pouvez voir l'ennemi, alors il peut aussi vous voir.

Mettons une simple fonction de mouvement en place. Ajouter le code suivant
dans la classe `Entity`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    def move(self, dx, dy):
        ...

+   def move_towards(self, target_x, target_y, game_map, entities):
+       dx = target_x - self.x
+       dy = target_y - self.y
+       distance = math.sqrt(dx ** 2 + dy ** 2)
+
+       dx = int(round(dx / distance))
+       dy = int(round(dy / distance))
+
+       if not (game_map.is_blocked(self.x + dx, self.y + dy) or
+                   get_blocking_entities_at_location(entities, self.x + dx, self.y + dy)):
+           self.move(dx, dy)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    def move(self, dx, dy):
        ...

    <span class="new-text">def move_towards(self, target_x, target_y, game_map, entities):
        dx = target_x - self.x
        dy = target_y - self.y
        distance = math.sqrt(dx ** 2 + dy ** 2)

        dx = int(round(dx / distance))
        dy = int(round(dy / distance))

        if not (game_map.is_blocked(self.x + dx, self.y + dy) or
                    get_blocking_entities_at_location(entities, self.x + dx, self.y + dy)):
            self.move(dx, dy)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous avons aussi besoin d'une fonction pour obtenir la distance entre l'entité
et sa cible.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    def move_towards(self, target_x, target_y, game_map, entities):
        ...

+   def distance_to(self, other):
+       dx = other.x - self.x
+       dy = other.y - self.y
+       return math.sqrt(dx ** 2 + dy ** 2)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    def move_towards(self, target_x, target_y, game_map, entities):
        ...

    <span class="new-text">def distance_to(self, other):
        dx = other.x - self.x
        dy = other.y - self.y
        return math.sqrt(dx ** 2 + dy ** 2)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Ces deux fonctions utilisent le module `math` donc nous devons l'importer.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
+import math


class Entity:
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="new-text">import math</span>


class Entity:
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Remplaçons notre fonction `take_turn` d'exemple avec celle qui va réellement
déplacer l'entité.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod


class BasicMonster:
-   def take_turn(self):
+   def take_turn(self, target, fov_map, game_map, entities):
-       print('The ' + self.owner.name + ' wonders when it will get to move.')
+       monster = self.owner
+       if libtcod.map_is_in_fov(fov_map, monster.x, monster.y):
+
+           if monster.distance_to(target) >= 2:
+               monster.move_towards(target.x, target.y, game_map, entities)
+
+           elif target.fighter.hp > 0:
+               print('The {0} insults you! Your ego is damaged!'.format(monster.name))
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="new-text">import tcod as libtcod</span>


class BasicMonster:
    <span class="crossed-out-text">def take_turn(self):</span>
    <span class="new-text">def take_turn(self, target, fov_map, game_map, entities):</span>
        <span class="crossed-out-text">print('The ' + self.owner.name + ' wonders when it will get to move.')</span>
        <span class="new-text">monster = self.owner
        if libtcod.map_is_in_fov(fov_map, monster.x, monster.y):

            if monster.distance_to(target) >= 2:
                monster.move_towards(target.x, target.y, game_map, entities)

            elif target.fighter.hp > 0:
                print('The {0} insults you! Your ego is damaged!'.format(monster.name))</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous devons aussi mettre à jour l'appel de `take_turn` dans `engine.py`

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-                   entity.ai.take_turn()
+                   entity.ai.take_turn(player, fov_map, game_map, entities)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                    <span class="crossed-out-text">entity.ai.take_turn()</span>
                    <span class="new-text">entity.ai.take_turn(player, fov_map, game_map, entities)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant notre ennemi va poursuivre le joueur et, s'il le rattrape lui hurler
des insultes \!

Si vous lancez le jeu projet, vous remarquerez quelque chose d'étrange à propos
de nos monstres : ils peuvent vous insulter depuis une case en diagonale mais
le joueur et les ennemis ne peuvent se déplacer que dans des directions
cardinales (nord, sud, est, ouest). Si les ennemis nous attaquaient vraiment
ils auraient un avantage injuste. Cela pourrait être un gameplay intéressant
mais nous le fixerons en permettant le déplacement et l'attaque dans les 8
directions pour toutes les entités.

Pour le joueur c'est assez simple, nous devons mettre à jour `handle_keys` pour
permettre un mouvement diagonal. Modifiez la partie mouvement de cette fonction
ainsi :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def handle_keys(key):
+   key_char = chr(key.c)

-   if key.vk == libtcod.KEY_UP:
+   if key.vk == libtcod.KEY_UP or key_char == 'k':
        return {'move': (0, -1)}
-  elif key.vk == libtcod.KEY_DOWN:
+   elif key.vk == libtcod.KEY_DOWN or key_char == 'j':
        return {'move': (0, 1)}
-   elif key.vk == libtcod.KEY_LEFT:
+   elif key.vk == libtcod.KEY_LEFT or key_char == 'h':
        return {'move': (-1, 0)}
-   elif key.vk == libtcod.KEY_RIGHT:
+   elif key.vk == libtcod.KEY_RIGHT or key_char == 'l':
        return {'move': (1, 0)}
+   elif key_char == 'y':
+       return {'move': (-1, -1)}
+   elif key_char == 'u':
+       return {'move': (1, -1)}
+   elif key_char == 'b':
+       return {'move': (-1, 1)}
+   elif key_char == 'n':
+       return {'move': (1, 1)}

    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def handle_keys(key):
    <span class="new-text">key_char = chr(key.c)</span>

    if key.vk == libtcod.KEY_UP<span class="new-text"> or key_char == 'k'</span>:
        return {'move': (0, -1)}
    elif key.vk == libtcod.KEY_DOWN<span class="new-text"> or key_char == 'j'</span>:
        return {'move': (0, 1)}
    elif key.vk == libtcod.KEY_LEFT<span class="new-text"> or key_char == 'h'</span>:
        return {'move': (-1, 0)}
    elif key.vk == libtcod.KEY_RIGHT<span class="new-text"> or key_char == 'l'</span>:
        return {'move': (1, 0)}
    <span class="new-text">elif key_char == 'y':
        return {'move': (-1, -1)}
    elif key_char == 'u':
        return {'move': (1, -1)}
    elif key_char == 'b':
        return {'move': (-1, 1)}
    elif key_char == 'n':
        return {'move': (1, 1)}</span>

    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

La première ligne récupère le caractère pressé sur le clavier. Cela sera commode
dans de futures étapes, quand nous allons écouter les commandes pour
l'inventaire et ramasser les objets.

Pour les mouvements diagonaux, nous avons implémenté les "vim keys" du
déplacement et conservé les fleches pour les déplacements cardinaux. Les "vim
keys" permettent de se déplacer en diagonale sans utiliser le pavé numérique.
De nombreux roguelikes implémentent les déplacements dans 8 directions via le
pavé numérique mais je préfère jouer sur un portable qui n'en a pas, aussi les
"vim keys" sont commodes.

Déplacer les ennemis dans huit directions sera un peu plus délicat. Pour ça,
nous allons utiliser un algorithme de recherche de chemin appelé A-star. Je vais
simplement copier le code puis les [extra de Roguebasin](http://www.roguebasin.com/index.php?title=Complete_Roguelike_Tutorial,_using_Python%2Blibtcod,_extras#A.2A_Pathfinding).
Je n'entrerai pas dans le détail à ce propos mais si vous voulez comprendre son
fonctionnement, [cliquez ici](https://en.wikipedia.org/wiki/A*_search_algorithm).

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    def move_towards(self, target_x, target_y, game_map, entities):
    ...

+   def move_astar(self, target, entities, game_map):
+       # Create a FOV map that has the dimensions of the map
+       fov = libtcod.map_new(game_map.width, game_map.height)
+
+       # Scan the current map each turn and set all the walls as unwalkable
+       for y1 in range(game_map.height):
+           for x1 in range(game_map.width):
+               libtcod.map_set_properties(fov, x1, y1, not game_map.tiles[x1][y1].block_sight,
+                                          not game_map.tiles[x1][y1].blocked)
+
+       # Scan all the objects to see if there are objects that must be navigated around
+       # Check also that the object isn't self or the target (so that the start and the end points are free)
+       # The AI class handles the situation if self is next to the target so it will not use this A* function anyway
+       for entity in entities:
+           if entity.blocks and entity != self and entity != target:
+               # Set the tile as a wall so it must be navigated around
+               libtcod.map_set_properties(fov, entity.x, entity.y, True, False)
+
+       # Allocate a A* path
+       # The 1.41 is the normal diagonal cost of moving, it can be set as 0.0 if diagonal moves are prohibited
+       my_path = libtcod.path_new_using_map(fov, 1.41)
+
+       # Compute the path between self's coordinates and the target's coordinates
+       libtcod.path_compute(my_path, self.x, self.y, target.x, target.y)
+
+       # Check if the path exists, and in this case, also the path is shorter than 25 tiles
+       # The path size matters if you want the monster to use alternative longer paths (for example through other rooms) if for example the player is in a corridor
+       # It makes sense to keep path size relatively low to keep the monsters from running around the map if there's an alternative path really far away
+       if not libtcod.path_is_empty(my_path) and libtcod.path_size(my_path) < 25:
+           # Find the next coordinates in the computed full path
+           x, y = libtcod.path_walk(my_path, True)
+           if x or y:
+               # Set self's coordinates to the next path tile
+               self.x = x
+               self.y = y
+       else:
+           # Keep the old move function as a backup so that if there are no paths (for example another monster blocks a corridor)
+           # it will still try to move towards the player (closer to the corridor opening)
+           self.move_towards(target.x, target.y, game_map, entities)
+
+           # Delete the path to free memory
+       libtcod.path_delete(my_path)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    def move_towards(self, target_x, target_y, game_map, entities):
    ...

    <span class="new-text">def move_astar(self, target, entities, game_map):
        # Create a FOV map that has the dimensions of the map
        fov = libtcod.map_new(game_map.width, game_map.height)

        # Scan the current map each turn and set all the walls as unwalkable
        for y1 in range(game_map.height):
            for x1 in range(game_map.width):
                libtcod.map_set_properties(fov, x1, y1, not game_map.tiles[x1][y1].block_sight,
                                           not game_map.tiles[x1][y1].blocked)

        # Scan all the objects to see if there are objects that must be navigated around
        # Check also that the object isn't self or the target (so that the start and the end points are free)
        # The AI class handles the situation if self is next to the target so it will not use this A* function anyway
        for entity in entities:
            if entity.blocks and entity != self and entity != target:
                # Set the tile as a wall so it must be navigated around
                libtcod.map_set_properties(fov, entity.x, entity.y, True, False)

        # Allocate a A* path
        # The 1.41 is the normal diagonal cost of moving, it can be set as 0.0 if diagonal moves are prohibited
        my_path = libtcod.path_new_using_map(fov, 1.41)

        # Compute the path between self's coordinates and the target's coordinates
        libtcod.path_compute(my_path, self.x, self.y, target.x, target.y)

        # Check if the path exists, and in this case, also the path is shorter than 25 tiles
        # The path size matters if you want the monster to use alternative longer paths (for example through other rooms) if for example the player is in a corridor
        # It makes sense to keep path size relatively low to keep the monsters from running around the map if there's an alternative path really far away
        if not libtcod.path_is_empty(my_path) and libtcod.path_size(my_path) < 25:
            # Find the next coordinates in the computed full path
            x, y = libtcod.path_walk(my_path, True)
            if x or y:
                # Set self's coordinates to the next path tile
                self.x = x
                self.y = y
        else:
            # Keep the old move function as a backup so that if there are no paths (for example another monster blocks a corridor)
            # it will still try to move towards the player (closer to the corridor opening)
            self.move_towards(target.x, target.y, game_map, entities)

            # Delete the path to free memory
        libtcod.path_delete(my_path)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Pour faire fonctionner ça, nous devons importer `libtcod` dans `entity.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
+import tcod as libtcod

import math
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="new-text">import tcod as libtcod</span>

import math
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Remarquez que si l'algorithme est incapable de trouver un chemin il va revenir
à notre fonction de mouvement précédente. Nous en avons donc toujours besoin.

Modifiez la fonction `take_turn` de `BasicMonster` pour utiliser cette nouvelle
fonction.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            ...
            if monster.distance_to(target) >= 2:
+               monster.move_astar(target, entities, game_map)
-               monster.move_towards(target.x, target.y, game_map, entities)
            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            if monster.distance_to(target) >= 2:
                <span class="new-text">monster.move_astar(target, entities, game_map)</span>
                <span style="color: red; text-decoration: line-through;">monster.move_towards(target.x, target.y, game_map, entities)</span>
            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant le joueur et les ennemis peuvent se déplacer en diagonale. Ceci étant
fait, il est temps d'implémenter un système de combat. Commençons par ajouter
une méthode au `Fighter` qui permette à l'entité de prendre des dégats.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class Fighter:
    def __init__(self, hp, defense, power):
        ...

+   def take_damage(self, amount):
+       self.hp -= amount
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class Fighter:
    def __init__(self, hp, defense, power):
        ...

    <span class="new-text">def take_damage(self, amount):
        self.hp -= amount</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Plutôt simple. Maintenant la fonction d'attaque, toujours dans `Fighter` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...

+   def attack(self, target):
+       damage = self.power - target.fighter.defense
+
+       if damage > 0:
+           target.fighter.take_damage(damage)
+           print('{0} attacks {1} for {2} hit points.'.format(self.owner.name.capitalize(), target.name, str(damage)))
+       else:
+           print('{0} attacks {1} but does no damage.'.format(self.owner.name.capitalize(), target.name))
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...

    <span class="new-text">def attack(self, target):
        damage = self.power - target.fighter.defense

        if damage > 0:
            target.fighter.take_damage(damage)
            print('{0} attacks {1} for {2} hit points.'.format(self.owner.name.capitalize(), target.name, str(damage)))
        else:
            print('{0} attacks {1} but does no damage.'.format(self.owner.name.capitalize(), target.name))</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Rien de très complexe dans ce système. On prend la puissance d'attaque (power)
de l'agresseur et on soustraie la défense du défenseur pour obtenir les dégâts
effectués. Si le dégât est supérieur à zéro, alors la cible reçoit des dégâts.

Nous pouvons enfin remplacer notre exemple antérieur \! Modifier l'exemple
du joueur dans `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
                if target:
-                   print('You kick the ' + target.name + ' in the shins, much to its annoyance!')
+                   player.fighter.attack(target)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                if target:
                    <span class="crossed-out-text">print('You kick the ' + target.name + ' in the shins, much to its annoyance!')</span>
                    <span class="new-text">player.fighter.attack(target)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

... et pour l'exemple de l'ennemi dans `BasicMonster`

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            ...
            elif target.fighter.hp > 0:
-               print('The {0} insults you! Your ego is damaged!'.format(monster.name))
+               monster.fighter.attack(target)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            elif target.fighter.hp > 0:
                <span class="crossed-out-text">print('The {0} insults you! Your ego is damaged!'.format(monster.name))</span>
                <span class="new-text">monster.fighter.attack(target)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant on peut attaquer les ennemis et ils peuvent rendre les coups \!

Aussi amusant que cela soit, nous devons marquer une pause et réfléchir au
design. Pour l'instant, nous affichons nos messages dans la console et, dans
l'étape suivante, nous utiliserons un journal de message plus classique. Aussi
nous devrons modifier l'état du jeu quand le joueur est tué en combat. Les
fonctions `attack` et `take_damage` doivent-elles recevoir le journal de message
ou l'état du jeu comme paramètre ? Doivent-elles manipuler ces objets ?

Il existe de multiples manières de traiter cela, pour ce tutoriel, nous allons
implémenter une liste `results` pour les fonctions de ce genre qui sera retourné
à `engine.py` et sera traité dans ce fichier. Nous faisons déjà quelque chose
de similaire dans `handle_keys`, cette fonction renvoie le résultat d'une touche
pressée, elle ne *déplace pas* le joueur.

Modifiez les fonctions `take_damage` et `attack` pour renvoyer un tableau de
résultats plutôt que d'afficher quoi que ce soit.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    def take_damage(self, amount):
+       results = []

        self.hp -= amount

+       if self.hp <= 0:
+           results.append({'dead': self.owner})
+
+       return results

    def attack(self, target):
+       results = []

        damage = self.power - target.fighter.defense

        if damage > 0:
-           target.fighter.take_damage(damage)
-           print('{0} attacks {1} for {2} hit points.'.format(self.owner.name.capitalize(), target.name, str(damage)))
+           results.append({'message': '{0} attacks {1} for {2} hit points.'.format(
+               self.owner.name.capitalize(), target.name, str(damage))})
+           results.extend(target.fighter.take_damage(damage))
        else:
-           print('{0} attacks {1} but does no damage.'.format(self.owner.name.capitalize(), target.name))
+           results.append({'message': '{0} attacks {1} but does no damage.'.format(
+               self.owner.name.capitalize(), target.name)})
+
+       return results
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>
    def take_damage(self, amount):
        <span class="new-text">results = []</span>

        self.hp -= amount

        <span class="new-text">if self.hp <= 0:
            results.append({'dead': self.owner})

        return results</span>

    def attack(self, target):
        <span class="new-text">results = []</span>

        damage = self.power - target.fighter.defense

        if damage > 0:
            <span style="color: red; text-decoration: line-through;">target.fighter.take_damage(damage)</span>
            <span style="color: red; text-decoration: line-through;">print('{0} attacks {1} for {2} hit points.'.format(self.owner.name.capitalize(), target.name, str(damage)))</span>
            <span class="new-text">results.append({'message': '{0} attacks {1} for {2} hit points.'.format(
                self.owner.name.capitalize(), target.name, str(damage))})
            results.extend(target.fighter.take_damage(damage))</span>
        else:
            <span style="color: red; text-decoration: line-through;">print('{0} attacks {1} but does no damage.'.format(self.owner.name.capitalize(), target.name))</span>
            <span class="new-text">results.append({'message': '{0} attacks {1} but does no damage.'.format(
                self.owner.name.capitalize(), target.name)})

        return results</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Découpons cette étape quelques morceaux. Dans `take_damage`, on ajoute un
dictionnaire à `results` si l'entité meurt après avoir pris des dégâts. la
liste résultante est renvoyée dans tous les cas (elle peut être vide).

Dans `attack`, nous créeons à nouveau une liste appelée `results` et nous y
ajoutons notre message qu'un dégât ait été pris ou non. Remarquez que dans le
bloc `if` nous utilisons `extend` pour ajouter les résultat de `take_damage`
à notre liste `results`.

La méthode `extend` est similaire à `append` mais elle garde la liste plate.
Ainsi nous évitons d'avoir quelque chose comme `[{'message': 'something'},
[{'message': 'something else'}]]`. Nous obtenons plutôt quelque chose comme :
[{'message': 'something'}, {'message': 'something else'}]`. Cela va simplifier
la boucle sur nos résultats.

Appliquons cette logique à la fonction `take_turn` de `BasicMonster`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class BasicMonster:
    def take_turn(self, target, fov_map, game_map, entities):
+       results = []

        monster = self.owner
        if libtcod.map_is_in_fov(fov_map, monster.x, monster.y):

            if monster.distance_to(target) >= 2:
                monster.move_astar(target, entities, game_map)

            elif target.fighter.hp > 0:
-               monster.fighter.attack(target)
+               attack_results = monster.fighter.attack(target)
+               results.extend(attack_results)
+
+       return results
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class BasicMonster:
    def take_turn(self, target, fov_map, game_map, entities):
        <span class="new-text">results = []</span>

        monster = self.owner
        if libtcod.map_is_in_fov(fov_map, monster.x, monster.y):

            if monster.distance_to(target) >= 2:
                monster.move_astar(target, entities, game_map)

            elif target.fighter.hp > 0:
                <span class="crossed-out-text">monster.fighter.attack(target)</span>
                <span class="new-text">attack_results = monster.fighter.attack(target)
                results.extend(attack_results)

        return results</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Que *faisons nous* avec cette liste `results` ? Modifiez `engine.py` pour réagir
aux résultats d'une attaque.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        fullscreen = action.get('fullscreen')

+       player_turn_results = []

        if move and game_state == GameStates.PLAYERS_TURN:
            dx, dy = move
            destination_x = player.x + dx
            destination_y = player.y + dy

            if not game_map.is_blocked(destination_x, destination_y):
                target = get_blocking_entities_at_location(entities, destination_x, destination_y)

                if target:
-                   player.fighter.attack(target)
+                   attack_results = player.fighter.attack(target)
+                   player_turn_results.extend(attack_results)
                else:
                    player.move(dx, dy)

                    fov_recompute = True

                game_state = GameStates.ENEMY_TURN

        if exit:
            return True

        if fullscreen:
            libtcod.console_set_fullscreen(not libtcod.console_is_fullscreen())

+       for player_turn_result in player_turn_results:
+           message = player_turn_result.get('message')
+           dead_entity = player_turn_result.get('dead')
+
+           if message:
+               print(message)
+
+           if dead_entity:
+               pass # We'll do something here momentarily

        if game_state == GameStates.ENEMY_TURN:
            for entity in entities:
                if entity.ai:
-                   entity.ai.take_turn(player, fov_map, game_map, entities)
+                   enemy_turn_results = entity.ai.take_turn(player, fov_map, game_map, entities)
+
+                   for enemy_turn_result in enemy_turn_results:
+                       message = enemy_turn_result.get('message')
+                       dead_entity = enemy_turn_result.get('dead')
+
+                       if message:
+                           print(message)
+
+                       if dead_entity:
+                           pass
+
+           else:
                game_state = GameStates.PLAYERS_TURN
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        fullscreen = action.get('fullscreen')

        <span class="new-text">player_turn_results = []</span>

        if move and game_state == GameStates.PLAYERS_TURN:
            dx, dy = move
            destination_x = player.x + dx
            destination_y = player.y + dy

            if not game_map.is_blocked(destination_x, destination_y):
                target = get_blocking_entities_at_location(entities, destination_x, destination_y)

                if target:
                    <span style="color: red; text-decoration: line-through;">player.fighter.attack(target)</span>
                    <span class="new-text">attack_results = player.fighter.attack(target)
                    player_turn_results.extend(attack_results)</span>
                else:
                    player.move(dx, dy)

                    fov_recompute = True

                game_state = GameStates.ENEMY_TURN

        if exit:
            return True

        if fullscreen:
            libtcod.console_set_fullscreen(not libtcod.console_is_fullscreen())

        <span class="new-text">for player_turn_result in player_turn_results:
            message = player_turn_result.get('message')
            dead_entity = player_turn_result.get('dead')

            if message:
                print(message)

            if dead_entity:
                pass # We'll do something here momentarily</span>

        if game_state == GameStates.ENEMY_TURN:
            for entity in entities:
                if entity.ai:
                    <span class="crossed-out-text">entity.ai.take_turn(player, fov_map, game_map, entities)</span>
                    <span class="new-text">enemy_turn_results = entity.ai.take_turn(player, fov_map, game_map, entities)

                    for enemy_turn_result in enemy_turn_results:
                        message = enemy_turn_result.get('message')
                        dead_entity = enemy_turn_result.get('dead')

                        if message:
                            print(message)

                        if dead_entity:
                            pass

            else:</span>
                <span style="color: blue">game_state = GameStates.PLAYERS_TURN</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

*\* Remarque : il y a encore une expression for-else. Il n'y a aucun `break`
pour l'instant donc le bloc `else` sera toujours exécuté. Mais nous l'ajouterons
dans un instant.*

Il n'y a pas eu beaucoup  de changement mais nous avons mis en place ce qu'il
faut pour la mort du joueur ou d'une autre entité. Implémentons ça maintenant.
Créons un nouveau fichier appelé `death_functions.py` et ajoutons y deux
fonctions :

{{< highlight py3 >}}
import tcod as libtcod

from game_states import GameStates


def kill_player(player):
    player.char = '%'
    player.color = libtcod.dark_red

    return 'You died!', GameStates.PLAYER_DEAD


def kill_monster(monster):
    death_message = '{0} is dead!'.format(monster.name.capitalize())

    monster.char = '%'
    monster.color = libtcod.dark_red
    monster.blocks = False
    monster.fighter = None
    monster.ai = None
    monster.name = 'remains of ' + monster.name

    return death_message
{{</ highlight >}}

Ces deux fonctions vont s'occuper de la mort du joueur et des monstres.
Elles sont différentes parce que la mort d'un monstre n'est pas quelque chose
de dramatique (nous en tuerons quelques uns...) mais la mort du joueur est
*très* importante (c'est un roguelike après tout \!).

Modifiez `engine.py` pour utiliser ces deux fonctions. Remplacez la section
`pass` comme ceci :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            ...
            if dead_entity:
-               pass
+               if dead_entity == player:
+                   message, game_state = kill_player(dead_entity)
+               else:
+                   message = kill_monster(dead_entity)
+
+               print(message)

        if game_state == GameStates.ENEMY_TURN:
            for entity in entities:
                if entity.ai:
                    enemy_turn_results = entity.ai.take_turn(player, fov_map, game_map, entities)

                    for enemy_turn_result in enemy_turn_results:
                        message = enemy_turn_result.get('message')
                        dead_entity = enemy_turn_result.get('dead')

                        if message:
                            print(message)

                        if dead_entity:
-                           pass
+                           if dead_entity == player:
+                               message, game_state = kill_player(dead_entity)
+                           else:
+                               message = kill_monster(dead_entity)
+
+                           print(message)
+
+                           if game_state == GameStates.PLAYER_DEAD:
+                               break
+
+                   if game_state == GameStates.PLAYER_DEAD:
+                       break
            else:
                game_state = GameStates.PLAYERS_TURN
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            if dead_entity:
                <span style="color: red; text-decoration: line-through;">pass</span>
                <span class="new-text">if dead_entity == player:
                    message, game_state = kill_player(dead_entity)
                else:
                    message = kill_monster(dead_entity)

                print(message)</span>

        if game_state == GameStates.ENEMY_TURN:
            for entity in entities:
                if entity.ai:
                    enemy_turn_results = entity.ai.take_turn(player, fov_map, game_map, entities)

                    for enemy_turn_result in enemy_turn_results:
                        message = enemy_turn_result.get('message')
                        dead_entity = enemy_turn_result.get('dead')

                        if message:
                            print(message)

                        if dead_entity:
                            <span style="color: red; text-decoration: line-through;">pass</span>
                            <span class="new-text">if dead_entity == player:
                                message, game_state = kill_player(dead_entity)
                            else:
                                message = kill_monster(dead_entity)

                            print(message)

                            if game_state == GameStates.PLAYER_DEAD:
                                break

                    if game_state == GameStates.PLAYER_DEAD:
                        break</span>
            else:
                game_state = GameStates.PLAYERS_TURN</pre>
{{</ original-tab >}}
{{</ codetab >}}

*\*Remarque : il y a l'expression break qui va éviter le 'else' de notre
'for-else'. Pourquoi ? Parce que si le joueur meurt nous ne voulons pas lui
rendre de tour une fois que les ennemis auront tous joué. D'autre part il n'y
aucune raison de continuer, le jeu est terminé.*

Souvenez-vous d'importer la fonction qui tue en haut de `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from components.fighter import Fighter
+from death_functions import kill_monster, kill_player
from entity import Entity, get_blocking_entities_at_location
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from components.fighter import Fighter
<span class="new-text">from death_functions import kill_monster, kill_player</span>
from entity import Entity, get_blocking_entities_at_location
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Aussi, nous devons ajouter la valeur `PLAYER_DEAD` à `GameStates` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class GameStates(Enum):
    PLAYERS_TURN = 1
    ENEMY_TURN = 2
+   PLAYER_DEAD = 3
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class GameStates(Enum):
    PLAYERS_TURN = 1
    ENEMY_TURN = 2
    <span class="new-text">PLAYER_DEAD = 3</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Lancez le projet maintenant. Les entités, y compris le joueur, vont mourir en
arrivant à 0 HP \! Quand le joueur meurt, on ne peut plus le déplacer mais on
peut toujours quitter le jeu. Nous avons enfin un vrai système de combat \!

C'est déjà un long chapitre mais nettoyons un peu les choses. Pour l'instant
nous ne savons pas combien le joueur a de HP avant sa mort. Plutôt que de
demander au joueur de faire les calculs mentalement nous pouvons ajouter une
petite barre de vie avec le code suivant à la fin de `render_all` juste avant
l'expression 'blit' (remarquez que le joueur doit être passé à `render_all`
pour l'instant).

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-def render_all(con, entities, game_map, fov_map, fov_recompute, screen_width, screen_height, colors):
+def render_all(con, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height, colors):
    ...
    for entity in entities:
        draw_entity(con, entity, fov_map)

+   libtcod.console_set_default_foreground(con, libtcod.white)
+   libtcod.console_print_ex(con, 1, screen_height - 2, libtcod.BKGND_NONE, libtcod.LEFT,
+                        'HP: {0:02}/{1:02}'.format(player.fighter.hp, player.fighter.max_hp))

    libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="crossed-out-text">def render_all(con, entities, game_map, fov_map, fov_recompute, screen_width, screen_height, colors):</span>
<span class="new-text">def render_all(con, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height, colors):</span>
    ...
    for entity in entities:
        draw_entity(con, entity, fov_map)

    <span class="new-text">libtcod.console_set_default_foreground(con, libtcod.white)
    libtcod.console_print_ex(con, 1, screen_height - 2, libtcod.BKGND_NONE, libtcod.LEFT,
                         'HP: {0:02}/{1:02}'.format(player.fighter.hp, player.fighter.max_hp))</span>

    libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)</pre>
{{</ original-tab >}}
{{</ codetab >}}

Mettez l'appel de `render_all` à jouer dans `engine.py`

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-render_all(con, entities, game_map, fov_map, fov_recompute, screen_width, screen_height, colors)
+render_all(con, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height, colors)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="crossed-out-text">render_all(con, entities, game_map, fov_map, fov_recompute, screen_width, screen_height, colors)</span>
<span class="new-text">render_all(con, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height, colors)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Une chose que vous avez certainement déjà remarqué est que les corps des ennemis
décédés "recouvrent" le joueur si on se déplace dessus. De toute évidence ce
n'est le comportement souhaité. Les entités qui agissent devraient toujours
apparaître au dessus des cadavres, des objets et des autres choses du donjon.
Pour résoudre ce problème ajoutons un Enum aux entités. Il décrira l'ordre dans
lequel elles doivent être dessinées. Les éléments faible priorité seront
dessinées en premier pour s'assurer qu'elles n'apparaissent jamais au dessus des
autres.

Ajoutez le code suivant à `render_functions.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

+from enum import Enum
+
+
+class RenderOrder(Enum):
+   CORPSE = 1
+   ITEM = 2
+   ACTOR = 3


def render_all(con, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height, colors):
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

<span class="new-text">from enum import Enum


class RenderOrder(Enum):
    CORPSE = 1
    ITEM = 2
    ACTOR = 3</span>


def render_all(con, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height, colors):
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Now modify the `__init__` function in `Entity` to take this into
account.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod
import math

+from render_functions import RenderOrder


class Entity:
    """
    A generic object to represent players, enemies, items, etc.
    """
-   def __init__(self, x, y, char, color, name, blocks=False, fighter=None, ai=None):
+   def __init__(self, x, y, char, color, name, blocks=False, render_order=RenderOrder.CORPSE, fighter=None, ai=None):
        self.x = x
        self.y = y
        self.char = char
        self.color = color
        self.name = name
        self.blocks = blocks
+       self.render_order = render_order
        self.fighter = fighter
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod
import math

<span class="new-text">from render_functions import RenderOrder</span>


class Entity:
    """
    A generic object to represent players, enemies, items, etc.
    """
    def __init__(self, x, y, char, color, name, blocks=False, <span class="new-text">render_order=RenderOrder.CORPSE,</span> fighter=None, ai=None):
        self.x = x
        self.y = y
        self.char = char
        self.color = color
        self.name = name
        self.blocks = blocks
        <span class="new-text">self.render_order = render_order</span>
        self.fighter = fighter
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant modifions l'initialisation des entités en commençant par `engine.py`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, fighter=fighter_component)
+player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, render_order=RenderOrder.ACTOR, fighter=fighter_component)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True, <span class="new-text">render_order=RenderOrder.ACTOR,</span> fighter=fighter_component)</pre>
{{</ original-tab >}}
{{</ codetab >}}

... N'oublions pas les imports sur le bord de la route :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-from render_functions import clear_all, render_all
+from render_functions import clear_all, render_all, RenderOrder
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>from render_functions import clear_all, render_all<span class="new-text">, RenderOrder</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Et maintenant les monstres de `game_map.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
                if randint(0, 100) < 80:
                    fighter_component = Fighter(hp=10, defense=0, power=3)
                    ai_component = BasicMonster()

-                   monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True,
-                                    fighter=fighter_component, ai=ai_component)
+                   monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True,
+                                    render_order=RenderOrder.ACTOR, fighter=fighter_component, ai=ai_component)
                else:
                    fighter_component = Fighter(hp=16, defense=1, power=4)
                    ai_component = BasicMonster()

-                   monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True, fighter=fighter_component,
-                                    ai=ai_component)
+                   monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True, fighter=fighter_component,
+                                    render_order=RenderOrder.ACTOR, ai=ai_component)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                if randint(0, 100) < 80:
                    fighter_component = Fighter(hp=10, defense=0, power=3)
                    ai_component = BasicMonster()

                    <span class="crossed-out-text">monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True,</span>
                                     <span class="crossed-out-text">fighter=fighter_component, ai=ai_component)</span>
                    <span class="new-text">monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True,
                                     render_order=RenderOrder.ACTOR, fighter=fighter_component, ai=ai_component)</span>
                else:
                    fighter_component = Fighter(hp=16, defense=1, power=4)
                    ai_component = BasicMonster()

                    <span class="crossed-out-text">monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True, fighter=fighter_component,</span>
                                     <span class="crossed-out-text">ai=ai_component)</span>
                    <span class="new-text">monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True, fighter=fighter_component,
                                     render_order=RenderOrder.ACTOR, ai=ai_component)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

... And the import:

{{< highlight py3 >}}
from render_functions import RenderOrder
{{</ highlight >}}

We'll also need to change the Entity's `render_order` when they die.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    monster.ai = None
    monster.name = 'remains of ' + monster.name
+   monster.render_order = RenderOrder.CORPSE
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
    <pre>    monster.ai = None
    monster.name = 'remains of ' + monster.name
    <span class="new-text">monster.render_order = RenderOrder.CORPSE</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

And, you guessed it, make sure you import:

{{< highlight py3 >}}
from render_functions import RenderOrder
{{</ highlight >}}

*\* Note: We're not changing the `render_order` on the player when it
dies; we actually **want** that corpse on top so we'll see it. It's more
dramatic that way\!*

Now let's implement the part in `render_all` that will actually take
this new variable into account.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    if fov_recompute:
        ...

+   entities_in_render_order = sorted(entities, key=lambda x: x.render_order.value)

-   for entity in entities:
+   for entity in entities_in_render_order:
        draw_entity(con, entity, fov_map)
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    if fov_recompute:
        ...

    <span class="new-text">entities_in_render_order = sorted(entities, key=lambda x: x.render_order.value)</span>

    <span class="crossed-out-text">for entity in entities:</span>
    <span class="new-text">for entity in entities_in_render_order:</span>
        draw_entity(con, entity, fov_map)
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant les cadavres seront dessinés en premier, ensuite les objets (quand
nous les aurons ajouté) et enfin les entités. Cela nous assure qu'on verra
d'abord les choses importantes.

Et c'est fait \! Ce fut une sacré étape mais vous en êtes sorti indemne \!
Lancez le projet et regardez combien de temps vous surviviez dans ce donjon
maudit qui est maintenant mortel. Avec un sytème de combat en place, nous avons
franchis un grand pas vers un vrai jeu roguelike.

Si vous voulez voir le code actuel entièrement, [cliquez ici](https://github.com/TStand90/roguelike_tutorial_revised/tree/part6).

[Cliquez ici pour vous rendre à la partie suivante de ce tutoriel.](/tutorials/tcod/part-7)

<script src="/js/codetabs.js"></script>
