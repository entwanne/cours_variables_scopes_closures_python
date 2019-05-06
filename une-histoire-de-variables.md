# Représentation des variables

Lorsqu'on exécute des calculs, appelle des fonctions ou effectue divers traitements, il est souvent intéressant de pouvoir en conserver le résultat. C'est tout le principe des variables : garder la trace d'une valeur.

Le concept de variables existe dans la majorité des langages de programmation, mais il peut prendre différentes formes. Les deux plus courantes vont être les variables sous forme de boîtes et les étiquettes.

Les boîtes sont la représentation que l'on rencontre dans des langages tels que le C, où une variable correspond à une case mémoire.
Il s'agit donc d'un nom associé à un emplacement mémoire, et assigner une valeur à la variable permet de stocker cette valeur à cet emplacement précis.
Deux variables distinctes correspondent à deux emplacements différents.

Les étiquettes sont la représentation utilisée par Python, elles peuvent sembler similaires aux boîtes à l'utilisation, mais diffèrent en bien des points.
En Python une variable est une étiquette -- juste un nom -- posée sur une valeur.
C'est-à-dire que la valeur existe quelque part en mémoire et qu'on vient lui attacher une étiquette.
On peut aisément placer plusieurs étiquettes sur une même valeur, mais aussi retirer une étiquette pour la placer sur une autre valeur.

La représentation des variables en Python est expliquée dans [cet article](http://foobarnbaz.com/2012/07/08/understanding-python-variables/) qui décrit très bien les choses.

# Déclaration et définition

Certains langages distinguent la déclaration et la définition d'une variable, ce n'est pas le cas en Python où les deux sont confondues. Mais il s'agit pourtant bien de deux concepts différents.

* Déclarer une variable, c'est indiquer au compilateur que tel nom existe dans tel contexte pour lui permettre de résoudre les utilisations de ce nom.
* Définir une variable, c'est lui assigner une valeur (soit en Python poser l'étiquette sur cette valeur).

`foo = 'bar'` en Python revient à déclarer une variable `foo` puis à la définir sur `'bar'`.
Par la suite, un `foo = 'rab'` dans le même contexte revient à redéfinir une variable déjà déclarée.
C'est-à-dire déplacer l'étiquette sur une autre valeur.

Pour être utilisée, une variable a besoin d'avoir été déclarée et définie, sans quoi vous rencontreriez une erreur de type `NameError` ou `UnboundLocalError` selon les cas.

```python
>>> def func():
...     print(x)
... 
>>> func()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in func
NameError: name 'x' is not defined
```

```python
>>> def func():
...     print(x)
...     x = 0
... 
>>> func()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in func
UnboundLocalError: local variable 'x' referenced before assignment
```

Vous pourriez vous demander pourquoi je tiens à marquer cette distinction si Python la masque, mais c'est parce que cela a de l'importance sur d'autres notions expliquées dans la suite du billet.

# Plusieurs étiquettes sur une même valeur (références multiples)

Comme je le disais, il est parfaitement envisageable d'avoir plusieurs étiquettes posées sur une même valeur, c'est même quelque chose de très courant.
Ça se produit même chaque fois que l'on assigne une variable à une autre.

```python
a = []
b = a
```

Ici nous créons une nouvelle liste  à laquelle nous ajoutons une première étiquette `a`. Puis nous ajoutons une seconde étiquette `b` sur cette même liste.
`a` et `b` référencent la même valeur, la même liste.

Cela peut être mis en évidence à l'aide de l'opérateur `is` :

```python
>>> a is b
True
```

Le résultat aurait été `False` si les deux listes avaient été distinctes :

```python
>>> c = []
>>> a is c
False
```

Notez que j'utilise ici des listes (objets mutables) afin de ne pas être être embêté dans mes exemples par les optimisations du compilateur, mais le fait que `b = a` ajoute une étiquette supplémentaire à la valeur existante est vrai pour tout type de valeur.
En d'autres termes, `a is b` sera toujours vrai après un `b = a`, quelle que soit la valeur initiale de `a`.

La conséquence de cela est que modifier `b` revient à modifier `a` (et inversement), puisque les deux étiquettes sont posées sur la même valeur :

```python
>>> b.append('value')
>>> a
['value']
>>> a += ['foobar']
>>> b
['value', 'foobar']
```

Dans ce dernier exemple, faites bien attention à l'opérateur `+=` qui n'est pas simplement un opérateur d'assignation mais peut aussi modifier la valeur existante.

Des cas de références multiples, on en rencontre très régulièrement en Python, en voici deux un peu plus insidieux.

### Multiplication de liste

```python
>>> table = [[0] * 4] * 3
>>> table[0][0] = 1
>>> table
[[1, 0, 0, 0], [1, 0, 0, 0], [1, 0, 0, 0]]
```

`table` est une liste contenant 3 fois la même valeur, modifier l'une d'elle revient à modifier les autres.

(Techniquement, `table[0]` est aussi une liste composée de 4 fois la même valeur, mais cette valeur étant immuable et donc redéfinie à chaque changement, il n'y a pas d'effet de bord.)

Pour pallier à ce problème, on utilisera plutôt une liste en intension avec un `range`.

```python
>>> table = [[0] * 4 for _ in range(3)]
>>> table[0][0] = 1
>>> table
[[1, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0]]
```

### Valeur par défaut d'un paramètre de fonction

```python
>>> def append_to_list(value, dest=[]):
...     dest.append(value)
...     return dest
... 
>>> append_to_list(10)
[10]
>>> append_to_list(15)
[10, 15]
```

Le problème ici est que les valeurs par défaut des paramètres sont définies une bonne fois pour toutes lors de la définition de la fonction, et conservées pour tous les appels.
Donc chaque appel à `append_to_list` utilisant la valeur par défaut référencera la même liste.

Pour contourner ce soucis, il est conseillé d'éviter les valeurs par défaut mutables, ou d'utiliser des sentinelles (`None` par exemple).

```python
>>> def append_to_list(value, dest=None):
...     if dest is None:
...         dest = []
...     dest.append(value)
...     return dest
... 
>>> append_to_list(10)
[10]
>>> append_to_list(15)
[15]
```