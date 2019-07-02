# Étiquettes = références

Plutôt que d'étiquettes, on parle plus couramment de références, mais l'idée est exactement la même : une variable est une référence vers une valeur. Deux variables distinctes peuvent référencer la même valeur. Une variable peut être réassignée pour référencer une valeur différente.

Chaque définition d'une variable en Python crée une nouvelle référence vers la valeur assignée.
Cela est vrai pour toute variable, incluant au passage les paramètres d'une fonction, qui deviennent lors de l'appel de nouvelles références vers les valeurs passées en arguments.
Il en est de même pour les valeurs insérées dans un conteneur (liste, *tuple*, dictionnaire, etc.) : c'est une référence vers la valeur qui est stockée dans le conteneur.

Tant qu'il existe au moins une référence vers une valeur, on dit que cette valeur est référencée. Une valeur référencée ne peut jamais être supprimée de la mémoire (cela poserait des problèmes pour les utilisations futures de la valeur via d'autres variables).

Comment alors supprimer une valeur ?

# Supprimer une valeur

Dans un premier temps il faut bien faire la distinction entre supprimer une variable et supprimer une valeur, les deux n'étant pas du tout équivalents.

Supprimer une variable, c'est faire en sorte que son nom n'existe plus. Ça se produit naturellement lorsque l'on sort du contexte dans lequel la variable est déclarée, c'est ce qu'il se passe pour les variables locales après l'exécution d'une fonction.

Cela peut aussi être déclenché manuellement à l'aide du mot-clé `del`. `del foo` a pour but de supprimer la variable `foo`, de faire en sorte que le nom `foo` ne corresponde plus à rien.

Quand une variable est supprimée, la référence vers la valeur est rompue. On dit que l'on déréférence la valeur.
Il y a d'autres moyens que la suppression de variable pour déréférencer une valeur : la réassignation est aussi très courante, comme dans le code suivant.

```python
>>> l = ['foo']
>>> l = ['bar'] # L'ancienne valeur de l est déréférencée
```

Le modèle mémoire de Python fonctionne à l'aide d'un compteur de références : chaque assignation d'une valeur à une variable incrémente ce compteur, et chaque suppression le décrémente.
Quand ce compteur atteint 0 (ce qui veut dire que la valeur n'est plus référencée par aucune variable, et donc plus accessible de nulle part dans le code), la valeur peut alors être supprimée en toute sécurité, et c'est le travail réalisé par le ramasse-miettes pour libérer la mémoire.

Ainsi, pour supprimer une valeur et libérer l'espace mémoire qu'elle occupe, il est nécessaire de la déréférencer totalement, de supprimer toutes les références vers cette valeur.
Cela concerne les variables aussi bien que les références stockées dans les conteneurs.
Donc un `del` sur une variable ne suffit pas si la valeur est toujours référencée par une autre variable, il faut aussi s'occuper des autres.

`del` peut d'ailleurs être utilisé pour supprimer tout type de référence et pas seulement les variables.

```python
>>> value = object() # value est une référence vers la valeur
>>> l = [value] # on crée une seconde référence depuis la liste
>>> d = {'key': value} # puis une troisième dans le dictionnaire
>>>
>>> del value # plus que deux références
>>> del l[0] # plus qu'une
>>> del d['key'] # plus du tout, la valeur est déréférencée
```

Quand Python supprime une valeur, il en appelle la méthode spéciale `__del__` (si elle en possède une).
Cette méthode permet d'intervenir juste avant la suppression de l'objet pour finaliser des traitements dessus.
Pour reprendre l'exemple précédent :

```python
>>> class Obj:
...     def __del__(self):
...         print('deleting', self)
...
>>> value = Obj()
>>> l = [value]
>>> d = {'key': value}
>>>
>>> del value
>>> del l[0]
>>> del d['key']
deleting <__main__.Obj object at 0x7f33ade3fba8>
```

On constate bien que ce n'est pas l'utilisation d'un `del` qui déclenche l'appel à `__del__`, mais bien le déréférencement total de notre valeur. La perte de toutes les références vers cette dernière.

Je n'ai utilisé ici que des déréférencements explicites, mais ils peuvent aussi être provoqués par des réassignations ou par un déréférencement parent (supprimer une liste revient à en déréférencer toutes les valeurs).

```python
>>> value = Obj()
>>> l = [value]
>>> d = {'key': value}
>>>
>>> value = None
>>> l[:] = []
>>> d['key'] = 20
deleting <__main__.Obj object at 0x7f33ade3fb38>
```

# Références cycliques

Dans tout cela, un cas qui peut être problématique est celui des références cycliques.
Que faire par exemple si un objet `obj1` contient une référence vers un objet `obj2`, qui contient lui-même une référence vers `obj1` ?

```python
>>> obj1 = Obj()
>>> obj2 = Obj()
>>> obj1.ref = obj2
>>> obj2.ref = obj1
>>> 
>>> del obj1
>>> del obj2
```

Comme vous le voyez, il ne se passe rien, les deux valeurs étant toujours référencées.
Mais à la sortie du programme, les références cycliques seront résolues et les valeurs supprimées.

```python
>>> ^D
deleting <__main__.Obj object at 0x7f8fcf561b70>
deleting <__main__.Obj object at 0x7f8fcf561ba8>
```

Il peut néanmoins être gênant de devoir attendre la fin du programme pour collecter ces valeurs (elles occupent inutilement de l'espace mémoire obligeant ainsi le programme à en allouer toujours plus), et dans ce genre de cas il est utile de pouvoir invoquer manuellement le ramasse-miettes.
Cela se fait à l'aide la méthode `collect` du module `gc`, qui renvoie le nombre de valeurs non atteignables trouvées et supprimées.

```python
>>> obj1 = Obj()
>>> obj2 = Obj()
>>> obj1.ref = obj2
>>> obj2.ref = obj1
>>> 
>>> del obj1
>>> del obj2
>>> 
>>> import gc
>>> gc.collect()
deleting <__main__.Obj object at 0x7f4077157b70>
deleting <__main__.Obj object at 0x7f4077157be0>
4
```

Les références cycliques sont assez courantes lorsqu'on travaille sur des représentations arborescentes et que l'on souhaite que les nœuds parents et enfants puissent se référencer.
C'est aussi le cas pour la gestion de données sous forme de graphes.

# Références faibles

Le problème des références cycliques provient du fait que le ramasse-miettes ne peut collecter les objets tant qu'ils sont référencés.
Une autre manière de le résoudre est alors d'utiliser des références qui n'empêchent pas ce ramasse-miettes de supprimer les valeurs.
On les appelle « références faibles » et elles sont fournies en Python par le module [`weakref`](https://docs.python.org/3/library/weakref.html).

Une référence faible est similaire à un appel de fonction qui renvoie l'objet si celui-ci est toujours référencé, ou `None` s'il a été supprimé.

```python
>>> import weakref
>>>
>>> obj1 = Obj()
>>> obj2 = Obj()
>>> obj1.ref = obj2
>>> obj2.ref = weakref.ref(obj1)
>>>
>>> obj2.ref
<weakref at 0x7f8de5d69408; to 'Obj' at 0x7f8de5d6b128>
>>> print(obj2.ref())
<__main__.Obj object at 0x7f8de5d6b128>
>>> obj2.ref() is obj1
True
>>>
>>> del obj1
deleting <__main__.Obj object at 0x7f8de5d6b128>
>>> print(obj2.ref())
None
```

-----------------

Quelques liens pour aller plus loin sur le sujet :

* https://rushter.com/blog/python-garbage-collector/
* https://docs.python.org/3/library/gc.html