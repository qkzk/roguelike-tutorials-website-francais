---
title: "Part 7 - Créer une interface"
date: 2019-03-30T09:33:53-07:00
draft: false
---

Avec chaque chapitre notre jeu est un peu plus jouable mais avant de progresser
sur le gameplay, nous devrions prendre un moment pour nous concentrer sur
l'aspect *esthétique*. Contrairement à ce que les traditionalistes du roguelike
pourraient vous dire, une bonne UI est pratique.

Commençons par la section sur l'HP. Avec relativement peu de code nous pouvons
ajouter une petite barre de vie qui nous dira combien il reste de santé au
joueur avant de mourir. Commençons par ajouter quelques variables utiles dans
`engine.py`

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    screen_height = 50

+   bar_width = 20
+   panel_height = 7
+   panel_y = screen_height - panel_height

    map_width = 80
-   map_height = 45
+   map_height = 43
    ...
    con = libtcod.console_new(screen_width, screen_height)
+   panel = libtcod.console_new(screen_width, panel_height)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    screen_height = 50

    <span class="new-text">bar_width = 20
    panel_height = 7
    panel_y = screen_height - panel_height</span>

    map_width = 80
    <span class="crossed-out-text">map_height = 45</span>
    <span class="new-text">map_height = 43</span>
    ...
    con = libtcod.console_new(screen_width, screen_height)
    <span class="new-text">panel = libtcod.console_new(screen_width, panel_height)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Nous créons une nouvelle console, `panel`, qui contiendra notre barre de vie
et le journal des messages. Nous avons aussi modifié la hauteur de la carte
afin de laisser un peu de place à notre barre de vie et notre futur journal de
message.

Maintenant il nous faut une faut une fonction qui dessine la barre de vie et
n'importe quelle barre qu'on puisse souhaiter. Vous pouvez ajouter une barre
de Mana ou de Stamina plus tard si vous le souhaitez aussi il est préférable
de la rendre la plus réutilisable possible. Ajoutez la suite à
`render_functions.py`, juste en dessous de l'enum `RenderOrder` mais au dessus
de `render_all`.

{{< highlight py3 >}}
def render_bar(panel, x, y, total_width, name, value, maximum, bar_color, back_color):
    bar_width = int(float(value) / maximum * total_width)

    libtcod.console_set_default_background(panel, back_color)
    libtcod.console_rect(panel, x, y, total_width, 1, False, libtcod.BKGND_SCREEN)

    libtcod.console_set_default_background(panel, bar_color)
    if bar_width > 0:
        libtcod.console_rect(panel, x, y, bar_width, 1, False, libtcod.BKGND_SCREEN)

    libtcod.console_set_default_foreground(panel, libtcod.white)
    libtcod.console_print_ex(panel, int(x + total_width / 2), y, libtcod.BKGND_NONE, libtcod.CENTER,
                             '{0}: {1}/{2}'.format(name, value, maximum))
{{</ highlight >}}

Maintenant utilisons cette fonction dans `render_all`. Retirez l'indication de
HP qu'on a ajouté plus tôt et ajoutez le code pour le panneau de statistiques
à la fin de la fonction.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-def render_all(con, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height, colors):
+def render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height, bar_width,
+              panel_height, panel_y, colors):
            ...
-   libtcod.console_set_default_foreground(con, libtcod.white)
-   libtcod.console_print_ex(con, 1, screen_height - 2, libtcod.BKGND_NONE, libtcod.LEFT,
-                            'HP: {0:02}/{1:02}'.format(player.fighter.hp, player.fighter.max_hp))

    libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)

+   libtcod.console_set_default_background(panel, libtcod.black)
+   libtcod.console_clear(panel)
+
+   render_bar(panel, 1, 1, bar_width, 'HP', player.fighter.hp, player.fighter.max_hp,
+              libtcod.light_red, libtcod.darker_red)
+
+   libtcod.console_blit(panel, 0, 0, screen_width, panel_height, 0, 0, panel_y)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="crossed-out-text">def render_all(con, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height, colors):</span>
<span class="new-text">def render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height, bar_width,
               panel_height, panel_y, colors):</span>
            ...
    <span class="crossed-out-text">libtcod.console_set_default_foreground(con, libtcod.white)</span>
    <span class="crossed-out-text">libtcod.console_print_ex(con, 1, screen_height - 2, libtcod.BKGND_NONE, libtcod.LEFT,</span>
                             <span class="crossed-out-text">'HP: {0:02}/{1:02}'.format(player.fighter.hp, player.fighter.max_hp))</span>

    libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)

    <span class="new-text">libtcod.console_set_default_background(panel, libtcod.black)
    libtcod.console_clear(panel)

    render_bar(panel, 1, 1, bar_width, 'HP', player.fighter.hp, player.fighter.max_hp,
               libtcod.light_red, libtcod.darker_red)

    libtcod.console_blit(panel, 0, 0, screen_width, panel_height, 0, 0, panel_y)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Ajoutez l'appel à `render_all` dans `engine.py`.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-       render_all(con, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height, colors)
+       render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height,
+                  bar_width, panel_height, panel_y, colors)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        <span class="crossed-out-text">render_all(con, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height, colors)</span>
        <span class="new-text">render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height,
                   bar_width, panel_height, panel_y, colors)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant on a une jolie barre de vie en bas de l'écran. Elle va décroître
quand le joueur perd des HP et croître quand on se soigne (ça arrive au prochain
chapitre).

Continuons d'avancer et créons un journal de messages. Ajouter les variables
suivantes à `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    ...
    panel_y = screen_height - panel_height

+   message_x = bar_width + 2
+   message_width = screen_width - bar_width - 2
+   message_height = panel_height - 1

    map_width = 80
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    panel_y = screen_height - panel_height

    <span class="new-text">message_x = bar_width + 2
    message_width = screen_width - bar_width - 2
    message_height = panel_height - 1</span>

    map_width = 80
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Pour implémenter un journal de message, nous avons besoin de deux classes :
une pour le journal et une pour les messages qu'il contient. Commencez par
créer un nouveau fichier appelé `game_messages.py`. Ajoutez-y le code suivant :

{{< highlight py3 >}}
import tcod as libtcod

import textwrap


class Message:
    def __init__(self, text, color=libtcod.white):
        self.text = text
        self.color = color


class MessageLog:
    def __init__(self, x, width, height):
        self.messages = []
        self.x = x
        self.width = width
        self.height = height

    def add_message(self, message):
        # Split the message if necessary, among multiple lines
        new_msg_lines = textwrap.wrap(message.text, self.width)

        for line in new_msg_lines:
            # If the buffer is full, remove the first line to make room for the new one
            if len(self.messages) == self.height:
                del self.messages[0]

            # Add the new line as a Message object, with the text and the color
            self.messages.append(Message(line, message.color))
{{< /highlight >}}

Cela fait beaucoup aussi examinons en détail.

`Message` est plutôt simple. On enregistre le message et la couleur associée.
Vous pouvez décider de ne pas passer de couleur, auquel cas, le blanc est
utilisé par défaut.

La classe `MessageLog` est la plus intéressante. Elle conserve une liste de
messages (de la classe `Message`), conserve les coordonnées en x (par commodité)
et sans hauteur et largeur. Hauteur et largeur sont pratiques pour savoir
quand couper le message du haut (les messages vont "défiler" avec l'arrivée de
nouveaux messages).

Dans la méthode `add_message`, on sépare le texte des messages en de multiples
lignes si c'est nécessaire, en utilisant la fonction `textwrap.wrap`. On peut
ensuite vérifier si le message et rempli et, si nécessaire, on efface la ligne
du haut. Enfin, on ajoute le nouveau message.

Commençons par mettre en place le nouveau journal de messages. Ajoutez un
nouveau journal à `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
    fov_map = initialize_fov(game_map)

+   message_log = MessageLog(message_x, message_width, message_height)

    key = libtcod.Key()
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    fov_map = initialize_fov(game_map)

    <span class="new-text">message_log = MessageLog(message_x, message_width, message_height)</span>

    key = libtcod.Key()</pre>
{{</ original-tab >}}
{{</ codetab >}}

Souvenez-vous d'importer aussi le `MessaegLog` en haut :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
from fov_functions import initialize_fov, recompute_fov
+from game_messages import MessageLog
from game_states import GameStates
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>from fov_functions import initialize_fov, recompute_fov
<span class="new-text">from game_messages import MessageLog</span>
from game_states import GameStates</pre>
{{</ original-tab >}}
{{</ codetab >}}

Notre journal de message étant implémenté, parcourons le projet pour retirer
les expressions `print` et les remplacer par des messages log.

Commençons par les fonctions 'death'. Dans `death_functions.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
import tcod as libtcod

+from game_messages import Message

from game_states import GameStates

from render_functions import RenderOrder


def kill_player(player):
    player.char = '%'
    player.color = libtcod.dark_red

-   return 'You died!', GameStates.PLAYER_DEAD
+   return Message('You died!', libtcod.red), GameStates.PLAYER_DEAD


def kill_monster(monster):
-   death_message = '{0} is dead!'.format(monster.name.capitalize())
+   death_message = Message('{0} is dead!'.format(monster.name.capitalize()), libtcod.orange)
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>import tcod as libtcod

<span class="new-text">from game_messages import Message</span>

from game_states import GameStates

from render_functions import RenderOrder


def kill_player(player):
    player.char = '%'
    player.color = libtcod.dark_red

    <span class="crossed-out-text">return 'You died!', GameStates.PLAYER_DEAD</span>
    <span class="new-text">return Message('You died!', libtcod.red), GameStates.PLAYER_DEAD</span>


def kill_monster(monster):
    <span class="crossed-out-text">death_message = '{0} is dead!'.format(monster.name.capitalize())</span>
    <span class="new-text">death_message = Message('{0} is dead!'.format(monster.name.capitalize()), libtcod.orange)</span>
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Ensuite, revenez dans `engine.py` et remplacez les expressions `print` comme
ceci :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
             ...
            (In the player's results loop)
            ...
            if dead_entity:
                if dead_entity == player:
                    message, game_state = kill_player(dead_entity)
                else:
                    message = kill_monster(dead_entity)

-               print(message)
+               message_log.add_message(message)
            ...
            (In the enemy results loop)
            ...
                        if dead_entity:
                            if dead_entity == player:
                                message, game_state = kill_player(dead_entity)
                            else:
                                message = kill_monster(dead_entity)

-                           print(message)
+                           message_log.add_message(message)
                            ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>             ...
            (In the player's results loop)
            ...
            if dead_entity:
                if dead_entity == player:
                    message, game_state = kill_player(dead_entity)
                else:
                    message = kill_monster(dead_entity)

                <span class="crossed-out-text">print(message)</span>
                <span class="new-text">message_log.add_message(message)</span>
            ...
            (In the enemy results loop)
            ...
                        if dead_entity:
                            if dead_entity == player:
                                message, game_state = kill_player(dead_entity)
                            else:
                                message = kill_monster(dead_entity)

                            <span class="crossed-out-text">print(message)</span>
                            <span class="new-text">message_log.add_message(message)</span>
                            ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant pour nos messages d'action, dans `fighter.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
        ...
        if damage > 0:
-           results.append({'message': '{0} attacks {1} for {2} hit points.'.format(self.owner.name.capitalize(),
-                                                                                   target.name, str(damage))})
+           results.append({'message': Message('{0} attacks {1} for {2} hit points.'.format(
+               self.owner.name.capitalize(), target.name, str(damage)), libtcod.white)})
            results.extend(target.fighter.take_damage(damage))
        else:
-           results.append({'message': '{0} attacks {1} but does no damage.'.format(self.owner.name.capitalize(),
-                                                                                   target.name)})
+           results.append({'message': Message('{0} attacks {1} but does no damage.'.format(
+               self.owner.name.capitalize(), target.name), libtcod.white)})

        return results
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        if damage > 0:
            <span class="crossed-out-text">results.append({'message': '{0} attacks {1} for {2} hit points.'.format(self.owner.name.capitalize(),</span>
                                                                                    <span class="crossed-out-text">target.name, str(damage))})</span>
            <span class="new-text">results.append({'message': Message('{0} attacks {1} for {2} hit points.'.format(
                self.owner.name.capitalize(), target.name, str(damage)), libtcod.white)})</span>
            results.extend(target.fighter.take_damage(damage))
        else:
            <span class="crossed-out-text">results.append({'message': '{0} attacks {1} but does no damage.'.format(self.owner.name.capitalize(),</span>
                                                                                    <span class="crossed-out-text">target.name)})</span>
            <span class="new-text">results.append({'message': Message('{0} attacks {1} but does no damage.'.format(
                self.owner.name.capitalize(), target.name), libtcod.white)})</span>

        return results</pre>
{{</ original-tab >}}
{{</ codetab >}}

Vous devez importer à la fois libtcod et Message pour que cela fonctionne :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
+import tcod as libtcod

+from game_messages import Message


class Fighter:
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="new-text">import tcod as libtcod

from game_messages import Message</span>


class Fighter:
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Et dans `engine.py` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
            ...
            (In the player's results loop)
            ...
            if message:
-               print(message)
+               message_log.add_message(message)
            ...
            (In the enemy results loop)
            ...
                        if message:
-                           print(message)
+                           message_log.add_message(message)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>            ...
            (In the player's results loop)
            ...
            if message:
                <span class="crossed-out-text">print(message)</span>
                <span class="new-text">message_log.add_message(message)</span>
            ...
            (In the enemy results loop)
            ...
                        if message:
                            <span class="crossed-out-text">print(message)</span>
                            <span class="new-text">message_log.add_message(message)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Super, maintenant nous ajoutons tous les messages dans le log. Cela étant dit,
rien n'apparaît encore. Modifions `render_all` pour afficher le journal de
message que nous avons crée.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-def render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height, bar_width,
-              panel_height, panel_y, colors):
+def render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width, screen_height,
+              bar_width, panel_height, panel_y, colors):
    ...
    libtcod.console_clear(panel)

+   # Print the game messages, one line at a time
+   y = 1
+   for message in message_log.messages:
+       libtcod.console_set_default_foreground(panel, message.color)
+       libtcod.console_print_ex(panel, message_log.x, y, libtcod.BKGND_NONE, libtcod.LEFT, message.text)
+       y += 1
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="crossed-out-text">def render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height, bar_width,</span>
               <span class="crossed-out-text">panel_height, panel_y, colors):</span>
<span class="new-text">def render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width, screen_height,
               bar_width, panel_height, panel_y, colors):</span>
    ...
    libtcod.console_clear(panel)

    <span class="new-text"># Print the game messages, one line at a time
    y = 1
    for message in message_log.messages:
        libtcod.console_set_default_foreground(panel, message.color)
        libtcod.console_print_ex(panel, message_log.x, y, libtcod.BKGND_NONE, libtcod.LEFT, message.text)
        y += 1</span>
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Modifiez l'appel à `render_all` depuis `engine.py` pour inclure le journal
de message :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-       render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height,
-                  bar_width, panel_height, panel_y, colors)
+       render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width,
+                  screen_height, bar_width, panel_height, panel_y, colors)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        <span class="crossed-out-text">render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, screen_width, screen_height,</span>
                   <span class="crossed-out-text">bar_width, panel_height, panel_y, colors)</span>
        <span class="new-text">render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width,
                   screen_height, bar_width, panel_height, panel_y, colors)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Lancez le projet maintenant. Tous les anciens "print" devraient apparaître
dans un journal de message déroulant. Dorenavent nous n'utiliserons plus de
print, nous ajouterons tout à notre journal de messages.

Et la suite ? Pourquoi pas un peu d'action à la souris ? Notre jeu ne contient
que des orcs et des trolls pour l'instant mais peut-être qu'un jour nous aurons
des douzaines (centaines ?) de monstres différents et de types d'objets. Ce
serait bien qu'on puisse voir ce qu'ils sont en déplaçant la souris sur eux.

Heureusement pour nous, on a déjà capturé les événements souris dans la variable
`mouse` juste au dessus de la boucle principale. Tout ce qu'il faut faire est
d'ajuster notre appel à `libtcod.sys_check_for_event` pour répondre à la souris
et d'écrire le code qui affiche le nom quand on déplace la souris au dessus de
quelquechose.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-       libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS, key, mouse)
+       libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS | libtcod.EVENT_MOUSE, key, mouse)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        <span class="crossed-out-text">libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS, key, mouse)</span>
        <span class="new-text">libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS | libtcod.EVENT_MOUSE, key, mouse)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Ajoutez la fonction suivante dans `render_functions.py` au dessus de
`render_bar` :

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
+def get_names_under_mouse(mouse, entities, fov_map):
+   (x, y) = (mouse.cx, mouse.cy)

+   names = [entity.name for entity in entities
+            if entity.x == x and entity.y == y and libtcod.map_is_in_fov(fov_map, entity.x, entity.y)]
+   names = ', '.join(names)

+   return names.capitalize()


def render_bar(panel, x, y, total_width, name, value, maximum, bar_color, back_color):
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="new-text">def get_names_under_mouse(mouse, entities, fov_map):
    (x, y) = (mouse.cx, mouse.cy)

    names = [entity.name for entity in entities
             if entity.x == x and entity.y == y and libtcod.map_is_in_fov(fov_map, entity.x, entity.y)]
    names = ', '.join(names)

    return names.capitalize()</span>


def render_bar(panel, x, y, total_width, name, value, maximum, bar_color, back_color):
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Maintenant nous allons à nouveau modifier notre fonction `render_all` (elle a
beaucoup changé dans ce chapitre, n'est-ce-pas ?) pour utiliser la souris et
utiliser notre nouvelle fonction.

{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-def render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width, screen_height,
-              bar_width, panel_height, panel_y, colors):
+def render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width, screen_height,
+              bar_width, panel_height, panel_y, mouse, colors):
    ...
    render_bar(panel, 1, 1, bar_width, 'HP', player.fighter.hp, player.fighter.max_hp,
               libtcod.light_red, libtcod.darker_red)

+   libtcod.console_set_default_foreground(panel, libtcod.light_gray)
+   libtcod.console_print_ex(panel, 1, 0, libtcod.BKGND_NONE, libtcod.LEFT,
+                            get_names_under_mouse(mouse, entities, fov_map))

    libtcod.console_blit(panel, 0, 0, screen_width, panel_height, 0, 0, panel_y)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre><span class="crossed-out-text">def render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width, screen_height,</span>
               <span class="crossed-out-text">bar_width, panel_height, panel_y, colors):</span>
<span class="new-text">def render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width, screen_height,
               bar_width, panel_height, panel_y, mouse, colors):</span>
    ...
    render_bar(panel, 1, 1, bar_width, 'HP', player.fighter.hp, player.fighter.max_hp,
               libtcod.light_red, libtcod.darker_red)

    <span class="new-text">libtcod.console_set_default_foreground(panel, libtcod.light_gray)
    libtcod.console_print_ex(panel, 1, 0, libtcod.BKGND_NONE, libtcod.LEFT,
                             get_names_under_mouse(mouse, entities, fov_map))</span>

    libtcod.console_blit(panel, 0, 0, screen_width, panel_height, 0, 0, panel_y)</pre>
{{</ original-tab >}}
{{</ codetab >}}

Et, bien-sûr, nous devons modifier l'appel à `render_all` dans `engine.py` pour
correspondre à notre nouvelle définition.


{{< codetab >}} {{< diff-tab >}} {{< highlight diff >}}
-       render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width,
-                  screen_height, bar_width, panel_height, panel_y, colors)
+       render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width,
+                  screen_height, bar_width, panel_height, panel_y, mouse, colors)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        <span class="crossed-out-text">render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width,</span>
                   <span class="crossed-out-text">screen_height, bar_width, panel_height, panel_y, colors)</span>
        <span class="new-text">render_all(con, panel, entities, player, game_map, fov_map, fov_recompute, message_log, screen_width,
                   screen_height, bar_width, panel_height, panel_y, mouse, colors)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Les graphismes de notre jeu sont bien meilleurs. Si vous espérez que votre jeu
soit joué par d'autres joueurs que vous (sinon c'est bien aussi \!) ce type de
changement sera d'une grande importance dans votre projet.

Si vous voulez voir le code actuel entièrement, [cliquez ici](https://github.com/TStand90/roguelike_tutorial_revised/tree/part7).

[Cliquez ici pour vous rendre à la partie suivante de ce tutoriel.](/tutorials/tcod/part-8)

<script src="/js/codetabs.js"></script>
