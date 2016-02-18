#Simuler en Python

_Ce document est librement inspiré [tutoriel NumPy de Nicolas Rougier](http://www.labri.fr/perso/nrougier/teaching/numpy/numpy.html) et est disponible avec son autorisation sous licence Creative Commons Attribution 3.0 United States License (CC-by) http://creativecommons.org/licenses/by/3.0/us_

La simulation numérique (ou informatique) permet de représenter dans une machine (ordinateur) un phénomène écologique ou physique que l'on veut étudier à moindre coût et sans danger. On distingue généralement deux types de simulation: ``simulation continue`` et ``simulation discrète``.

Le principe d'une ``simulation discrète`` d'un phénonème physique ou écologique consiste à représenter l'état initial de la simulation, puis ensuite de construire une fonction qui à partir d'un état précédent va déterminer l'état suivant de la simulation.
L'application de cette fonction permet de passer du temps t0 au temps t1, puis t2, etc ... Généralement, on arrrête la simulation au bout d'un certain nombre d'application de la fonction (n) déterminé à l'avance.

Au contraire une ``simulation continue`` permet de représenter de manière continue les changements d'un système physique ou biologique. Généralement on emploie pour cela, des [équations différentielles](http://fr.wikipedia.org/wiki/%C3%89quation_diff%C3%A9rentielle).

On ne va s'intéresser ici qu'à des simulations discrètes. Dans une simulation discrète, le temps est discrétisé en durée similaire. Chaque étape peut correspondre en fonction du phénomène considérée à une durée de 1 ms, 1s, 1 jour ou bien 1000 ans.

Nous allons illustrer ici ce principe au moyen d'une simulation sous la forme d'une simulation écologique appellée Jeu de la Vie représentant l'évolution de cellules qui naissent ou qui meurent au cours du temps.

##Phénomène à simuler: le Jeu de la Vie

Le [Jeu de la Vie](https://fr.wikipedia.org/wiki/Jeu_de_la_vie) est un des premiers exemples d'automates cellulaires (voir figure ci-dessous) construit par John Conway en 1970. Ces automates cellulaires peuvent être considérés comme un tableau de cellules qui sont connectées les unes aux autres par la notion de voisinage.

Ce "jeu" est un fait un jeu à zéro joueur, car son évolution est déterminé uniquement par son état inital et ne nécessite pas d'entrées de joueurs humains. La seule façon d'interagir avec un Jeu de la Vie est de créer une configuration initiale et d'observer comment elle évolue au cours du temps.

L'univers (ou l'état) du Jeu de la Live est une grille à deux dimensions de taille infinie, composé de cellules carrées. Chaque cellule peut contenir l'un des deux états possibles: vivant ou mort, que l'on représente par les valeurs entières 0 ou 1.

![image](http://www.labri.fr/perso/nrougier/teaching/numpy/figures/game-of-life.png)

Chaque cellule interagit avec ces 8 voisins, qui sont les cellules directement adjacentes horizontalement, verticalement et en diagonale. A chaque étape du temps, les règles suivantes vont s'appliquer :

1. Une cellule vivante avec moins de deux voisines vivantes, meurt d'isolement,
2. Une cellule vivante avec plus de 3 cellules voisines vivantes, meurt d'étouffement,
3. Une cellule vivante avec 2 ou 3 cellules voisines vivantes, reste inchangée à la prochaine génération,
4. Une cellule morte avec exactement 3 cellules vivantes, devient une cellule vivante.

L'état de départ est constitué par une forme initiale. La première génération est créér en appliquant les règles ci-dessus simultanément à toutes les cellules de l'état de départ: naissances et morts sont effectués simultanément, and le moment où cela se déroule est appellé tick (de simulation). Les règles continuent d'être appliquées pour créer les futures générations.

Pour commencer, nous allons utiliser un état de départ très simple, appellé "planeur" (glider) qui est connu pour se déplacer diagonalement au bout de 4 itérations comme indiqué ci-dessous :

![image](http://www.labri.fr/perso/nrougier/teaching/numpy/figures/glider-00.png)
![image](http://www.labri.fr/perso/nrougier/teaching/numpy/figures/glider-01.png)
![image](http://www.labri.fr/perso/nrougier/teaching/numpy/figures/glider-02.png)
![image](http://www.labri.fr/perso/nrougier/teaching/numpy/figures/glider-03.png)
![image](http://www.labri.fr/perso/nrougier/teaching/numpy/figures/glider-04.png)
![image](http://www.labri.fr/perso/nrougier/teaching/numpy/figures/glider-05.png)

Cette propriété va nous permettre de débogguer visuellement plus facilement nos programmes.

La première question à se poser pour faire cette simulation est comment représenter un état, ici l'ensemble des cellules à un instant donné. En Python, il est possible d'utiliser le type list ou array pour représenter des tableaux a une ou plusieurs dimensions.

La bibliothèque scientifique ``NumPy`` est une alternative qui permet de manipuler très efficacemment des tableaux de grande taille en Python: http://www.numpy.org/

La première chose à faire est de créer un tableau NumPy afin de contenir les cellules (``cells``). Ceci peut être fait facilement de la façon suivante :

```python
>>> import numpy as np
>>> cells = np.array([[0,0,0,0,0,0],
              [0,0,0,1,0,0],
              [0,1,0,1,0,0],
              [0,0,1,1,0,0],
              [0,0,0,0,0,0],
              [0,0,0,0,0,0]])
```

Il existe de nombreuses autres façons de créer un tableau NumPy : http://docs.scipy.org/doc/numpy/reference/routines.array-creation.html

Notez que nous n'avons pas spécifié le type des données contenues dans le tableau, NumPy a choisi pour nous. Comme tous les éléments sont des entiers, NumPy a choisi le type entier (integer). Ceci peut se vérifier facilement :

```python
>>> print(cells.dtype)
int64
```

On peut facilement vérifier la taille d'un tableau, ici par exemple 6x6 :

```python
>>> print(cells.shape)
(6, 6)
```

Chaque élément de ``cells`` peut être accédé en utilisant un index de ligne et de colonne (en suivant cet ordre) :

```python
>>> print(cells[0,5])
0
```

Il est également possible d'accéder à une sous-partie d'un tableau, en utilsant la notation dite slice :

```python
>>> print(cells[1:5,1:5])
[[0 0 1 0]
 [1 0 1 0]
 [0 1 1 0]
 [0 0 0 0]]
```

Dans l'exemple ci-dessous, nous avons extrait une sous-partie de ``cells`` de la ligne 1 à 5 et de la collonne 1 à 5. Il est important de bien comprendre qu'il s'agit vraiment d'une sous-ensemble de ``cells`` dans le sens où chaque modification de la sous-partie va avoir un impact direct sur ``cells`` :

```python
>>> a = cells[1:5,1:5]
>>> a[0,0] = 9
>>> print(a)
[[9 0 1 0]
 [1 0 1 0]
 [0 1 1 0]
 [0 0 0 0]]

>>> print(cells)
[[0 0 0 0 0 0]
 [0 9 0 1 0 0]
 [0 1 0 1 0 0]
 [0 0 1 1 0 0]
 [0 0 0 0 0 0]
 [0 0 0 0 0 0]]
```

Nous avons modifié la valeur de ``a[0,0]`` à 9 et nous voyons un changement immédiat dans ``cells[1,1]`` parce que ``a[0,0]`` correspond à ``cells[1,1]``. Ceci peut paraître trivial avec des tableaux si simples, mais les choses peuvent devenir plus complexe comme nous le verrons plus tard. En cas de doute, il possible de vérifier rapidement, si un tableau est une partie d'un autre :

```python
>>> print(cells.base)
None
>>> print(a.base is cells)
True
```

N'oublions pas de remettre la valeur de a[0,0] à 0 :

```python
>>> a[0,0] = 0
```

### Compter les voisins
Nous avons besoin d'une fonction pour compter les voisins d'une cellule.

```python
def compute_neighbours(cells):
    rows, cols = len(cells), len(cells[0])
    count  = np.zeros([rows, cols], int)
    for x in range(1,cols-1):
        for y in range(1,rows-1):
            count[y][x] = cells[y-1][x-1]+cells[y][x-1]+cells[y+1][x-1] \
                      + cells[y-1][x]              +cells[y+1][x]   \
                      + cells[y-1][x+1]+cells[y][x+1]+cells[y+1][x+1]
    return count
```

Avec NumPy, il est possible d'utiliser des opérations qui portent sur l'ensemble du tableau, plutôt que faire des itérations, ce que l'on appelle la **vectorisation**. Voir version ici : http://www.labri.fr/perso/nrougier/teaching/numpy/numpy.html#the-game-of-life

###Faire des itérations

Construisons la fonction ``iterate`` qui permet d'application les règles sur ``cells`` pour produire une nouvelle génération.

```python
def iterate(cells):
    rows,cols = len(cells), len(cells[0])
    N = compute_neighbours(cells)
    for x in range(1,cols-1):
        for y in range(1,rows-1):
            if cells[y][x] == 1 and (N[y][x] < 2 or N[y][x] > 3):
                cells[y][x] = 0
            elif cells[y][x] == 0 and N[y][x] == 3:
                cells[y][x] = 1
    return cells
```
Appliquer la fonction iterate sur le tableau ``cells`` définit précédemment.

**Question: Expliquer comment la fonction ``iterate`` fonctionne ?**

## Lancer une simulation
Essayons de lancer le Jeu de la Vie sur une grille beaucoup plus grande.
Pour cela générons une grille remplie de 0 et 1 placé de manière aléatoire.

La bibliothèque ``random`` permet de générer des nombres aléatoires.
Voir documentation ici: https://docs.python.org/3/library/random.html

Quelques unes des fonctions les plus utiles:

* ``random.seed()`` : permet d'initialiser le générateur de nombres aléatoires
* ``random.random()`` : retourne un nombre réél aléatoire dans l'intervalle [0, 1]
* ``random.randrange(a,b)``: retourne un nombre entier compris entre a et b (inclus).

**Question: Pourquoi les nombres générés par cette bibliothèque sont des nombres pseudo-aléatoires ?**

La bibliothèque NumPy complète la bibliothèque ``random`` par des fonctions comme la fonction ``np.random.randint`` qui permet de générer des nombres aléatoires pour remplir un tableau (voir documentation ici : http://docs.scipy.org/doc/numpy/reference/routines.random.html) :

```python
>>> cells = np.random.randint(0, 2, (256,512))
```

**Question: donner une explication des paramètres de la fonction ``np.random.randint`` ?**

Effectuons 10 itérations :

```python
>>> for i in range(10):
   iterate(cells)
```

et affichons les résultats :

```python
>>> import matplotlib.pyplot as plt
>>> size = np.array(cells.shape)
>>> dpi = 72.0
>>> figsize= size[1]/float(dpi),size[0]/float(dpi)
>>> fig = plt.figure(figsize = figsize, dpi = dpi, facecolor = "white")
>>> fig.add_axes([0.0, 0.0, 1.0, 1.0], frameon = False)
>>> plt.imshow(cells, interpolation = 'nearest', cmap = plt.cm.gray_r)
>>> plt.xticks([]), plt.yticks([])
>>> plt.show()
```

![image](http://www.labri.fr/perso/nrougier/teaching/numpy/figures/game-of-life-big.png)

Nous utilisons pour l'affichage la bibliothèque Matplotlib que nous aborderons plus en détail dans la prochaine fiche.

Vous trouverez une version graphique du Jeu de la Vie (écrit en JavaScript) ici : http://www.grappa.univ-lille3.fr/~torre/Enseignement/TPs/JavaScript/jeu-de-la-vie-js-canvas-html5/

Pour vous entrainez à l'utilisation de NumPy, vous pouvez essayer les exercices disponibles ici : https://github.com/rougier/numpy-100