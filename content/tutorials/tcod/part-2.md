---
title: "Part 2 - Une entité générique, les fonctions de rendu et la carte]"
date: 2019-03-30T08:39:20-07:00
draft: false
---

Maintenant que l'on peut déplacer notre petit symbole "@", on doit lui donner un
*cadre*. Mais avant ça pensons un instant à l'objet joueur lui même.

Pour l'instant, nous représentons simplement le joueur avec un '@' et ses
coordonnées x et y. On devrait regrouper ces éléments dans un objet avec
d'autres données et des fonctions qui lui correspondent.

Créons une classe générique qui représente non seulement le joueur mais aussi
*tous les éléments* de notre monde de jeu. Ennemis, items et tout autre entité
à laquelle on pourrait penser feront parties de cette classe qui s'appellera
`Entity`.

Créez un nouveau fichier et nommez le `entity.py`. Dans ce fichier, ajouter la
classe suivante :

{{< highlight py3 >}}
class Entity:
    """
    A generic object to represent players, enemies, items, etc.
    """
    def __init__(self, x, y, char, color):
        self.x = x
        self.y = y
        self.char = char
        self.color = color

    def move(self, dx, dy):
        # Move the entity by a given amount
        self.x += dx
        self.y += dy
{{</ highlight >}}

C'est plutôt explicite. La classe `Entity` qui contient les coordonnées x et y
ainsi que le caractère (le symbole `@` en ce qui concerne le joueur) et la
couleur (blanche par défaut pour le joueur). Nous avons aussi une méthode
appelée `move` qui permettra à l'entité d'être déplacée selon des coordonnées
x et y données.

Mettons cette nouvelle classe en action \! Modifiez la première partie de
`engine.py` pour qu'elle ressemble à ceci :

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
import tcod as libtcod

+from entity import Entity
from input_handlers import handle_keys


def main():
    screen_width = 80
    screen_height = 50

-   player_x = int(screen_width / 2)
-   player_y = int(screen_height / 2)

+   player = Entity(int(screen_width / 2), int(screen_height / 2), '@', libtcod.white)
+   npc = Entity(int(screen_width / 2 - 5), int(screen_height / 2), '@', libtcod.yellow)
+   entities = [npc, player]
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

<span class="new-text">from entity import Entity</span>
from input_handlers import handle_keys


def main():
    screen_width = 80
    screen_height = 50

    <span class="crossed-out-text">player_x = int(screen_width / 2)</span>
    <span class="crossed-out-text">player_y = int(screen_height / 2)</span>

    <span class="new-text">player = Entity(int(screen_width / 2), int(screen_height / 2), '@', libtcod.white)
    npc = Entity(int(screen_width / 2 - 5), int(screen_height / 2), '@', libtcod.yellow)
    entities = [npc, player]</span>
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

On importe la classe `Entity` dans `engine.py` et on l'emploie pour initialiser
le joueur et un nouveau NPC. On range ces éléments dans une liste qui contiendra
toutes les entités sur la carte.

Modifiez aussi la partie qui gère le mouvement de façon à ce que ce soit la
classe Entity qui s'en occupe.

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
        if move:
            dx, dy = move
-           player_x += dx
-           player_x += dy
+           player.move(dx, dy)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        if move:
            dx, dy = move
            <span class="crossed-out-text">player_x += dx</span>
            <span class="crossed-out-text">player_x += dy</span>
            <span class="new-text">player.move(dx, dy)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Enfin, mettez à jour les fonctions de dessin pour utiliser le nouvel objet
joueur.

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
    while not libtcod.console_is_window_closed():
        libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS, key, mouse)

        libtcod.console_set_default_foreground(con, libtcod.white)
-       libtcod.console_put_char(con, player_x, player_y, '@', libtcod.BKGND_NONE)
+       libtcod.console_put_char(con, player.x, player.y, '@', libtcod.BKGND_NONE)
        libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)
        libtcod.console_set_default_foreground(0, libtcod.white)
-       libtcod.console_put_char(0, player_x, player_y, '@', libtcod.BKGND_NONE)
+       libtcod.console_put_char(0, player.x, player.y, '@', libtcod.BKGND_NONE)
        libtcod.console_flush()

-       libtcod.console_put_char(con, player_x, player_y, ' ', libtcod.BKGND_NONE)
-       libtcod.console_put_char(0, player_x, player_y, ' ', libtcod.BKGND_NONE)
+       libtcod.console_put_char(con, player.x, player.y, ' ', libtcod.BKGND_NONE)
+       libtcod.console_put_char(0, player.x, player.y, ' ', libtcod.BKGND_NONE)

        action = handle_keys(key)

{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    while not libtcod.console_is_window_closed():
        libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS, key, mouse)

        libtcod.console_set_default_foreground(con, libtcod.white)
        <span class="crossed-out-text">libtcod.console_put_char(con, player_x, player_y, '@', libtcod.BKGND_NONE)</span>
        <span class="new-text">libtcod.console_put_char(con, player.x, player.y, '@', libtcod.BKGND_NONE)</span>
        libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)
        libtcod.console_set_default_foreground(0, libtcod.white)
        <span class="crossed-out-text">libtcod.console_put_char(0, player_x, player_y, '@', libtcod.BKGND_NONE)</span>
        <span class="new-text">libtcod.console_put_char(0, player.x, player.y, '@', libtcod.BKGND_NONE)</span>
        libtcod.console_flush()

        <span class="crossed-out-text">libtcod.console_put_char(con, player_x, player_y, ' ', libtcod.BKGND_NONE)</span>
        <span class="crossed-out-text">libtcod.console_put_char(0, player_x, player_y, ' ', libtcod.BKGND_NONE)</span>
        <span class="new-text">libtcod.console_put_char(con, player.x, player.y, ' ', libtcod.BKGND_NONE)</span>
        <span class="new-text">libtcod.console_put_char(0, player.x, player.y, ' ', libtcod.BKGND_NONE)</span>

        action = handle_keys(key)
    </pre>
{{</ original-tab >}}
{{</ codetab >}}

On doit modifier la manière dont l'entité est dessinée à l'écran. Si vous
exécutez le code maintenant, seul le joueur est dessiné. Ecrivons quelques
fonctions qui dessinent à la fois le joueur mais aussi toute entité de la liste
des entités.

Créez un nouveau fichier appelé `render_functions.py`. Il contiendra nos
fonctions de dessin et une fonction nettoyant l'écran. Ajoutez le code suivant
dans ce fichier.

{{< highlight py3 >}}
import tcod as libtcod


def render_all(con, entities, screen_width, screen_height):
    # Draw all entities in the list
    for entity in entities:
        draw_entity(con, entity)

    libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)


def clear_all(con, entities):
    for entity in entities:
        clear_entity(con, entity)


def draw_entity(con, entity):
    libtcod.console_set_default_foreground(con, entity.color)
    libtcod.console_put_char(con, entity.x, entity.y, entity.char, libtcod.BKGND_NONE)


def clear_entity(con, entity):
    # erase the character that represents this object
    libtcod.console_put_char(con, entity.x, entity.y, ' ', libtcod.BKGND_NONE)
{{</ highlight >}}

Voici un découpage rapide de ce que font ces fonctions :

  - La fonction `render_all` est celle qui sera appelée de notre boucle de
    jeu pour dessiner les entités et, dans un instant, la carte. Pour l'instant
    elle prend la console (con), une liste d'entité et les dimensions
    (hauteur/largeur) de l'écran en paramètres et elle appelle la fonction
    `draw_entity` sur chaque élément. Ensuite elle colle (blit) les changements
    à l'écran.
  - `draw_entity` est ce qui réalise vraiment le dessin. Le code devrait être
    très proche de ce qui est dans notre boucle de jeu actuellement à ceci près
    qu'elle utilise les variables de l'entité (x, y, char et color) pour faire
    le dessin. Cela est assez flexible en théorie pour dessiner n'importe quelle
    entité qu'on lui donne.
  - `clear_all` et ce qu'on utilisera pour nettoyer les entités après les avoir
    dessinées à l'écran. C'est simplement une boucle qui appelle une autre
    fonction.
  - `clear_entity` est justement cette fonction. Elle nettoie les entités de
    l'écran (de façon à ce qu'elle bouge sans laisser une trace derrière elle).

Maitenant qu'on a quelques fonctions pour nous aider à dessiner les entités
mettons les en action. Réalisez les modifications dans la partie où le joueur
est dessiné dans `engine.py`

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
        ...
    libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS, key, mouse)

-   libtcod.console_set_default_foreground(con, libtcod.white)
-   libtcod.console_put_char(con, player.x, player.y, '@', libtcod.BKGND_NONE)
-   libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)
+   render_all(con, entities, screen_width, screen_height)

    libtcod.console_flush()

-   libtcod.console_put_char(con, player.x, player.y, ' ', libtcod.BKGND_NONE)
+   clear_all(con, entities)

    action = handle_keys(key)
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
    libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS, key, mouse)

    <span class="crossed-out-text">libtcod.console_set_default_foreground(con, libtcod.white)</span>
    <span class="crossed-out-text">libtcod.console_put_char(con, player.x, player.y, '@', libtcod.BKGND_NONE)</span>
    <span class="crossed-out-text">libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)</span>
    <span class="new-text">render_all(con, entities, screen_width, screen_height)</span>

    libtcod.console_flush()

    <span class="crossed-out-text">libtcod.console_put_char(con, player.x, player.y, ' ', libtcod.BKGND_NONE)</span>
    <span class="new-text">clear_all(con, entities)</span>

    action = handle_keys(key)
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

N'oubliez pas d'importez `render_all` et `clear_all` en haut de votre fichier.
Votre partie d'imports devrait ressemble à quelque chose comme cela :

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
import tcod as libtcod

from entity import Entity
from input_handlers import handle_keys
+from render_functions import clear_all, render_all
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

from entity import Entity
from input_handlers import handle_keys
<span class="new-text">from render_functions import clear_all, render_all</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Si vous exécutez le projet maintenant, vous devriez voir votre symbole '@'
accompagné d'un autre symbole jaune représentant notre NPC. Il ne fait rien
pour l'instant mais nous avons une méthode permettant dessiner plus d'un
personnage à l'écran.

Il est temps de changer de vitesse et de mettre la carte en place. La carte est
un tableau 2d d'objets Tile (tuile). Les tuiles aurons quelques propriétés qui
définissent si on peut voir à travers ou voir à travers.

On devrait commencer en définissant la taille de notre carte. Ajoutez ces
variables juste après avoir défini la hauteur et la largeur de l'écran.

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
    ...
    screen_height = 50
+   map_width = 80
+   map_height = 45
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    screen_height = 50
    <span class="new-text">map_width = 80
    map_height = 45</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Assez simple. On doit maintenant trouver une place pour notre classe Tile
et d'autres classes. Je préfère ranger les classes similaires dans le même
dossier donc créez un nouveau package Python (c'est un dossier avec un fichier
appelé `__init__.py`, ce fichier étant vide dans notre cas) appelé
`map_objects`. Dans ce dossier, crées un fichier `tile.py` et ajoutez-y le code
suivant.

{{< highlight py3 >}}
class Tile:
    """
    A tile on a map. It may or may not be blocked, and may or may not block sight.
    """
    def __init__(self, blocked, block_sight=None):
        self.blocked = blocked

        # By default, if a tile is blocked, it also blocks sight
        if block_sight is None:
            block_sight = blocked

        self.block_sight = block_sight

{{</ highlight >}}

Rien de très compliqué ici. La classe `Tile` contient l'information selon
laquelle elle est bloquante (blocked, si elle l'est, vous ne pouvez vous
déplacer dedans et l'information nous permettant de savoir si on peut voir à
travers (`block_sight` pour notre algorithme de champ de vision FOV). Remarquez
qu'il n'est pas nécessaire de passer `block_sight` à chaque fois ; ce
paramètre supposé être le même que `blocked`. En séparant les deux, une tuile
peut être transparente sans pouvoir être traversée (un puis de lave,
peut-être ?) ou inversement (une pièce sombre, par exemple).

Maintenant qu'on a une classe tuile, il nous faut un conteneur pour garder
nos tuiles. Créeons une classe `GameMap` qui contiendra notre tableau 2d de
tuiles ainsi que quelques méthodes pour régler et intéragir avec elles.
Créez un fichier dans le dossier map_objects et appelez le `game_map.py`.
Ajoutez-y le code suivant :

{{< highlight py3 >}}
from map_objects.tile import Tile


class GameMap:
    def __init__(self, width, height):
        self.width = width
        self.height = height
        self.tiles = self.initialize_tiles()

    def initialize_tiles(self):
        tiles = [[Tile(False) for y in range(self.height)] for x in range(self.width)]

        tiles[30][22].blocked = True
        tiles[30][22].block_sight = True
        tiles[31][22].blocked = True
        tiles[31][22].block_sight = True
        tiles[32][22].blocked = True
        tiles[32][22].block_sight = True

        return tiles
{{</ highlight >}}

On lui passe la largeur et la hauteur de la carte (définies dans notre moteur)
et on initialise un tableau 2d de tuiles, réglées sur non bloquantes par défaut.
On règle quelques tuiles comme étant bloquantes, pour démontrer le principe.
J'ai conservé les réglages des tuiles hors de la fonction `__init__` pour deux
raisons. D'une part parce qu'on peut l'appeler en dehors de l'initialisation,
d'autre part parce que je prefère avoir des fonctions `__init__` aussi simples
que possible.

Revenez sur `engine.py` où nous allons faire quelques changement pour
initialiser la carte et l'afficher à l'écran.

D'abord, nous définissons quelle couleur employer pour les tuiles bloquantes et
non bloquantes. Définissons un dictionnaire qui contient les couleurs qu'on
utilisera pour l'instant (il s'agrandira avec l'avancée du tutoriel).

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
    ...
    map_height = 45

+   colors = {
+       'dark_wall': libtcod.Color(0, 0, 100),
+       'dark_ground': libtcod.Color(50, 50, 150)
+   }

    player = Entity(int(screen_width / 2), int(screen_height / 2), '@', libtcod.white)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    map_height = 45

    <span class="new-text">colors = {
        'dark_wall': libtcod.Color(0, 0, 100),
        'dark_ground': libtcod.Color(50, 50, 150)
    }</span>

    player = Entity(int(screen_width / 2), int(screen_height / 2), '@', libtcod.white)</pre>
{{</ original-tab >}}
{{</ codetab >}}

Ces couleurs nous serviront pour les murs et le sol en dehors du champ de vision
quand on y sera (d'où le 'dark' de leurs noms).

Maintenant nous allons initialiser la carte de jeu elle même. Cela peut-être
placé n'importe où avant la boucle principale. J'ajoute la mienne juste en
dessous de l'initialisation de la console.

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
    con = libtcod.console_new(screen_width, screen_height)

+   game_map = GameMap(map_width, map_height)

    key = libtcod.Key()
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    con = libtcod.console_new(screen_width, screen_height)

    <span class="new-text">game_map = GameMap(map_width, map_height)</span>

    key = libtcod.Key()</pre>
{{</ original-tab >}}
{{</ codetab >}}

N'oublions pas d'importer l'objet GameMap de façon à l'utiliser dans le moteur.

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
from entity import Entity
from input_handlers import handle_keys
+from map_objects.game_map import GameMap
from render_functions import clear_all, render_all
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>from entity import Entity
from input_handlers import handle_keys
<span class="new-text">from map_objects.game_map import GameMap</span>
from render_functions import clear_all, render_all</pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant que notre objet carte est prêt passons le à `render_all` de façon
à le dessiner. Nous passerons aussi le dictionnaire `colors` parce que
`render_all` aura besoin de connaître les couleurs des éléments de la carte.
Remarquez que l'ordre dans lequel vous passez ces arguments n'a pas d'importance
cela doit simplement être cohérent avec la définition de la fonction quand vous
l'appelez.

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
-       render_all(con, entities, screen_width, screen_height)
+       render_all(con, entities, game_map, screen_width, screen_height, colors)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        <span class="crossed-out-text">render_all(con, entities, screen_width, screen_height)</span>
        <span class="new-text">render_all(con, entities, game_map, screen_width, screen_height, colors)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Ouvrez `render_functions.py` et modifiez `render_all` ainsi :

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
-def render_all(con, entities, screen_width, screen_height):
+def render_all(con, entities, game_map, screen_width, screen_height, colors):
+   # Draw all the tiles in the game map
+   for y in range(game_map.height):
+       for x in range(game_map.width):
+           wall = game_map.tiles[x][y].block_sight
+
+           if wall:
+               libtcod.console_set_char_background(con, x, y, colors.get('dark_wall'), libtcod.BKGND_SET)
+           else:
+               libtcod.console_set_char_background(con, x, y, colors.get('dark_ground'), libtcod.BKGND_SET)
+
    # Draw all entities in the list
    for entity in entities:
        draw_entity(con, entity)

    libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="crossed-out-text">def render_all(con, entities, screen_width, screen_height):</span>
<span class="new-text">def render_all(con, entities, game_map, screen_width, screen_height, colors):
    # Draw all the tiles in the game map
    for y in range(game_map.height):
        for x in range(game_map.width):
            wall = game_map.tiles[x][y].block_sight

            if wall:
                libtcod.console_set_char_background(con, x, y, colors.get('dark_wall'), libtcod.BKGND_SET)
            else:
                libtcod.console_set_char_background(con, x, y, colors.get('dark_ground'), libtcod.BKGND_SET)
    </span>
    # Draw all entities in the list
    for entity in entities:
        draw_entity(con, entity)

    libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)</pre>
{{</ original-tab >}}
{{</ codetab >}}

`render_all` boucle maintenant sur les tuiles de la carte de jeu et vérifie si
elles bloquent la vue ou non. Si c'est le cas, il dessine la tuile comme un mur
et sinon comme le sol.

Lancez le projet maintenant et vous devriez voir la carte dessinée avec des
couleurs. Vous verrez nos trois blocs de mur mais il y a un problème : vous
pouvez bouger à travers le mur !

Nous devons ajouter deux choses avant de conclure cette étape. Modifiez la
partie où la fonction de déplacement du joueur est appelée pour qu'elle
ressemble à ceci :

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
            if not game_map.is_blocked(player.x + dx, player.y + dy):
                player.move(dx, dy)
-           player.move(dx, dy)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            <span class="new-text">if not game_map.is_blocked(player.x + dx, player.y + dy):
                player.move(dx, dy)</span>
            <span class="crossed-out-text">player.move(dx, dy)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

*\* Remarquez le changement d'indentation pour `plaer.move(dx, dy)`. En Python
l'indentation est importante !*

Maintenant on doit simplement crée la méthode `is_blocked` dans la carte du jeu.
Ouvrez le fichier `game_map.py` et ajouter cette méthode à la classe :

{{< highlight py3 >}}
    def is_blocked(self, x, y):
        if self.tiles[x][y].blocked:
            return True

        return False
{{</ highlight >}}

*\* Remarque : vous pouvez raccourcir la fonction `is_blocked` en écrivant
simplement `return self.tiles[x][y].blocked` mais nous changerons cette fonction
pour qu'elle vérifie plus de choses plus tard aussi nous prenons un chemin
plus explicite.*

Lancez le projet à nouveau et vous serez bloqués par les murs.

Cela fera l'affaire pour ce tutoriel. Il n'y parait peut-être pas mais nous
avons fait ce qu'il faut pour créer un donjon réalise dans la prochaine partie.


Si vous voulez voir le code actuel entièrement, [cliquez ici](https://github.com/TStand90/roguelike_tutorial_revised/tree/part2).

[Cliquez ici pour vous rendre à la partie suivante de ce tutoriel.](/tutorials/tcod/part-2)

<script src="/js/codetabs.js"></script>
