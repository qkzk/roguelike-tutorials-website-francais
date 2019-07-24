---
title: "Partie 0 - Configuration"
date: 2019-03-30T09:53:48-07:00
draft: false
---

#### Connaissances préalables

Ce tutoriel suppose quelques connaissances en programmation en général et avec
Python. Si vous n'avez jamais utilisé Python, ce tutoriel peut vous sembler
confus. Il existe de nombreuses ressources en ligne pour apprendre la
programmation et Python (trop pour être citées ici) et je vous recommande de
découvrir au minimum les objets et les fonctions en Python avant de commencer la
lecture de ce tutoriel.

... Bien sûr, nombreux sont ceux qui n'ont pas tenu compte de ce conseil et
s'en sont sortis indemne aussi, ignorez ce dernier paragraphe si vous êtes
audacieux \!

#### Installation

Pour suivre ce tutoriel, vous aurez besoin de Python 3.5 au moins. La dernière
version de Python est recommander (3.7 en mars 2019).
**Remarque : ce tutoriel n'est pas compatible avec Python 2**

[Téléchargez Python ici](https://www.python.org/downloads/).

Vous aurez aussi besoin de la dernière version de la librairie TCOD sur laquelle
ce tutoriel est conçu.

[Les instructions pour installer TCOD sont disponibles ici.](https://python-tcod.readthedocs.io/en/latest/installation.html)


Bien qu'il soit parfaitement possible d'installer TCOD et de réaliser ce
tutoriel autrement, je vous recommande d'utiliser un environnement virtuel.
[Documentation sur ce sujet ici.](https://docs.python.org/3/library/venv.html)

#### Éditeurs

N'importe quel éditeur permet d'écrire du Python. Vous pouvez même utilisez
Notepad si vous le souhaitez. En ce qui me concerne, je suis fan de
[Pycharm](https://www.jetbrains.com/pycharm/) et de [Visual Studio Code](https://code.visualstudio.com/)
. Quel que soit votre choix, je vous recommande d'en utiliser permettant de détecter les erreurs de syntaxe. Je travaille en Python depuis plus
de cinq ans et je fais encore ce type d'erreurs tout le temps.

#### Vous assurer que Python fonctionne

Pour vérifier que votre installation de Python 3 et TCOD fonctionne, créer un
nouveau fichier (dans n'importe quel dossier que vous prévoyez d'employer pour
ce tutoriel) appelé `engine.py`, en saisissez le texte suivant :

```python
import tcod as libtcod


def main():
    print('Hello World!')


if __name__ == '__main__':
    main()
```

Exécuter ce fichier dans votre terminal (ou dans votre éditeur, si possible):

`python engine.py`

Si vous n'utilisez pas d'environement virtuel, la commande ressemblera sûrement
à :

`python3 engine.py`

Vous devriez vous "Hello World\!" affiché dans votre terminal. Si vous avez une
erreur, il y a sûrement un problème avec votre installation de Python ou de
TCOD.

### Télécharger le fichier image

Pour ce tutoriel, nous aurons besoin d'une image. Celle par défaut est fournie
ici.

![Font File](/images/arial10x10.png "Arial 10x10")

Faîtes un clic droit sur l'image et enregistrez la dans le dossier où vous
souhaitez écrire votre code. Si l'image ci-dessus ne s'affiche pas, elle est
[aussi disponible ici](https://raw.githubusercontent.com/TStand90/roguelike_tutorial_revised/master/arial10x10.png).

### À propos de ce site

Les extraits de code de ce site sont présentés de façon à exposer exactement ce
que l'utilisateur doit ajouter à un fichier à chaque moment. Quand un
utilisateur doit créer un fichier depuis zéro et y saisir du code, il sera
présenté avec la syntaxe usuelle de Python, ainsi :

{{< highlight py3 >}}
class Fighter:
    def __init__(self, hp, defense, power):
        self.max_hp = hp
        self.hp = hp
        self.defense = defense
        self.power = power
{{</ highlight >}}

**Extrait de la partie 6*.

La plupart du temps, vous éditerez un fichier qui contiendra déjà du code.
Dans ce cas, le code sera affiché ainsi :

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

**Aussi extrait de la partie 6.*

Cliquer un des boutons au dessus des sections de code change le "style" non
seulement pour ce bloc de code mais aussi pour le site entier. Vous pouvez
alterner quand vous le souhaitez.

Dans l'extrait ci-dessus, vous devriez retirer l'ancienne définition de
`__init__` et la remplacer par la nouvelle. Ensuite, vous devriez ajouter les
lignes nécessaires en dessous. Les deux styles présentent la même idée.

Mais quelle différence ? Le style "Diff" montre le code tel que vous le verriez
dans un diff Git (d'où le nom). Cela montre des plus et des moins sur le côté
pour montrer si vous devriez ajouter ou soustraire des lignes d'un fichier. Le
style "Original" montre la même chose mais les lignes à retirer sont barrées et
il n'y a ni plus ni moins.


L'avantage du style "Diff" est qu'il ne dépend d'aucune couleurs pour souligner
ce qui doit être ajouté, le rendant plus accessible en général. L'inconvénient
est qu'il est parfois impossible d'afficher correctement l'indentation. Les plus
et moins prennent de la place ainsi, dans une section de code telle que
celle-ci, prenez soin de laisser un espace pour le plus dans votre code (il ne
doit pas y avoir d'espace avant "from"):

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

Le style "Original" n'utilise pas de symboles + et - et n'a pas de problème
d'indentation, simplifiant quelque peu les copiers coller de code.

Le style que vous employez est une question de préférence personnelle. Le code
en lui même de ce tutoriel reste le même.

### Obtenir de l'aide

N'hésite pas à consulter le [Roguelike Development Subreddit](https://www.reddit.com/r/roguelikedev) pour de l'aide. Il y a aussi un lien vers le canal Discord.

-----

### Prêt à démarrer ?

Une fois que vous êtes prêts, vous pouvez vous rendre dans la [Partie 1](/tutorials/tcod/part-1).

<script src="/js/codetabs.js"></script>
