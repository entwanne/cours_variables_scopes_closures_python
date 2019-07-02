# Scope

Une variable n'est certes qu'un nom posé sur une valeur, mais plusieurs variables portant le même nom peuvent coexister au sein d'un programme.
Un nom de variable est pourtant unique, mais seulement dans le contexte où il est déclaré, dans son *scope*.

Une variable peut en effet être déclarée au niveau global d'un module et ainsi être accessible depuis n'importe où dans ce module (on dit que sa portée est globale), mais elle peut aussi être déclarée dans le *scope* d'une fonction et accessible depuis cette fonction uniquement (portée locale).

Il est donc tout à fait possible d'avoir une variable locale à une fonction portant le même nom qu'une variable du module, comme dans l'exemple suivant pour la fonction `g`.
La variable locale prendra la priorité sur la globale lors de la résolution du nom par le compilateur.

```python
>>> a = 0
>>> def f():
...     print('f: a =', a)
... 
>>> def g():
...     a = 1
...     print('g: a =', a)
... 
>>> print('global: a =', a)
global: a = 0
>>> f()
f: a = 0
>>> g()
g: a = 1
>>> print('global: a =', a)
global: a = 0
```

Le *scope* de déclaration définit la portée dans laquelle est accessible une variable.
Cette portée inclut le contexte courant, mais aussi tous les *scopes* enfants.
L'exemple précédent ne présentait que deux niveaux, global et local, mais il y en a en réalité une multitude, les *scopes* de fonctions pouvant s'imbriquer.

```python
>>> def outer():
...     def inner():
...         print(var)
...     var = 10
...     inner()
... 
>>> outer()
10
```

Ici, une variable déclarée dans le *scope* de la fonction `outer` est accessible dans `inner`. Et il en serait de même si `inner` définissait encore une sous-fonction, celle-ci aurait accès à `var` déclarée dans un *scope* parent.

L'inverse n'est pas vrai, une variable déclarée dans un *scope* enfant n'est pas accessible depuis le parent.

```python
>>> def outer():
...     def inner():
...         var = 10
...     print(var)
... 
>>> outer()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 4, in outer
NameError: name 'var' is not defined
```

Lorsqu'on quitte un *scope*, le *scope* ainsi que les variables qu'il contient sont détruit·e·s, ce qui permet de supprimer les valeurs si elles ne sont plus référencées.

```python
>>> class Obj:
...     def __del__(self):
...         print('deleting', self)
... 
>>> def func():
...     obj1 = Obj()
...
>>> func()
deleting <__main__.Obj object at 0x7f1f6eeedcc0>
```

Bien sûr, ça ne s'applique pas au cas où la fonction renverrait une valeur qui serait assignée à une variable, puisque l'on en conserverait alors une référence.

```python
>>> def func():
...     obj1 = Obj()
...     obj2 = Obj()
...     return obj1
... 
>>> ret = func()
deleting <__main__.Obj object at 0x7f1f6eeedcc0>
>>> del ret
deleting <__main__.Obj object at 0x7f1f6eeedc88>
```

Ce que cela nous montre aussi, c'est que bien qu'une variable soit associée à un *scope*, sa valeur peut, elle, transiter de *scope* en *scope* et ainsi remonter vers les parents à l'aide du `return`.
La variable référençant la valeur change (on passe de `obj1` à `ret`), mais la valeur reste toujours la même (elle n'est pas copiée).

# Variables locales et extérieures

Le corps d'une fonction a donc accès aux variables définies dans le *scope* courant (les variables locales), mais aussi à celles des *scopes* parents (les variables extérieures).
Ces variables extérieures ne se limitent pas aux variables globales : dans le premier exemple donné plus haut sur `outer` et `inner`, `inner` accède à une variable `var` qui est définie dans `outer`.

Il ne s'agit donc pas d'une variable globale (elle n'existe pas dans le *scope* global du module), mais d'une variable locale d'`outer`, et donc une variable extérieure (non-locale) à `inner`.

Les variables extérieures peuvent être utilisées de la même manière que les variables locales.
Et lors de la résolution du nom si elles sont plusieurs à porter le même nom, c'est la variable du *scope* parent le plus proche qui sera utilisée en priorité.

```python
>>> var = 10
>>> def outer():
...     var = 20
...     def inner():
...         print(var)
...     inner()
... 
>>> outer()
20
>>> var
10
```

Cet exemple nous amène à une petite subtilité des variables extérieures.
Nous y voyons que nous pouvons définir dans une fonction une nouvelle variable portant le nom d'une variable extérieure (sans que cela n'affecte la variable extérieure).
Comment alors redéfinir la valeur d'une variable extérieure ?

Il est dans ce cas nécessaire d'indiquer à Python qu'une variable de ce nom est déjà déclarée à l'extérieur.
Pour cela, deux mots-clés sont à notre disposition : `global` et `nonlocal`.

Le premier permet d'indiquer qu'une variable est globale, et Python comprendra qu'elle devra être déclarée et définie dans le *scope* global.
Il s'utilise suivi d'un ou plusieurs noms de variables : `global foo` ou `global foo, bar, baz`.

```python
>>> x = 5
>>> def define_globals():
...     global x
...     x = 10
... 
>>> x
5
>>> define_globals()
>>> x
10
```

On notera que la variable n'a pas besoin d'avoir déjà été définie dans le *scope* global pour être utilisée dans notre fonction.

```python
>>> def define_globals():
...     global y
...     y = 2
... 
>>> y
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'y' is not defined
>>> define_globals()
>>> y
2
```

Le mot-clé `nonlocal` est similaire mais pour indiquer qu'une variable existe dans un *scope* local parent. Au contraire de `global`, cette variable doit donc nécessairement avoir été définie dans un *scope* parent (autre que le *scope* global).

```python
>>> def outer():
...     x = 0
...     def inner():
...         y = 0
...         def inception():
...             nonlocal y
...             y = 10
...         inception()
...         nonlocal x
...         x = y
...     inner()
...     return x
... 
>>> outer()
10
```

L'omission de l'un de ces deux `nonlocal` aurait pour effet de redéclarer une variable locale (`x` ou `y`), et donc ne permettrait pas la remontée de la valeur `10`.

Si plusieurs *scopes* parents déclarent une variable du même nom, c'est la variable du *scope* le plus proche qui sera utilisée, comme c'est le cas pour tout accès à une variable extérieure.

-------------------------

Par ailleurs, il est souvent dit que les mots-clés `global` et `nonlocal` sont nécessaires pour modifier une variable extérieure, pour y accéder en écriture.

Cela est faux, ils ne sont nécessaires que pour la redéfinir.
Il est par exemple parfaitement possible d'utiliser depuis une fonction la méthode `append` d'une liste définie au niveau global, sans avoir à utiliser le mot-clé `global`.

```python
>>> values = []
>>> def append_value(value):
...     values.append(value)
... 
>>> append_value(0)
>>> append_value(1)
>>> append_value(2)
>>> values
[0, 1, 2]
```

Ainsi, les utilisations de `global` ou `nonlocal` sont en réalité plutôt rares, et il est généralement déconseillé d'utiliser ces mots-clés (ils prêtent à confusion).
Mais leur connaissance permet de résoudre certains cas problématiques.

# Closure (fermeture)

Je disais quelques paragraphes plus haut que le contexte local à une fonction est détruit à la sortie de cette fonction.
Je précisais ensuite que cela n'entraînait pas nécessairement la destruction de toutes les valeurs de ce contexte si certaines étaient toujours référencées, comme ça peut être le cas de la valeur de retour de la fonction.

Un autre cas intéressant est celui des *closures* (fermetures). Celles-ci vont permettre de capturer des variables locales pour qu'elles soient toujours accessibles dans un contexte particulier.

Je sais que ces explications ne sont pas très claires et je préfère alors vous fournir tout de suite un exemple :

```python
>>> def cached_addition():
...     cache = {}
...     def addition(x, y):
...         if (x, y) not in cache:
...             print(f'Computing {x} + {y}')
...             cache[x, y] = x + y
...         return cache[x, y]
...     return addition
... 
>>> addition = cached_addition()
>>> addition(1, 2)
Computing 1 + 2
3
>>> addition(1, 2)
3
>>> addition(2, 3)
Computing 2 + 3
5
```

Ce n'est pas l'exemple le plus utile, mais on pourrait imaginer une fonction plus complexe à la place de l'addition, dont la mise en cache du résultat serait nécessaire.

L'idée ici est que notre fonction `cached_addition` retourne à chaque appel une nouvelle fonction `addition` créée dynamiquement, utilisant un cache particulier.
Ce cache est une variable définie localement dans `cached_addition` et donc accessible depuis `addition`.

Cependant, une fois l'appel à `cached_addition` terminé, son *scope* local est détruit, ce qui doit impliquer la destruction des valeurs qu'il contient.
Ici, on voit bien que `cache` lui survit puisqu'il continue à être utilisé sans problème par la fonction `addition`.

Ce qu'il se passe c'est que la fonction `addition` crée une *closure* qui emprisonne les variables locales des *scopes* parents qu'elle utilise. Cela permet à ces valeurs d'être toujours référencées.
On peut d'ailleurs constater que notre fonction `addition` possède un attribut spécial `__closure__`.

```python
>>> addition.__closure__
(<cell at 0x7f3a700a5d98: dict object at 0x7f3a70174fc0>,)
>>> addition.__closure__[0].cell_contents
{(1, 2): 3, (2, 3): 5}
```

L'intérêt des *closures*, c'est que plusieurs appels à `cached_addition` distincts renverront des fonctions utilisant un cache différent, parce qu'il s'agira à chaque fois d'une nouvelle variable locale.

```python
>>> other_addition = cached_addition()
>>> other_addition(1, 2)
Computing 1 + 2
3
```

Le mécanisme de *closures* est souvent utilisé au sein de décorateurs puisqu'il permet de facilement attacher des variables à une fonction créée dynamiquement (quelques exemples peuvent être trouvés [ici](https://zestedesavoir.com/tutoriels/954/notions-de-python-avancees/5-exercises/2-3-decorators/)).

Nous avons vu que la *closure* permettait la persistance en mémoire de certaines valeurs en les capturant.
Mais cette *closure* n'est pas éternelle et disparaît naturellement quand elle est elle-même déréférencée, c'est à dire quand la fonction qui emprisonne ces valeurs disparaît.

```python
>>> class Obj:
...     def __del__(self):
...         print('deleting', self)
... 
>>> def outer():
...     obj = Obj()
...     def inner():
...         print(obj)
...     return inner
... 
>>> func1 = outer()
>>> func2 = outer()
>>> func1()
<__main__.Obj object at 0x7f58ba68dd68>
>>> func2()
<__main__.Obj object at 0x7f58ba68de10>
>>> del func1
deleting <__main__.Obj object at 0x7f58ba68dd68>
>>> del func2
deleting <__main__.Obj object at 0x7f58ba68de10>
```

Et pour terminer sur les *closures* voici un autre billet expliquant le concept avec des exemples en Python et en JS : http://sametmax.com/closure-en-python-et-javascript/.

# Délimitation d'un scope

On pourrait croire, comme c'est le cas dans d'autres langages, qu'un nouveau *scope* est créé pour chaque bloc de code (identifié en Python par le niveau d'indentation).
Mais le code suivant, plutôt courant, serait alors invalide :

```python
>>> items = [1, 2, 3, ...]
>>> if items:
...     value = items[0]
... else:
...     value = None
... 
>>> value
1

```

On remarque bien ici que notre variable `value` est définie dans le *scope* courant et non dans un *scope* fils créé spécialement pour le bloc conditionnel. Le niveau d'indentation n'a pas d'incidence sur le *scope*.

En fait, outre les modules, qui forment le *scope* global, seules les classes et les fonctions permettent de définir de nouveaux *scopes*. Ce n'est donc pas le cas des autres blocs de code comme les conditions, les boucles ni même les blocs `with`.

Cela peut être particulièrement trompeur pour les boucles `for`, et notamment pour leur variable d'itération qui n'est donc pas propre à la boucle. Elle est définie et existe à l'extérieur.

```python
>>> for i in range(10):
...     pass
... 
>>> i
9
```

Ce qui peut mener à des cas plus embêtants si l'on imaginait que cette variable serait capturée dans une *closure* propre à la boucle.

```python
>>> functions = []
>>> for i in range(3):
...     def add_func(x):
...         return i + x
...     functions.append(add_func)
... 
>>> functions
[<function add_func at 0x7ff229851268>, <function add_func at 0x7ff2298512f0>, <function add_func at 0x7ff229851378>]
>>> [f(0) for f in functions]
[2, 2, 2]
```

Toutes les fonctions `f` renvoient la même valeur pour 0 car toutes accèdent à la même variable `i`, qui vaut 2 en sortie de boucle.

La solution dans ce genre de cas est donc de créer une fonction englobante et de l'appeler afin de tirer profit du mécanisme des *closures*. L'exemple précédent pourrait être corrigé comme suit.

```python
>>> functions = []
>>> for i in range(3):
...     def get_add_func(i):
...         def add_func(x):
...             return i + x
...         return add_func
...     functions.append(get_add_func(i))
... 
>>> [f(0) for f in functions]
[0, 1, 2]
```

La fonction `get_add_func` (qui pourrait tout aussi bien être placée en dehors de la boucle, ce qui serait même préférable) possède un paramètre `i` qui sera capturé dans la *closure* de la sous-fonction `add_func`.
Ainsi, à chaque tour de boucle `get_add_func` est appelée avec une valeur différente, et c'est cette valeur qui est chaque fois emprisonnée.


# Variables non déclarées ou non définies

Au tout début de ce billet j'évoquais les exceptions `NameError` et `UnboundLocalError` qui peuvent être levées lors de l'accès à une variable.

La première (`NameError`) est levée si le nom d'une variable n'existe pas, c'est-à-dire qu'elle n'est déclarée ni dans le *scope* courant, ni dans les parents.

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

La seconde (`UnboundLocalError`) est plus subtile et survient pour une variable déclarée mais non définie.

Pour rappel, l'instruction `x = 10` a pour effet de déclarer la variable `x` puis de la définir avec pour valeur 10.
Ces deux opérations n'interviennent pas au même moment : la définition se fait au moment où cette instruction est rencontrée, mais la déclaration est valable pour tout le *scope*. C'est donc comme si la variable avait été déclarée au tout début du *scope*.

Ainsi, prenons l'exemple suivant :

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

Nous essayons ici d'afficher la valeur de `x` mais celle-ci n'est pas encore définie. En revanche, le nom de notre variable existe bel et bien, car elle est déclarée dans le *scope* courant, à la ligne suivante.

Cette erreur est très courante quand on souhaite redéfinir une variable globale après l'avoir utilisée, mais en omettant le mot-clé `global`.

```python
>>> var = 0
>>> 
>>> def func1():
...     print(var)
...     var = 10
... 
>>> func1()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 2, in func1
UnboundLocalError: local variable 'var' referenced before assignment
>>> 
>>> def func2():
...     global var
...     print(var)
...     var = 10
... 
>>> func2()
0
>>> var
10
```