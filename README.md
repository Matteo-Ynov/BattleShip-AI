# BattleShip AI

## Sommaire :
1. Présentation du projet
2. Language et librairies utilisés
3. Lancement du programme
4. Différences entre les niveaux de difficultés
5. Présentation des classes
6. Méthodes importantes
7. Captures d'écran

## I) Présentation du projet

Notre projet est une reproduction du célébre jeu de la bataille navale, où 2 joueurs s'affrontent avec un plateau de 10 cases par 10 cases chacun, où sont placés 5 bateaux. Le but est de détruire tous les bateaux de l'adversaire avant qu'ils ne fassent de même. Contrairement à la bataille navale classique, notre projet permet de jouer contre une intelligence artificielle possédants 3 niveaux de difficulté et utilisant les probabilités afin de gagner.

## II) Language et librairies utilisés

Nous avons utilisé Python 3.7 afin de mener à bien le projet de par l'orientation objet de ce language. Nous utilisons Pygame pour l'interface graphique et numpy afin de représenter les grilles de jeu.

## III) Lancement du programme

Toutes les librairies utilisées peuvent être installées avec la commande :
```
pip install -r requirements.txt
``` 
depuis l'intérieur du dossier. 

Pour lancer le programme, utilisez la commande :
```
python BattleShip.py [easy, medium, hard]
```
L'écran de placement de bateau s'affichera alors et vous pourrez cliquer sur ces derniers afin de les positionner sur la grille. Un clic gauche permettra de changer l'orientation du bateau sélectionné. Quand tous les bateaux sont placés, un bouton de validation apparaîtra et la partie pourront commencer.

## IV) Différences entre les niveaux de difficultés
Le jeu comprend 3 niveaux de difficulté : facile, moyen et difficile. Selon la difficulté choisit l'IA aura accès à des outils plus ou moins puissants afin de gagner. En plus de ces outils, chaque niveau de difficulté augmentera le nombre de simulations réalisé par l'IA
* En facile, l'IA va tirer aléatoirement mais cherchera quand même à finir les bateaux partiellement coulés.
* En moyen, l'IA ne va plus tirer aléatoirement mais utiliser les statistiques afin de déterminer le coup ayant la plus grande probabilité de toucher un bateau.
* En difficile, l'IA agira de la même manière qu'en moyen mais tirera avec un patterne de 
grille : chaque bateau faisant au moins 2 de longueur, tirer de cette manière garantie de toucher tous les bateaux en un minimum de coup :
![](https://i.imgur.com/3ch7Hx7.png)



## V) Présentation des classes

Le programme comporte deux classes : Player() et Board()
* Player() est la plus importante étant donné qu'elle comprend toutes les informations des joueurs ( plateau respectif, nombre de bateau restant, etc.. ) ainsi que les méthodes principales du programme. Elle représente le joueur et l'intelligence artificielle qui partagent les méthodes de tir, de destruction et de mise à jour des plateaux de jeu. Elles possédent des méthodes supplémentaires dédiés uniquement a l'IA comme la placement aléatoire, l'analyse de plateau et la recherche de bateau.
* Board() est la classe contenant les plateaux de jeu. Elle permet d'en créer puis de les modifier ou d'obtenir une copie dans certains cas.

## VI) Méthodes importantes

Afin de mieux comprendre les méthodes voici comment se décomposent les plateaux de jeu : 
* 0 : case d'océan non testé
* 1 à 5 : bateau non trouvé
* 6 : case d'océan testé
* 7 à 11 : bateau trouvé
* 12 : bateau coulé

### Méthodes communes : 

* **SendHit()** prend en paramètres la position choisie par le joueur (x, y) ainsi que le joueur attaqué.
```Py
def SendHit(self, x, y, Player):
    if self.validHit(x, y):
        id_boat = int(Player.defense_board.board[x][y])
        if id_boat == 0:
            self.attack_board.board[x][y] = 6
            Player.defense_board.board[x][y] = 6
            return (False, False, False)
        else:
            Player.defense_board.board[x][y] = 12
            self.attack_board.board[x][y] = id_boat
            self.destroyed[id_boat - 1][1] += 1
            if self.CheckDestroyed(id_boat - 1):
                self.Destroy(id_boat)
                self.destroyed_boats += 1
                if self.destroyed_boats == 5:
                    return (True, True, True)
            return (True, True, False)
    else:
        return (False, False, False)
```
Elle retourne 3 valeurs Booléennes correspondant à (validité du coup ?, bateau touché ?, partie terminée ?). Elle fait appel a **ValidHit()**
```py
def validHit(self, x, y):
    if 0 <= x <= 9 and 0 <= y <= 9:
        if self.attack_board.board[x][y] == 0:
            return True
    return False
```
qui va vérifier que le coup peut être envoyé, ainsi qu'à **CheckDestroyed()**
```py
def CheckDestroyed(self, id_boat):
    if list(self.boat[id_boat]) == self.destroyed[id_boat]:
        return True
    return False

```
et **Destroy()** 
```py
def Destroy(self, boat):
    for i in range(10):
        for j in range(10):
            if int(self.attack_board.board[i][j]) == boat:
                self.attack_board.board[i][j] = 12
```
qui chercheront et mettront à jour les bateaux entièrement coulés.

---

### Méthodes dédiés a l'IA : 

* **search_best_move()** est la fonction la plus importante du programme. C'est elle qui va trouver le coup le plus intéressant à faire jouer à l'IA
```py 
def search_best_move(self):
    self.dico = {}
    list_boat = []
    # Add all the remaining boat of the opponent defense board to a list
    for boat in self.destroyed:
        if not self.CheckDestroyed(boat[0] - 1):
            list_boat.append(self.boat[boat[0] - 1])
    # Launch the monte-carlo simulation depth times
    for _ in range(self.depth):
        for boat in list_boat:
            self.monte_carlo(boat[1])
    # Let the hunt begin
    if self.difficulty != "easy":
        self.hunt()

    # Get the best move with the heatmap
    coo = list(self.dico.keys())[
        list(self.dico.values()).index(max(self.dico.values()))
    ]
    if coo[0] < 0 or coo[1] < 0:
        del self.dico[coo]
    return list(self.dico.keys())[
        list(self.dico.values()).index(max(self.dico.values()))
    ]
```
Elle se décompose en plusieurs parties : 
1. Déterminer le nombre de bateaux restant à l'adversaire.
2. Effectuer une simulation Monte-Carlo avec ces informations et sa grille de jeu actuel afin de créer une Heatmap
3. Modifier les probabilités de la Heatmap autour des bateaux partiellement coulés
4. Trouver la case dont la probabilité est la plus élevée et la retournez

---
Une simulation **monte_carlo()** consiste à obtenir des résultats précis en partant d'aléatoire. Dans notre application de cette simulation à la bataille navale, nous transmettons à la fonction chacun des bateaux parmi ceux qui n'ont pas encore été coulés. La fonction va alors essayer de le placer à un endroit aléatoire, dans une orientation aléatoire : s'il y arrive nous ajoutons une probabilité dans notre dictionnaire ( sous la forme de : {(x,y): proba, (x,y): proba}).
```py
def monte_carlo(self, boat):
    placed = False
    board = self.attack_board.get_board(copy=True)
    tries = 0
    while placed == False:
        if tries == 100:
            placed = True
        vertical = randint(0, 1)
        if vertical:
            x, y = randint(0, 9 - boat + 1), randint(0, 9)
            valid = True
            for i in range(x, x + boat):
                if board[i][y] != 0:
                    valid = False
            if valid == True:
                for i in range(x, x + boat):
                    if (
                        (i % 2 == 0 and y % 2 == 1)
                        or (i % 2 == 1 and y % 2 == 0)
                        or self.difficulty in ["easy", "medium"]
                    ):
                        if (i, y) in self.dico:
                            self.dico[(i, y)] += 1
                        else:
                            if board[i][y] == 0:
                                self.dico[(i, y)] = 1
                    placed = True
        else:
            x, y = randint(0, 9), randint(0, 9 - boat + 1)
            valid = True
            for i in range(y, y + boat):
                if board[x][i] != 0:
                    valid = False
            if valid == True:
                for i in range(y, y + boat):
                    if (
                        (i % 2 == 0 and x % 2 == 1)
                        or (i % 2 == 1 and x % 2 == 0)
                        or self.difficulty in ["easy", "medium"]
                    ):
                        if (x, i) in self.dico:
                            self.dico[(x, i)] += 1
                        else:
                            if board[x][i] == 0:
                                self.dico[(x, i)] = 1
                    placed = True
        tries += 1
```

En difficulté moyenne, la heatmap ressemble généralement à ça lors du premier coup:

``{(4, 9): 1490, (5, 9): 1441, (6, 9): 1330, (7, 9): 1190, (8, 9): 951, (7, 0): 1323, (7, 1): 1695, (7, 2): 1937, (7, 3): 2000, (6, 7): 2000, (7, 7): 1886, (8, 7): 1589, (5, 0): 1563, (6, 0): 1487, (0, 0): 642, (1, 0): 962, (3, 7): 2009, (4, 7): 2090, (5, 7): 2058, (5, 1): 1893, (5, 2): 2237, (5, 3): 2312, (5, 4): 2374, (6, 1): 1803, (8, 1): 1391, (4, 6): 2379, (5, 6): 2328, (6, 6): 2204, (8, 2): 1639, (8, 3): 1765, (1, 8): 1393, (2, 8): 1688, (3, 
8): 1765, (4, 8): 1815, (5, 8): 1801, (0, 5): 1484, (0, 6): 1407, (0, 7): 1259, (0, 8): 992, (0, 2): 1316, (1, 2): 1644, (2, 2): 1879, (9, 9): 654, (4, 0): 1592, (8, 0): 1051, (0, 1): 992, (0, 3): 1427, (0, 4): 1490, (1, 5): 
1883, (2, 5): 2142, (3, 5): 2246, (4, 2): 2268, (4, 3): 2300, (4, 1): 1919, (4, 4): 2433, (6, 3): 2184, (6, 4): 2268, (6, 5): 2274, (5, 5): 2375, (2, 4): 2152, (3, 4): 2286, (2, 6): 2076, (2, 7): 1940, (2, 9): 1307, (0, 9): 675, (1, 9): 1024, (8, 6): 1785, (9, 6): 1467, (1, 1): 1331, (1, 3): 1765, (1, 4): 1893, (9, 5): 1530, (9, 7): 1281, (9, 8): 1048, (8, 8): 1312, (7, 4): 2116, (8, 4): 1896, (8, 5): 1911, (6, 2): 2165, (3, 2): 2126, (9, 2): 1297, (9, 3): 1433, (9, 4): 1483, (3, 6): 2192, (9, 1): 1077, (1, 7): 1628, (6, 8): 1718, (9, 0): 698, (7, 5): 2137, (7, 6): 1981, (3, 1): 1764, (3, 3): 2193, (2, 0): 1256, (1, 6): 1770, (2, 3): 1968, (4, 5): 2325, (7, 8): 1592, (3, 0): 1480, (3, 9): 1426, (2, 1): 1587}``

Représentation graphique :
![](https://i.imgur.com/Zb43qOi.png)

---
Si la difficulté n'est pas réglée sur facile, la fonction **hunt()** s'exécute et va chercher les bateaux partiellement coulés afin de terminer leur destruction.
```py
def hunt(self):
    board = self.attack_board.board
    for i in range(10):
        for j in range(10):
            neighbours = []
            if int(board[i][j]) not in [0, 6, 12]:
                for adj_x, adj_y in [
                    (i - 1, j),
                    (i + 1, j),
                    (i, j - 1),
                    (i, j + 1),
                ]:
                    try:
                        if board[adj_x][adj_y] == board[i][j]:
                            neighbours.append((adj_x, adj_y))
                    except:
                        pass
                if len(neighbours) == 0:
                    for adj_x, adj_y in [
                        (i - 1, j),
                        (i + 1, j),
                        (i, j - 1),
                        (i, j + 1),
                    ]:
                        try:
                            if int(board[adj_x][adj_y]) == 0:
                                if (adj_x, adj_y) in self.dico:
                                    self.dico[(adj_x, adj_y)] += self.depth
                                else:
                                    self.dico[(adj_x, adj_y)] = self.depth
                        except:
                            pass
                else:
                    for x, y in neighbours:
                        xaxys = x - i
                        yaxys = y - j
                        if xaxys != 0:
                            try:
                                if int(board[i - xaxys][j]) == 0:
                                    if (i - xaxys, j) in self.dico:
                                        self.dico[(i - xaxys, j)] += self.depth
                                    else:
                                        self.dico[(i - xaxys, j)] = self.depth
                            except:
                                pass
                        elif yaxys != 0:
                            try:
                                if int(board[i][j - yaxys]) == 0:
                                    if (i, j - yaxys) in self.dico:
                                        self.dico[(i, j - yaxys)] += self.depth
                                    else:
                                        self.dico[(i, j - yaxys)] = self.depth
                            except:
                                pass
```
La Heatmap modifiée ressemble alors à ça :
![](https://i.imgur.com/kteWLPg.png)


## VII) Captures d'écran

### Écran de placement des bateaux
![](https://i.imgur.com/JNvk36R.png)

---
### Bateaux placés prêts à être validé
![](https://i.imgur.com/aIaHQzZ.png)

---
### Début de partie
![](https://i.imgur.com/GwgEdwe.png)

---
### Partie commencé
![](https://i.imgur.com/qmHtoi0.png)

---
### Représentation graphique de la pensée de l'IA durant la partie
![](https://i.imgur.com/TbJ6AWq.gif)

