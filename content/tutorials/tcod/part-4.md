---
title: "Partie 4 - Champ de vision"
date: 2019-03-30T09:33:44-07:00
draft: false
---

Nous avons un donjon maintenant et on peut s'y déplacer librement mais est-ce
vraiment de l'*exploration* si on peut le voir entièrement dès le départ ?

La plupart des roguelikes (pas tous \!) ne vous laissent voir qu'un certain
espace autour de votre personnage et le notre va respecter ce principe. Nous
devons implémenter une manière de calculer le "champ de vision" (FOV) de notre
aventurier et, heureusement, libtcod rend les choses aisées \!

Nous devons définir quelques variables avant d'avancer. Ajoutez les à la même
section que votre variable d'écran et de carte :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    max_rooms = 30

+   fov_algorithm = 0
+   fov_light_walls = True
+   fov_radius = 10

    colors = {
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    max_rooms = 30

    <span class="new-text">fov_algorithm = 0
    fov_light_walls = True
    fov_radius = 10</span>

    colors = {
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

'0' est juste l'algorithme utilisé par libtcod, il y en a d'autres et je vous
invite à les essayer plus tard. `fov_light_walls` nous indique s'il faut
éclairer ('light_up') les murs que nous voyons. Vous pouvez les changer si vous
n'aimez pas ce que vous voyez. `fov_radius` est plutôt évident et nous indique
à quelle distance on peut voir.

On doit aussi mettre à jour le dictionnaire `colors` parce que nous avons deux
couleurs supplémentaires pour la version éclairée ('light') des murs et du sol.
Les murs et le sol dans notre champ de vision seront éclairés de façon à les
distinguer de ceux qu'on ne peut voir.

{{< highlight py3 >}}
    colors = {
        'dark_wall': libtcod.Color(0, 0, 100),
        'dark_ground': libtcod.Color(50, 50, 150),
        'light_wall': libtcod.Color(130, 110, 50),
        'light_ground': libtcod.Color(200, 180, 50)
    }
{{</ highlight >}}

*\* N'oubliez pas d'ajouter la virgule apres le 'dark\_ground' sinon Python va
renvoyer une erreur !*

Si vous n'aimez pas ces couleurs, n'hésitez pas à les changer selon vos gouts.

Les choses à savoir à propos du champ de vision est qu'il n'a pas besoin d'être
calculé à chaque tour. Ce serait du gachis \! Nous devons seulement le changer
quand le joueur se déplace. Attaquer, utiliser un item ou seulement rester sur
place pour un tour ne change pas le FOV. Nous pouvons mettre ça en place avec
un booléen qu'on appellera `fov_recompute` et qui nous dira s'il faut recalculer.
On peut ensuite le définir quelque part avant la boucle de jeu (j'ai ajouté le
mien juste après l'initialisation de la carte).

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    game_map.make_map(max_rooms, room_min_size, room_max_size, map_width, map_height, player)

+   fov_recompute = True

    key = libtcod.Key()
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    game_map.make_map(max_rooms, room_min_size, room_max_size, map_width, map_height, player)

    <span class="new-text">fov_recompute = True</span>

    key = libtcod.Key()
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

C'est `True` par défaut parce nous devons le calculer juste après le lancement
du jeu.

Maintenant initialisons notre champ de vision, que nous garderons dans une
variable appelée `fov_map`. `fov_map` devra non seulement être initialisée mais
recalculée quand le joueur se déplace. Conservons ces fonctions hors de
`engine.py` et ajoutons les à un nouveau fichier, appelé `fov_functions.py`.
Dans ce fichier ajouter les éléments suivants :

{{< highlight py3 >}}
import tcod as libtcod


def initialize_fov(game_map):
    fov_map = libtcod.map_new(game_map.width, game_map.height)

    for y in range(game_map.height):
        for x in range(game_map.width):
            libtcod.map_set_properties(fov_map, x, y, not game_map.tiles[x][y].block_sight,
                                       not game_map.tiles[x][y].blocked)

    return fov_map
{{</ highlight >}}

Appelons cette fonction dans `engine.py` et conservons ce résultat dans
`fov_map`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    fov_recompute = True

+   fov_map = initialize_fov(game_map)

    key = libtcod.Key()
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    fov_recompute = True

    <span class="new-text">fov_map = initialize_fov(game_map)</span>

    key = libtcod.Key()
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

N'oubliez pas d'importer cette fonction.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from entity import Entity
+from fov_functions import initialize_fov
from input_handlers import handle_keys
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from entity import Entity
<span class="new-text">from fov_functions import initialize_fov</span>
from input_handlers import handle_keys
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Tant qu'on s'occupe de ça, modifiez la section où le joueur se déplace pour
régler `fov_recompute` à True.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
                ...
                player.move(dx, dy)

+               fov_recompute = True
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                ...
                player.move(dx, dy)

                <span class="new-text">fov_recompute = True</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Mais *où* se déroule ce calcul ? Pour ça, ajoutons une nouvelle fonction à
`fov_functions.py` pour faire ce recalcul. La fonction de recalcul va modifier
la variable `fov_map` en fonction de la position du joueur, du rayon de la
lumière ambiente, du fait d'éclairer les murs ou non et de l'algorithme qu'on
utilise.

Cela fait beaucoup de variables mais considérons ceci, dans votre jeu vous allez
choisir un algorithme de FOV et le conserver. Aussi, éclairer les murs ou non
ne changera pas durant le cours du jeu. Pourquoi ne pas créer notre fonction
avec des arguments par défaut ? Ainsi nous pourrons passer les variables
`light_walls` et `algorithm` si on le souhaite mais sinon, une valeur par défaut
est choisie. Cela ressemble à ça :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def initialize_fov(game_map):
    ...

+def recompute_fov(fov_map, x, y, radius, light_walls=True, algorithm=0):
+   libtcod.map_compute_fov(fov_map, x, y, radius, light_walls, algorithm)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>def initialize_fov(game_map):
    ...

<span class="new-text">def recompute_fov(fov_map, x, y, radius, light_walls=True, algorithm=0):
    libtcod.map_compute_fov(fov_map, x, y, radius, light_walls, algorithm)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Aussi quand on appelle la fonction, nous devons lui passer fov\_map, x, y et le
rayon mais il n'est pas indispensable de lui passer light\_walls ou algirithm.
Dans mon fichier `engine.py`, je lui passe néanmoins mais n'y êtes pas tenu
(vous pouvez aussi changer les valeurs par défaut pour celles que vous
souhatiez)

Quel que soit votre choix, mettez à jour votre recalcul du FOV dans `engine.py`
ainsi

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS, key, mouse)

+       if fov_recompute:
+           recompute_fov(fov_map, player.x, player.y, fov_radius, fov_light_walls, fov_algorithm)

        render_all(con, entities, game_map, screen_width, screen_height, colors)
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS, key, mouse)

        <span style="color:green">if fov_recompute:
            recompute_fov(fov_map, player.x, player.y, fov_radius, fov_light_walls, fov_algorithm)</span>

        render_all(con, entities, game_map, screen_width, screen_height, colors)
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

... Et, bien sûr, nous devons importer cette fonction :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
...
from entity import Entity
-from fov_functions import initialize_fov
+from fov_functions import initialize_fov, recompute_fov
from input_handlers import handle_keys
...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>...
from entity import Entity
from fov_functions import initialize_fov<span style="color: green;">, recompute_fov</span>
from input_handlers import handle_keys
...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant, une fois que le joueur se déplace convenablement, le champ de vision
sera réglé pour être recalculé mais rien ne se produira si nous n'ajoutons pas
quelque chose.

Notre champ de vision étant recalculé, nous devons *l'afficher* (si vous
exécutez le code maintenant, vous ne remarquerez aucun changement visible).
Ouvrez `render_functions.py` et changez la fonction `render_all` ainsi :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def render_all(con, entities, game_map, screen_width, screen_height, colors):
+def render_all(con, entities, game_map, fov_map, fov_recompute, screen_width, screen_height, colors):
-   for y in range(game_map.height):
+   if fov_recompute:
-       for x in range(game_map.width):
+       for y in range(game_map.height):
-           wall = game_map.tiles[x][y].block_sight
-
-           if wall:
-               libtcod.console_set_char_background(con, x, y, colors.get('dark_wall'), libtcod.BKGND_SET)
-           else:
-               libtcod.console_set_char_background(con, x, y, colors.get('dark_ground'), libtcod.BKGND_SET)
+           for x in range(game_map.width):
+               visible = libtcod.map_is_in_fov(fov_map, x, y)
+               wall = game_map.tiles[x][y].block_sight

+               if visible:
+                   if wall:
+                       libtcod.console_set_char_background(con, x, y, colors.get('light_wall'), libtcod.BKGND_SET)
+                   else:
+                       libtcod.console_set_char_background(con, x, y, colors.get('light_ground'), libtcod.BKGND_SET)
+               else:
+                   if wall:
+                       libtcod.console_set_char_background(con, x, y, colors.get('dark_wall'), libtcod.BKGND_SET)
+                   else:
+                       libtcod.console_set_char_background(con, x, y, colors.get('dark_ground'), libtcod.BKGND_SET)

    # Draw all entities in the list
    for entity in entities:
        draw_entity(con, entity)

    libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="crossed-out-text">def render_all(con, entities, game_map, screen_width, screen_height, colors):</span>
<span class="new-text">def render_all(con, entities, game_map, fov_map, fov_recompute, screen_width, screen_height, colors):</span>
    <span class="new-text">if fov_recompute:</span>
        <span style="color: blue">for y in range(game_map.height):
            for x in range(game_map.width):</span>
                <span class="new-text">visible = libtcod.map_is_in_fov(fov_map, x, y)</span>
                <span style="color: blue">wall = game_map.tiles[x][y].block_sight</span>

                <span class="new-text">if visible:
                    if wall:
                        libtcod.console_set_char_background(con, x, y, colors.get('light_wall'), libtcod.BKGND_SET)
                    else:
                        libtcod.console_set_char_background(con, x, y, colors.get('light_ground'), libtcod.BKGND_SET)
                else:</span>
                    <span style="color: blue">if wall:
                        libtcod.console_set_char_background(con, x, y, colors.get('dark_wall'), libtcod.BKGND_SET)
                    else:
                        libtcod.console_set_char_background(con, x, y, colors.get('dark_ground'), libtcod.BKGND_SET)</span>

    # Draw all entities in the list
    for entity in entities:
        draw_entity(con, entity)

    libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)</pre>
{{</ original-tab >}}
{{</ codetab >}}

*\*Remarque : le bleu est la couleur des lignes qui sont identiques aux
précédentes mais qui ont été indentées différemment. Les blocs if du
`fov_recompute` et `visible` augmentent l'indentation. Souvenez-vous, en Python
l'indentation est indispensable. \!*

Maintenant notre fonction `render_all` va afficher les tuiles différemment,
selon qu'elles soient dans notre champ de vision  ou non. Si une tuile est dans
`fov_map`, on la dessine avec la couleur `light` et sinon on la dessine avec la
version `dark`.

La définition de `render_all` a changé aussi assurez-vous de la mettre à jour
dans `engine.py`. Tant qu'on y est, réglons `fov_recompute à `False` après avoir
appelé `render_all`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
-       render_all(con, entities, game_map, screen_width, screen_height, colors)
+       render_all(con, entities, game_map, fov_map, fov_recompute, screen_width, screen_height, colors)
+
+       fov_recompute = False
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        <span class="crossed-out-text">render_all(con, entities, game_map, screen_width, screen_height, colors)</span>
        <span class="new-text">render_all(con, entities, game_map, fov_map, fov_recompute, screen_width, screen_height, colors)

        fov_recompute = False</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Run the project now. The player's field of view is now visible\! But,
despite being able to "see" the FOV, it still doesn't really *do*
anything. We can still see the entire map, along with our NPC. Luckily,
the changes we have to make to fix this are fairly minimal.

Lancez le projet maintenant. Le champ de vision du joueur est maintenant
visible \! Mais bien qu'on soit capable de "voir" le FOV, cela ne *change rien*.
On peut toujours voir l'intégralité de la carte ainsi que notre NPC.
Heureusement, les changements pour y parvenir sont minimaux.

Commençons avec le NPC. On devrait simplement être capable de modifier notre
fonction `draw_entity`pour tenir compte du champ de vision, ce qui devrait
résoudre notre problème.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
def draw_entity(con, entity):
+def draw_entity(con, entity, fov_map):
-   libtcod.console_set_default_foreground(con, entity.color)
-   libtcod.console_put_char(con, entity.x, entity.y, entity.char, libtcod.BKGND_NONE)
+   if libtcod.map_is_in_fov(fov_map, entity.x, entity.y):
+       libtcod.console_set_default_foreground(con, entity.color)
+       libtcod.console_put_char(con, entity.x, entity.y, entity.char, libtcod.BKGND_NONE)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="crossed-out-text">def draw_entity(con, entity):</span>
<span class="new-text">def draw_entity(con, entity, fov_map):
    if libtcod.map_is_in_fov(fov_map, entity.x, entity.y):</span>
        <span style="color: blue">libtcod.console_set_default_foreground(con, entity.color)
        libtcod.console_put_char(con, entity.x, entity.y, entity.char, libtcod.BKGND_NONE)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

*\* A nouveau, les lignes bleues contiennent du code inchangé hormis en ce qui
concerne son indentation.*

Aussi soyez sûrs de mettre à jour la partie où on appelle la fonction :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    for entity in entities:
-       draw_entity(con, entity)
+       draw_entity(con, entity, fov_map)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    for entity in entities:
        <span class="crossed-out-text">draw_entity(con, entity)</span>
        <span class="new-text">draw_entity(con, entity, fov_map)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Lancez le projet à nouveau et vous ne verrez plus le NPC à moins qu'il soit
dans votre champ de vision;

Maintenant la carte. Dans un roguelike traditionnel, votre joueur ne voit que
son champ de vision mais il se souvient des zones qu'il a déjà exploré. Nous
pouvons réaliser ça en ajoutant une variable `explored` à notre classe `Tile`.
Modifiez la fonction `__init__` dans `Tile` pour inclure cette nouvelle
variable :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        self.block_sight = block_sight

+       self.explored = False
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        self.block_sight = block_sight

        <span class="new-text">self.explored = False</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Cette nouvelle variable doit être prise en compte dans notre fonction
`render_all`. Faisons le immédiatement. On ne dessinera que les tuiles hors du
champ de vision que si on les déjà explorées. Aussi, chaque tuile figurant dans
notre champ de vision sera marquée comme explorée.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
                ...
                visible = libtcod.map_is_in_fov(fov_map, x, y)
                wall = game_map.tiles[x][y].block_sight

                if visible:
                    if wall:
                        libtcod.console_set_char_background(con, x, y, colors.get('light_wall'), libtcod.BKGND_SET)
                    else:
                        libtcod.console_set_char_background(con, x, y, colors.get('light_ground'), libtcod.BKGND_SET)

+                   game_map.tiles[x][y].explored = True
-               else:
+               elif game_map.tiles[x][y].explored:
                    if wall:
                        libtcod.console_set_char_background(con, x, y, colors.get('dark_wall'), libtcod.BKGND_SET)
                    else:
                        libtcod.console_set_char_background(con, x, y, colors.get('dark_ground'), libtcod.BKGND_SET)
                    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>                ...
                visible = libtcod.map_is_in_fov(fov_map, x, y)
                wall = game_map.tiles[x][y].block_sight

                if visible:
                    if wall:
                        libtcod.console_set_char_background(con, x, y, colors.get('light_wall'), libtcod.BKGND_SET)
                    else:
                        libtcod.console_set_char_background(con, x, y, colors.get('light_ground'), libtcod.BKGND_SET)

                    <span class="new-text">game_map.tiles[x][y].explored = True</span>
                <span class="crossed-out-text">else:</span>
                <span class="new-text">elif game_map.tiles[x][y].explored:</span>
                    if wall:
                        libtcod.console_set_char_background(con, x, y, colors.get('dark_wall'), libtcod.BKGND_SET)
                    else:
                        libtcod.console_set_char_background(con, x, y, colors.get('dark_ground'), libtcod.BKGND_SET)
                    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous avons maintenant un vrai donjon explorable \! Il est vrai qu'il n'y a
surement pas grand chose à y faire pour l'instant mais c'est un grand pas vers
un jeu fonctionnel. Dans les prochaines parties, nous remplirons le donjon
de monstres (hostiles ?) qu'on peut taper.


Si vous voulez voir le code actuel entièrement, [cliquez ici](https://github.com/TStand90/roguelike_tutorial_revised/tree/part4).

[Cliquez ici pour vous rendre à la partie suivante de ce tutoriel.](/tutorials/tcod/part-5)

<script src="/js/codetabs.js"></script>
