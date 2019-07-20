---
title: "Partie 5 - Ajouter des ennemis et leur donner des coups de pied (sans faire mal)"
date: 2019-03-30T09:33:48-07:00
draft: false
---

Qu'est-ce qu'un donjon sans monstre à cogner ? Ce chapitre se concentrera sur
la disposition des ennemis à travers le donjon et les configurer pour qu'on
puisse les attaquer (les vrais combats seront abordés plus tard). Pour
commencer nous aurons besoin d'une fonction pour positionner les ennemis dans
le donjon. Appelons la `place_entities` et ajoutons la à la classe `GameMap`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    def create_v_tunnel(self, y1, y2, x):
        ...

+   def place_entities(self, room, entities, max_monsters_per_room):
+       # Get a random number of monsters
+       number_of_monsters = randint(0, max_monsters_per_room)
+
+       for i in range(number_of_monsters):
+           # Choose a random location in the room
+           x = randint(room.x1 + 1, room.x2 - 1)
+           y = randint(room.y1 + 1, room.y2 - 1)
+
+           if not any([entity for entity in entities if entity.x == x and entity.y == y]):
+               if randint(0, 100) < 80:
+                   monster = Entity(x, y, 'o', libtcod.desaturated_green)
+               else:
+                   monster = Entity(x, y, 'T', libtcod.darker_green)
+
+               entities.append(monster)

    def is_blocked(self, x, y):
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    def create_v_tunnel(self, y1, y2, x):
        ...

    <span class="new-text">def place_entities(self, room, entities, max_monsters_per_room):
        # Get a random number of monsters
        number_of_monsters = randint(0, max_monsters_per_room)

        for i in range(number_of_monsters):
            # Choose a random location in the room
            x = randint(room.x1 + 1, room.x2 - 1)
            y = randint(room.y1 + 1, room.y2 - 1)

            if not any([entity for entity in entities if entity.x == x and entity.y == y]):
                if randint(0, 100) < 80:
                    monster = Entity(x, y, 'o', libtcod.desaturated_green)
                else:
                    monster = Entity(x, y, 'T', libtcod.darker_green)

                entities.append(monster)</span>

    def is_blocked(self, x, y):
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Dans cette fonction on choisit un nombre aléatoire d'ennemis à disposer, entre
0 et le maximum qu'on choisit. Ensuite on tire un x et y aléatoire et, si
aucun ennemi n'est déjà à cet endroit, on y place un monstre. Il y à 80% de
chance que ce soit un Orc et 20% de chance que ce soit un Troll.

Nous aurons besoin d'importer à la fois `libtcod` et la classe `Entity`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
+import tcod as libtcod
from random import randint

+from entity import Entity
from map_objects.rectangle import Rect
from map_objects.tile import Tile
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
    <pre><span class="new-text">import tcod as libtcod</span>
from random import randint

<span class="new-text">from entity import Entity</span>
from map_objects.rectangle import Rect
from map_objects.tile import Tile</pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant modifions notre fonction `make_map` pour y inclure la fonction
`place_entities`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
                        ...
                        self.create_h_tunnel(prev_x, new_x, new_y)

+               self.place_entities(new_room, entities, max_monsters_per_room)

                rooms.append(new_room)
                ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                        ...
                        self.create_h_tunnel(prev_x, new_x, new_y)

                <span class="new-text">self.place_entities(new_room, entities, max_monsters_per_room)</span>

                rooms.append(new_room)
                ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Parce que nous avons maintenant besoin des variables `entities` et
`max_monsters_per_room`, nous devrions modifier la définition de la fonction
`make_map` pour les y inclure.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-   def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player):
+   def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities,
+                max_monsters_per_room):
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    <span class="crossed-out-text">def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player):</span>
    <span class="new-text">def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities,
                 max_monsters_per_room):</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Tout est bon ici, maintenant nous devons modifier `engine.py` pour tenir compte
de cette nouvelle fonction `make_map`. Aussi nous aurons besoin de créer la
variable `max_room_per_monsters` avant d'appeler la fonction. Enfin nous
changerons notre liste `entities` pour qu'elle ne contienne que le joueur,
retirant complètement notre NPC d'exemple utilisé plus tôt.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    fov_radius = 10

+   max_monsters_per_room = 3

    colors = {
        'dark_wall': libtcod.Color(0, 0, 100),
        'dark_ground': libtcod.Color(50, 50, 150),
        'light_wall': libtcod.Color(130, 110, 50),
        'light_ground': libtcod.Color(200, 180, 50)
    }

-   player = Entity(int(screen_width / 2), int(screen_height / 2), '@', libtcod.white)
-   npc = Entity(int(screen_width / 2 - 5), int(screen_height / 2), '@', libtcod.yellow)
-   entities = [npc, player]
+   player = Entity(0, 0, '@', libtcod.white)
+   entities = [player]

    libtcod.console_set_custom_font('arial10x10.png', libtcod.FONT_TYPE_GREYSCALE | libtcod.FONT_LAYOUT_TCOD)

    libtcod.console_init_root(screen_width, screen_height, 'libtcod tutorial revised', False)

    con = libtcod.console_new(screen_width, screen_height)

    game_map = GameMap(map_width, map_height)
-   game_map.make_map(max_rooms, room_min_size, room_max_size, map_width, map_height, player)
+   game_map.make_map(max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities, max_monsters_per_room)

    fov_recompute = True
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    fov_radius = 10

    <span class="new-text">max_monsters_per_room = 3</span>

    colors = {
        'dark_wall': libtcod.Color(0, 0, 100),
        'dark_ground': libtcod.Color(50, 50, 150),
        'light_wall': libtcod.Color(130, 110, 50),
        'light_ground': libtcod.Color(200, 180, 50)
    }

    <span class="crossed-out-text">player = Entity(int(screen_width / 2), int(screen_height / 2), '@', libtcod.white)</span>
    <span class="crossed-out-text">npc = Entity(int(screen_width / 2 - 5), int(screen_height / 2), '@', libtcod.yellow)</span>
    <span class="crossed-out-text">entities = [npc, player]</span>
    <span class="new-text">player = Entity(0, 0, '@', libtcod.white)</span>
    <span class="new-text">entities = [player]</span>

    libtcod.console_set_custom_font('arial10x10.png', libtcod.FONT_TYPE_GREYSCALE | libtcod.FONT_LAYOUT_TCOD)

    libtcod.console_init_root(screen_width, screen_height, 'libtcod tutorial revised', False)

    con = libtcod.console_new(screen_width, screen_height)

    game_map = GameMap(map_width, map_height)
    <span class="crossed-out-text">game_map.make_map(max_rooms, room_min_size, room_max_size, map_width, map_height, player)</span>
    <span class="new-text">game_map.make_map(max_rooms, room_min_size, room_max_size, map_width, map_height, player, entities, max_monsters_per_room)</span>

    fov_recompute = True
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Lancez le projet maintenant et vous devriez voir quelques orcs et trolls peupler
notre donjon \!

Un problème évident avec les monstres qu'on vient d'inventer (au dela du fait
qu'ils sont totalement inanimés...) est que le joueur peut les traverser. À
moins que vous n'ayez l'intention de créer un jeu sur un fantôme qui traverse
des monstre pour en prendre possession (ça ne semble pas être une si mauvaise
idée \!), ce n'est pas ce que nous souhaitons. Si le joueur "se déplace dans" un
ennemi, on devrait l'attaquer !

On pourrait croire qu'il suffit de vérifier si on se déplacer dans une Entity et
l'attaquer si c'est le cas mais nous aurons besoin que certaines entités ne
bloquent pas le mouvement. Pourquoi ? Parce que nous utiliserons cette classe
Entity pour représenter les objets et nous voudrons marcher dessus pour les
ramasser. Aussi il semble qu'il nous faille un attribut de classe nous disant
si l'entité bloque le mouvement ou non.

Modifions la classe `Entity` pour y inclure la variable "block". Profitons de
cette occasion pour passer à la classe le nom ("name") pour l'entité. Nous
en aurons besoin d'ici peu.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class Entity:
-   def __init__(self, x, y, char, color):
+   def __init__(self, x, y, char, color, name, blocks=False):
        self.x = x
        self.y = y
        self.char = char
        self.color = color
+       self.name = name
+       self.blocks = blocks

    def move(self, dx, dy):
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class Entity:
    <span class="crossed-out-text">def __init__(self, x, y, char, color):</span>
    <span class="new-text">def __init__(self, x, y, char, color, name, blocks=False):</span>
        self.x = x
        self.y = y
        self.char = char
        self.color = color
        <span class="new-text">self.name = name
        self.blocks = blocks</span>

    def move(self, dx, dy):
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Remarquons que "blocks" est optionnel. Si nous ne la déclarons pas à
l'initialisation, elle sera False par défaut.

Revenons à `game_map.py` et modifions la méthode `place_entities` où nous
déclarons nos monstres.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            if randint(0, 100) < 80:
-               monster = Entity(x, y, 'o', libtcod.desaturated_green)
+               monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True)
            else:
-               monster = Entity(x, y, 'T', libtcod.darker_green)
+               monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            if randint(0, 100) < 80:
                <span class="crossed-out-text">monster = Entity(x, y, 'o', libtcod.desaturated_green)</span>
                <span class="new-text">monster = Entity(x, y, 'o', libtcod.desaturated_green, 'Orc', blocks=True)</span>
            else:
                <span class="crossed-out-text">monster = Entity(x, y, 'T', libtcod.darker_green)</span>
                <span class="new-text">monster = Entity(x, y, 'T', libtcod.darker_green, 'Troll', blocks=True)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous devons aussi mettre à jour l'initialisation du joueur dans `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-   player = Entity(0, 0, '@', libtcod.white)
+   player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    <span class="crossed-out-text">player = Entity(0, 0, '@', libtcod.white)</span>
    <span class="new-text">player = Entity(0, 0, '@', libtcod.white, 'Player', blocks=True)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Avec nos nouveaux attributs nous devons nous assurer qu'une entité ne bloque
pas le chemin quand nous essayons de nous déplacer sur une tuile. Cela nous
aiderait d'avoir une fonction qui renvoie l'entité bloquante en lui donnant
la liste des entités et les coordonnées x et y. Nous l'ajouterons à `entity.py`
mais pas à la classe `Entity` elle même. La raison est que c'est une fonction
en rapport avec les entités mais pas à une entité en particulier aussi elle
n'appartient pas à la classe.

Ajoutez la fonction à `entity.py` comme ceci :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class Entity:
    ...


+def get_blocking_entities_at_location(entities, destination_x, destination_y):
+   for entity in entities:
+       if entity.blocks and entity.x == destination_x and entity.y == destination_y:
+           return entity
+
+   return None
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class Entity:
    ...


<span class="new-text">def get_blocking_entities_at_location(entities, destination_x, destination_y):
    for entity in entities:
        if entity.blocks and entity.x == destination_x and entity.y == destination_y:
            return entity

    return None</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

La fonction boucle sur les entités et, si l'une est bloquante et placée aux
x et y indiqués, on la renvoie. Si aucune ne correspond on renvoie "None".
Remarquez que la fonction suppose qu'une seule entité est placée à cette
position. Cela ne devrait pas poser de problème car nous nous assurerons que
deux entités ne peuvent se déplacer sur une même tuile.

Ceci étant fait, revenons à notre fonction de déplacement. Modifiez le code
qui déplace le joueur dans `engine.py` comme ceci :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        if move:
            dx, dy = move

-           player.move(dx, dy)
-
-           fov_recompute = True

+           destination_x = player.x + dx
+           destination_y = player.y + dy

-           if not game_map.is_blocked(player.x + dx, player.y + dy):
+           if not game_map.is_blocked(destination_x, destination_y):
+               target = get_blocking_entities_at_location(entities, destination_x, destination_y)
+
+               if target:
+                   print('You kick the ' + target.name + ' in the shins, much to its annoyance!')
+               else:
+                   player.move(dx, dy)
+
+                   fov_recompute = True
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        if move:
            dx, dy = move
            <span class="new-text">destination_x = player.x + dx
            destination_y = player.y + dy</span>

            <span class="crossed-out-text">if not game_map.is_blocked(player.x + dx, player.y + dy):</span>
            <span class="new-text">if not game_map.is_blocked(destination_x, destination_y):</span>
                <span class="new-text">target = get_blocking_entities_at_location(entities, destination_x, destination_y)

                if target:
                    print('You kick the ' + target.name + ' in the shins, much to its annoyance!')
                else:</span>
                    <span style="color: blue">player.move(dx, dy)

                    fov_recompute = True</span>
</pre>
{{</ original-tab >}}
{{</ codetab >}}

Assurez vous d'importer la fonction `get_blocking_entities_at_location` en haut
de `engine.py`

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-   from entity import Entity
+   from entity import Entity, get_blocking_entities_at_location
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>from entity import Entity<span class="new-text">, get_blocking_entities_at_location</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant le joueur est bloqué quand il essaye de traverser une autre entité.
Nous affichons un texte humoristique (hey, je trouve ça drôle \!) pour
l'instant. Nous implémenterons un vrai combat dans le chapitre suivant.

Notre joueur ne devrait pouvoir se déplacer que durant son tour et ce principe
s'applique aux monstres. Nous aurons besoin d'une variable pour savoir à qui est
le tour. Nous pourrions conserver une chaîne dans cette variable comme
'players\_turn' ou 'enemy_turn' mais c'est succeptible de créer des erreurs. Si
vous faîtes une typo en écrivant ces chaîne, vous allez créer des bugs.
N'oublions pas que le nombre d'états du jeu va augmenter et nous aurons besoin
d'une meilleure manière de les enregistrer.

Nous allons enregistrer ces états avec un Enum. Un "Enum" est un ensemble de
valeurs qui ne changent pas et qu'on peut énumérer, parfait pour les états du
jeu. Créer un nouveau fichier appelé `game_states.py` et ajoutez-y la classe
suivante :

{{< highlight py3 >}}
from enum import Enum


class GameStates(Enum):
    PLAYERS_TURN = 1
    ENEMY_TURN = 2
{{</ highlight >}}

Cela rendra nos changements d'états du jeu plus faciles à gérer, en particulier
quand nous en aurons plus de deux.

*\* Remarque : les nombres associés n'ont pas de sens précis. En fait, si vous
Python 3.6 ou une version ultérieure, vous pouvez utiliser 'auto' pour
incrémenter un nombre automatiquement. Vérifiez si c'est possible pour vous.*

Mettons ces nouveaux enum `GameStates` en action. Commencez par les importer
en haut.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from fov_functions import initialize_fov, recompute_fov
+from game_states import GameStates
from input_handlers import handle_keys
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
    <pre>...
from fov_functions import initialize_fov, recompute_fov
<span class="new-text">from game_states import GameStates</span>
from input_handlers import handle_keys
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Enfin, créez une variable appelée `game_state` sera d'abord réglée sur le
tour du joueur.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    mouse = libtcod.Mouse()

+   game_state = GameStates.PLAYERS_TURN

    while not libtcod.console_is_window_closed():
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    mouse = libtcod.Mouse()

    <span class="new-text">game_state = GameStates.PLAYERS_TURN</span>

    while not libtcod.console_is_window_closed():
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Selon que ce soit le tour du joueur ou non, nous voulons contrôler le mouvement
du joueur. Le joueur ne peut se déplacer que durant son tour aussi modifions
notre section `if move:` pour en tenir compte. Une fois que notre joueur aura
réussi à se déplacer, nous allons passer l'état à `ENEMY_TURN`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-       if move:
+       if move and game_state == GameStates.PLAYERS_TURN:
            dx, dy = move
            destination_x = player.x + dx
            destination_y = player.y + dy

            if not game_map.is_blocked(destination_x, destination_y):
                target = get_blocking_entities_at_location(entities, destination_x, destination_y)

                if target:
                    print('You kick the ' + target.name + ' in the shins, much to its annoyance!')
                else:
                    player.move(dx, dy)

                    fov_recompute = True

+               game_state = GameStates.ENEMY_TURN
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        <span class="crossed-out-text">if move:</span>
        <span class="new-text">if move and game_state == GameStates.PLAYERS_TURN:</span>
            dx, dy = move
            destination_x = player.x + dx
            destination_y = player.y + dy

            if not game_map.is_blocked(destination_x, destination_y):
                target = get_blocking_entities_at_location(entities, destination_x, destination_y)

                if target:
                    print('You kick the ' + target.name + ' in the shins, much to its annoyance!')
                else:
                    player.move(dx, dy)

                    fov_recompute = True

                <span class="new-text">game_state = GameStates.ENEMY_TURN</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Si vous lancez le projet maintenant, votre joueur sera capable de se déplacer
une fois... et sera bloqué pour toujours. C'est parce que nous devons
implémenter les mouvements des ennemis et rendre le `game_state` au joueur
ensuite. Remarquez que vous *pouvez* quitter le jeu ou le passer en plein écran
parce que nous n'empéchons pas le joueur d'accomplir ces choses quand ça n'est
pas son tour.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        if fullscreen:
            libtcod.console_set_fullscreen(not libtcod.console_is_fullscreen())

+       if game_state == GameStates.ENEMY_TURN:
+           for entity in entities:
+               if entity != player:
+                   print('The ' + entity.name + ' ponders the meaning of its existence.')
+
+           game_state = GameStates.PLAYERS_TURN
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        if fullscreen:
            libtcod.console_set_fullscreen(not libtcod.console_is_fullscreen())

        <span class="new-text">if game_state == GameStates.ENEMY_TURN:
            for entity in entities:
                if entity != player:
                    print('The ' + entity.name + ' ponders the meaning of its existence.')

            game_state = GameStates.PLAYERS_TURN</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

C'est assez simple. Supposons que c'est le tour de l'ennemi, nous parcourons
chaque entité, à l'exception du joueur, et nous leur donnons le tour. Pour
l'instant nous n'avons pas d'AI pour nos ennemis donc ils restent immobiles à
contempler leurs vies. Dans le prochain chapitre nous leur donnerons un
comportement plus intéressant mais pour l'instant cela servira d'exemple.

Si vous voulez voir le code actuel entièrement, [cliquez ici](https://github.com/TStand90/roguelike_tutorial_revised/tree/part5).

[Cliquez ici pour vous rendre à la partie suivante de ce tutoriel.](/tutorials/tcod/part-6)

<script src="/js/codetabs.js"></script>
