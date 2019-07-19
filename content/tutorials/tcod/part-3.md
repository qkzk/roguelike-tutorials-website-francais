---
title: "Part 3 - Generating a dungeon"
date: 2019-03-30T08:39:22-07:00
draft: false
---

Bienvenue à nouveau dans le tutoriel Roguelike revisité \! Dans cette étape nous
allons franchir un pas **très** important vers un vrai jeu fonctionnel : créer
un donjon procédural !

Vous souvenez-vous du petit mur crée pour la démonstration dans la partie
précédente ? Nous n'en avons plus besoin aussi enlevons le.

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
-       tiles[30][22].blocked = True
-       tiles[30][22].block_sight = True
-       tiles[31][22].blocked = True
-       tiles[31][22].block_sight = True
-       tiles[32][22].blocked = True
-       tiles[32][22].block_sight = True
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        <span class="crossed-out-text">tiles[30][22].blocked = True</span>
        <span class="crossed-out-text">tiles[30][22].block_sight = True</span>
        <span class="crossed-out-text">tiles[31][22].blocked = True</span>
        <span class="crossed-out-text">tiles[31][22].block_sight = True</span>
        <span class="crossed-out-text">tiles[32][22].blocked = True</span>
        <span class="crossed-out-text">tiles[32][22].block_sight = True</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous avons aussi besoin de faire un petit changement à la liste par
compréhension qui crée nos tuiles.

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
-       tiles = [[Tile(False) for y in range(self.height)] for x in range(self.width)]
+       tiles = [[Tile(True) for y in range(self.height)] for x in range(self.width)]
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        <span class="crossed-out-text">tiles = [[Tile(False) for y in range(self.height)] for x in range(self.width)]</span>
        <span class="new-text">tiles = [[Tile(True) for y in range(self.height)] for x in range(self.width)]</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Pourquoi changer `False` en `True` ? Jusque là, nous réglions chaque tuile
pour être franchissable par défaut de façon à nous déplacer facilement.
Aussi nous passion `False` à la classe `Tile` de façon a rendre l'attribut
`blocked` en False.

Cependant notre algorithme de génération fonctionne à l'envers : on crée une
pièce remplie de murs et on creuse les sections alors qu'on avance. Aussi,
on initialise nos tuiles pour qu'elles bloquent par défaut. Pour information,
tous les algorithmes de génération que j'ai vu fonctionnent ainsi.

Avant d'attaquer l'algorithme nous devons faire une chose en plus : définir une
classe d'aide pour nos "cartes". Ce sera une classe basique qui contiendra un
peu d'information à propos des dimensions et que nous appellerons `Rect` (pour
rectangle). Créez un nouveau fichier dans le dossier `map_objects` et appelez-le
`rectangle.py`. Saisissez-y le code suivant.


{{< highlight py3 >}}
class Rect:
    def __init__(self, x, y, w, h):
        self.x1 = x
        self.y1 = y
        self.x2 = x + w
        self.y2 = y + h
{{</ highlight >}}

La fonction `__init__` prend les coordonnées x et y du coin supérieur gauche et
calcule le coin inférieur droit avec la largeur et la hauteur données en
paramètres w et h. Nous ajouterons plus de choses à cette classe dans peu de
temps mais c'est tout ce dont on a besoin pour commencer.

Maintenant, si nous voulons "creuser" un paquet de pièces pour créer notre
donjon, nous avons besoin d'une fonction pour créer une pièce. Cette fonction
doit prendre un argument, appelé `room` qui doit être de la classe `Rect` que
nous venons de créer. De x1 à x2 et de y1 à y2 nous voulons que chaque tuile
dans le `Rect` ne soit pas bloquante de façon à ce que le joueur puisse s'y
déplacer. Nous pouvons ajouter cette fonction dans la classe `GameMap` puisque
nous manipulerons la liste des tuiles de la carte.

Voici ce qu'on obtient dans cette fonction :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    def initialize_tiles(self):
        ...

+   def create_room(self, room):
+       # go through the tiles in the rectangle and make them passable
+       for x in range(room.x1 + 1, room.x2):
+           for y in range(room.y1 + 1, room.y2):
+               self.tiles[x][y].blocked = False
+               self.tiles[x][y].block_sight = False

    def is_blocked(self, x, y):
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    def initialize_tiles(self):
        ...

    <span class="new-text">def create_room(self, room):
        # go through the tiles in the rectangle and make them passable
        for x in range(room.x1 + 1, room.x2):
            for y in range(room.y1 + 1, room.y2):
                self.tiles[x][y].blocked = False
                self.tiles[x][y].block_sight = False</span>

    def is_blocked(self, x, y):
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

*\*Remarque : `initialize_tiles` et `is_blocked` sont réduites pour rendre les
choses concises.

Pourquoi les + 1 de room.x1 et room.y1 ? Pensons à ce que nous disons à notre
programme quand nous voulons une pièce aux coordonnées (1, 1) qui aille jusque
(6, 6). On pourrait supposer qu'on creuse une pièce comme celle-ci
(souvenez-vous que les listes sont indexées à partir de 0) donc (0, 0) est un
mur dans notre cas) :


```
  0 1 2 3 4 5 6 7
0 # # # # # # # #
1 # . . . . . . #
2 # . . . . . . #
3 # . . . . . . #
4 # . . . . . . #
5 # . . . . . . #
6 # . . . . . . #
7 # # # # # # # #
```

Tout cela est bel et bon mais que se passe-t-il si on ajoute une pièce juste
à côté ? Imaginons une pièce qui commence en (7, 1) et aille jusque (9, 6).

```
  0 1 2 3 4 5 6 7 8 9 10
0 # # # # # # # # # # #
1 # . . . . . . . . . #
2 # . . . . . . . . . #
3 # . . . . . . . . . #
4 # . . . . . . . . . #
5 # . . . . . . . . . #
6 # . . . . . . . . . #
7 # # # # # # # # # # #
```

Aucun mur ne les sépare \! Cela veut dire que si deux pièces sont côte à côte,
il n'y aura aucun mur entre elles \! Pour faire simple, notre fonction doit
tenir compte des murs quand on creuse une pièce. Ainsi si on a un rectangle de
coordonnées x1 = 1, x2 = 6, y1 = 1 et y2 = 6, alors la pièce devrait ressembler
à cela :

```
  0 1 2 3 4 5 6 7
0 # # # # # # # #
1 # # # # # # # #
2 # # . . . . # #
3 # # . . . . # #
4 # # . . . . # #
5 # # . . . . # #
6 # # # # # # # #
7 # # # # # # # #
```

Cela nous assure qu'on aura au moins une tuile de mur d'épaisseur entre les
pièces à moins qu'on souhaite créer des pièces qui se superposent. De façon à y
parvenir on ajoute + 1 à x1 et y1.

*\* Note: In case you're wondering, we don't subtract 1 from x2 and y2
because Python's range function does not include the 'end' value in its
range. For example, range(0, 10) would give us \[0, 1, 2, 3, 4, 5, 6, 7,
8, 9\].*

*\* Remarque : si vous vous posiez la question, on n'a pas besoin de soustraire
1 de x2 et y2 parce que la fonction range de Python n'inclut pas les valeurs
de fin dans son intervalle. Par exemple range(0, 10) nous donne \[0, 1, 2, 3, 4,
 5, 6, 7, 8, 9\].*

Créons des pièces \! Nous avons besoin d'une fonction dans `GameMap` pour
générer notre carte donc ajoutons en une :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    def initialize_tiles(self):
        ...

+   def make_map(self):
+       # Create two rooms for demonstration purposes
+       room1 = Rect(20, 15, 10, 15)
+       room2 = Rect(35, 15, 10, 15)
+
+       self.create_room(room1)
+       self.create_room(room2)

    def create_room(self, room):
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    def initialize_tiles(self):
        ...

    <span class="new-text">def make_map(self):
        # Create two rooms for demonstration purposes
        room1 = Rect(20, 15, 10, 15)
        room2 = Rect(35, 15, 10, 15)

        self.create_room(room1)
        self.create_room(room2)</span>

    def create_room(self, room):
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

On doit importer la classe `Rect` dans le fichier `game_map` de façon à ce que
ça fonctionne. En haut de votre fichier, modifiez votre section d'import :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
+from map_objects.rectangle import Rect
from map_objects.tile import Tile
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
        <pre><span class="new-text">from map_objects.rectangle import Rect</span>
from map_objects.tile import Tile</pre>
{{</ original-tab >}}
{{</ codetab >}}

Finally, modify `engine.py` to actually call the new `make_map`
function.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    game_map = GameMap(map_width, map_height)
+   game_map.make_map()
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
    <pre>    game_map = GameMap(map_width, map_height)
    <span class="new-text">game_map.make_map()</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

C'est le bon moment pour exécuter votre code et vous assurer que tout fonctionne
comme prévu. Les changements effectués mettent deux pièces d'exemple sur la
carte avec notre joueur au centre de l'une d'entre elle (notre pauvre NPC est
coincé dans un mur, cela dit).

Je pense que vous aurez remarqué que les pièces ne sont pas reliées. Quel est
l'intérêt d'avoir un donjon si on est enfermé dans une pièce ? Pas d'inquiétude,
écrivons un peu de code pour créer un tunnel d'une pièce à l'autre. Ajoutez la
méthode suivante à `GameMap` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    def create_room(self, room):
        ...

+   def create_h_tunnel(self, x1, x2, y):
+       for x in range(min(x1, x2), max(x1, x2) + 1):
+           self.tiles[x][y].blocked = False
+           self.tiles[x][y].block_sight = False
+
+   def create_v_tunnel(self, y1, y2, x):
+       for y in range(min(y1, y2), max(y1, y2) + 1):
+           self.tiles[x][y].blocked = False
+           self.tiles[x][y].block_sight = False

    def is_blocked(self, x, y):
        ...

{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    def create_room(self, room):
        ...

    <span class="new-text">def create_h_tunnel(self, x1, x2, y):
        for x in range(min(x1, x2), max(x1, x2) + 1):
            self.tiles[x][y].blocked = False
            self.tiles[x][y].block_sight = False

    def create_v_tunnel(self, y1, y2, x):
        for y in range(min(y1, y2), max(y1, y2) + 1):
            self.tiles[x][y].blocked = False
            self.tiles[x][y].block_sight = False</span>

    def is_blocked(self, x, y):
        ...
        </pre>
{{</ original-tab >}}
{{</ codetab >}}

Let's put this code to use by drawing a tunnel between our two rooms.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        self.create_room(room2)

+       self.create_h_tunnel(25, 40, 23)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        self.create_room(room2)

        <span class="new-text">self.create_h_tunnel(25, 40, 23)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant qu'on a démontré que nos fonctions de pièces et de tunnels
fonctionnent comme prévues, il est temps de passer à un vrai algorithme de
génération de donjons. Le notre sera plutôt simple : on place des pièces une à
la fois en nous assurant qu'elles ne se superposent pas et on les relie avec des
tunnels.

Nous aurons besoin de deux fonctions dans la classe `Rect` pour nous assurer que
les deux rectangles (rooms) ne se superposent pas. Saisissez les méthodes dans
la classe `Rect` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
class Rect:
    def __init__(self, x, y, w, h):
        self.x1 = x
        self.y1 = y
        self.x2 = x + w
        self.y2 = y + h

+   def center(self):
+       center_x = int((self.x1 + self.x2) / 2)
+       center_y = int((self.y1 + self.y2) / 2)
+       return (center_x, center_y)
+
+   def intersect(self, other):
+       # returns true if this rectangle intersects with another one
+       return (self.x1 <= other.x2 and self.x2 >= other.x1 and
+               self.y1 <= other.y2 and self.y2 >= other.y1)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>class Rect:
    def __init__(self, x, y, w, h):
        self.x1 = x
        self.y1 = y
        self.x2 = x + w
        self.y2 = y + h

    <span class="new-text">def center(self):
        center_x = int((self.x1 + self.x2) / 2)
        center_y = int((self.y1 + self.y2) / 2)
        return (center_x, center_y)

    def intersect(self, other):
        # returns true if this rectangle intersects with another one
        return (self.x1 <= other.x2 and self.x2 >= other.x1 and
                self.y1 <= other.y2 and self.y2 >= other.y1)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Ne vous souciez pas trop des détails ici. Sachez simplement que la méthode
'center' renvoie le point central d'un rectangle et qu'`intersect` nous indique
si deux rectangles se rencontrent.

Nous aurons besoin de quelques variables pour régler les dimensions minimales
des pièces ainsi que le nombre maximal de pièces qu'un étage peut contenir.
Ajoutez les éléments suivants à `engine.py`

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    map_height = 45

+   room_max_size = 10
+   room_min_size = 6
+   max_rooms = 30

    colors = {
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    map_height = 45

    <span class="new-text">room_max_size = 10
    room_min_size = 6
    max_rooms = 30</span>

    colors = {
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Enfin, il est temps de modifier `make_map` pour créer notre donjon \!
Vous pouvez enlever complètement notre ancienne implémentation et la remplacer
par la suivante :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-   def make_map(self):
+   def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player):
-       room1 = Rect(20, 15, 10, 15)
-       room2 = Rect(35, 15, 10, 15)

-       self.create_room(room1)
-       self.create_room(room2)

-       self.create_h_tunnel(25, 40, 23)

+       rooms = []
+       num_rooms = 0
+
+       for r in range(max_rooms):
+           # random width and height
+           w = randint(room_min_size, room_max_size)
+           h = randint(room_min_size, room_max_size)
+           # random position without going out of the boundaries of the map
+           x = randint(0, map_width - w - 1)
+           y = randint(0, map_height - h - 1)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    <span class="crossed-out-text">def make_map(self):</span>
    <span class="new-text">def make_map(self, max_rooms, room_min_size, room_max_size, map_width, map_height, player):</span>
        <span class="crossed-out-text">room1 = Rect(20, 15, 10, 15)</span>
        <span class="crossed-out-text">room2 = Rect(35, 15, 10, 15)</span>

        <span class="crossed-out-text">self.create_room(room1)</span>
        <span class="crossed-out-text">self.create_room(room2)</span>

        <span class="crossed-out-text">self.create_h_tunnel(25, 40, 23)</span>

        <span class="new-text">rooms = []
        num_rooms = 0

        for r in range(max_rooms):
            # random width and height
            w = randint(room_min_size, room_max_size)
            h = randint(room_min_size, room_max_size)
            # random position without going out of the boundaries of the map
            x = randint(0, map_width - w - 1)
            y = randint(0, map_height - h - 1)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Les variables que nous créons ici seront celles que nous utiliserons pour
créer nos pièces dans un instant. `randint` nous donne un entier aléatoire entre
les valeurs indiquées. Dans notre cas nous voulons que que la largeur et la
hauteur soient entre les minimums et maximums et que notre x et y soient entre
les bornes de la carte.

Nous devons aussi importer `randint` de `random` en haut du fichier.
Votre section d'import pour `game_map.py` devrait maintenant ressembler à
quelque chose comme ceci :

{{< highlight py3 >}}
from random import randint

from map_objects.rectangle import Rect
from map_objects.tile import Tile
{{</ highlight >}}

Dernière étape avant d'avancer : nous devons mettre à jour l'appel de `make_map`
dans `engine.py`, nous utilisons des variables qui n'existaient pas jusque là.
Modifiez le pour qu'il ressemble à :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    game_map = GameMap(map_width, map_height)
-   game_map.make_map()
+   game_map.make_map(max_rooms, room_min_size, room_max_size, map_width, map_height, player)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    game_map = GameMap(map_width, map_height)
    <span class="crossed-out-text">game_map.make_map()</span>
    <span class="new-text">game_map.make_map(max_rooms, room_min_size, room_max_size, map_width, map_height, player)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant nous allons mettre notre classe `Rect` en action en lui passant les
variables créees. Ensuite, nous pourrons vérifier s'il rencontre une autre
pièce. Si c'est le cas, nous ne voulons pas l'ajouter aux pièces et on s'en
débarrasse simplement.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            ...
            y = randint(0, map_height - h - 1)

+           # "Rect" class makes rectangles easier to work with
+           new_room = Rect(x, y, w, h)
+
+           # run through the other rooms and see if they intersect with this one
+           for other_room in rooms:
+               if new_room.intersect(other_room):
+                   break
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            y = randint(0, map_height - h - 1)

            <span class="new-text"># "Rect" class makes rectangles easier to work with
            new_room = Rect(x, y, w, h)

            # run through the other rooms and see if they intersect with this one
            for other_room in rooms:
                if new_room.intersect(other_room):
                    break</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Si la pièce *n'en rencontre pas* d'autre alors nous devons la créer. Plutôt que
d'introduire un booléen (True/False) pour garder ça en mémoire, on peut
simplement utiliser une expression for-else \! C'est une particularité
spécifique et méconnue de Python qui dit simplement "si la boucle n'a pas été
interrompue par un 'break', alors fait ceci". Nous ajoutons notre code de
construction de la pièce dans l'expression 'else' juste après la boucle 'for'.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            ...
            for other_room in rooms:
                if new_room.intersect(other_room):
                    break
+           else:
+               # this means there are no intersections, so this room is valid
+
+               # "paint" it to the map's tiles
+               self.create_room(new_room)
+
+               # center coordinates of new room, will be useful later
+               (new_x, new_y) = new_room.center()
+
+               if num_rooms == 0:
+                   # this is the first room, where the player starts at
+                   player.x = new_x
+                   player.y = new_y
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            for other_room in rooms:
                if new_room.intersect(other_room):
                    break
            <span class="new-text">else:
                # this means there are no intersections, so this room is valid

                # "paint" it to the map's tiles
                self.create_room(new_room)

                # center coordinates of new room, will be useful later
                (new_x, new_y) = new_room.center()

                if num_rooms == 0:
                    # this is the first room, where the player starts at
                    player.x = new_x
                    player.y = new_y</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous créons la pièce et conservons les coordonnées de son centre. Si c'est la
première pièce créée on y place le joueur en son centre. Nous allons utiliser
ces coordonnés de centre dans un instant pour créer nos tunnels.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
                ...
                if num_rooms == 0:
                    # this is the first room, where the player starts at
                    player.x = new_x
                    player.y = new_y
+               else:
+                   # all rooms after the first:
+                   # connect it to the previous room with a tunnel
+
+                   # center coordinates of previous room
+                   (prev_x, prev_y) = rooms[num_rooms - 1].center()
+
+                   # flip a coin (random number that is either 0 or 1)
+                   if randint(0, 1) == 1:
+                       # first move horizontally, then vertically
+                       self.create_h_tunnel(prev_x, new_x, prev_y)
+                       self.create_v_tunnel(prev_y, new_y, new_x)
+                   else:
+                       # first move vertically, then horizontally
+                       self.create_v_tunnel(prev_y, new_y, prev_x)
+                       self.create_h_tunnel(prev_x, new_x, new_y)
+
+               # finally, append the new room to the list
+               rooms.append(new_room)
+               num_rooms += 1
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                ...
                if num_rooms == 0:
                    # this is the first room, where the player starts at
                    player.x = new_x
                    player.y = new_y
                <span class="new-text">else:
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

                # finally, append the new room to the list
                rooms.append(new_room)
                num_rooms += 1</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Ce bloc 'else' traite tous les cas où nous avons déjà crée au moins une pièce.
De manière à pouvoir parcourir notre donjon, on doit s'assurer que les tunnels
soient bien connectés. On récupère le centre de la pièce précédente et, selon
un choix aléatoire (entre pile ou face, si vous voulez), on creuse notre tunnel
verticalement puis horizontalement ou le contraire. Une fois tout ceci réalisé
on ajoute la pièce à notre liste de pièces 'rooms' et on incrémente le nombre
de pièces.

Et voilà \! Voici notre algorithme de génération de donjons, plutôt simple, mais
qui fonctionne. Lancez le projet et vous devriez vous trouver dans un donjon
procédural \! Remarquez que le NPC n'est pas placé intelligemment et peut être
bloqué dans un mur ou non.

Si vous voulez voir le code actuel entièrement, [cliquez ici](https://github.com/TStand90/roguelike_tutorial_revised/tree/part3).

[Cliquez ici pour vous rendre à la partie suivante de ce tutoriel.](/tutorials/tcod/part-4)

<script src="/js/codetabs.js"></script>
