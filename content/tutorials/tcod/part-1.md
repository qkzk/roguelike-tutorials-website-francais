---
title: "Partie 1 - Dessiner le caractère ‘@’ et le déplacer"
date: 2019-03-30T08:39:15-07:00
draft: false
---

Bienvenue dans la partie 1 du **Tutoriel Roguelike revesité \!** Cette série va
vous aider à créer votre tout premier jeu roguelike, écrit en Python.

Ce tutoriel est largement inspiré de celui [trouvé sur Roguebasin](http://www.roguebasin.com/index.php?title=Complete_Roguelike_Tutorial,_using_python%2Blibtcod). Une grande
partie des décisions de conception furent prises afin de conserver une
progression commune avec celui-ci (du moins en ce concerne la composition des
chapitres et la direction générale). Ce tutoriel n'aurait pu être possible sans
le travail des concepteurs d'origine ainsi que celui des merveilleux
contributeurs de libtcod et python-tcod.


Nous supposons dans cette partie que vous avez déjà effectué la [partie 0](/tutorials/tcod/part-0)
et être prêt à commencer. Sinon, rendez-vous sur cette page est vérifiez votre
installation de Python et de TCOD. Assurez-vous d'avoir un fichier `engine.py`
dans le dossier choisi pour votre travail.

Supposant que vous avez accompli tout ça, commençons. Modifiez (ou créez si
vous ne l'avez pas déjà fait) le fichier `engine.py` pour qu'il ressemble à ça :

{{< highlight py3 >}}
import tcod as libtcod


def main():
    print('Hello World!')


if __name__ == '__main__':
    main()
{{</ highlight >}}

Vous pouvez exécuter le programme comme n'importe quel programme Python mais,
pour les novices, vous le faîtes en tapant `python engine.py` dans le terminal.
Si vous avez à la fois Python 2 et 3 d'installé sur votre machine, il se peut
que vous deviez taper `python3 engine.py` pour le lancer (cela dépend de votre
python par défaut et vous utilisez un environnement virtuel ou non).

D'accord, ce n'est pas le programme le plus enthousiasmant qui soit, je le
reconnais, mais nous avons déjà une différence majeure avec l'autre tutoriel.
C'est cette chose étrange ici :

{{< highlight py3 >}}
if __name__ == '__main__':
    main()
{{< /highlight >}}

Qu'est-ce que ça fait ? Simplement, nous disons que nous n'allons lancer
la fonction principale "main" que si nous lançons explicitement le script avec
la commande `python engine.py`. Ce n'est pas fondamental que vous compreniez ça
maintenant mais si vous tenez à comprendre, cette réponse sur [Stackoverflow] (https://stackoverflow.com/a/419185) donne un bon aperçu.

Assurez-vous que le programme précédent tourne (sinon, c'est probablement un
soucis d'installation de libtcod). Une fois que cela est accompli, nous pouvons
allez vers des étapes un peu plus intéressantes. La première étape importante
dans la conception d'un roguelike est d'obtenir un caractère '@' à l'écran et
de le déplacer aussi allons-y.

Modifiez `engine.py` pour qu'il ressemble à ça :

{{< highlight py3 >}}
import tcod as libtcod


def main():
    screen_width = 80
    screen_height = 50

    libtcod.console_set_custom_font('arial10x10.png', libtcod.FONT_TYPE_GREYSCALE | libtcod.FONT_LAYOUT_TCOD)

    libtcod.console_init_root(screen_width, screen_height, 'libtcod tutorial revised', False)

    while not libtcod.console_is_window_closed():
        libtcod.console_set_default_foreground(0, libtcod.white)
        libtcod.console_put_char(0, 1, 1, '@', libtcod.BKGND_NONE)
        libtcod.console_flush()

        key = libtcod.console_check_for_keypress()

        if key.vk == libtcod.KEY_ESCAPE:
            return True


if __name__ == '__main__':
    main()
{{</ highlight >}}

Exécutez `engine.py` à nouveau et vous devriez vous un '@' à l'écran. Une fois
que vous êtes complètement repu de la gloire provoquée par l'écran devant vous,
vous pouvez presser la touche \`Esc\` pour quitter le programme.

Il se passe beaucoup de choses ici, aussi découpons ligne par ligne.

{{< highlight py3 >}}
    screen_width = 80
    screen_height = 50
{{</ highlight >}}

C'est assez simple. On définit quelques variables pour la dimension de l'écran.
Éventuellement, nous pourrions charger ces valeurs depuis un fichier JSON plutôt
que de les coder en dur dans les sources, mais nous ne en soucierons pas avant
d'avoir plus de variables.

{{< highlight py3 >}}
    libtcod.console_set_custom_font('arial10x10.png', libtcod.FONT_TYPE_GREYSCALE | libtcod.FONT_LAYOUT_TCOD)
{{</ highlight >}}

Ici, nous disons à libtcod quelle police employer. La partie `'arial10x10.png'`
est le fichier qu'on lit (il devrait exister dans votre dossier de projet). Les
deux autres parties disent à libtcod quel type de fichier nous lisons.

{{< highlight py3 >}}
    libtcod.console_init_root(SCREEN_WIDTH, SCREEN_HEIGHT, 'libtcod tutorial revised', False)
{{</ highlight >}}

Cette ligne est ce qui crée l'écran. Nous lui donnons les valeurs `screen_width`
et `screen_height` définies plus haut (80 et 50, respectivement), ainsi qu'un
titre (changez le si vous connaissez déjà le nom de votre jeu) et une valeur
booléenne qui indique à libtcod si le jeu est plein écran ou non.

{{< highlight py3 >}}
    while not libtcod.console_is_window_closed():
{{</ highlight >}}

C'est ce qu'on appelle la 'boucle de jeu'. C'est simplement une boucle qui ne va
jamais s'interrompre tant qu'on n'aura pas fermé la fenêtre. Chaque jeu a une
boucle de jeu d'une manière ou d'une autre.

{{< highlight py3 >}}
        libtcod.console_set_default_foreground(0, libtcod.white)
{{</ highlight >}}

Cette ligne indique à libtcod de régler la couleur de notre symbole '@'. Si vous
souhaitez qu'il soit d'une autre couleur, remplacez `libtcod.white` par quelque
chose comme `libtcod.red` et voyez ce que cela donne. Le '0' dans cette fonction
est la console sur laquelle nous écrivons. Nous en reparlerons plus tard.

{{< highlight py3 >}}
        libtcod.console_put_char(0, 1, 1, '@', libtcod.BKGND_NONE)
{{</ highlight >}}

Le premier argument est '0' (à nouveau, la console sur laquelle nous écrivons).
Les deux suivants sont des coordonnées x et y, dans ce cas 1 et 1 (essayez de
les remplacer pour voir ce que cela donne). Ensuite nous affichons le symbole
'@' et réglons le fond sur 'none' avec `libtcod.BKGND_NONE`.

{{< highlight py3 >}}
        libtcod.console_flush()
{{</ highlight >}}

This is the part that presents everything on the screen. Pretty
straightforward.

{{< highlight py3 >}}
        key = libtcod.console_check_for_keypress()

        if key.vk == libtcod.KEY_ESCAPE:
            return True
{{</ highlight >}}

Cette partie nous permet de quitter gracieusement le jeu (sans planter) en
pressant la touche `Esc`. La fonction `libtcod.console_check_for_keypress()`
récupère une saisie clavier du programme que nous stockons dans la variable
`key`. Ensuite, nous vérifions si la touche pressée est `Esc` ou non. Si c'est
le cas, nous quittons la boucle, ce qui termine le programme.

Maintenant que nous avons notre symbole `@` dessiné, déplaçons le \!

Nous devons connaître la position du joueur, donc nous allons créer deux
variables, `player_x` et `player_y`.

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
    ...
    screen_height = 50
+
+   player_x = int(screen_width / 2)
+   player_y = int(screen_height / 2)
+
    libtcod.console_set_custom_font('arial10x10.png', libtcod.FONT_TYPE_GREYSCALE | libtcod.FONT_LAYOUT_TCOD)
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    screen_height = 50
    <span class="new-text">
    player_x = int(screen_width / 2)
    player_y = int(screen_height / 2)
    </span>
    libtcod.console_set_custom_font('arial10x10.png', libtcod.FONT_TYPE_GREYSCALE | libtcod.FONT_LAYOUT_TCOD)
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

*Remarque: Les trois points désignent une partie omise du code. J'inclurai des
lignes autour du code à insérer afin que vous sachiez exactement où écrire les
nouvelles parties de code mais je ne montrerai pas le fichier entier à chaque
fois. Les lignes vertes indiquent le code que vous devriez ajouter.*

Nous plaçons le joueur en plein milieu de l'écran. Que fait la fonction `int()`
? Et bien, Python 3 ne tronque pas automatiquement la division comme Python 2
aussi nous devons convertir le résultat (un flottant) en entier. Sinon libtcod
va renvoyer une erreur.

Nous devons aussi modifier la commande affichant le symbole '@' afin d'employer
ces nouvelles coordonnées.

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
        ...
        libtcod.console_set_default_foreground(0, libtcod.white)
-       libtcod.console_put_char(0, 1, 1, '@', libtcod.BKGND_NONE)
+       libtcod.console_put_char(0, player_x, player_y, '@', libtcod.BKGND_NONE)
        libtcod.console_flush()
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        libtcod.console_set_default_foreground(0, libtcod.white)
        <span class="crossed-out-text">libtcod.console_put_char(0, 1, 1, '@', libtcod.BKGND_NONE)</span>
        <span class="new-text">libtcod.console_put_char(0, player_x, player_y, '@', libtcod.BKGND_NONE)</span>
        libtcod.console_flush()
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

*Remarque : les lignes en rouge signalent le code qui a été enlevé.*

Exécutez le programme maintenant et vous devriez voir le '@' au centre de
l'écran. Occupons nous maintenant de le rendre mobile.

Ajoutez les deux lignes suivantes juste au dessus de la boucle principale.

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
    ...
    libtcod.console_init_root(screen_width, screen_height, 'libtcod tutorial revised', False)

+   key = libtcod.Key()
+   mouse = libtcod.Mouse()

    while not libtcod.console_is_window_closed():
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    libtcod.console_init_root(screen_width, screen_height, 'libtcod tutorial revised', False)

    <span class="new-text">key = libtcod.Key()
    mouse = libtcod.Mouse()</span>

    while not libtcod.console_is_window_closed():
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Ainsi que les noms le suggèrent, ces variables vont contenir nos saisies clavier
et souris. Nous n'implémenterons pas la souris immédiatement mais la fonction
que nous allons ajouter en tient compte aussi nous pouvons l'ajouter.

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
    ...
    while not libtcod.console_is_window_closed():
+       libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS, key, mouse)

        libtcod.console_set_default_foreground(0, libtcod.white)
    ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    while not libtcod.console_is_window_closed():
        <span class="new-text">libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS, key, mouse)</span>

        libtcod.console_set_default_foreground(0, libtcod.white)
    ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

C'est la fonction qui capture les nouveaux "événements" (saisie clavier).
Elle va mettre à jour les variables `key` et `mouse` avec ce que saisit
l'utilisateur. À nouveau seule `key` nous intéresse pour l'instant.

D'accord, nous mettons `key` à jour avec la saisie de l'utilisateur. Mais qu'en
*faisons* nous ? Cela va traduire les saisies en action de jeu.

Pour l'instant, ce tutoriel n'a pas vraiment divergé de celui d'origine mais
voici un changement radical. Nous sommes sur le point de définir une fonction
appelée `handle_keys` qui va gérer les saisies clavier. Nous *pourrions*
l'ajouter à notre fichier `engine.py`... mais est-ce la place de cette
fonction ? Je pense que non. Le moteur (la boucle principale) capture les
saisies et devrait en faire quelque chose mais traduire de l'un vers l'autre
ne le concerne pas.

Aussi, plutôt que d'ajouter la fonction `handle_keys` à `engine.py`, créez un
nouveau fichier, appelé `input_handlers.py`. Placez le code suivant dans ce
nouveau fichier.

{{< highlight py3 >}}
import tcod as libtcod


def handle_keys(key):
    # Movement keys
    if key.vk == libtcod.KEY_UP:
        return {'move': (0, -1)}
    elif key.vk == libtcod.KEY_DOWN:
        return {'move': (0, 1)}
    elif key.vk == libtcod.KEY_LEFT:
        return {'move': (-1, 0)}
    elif key.vk == libtcod.KEY_RIGHT:
        return {'move': (1, 0)}

    if key.vk == libtcod.KEY_ENTER and key.lalt:
        # Alt+Enter: toggle full screen
        return {'fullscreen': True}

    elif key.vk == libtcod.KEY_ESCAPE:
        # Exit the game
        return {'exit': True}

    # No key was pressed
    return {}
{{</ highlight >}}

C'est un gros morceau à comprendre en une seule fois aussi, à nouveau, découpons
le en différentes parties.

{{< highlight py3 >}}
def handle_keys(key):
{{</ highlight >}}

On crée une fonction appelée `handle_keys` qui prend un paramètre, `key`. `key`
désigne cette fois la touche capturée plus tôt.

{{< highlight py3 >}}
    if key.vk == libtcod.KEY_UP:
{{</ highlight >}}

Cette expression if (ainsi que les autres elifs) nous dit quelle touche a été
pressée. Pour l'instant, c'est l'une des flèches pour le mouvement. Ce qui est
plus intéressant est le code dans cette expression if.

{{< highlight py3 >}}
    return {'move': (0, -1)}
{{</ highlight >}}

Que faisons nous ici ? Quand nous retournons de cette fonction, le moteur va
devoir faire quelque chose. Dans ce cas, nous voulons déplacer notre personnage.
Mais si nous voulons utiliser une autre touche ? Alors nous ne voulons peut-être
pas bouger mais utiliser un item, lancer un sort ou quitter le jeu. Une manière
de traiter toutes ces possibilités est de renvoyer un dictionnaire depuis cette
fonction. Le moteur le lira et décidera ce qu'il en fera.

Pour ce faire, nous retournons un dictionnaire avec la clé `'move'` et pour
valeur une paire de nombres. Ces nombres dirons au moteur dans quelle direction
déplacer le joueur. Par exemple, la touche 'haut' va déplacer le joueur de '0'
selon les x et '-1' selon les y.

{{< highlight py3 >}}
    if key.vk == libtcod.KEY_ENTER and key.lalt:
        # Alt+Enter: toggle full screen
        return {'fullscreen': True}
    elif key.vk == libtcod.KEY_ESCAPE:
        # Exit the game
        return {'exit': True}
{{</ highlight >}}

Voici les actions sans mouvement que nous acceptons pour l'instant. Si
l'utilisateur presse 'ALT+enter', le jeu va passer en mode plein écran. Si
l'utilisateur presse 'Esc', le jeu va quitter.

{{< highlight py3 >}}
    return {}
{{</ highlight >}}

Parce que notre moteur attend un dictionnaire, nous devons renvoyer
*quelque chose*, même si rien ne s'est produit.

Cela peut sembler surprenant mais cela prendra sens dans quelques insants.
Retournons à ntore fichier `engine.py` pour appeler la fonction `handle_keys`.

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
        ...
        libtcod.console_flush()

-       key = libtcod.console_check_for_keypress()
+       action = handle_keys(key)
+
+       move = action.get('move')
+       exit = action.get('exit')
+       fullscreen = action.get('fullscreen')

+       if move:
+           dx, dy = move
+           player_x += dx
+           player_y += dy

-       if key.vk == libtcod.KEY_ESCAPE:
+       if exit:
+           return True
+
+       if fullscreen:
+           libtcod.console_set_fullscreen(not libtcod.console_is_fullscreen())
        ...
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>        ...
        libtcod.console_flush()

        <span class="crossed-out-text">key = libtcod.console_check_for_keypress()</span>
        <span class="new-text">action = handle_keys(key)

        move = action.get('move')
        exit = action.get('exit')
        fullscreen = action.get('fullscreen')</span>

        <span class="new-text">if move:
            dx, dy = move
            player_x += dx
            player_y += dy</span>

        <span class="crossed-out-text">if key.vk == libtcod.KEY_ESCAPE:</span>
        <span class="new-text">if exit:
            return True

        if fullscreen:
            libtcod.console_set_fullscreen(not libtcod.console_is_fullscreen())</span>
        ...</pre>
{{</ original-tab >}}
{{</ codetab >}}

Remarque : Je ferai apparaître en rouge les lignes effacées. Dans ce cas,
retirez les lignes : `key = libtcod.console_check_for_keypress()` et `if key.vk ==
libtcod.KEY_ESCAPE`

Aussi, assurez-vous d'importer les fonctions `handle_keys` en haut de
`engine.py`

{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
import tcod as libtcod

+from input_handlers import handle_keys
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
    <pre>import tcod as libtcod

<span class="new-text">from input_handlers import handle_keys</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Esperons maintenant que le dictionnaire de `handle_keys` prenne un peu plus de
sens. On capture la valeur de retour de `handle_keys` dans la variable
`action` (qui devrait être un dictionnaire, quoi qu'on ait pressé) et on
vérifie quelle touche quelle touche elle contient. Si elle contient une clé
appelée `move` alors on sait qu'il faut chercher les coordonnées (x, y).
Si elle contient 'exit', alors on sait qu'il faut quitter le jeu.

Essayez d'exécuter le fichier engine.py maintenant. Vous devriez pouvoir vous
déplacer. Amusant !

Une dernière étape avant d'avancer. Regardez la fonction de dessin. Remarquez
le premier argument qui est '0'. Cela représente la console dans laquelle on
écrit. 0 est celle par défaut. Plutôt que d'écrire sur celle par défaut nous
voulons préciser sur quelle console écrire après en avoir crée une nouvelle.
La raison derrière est qu'il sera plus facile d'en créer pour dessiner dessus
plus tard. Cela sera particulièrement utile quand nous aborderons la partie
relative à l'interface graphique de cette série.

Modifiez le fichier `engine.py` ainsi :


{{< codetab >}}
{{< diff-tab >}}
{{< highlight diff >}}
    ...
    libtcod.console_init_root(screen_width, screen_height, 'libtcod tutorial revised', False)

+   con = libtcod.console_new(screen_width, screen_height)

    key = libtcod.Key()
    mouse = libtcod.Mouse()

    while not libtcod.console_is_window_closed():
        libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS, key, mouse)
+
+       libtcod.console_set_default_foreground(con, libtcod.white)
+       libtcod.console_put_char(con, player_x, player_y, '@', libtcod.BKGND_NONE)
+       libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)
-       libtcod.console_set_default_foreground(0, libtcod.white)
-       libtcod.console_put_char(0, player_x, player_y, '@', libtcod.BKGND_NONE)
        libtcod.console_flush()
+
+       libtcod.console_put_char(con, player_x, player_y, ' ', libtcod.BKGND_NONE)
-       libtcod.console_put_char(0, player_x, player_y, ' ', libtcod.BKGND_NONE)
{{</ highlight >}}
{{</ diff-tab >}}
{{< original-tab >}}
<pre>    ...
    libtcod.console_init_root(screen_width, screen_height, 'libtcod tutorial revised', False)

    <span class="new-text">con = libtcod.console_new(screen_width, screen_height)</span>

    key = libtcod.Key()
    mouse = libtcod.Mouse()

    while not libtcod.console_is_window_closed():
        libtcod.sys_check_for_event(libtcod.EVENT_KEY_PRESS, key, mouse)
        <span class="new-text">
        libtcod.console_set_default_foreground(con, libtcod.white)
        libtcod.console_put_char(con, player_x, player_y, '@', libtcod.BKGND_NONE)
        libtcod.console_blit(con, 0, 0, screen_width, screen_height, 0, 0, 0)</span>
        <span class="crossed-out-text">libtcod.console_set_default_foreground(0, libtcod.white)</span>
        <span class="crossed-out-text">libtcod.console_put_char(0, player_x, player_y, '@', libtcod.BKGND_NONE)</span>
        libtcod.console_flush()
        <span class="new-text">
        libtcod.console_put_char(con, player_x, player_y, ' ', libtcod.BKGND_NONE)</span>
        <span class="crossed-out-text">libtcod.console_put_char(0, player_x, player_y, ' ', libtcod.BKGND_NONE)</span></pre>
{{</ original-tab >}}
{{</ codetab >}}

Cela conclut la première partie de ce tutoriel \! Si vous utilisez git ou une
autre forme de contrôle de version (et je vous le recommande), vous devriez
faire un commit de vos changements maintenant.

Si vous voulez voir le code actuel entièrement, [cliquez ici](https://github.com/TStand90/roguelike_tutorial_revised/tree/part1).
Les fichiers que vous souhaitez consulter sont `engine.py` et
`input_handlers.py`.

[Cliquez ici pour vous rendre à la partie suivante de ce tutoriel.](/tutorials/tcod/part-2)

<script src="/js/codetabs.js"></script>
