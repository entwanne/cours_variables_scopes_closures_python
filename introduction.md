Après plusieurs années à errer sur des forums d'entraide spécialisés en Python, il est courant de retomber régulièrement sur les mêmes problèmes.

Et s'il est un sujet qui concentre beaucoup d'incompréhension et d'interrogations, c'est celui de la gestion des variables.
J'ai en tête de nombreux sujets demandant pourquoi telle valeur est modifiée alors qu'elle ne devrait pas, d'utilisation du mot-clé `global` à tire-larigot, etc. C'est suite à [ce sujet](https://openclassrooms.com/forum/sujet/fonctionnement-des-decorateurs-python) qui rassemble quelques unes de ces erreurs que je me suis décidé à rédiger ce billet.

Les variables sont l'un des premiers mécanismes que l'on rencontre en Python, mais on peut facilement constater que leur fonctionnement est souvent mal compris (car malheureusement souvent mal enseigné aussi).
Elles forment un point d'incompréhension majeur du langage dont vont découler beaucoup d'autres, notamment en raison de la gestion des scopes et des variables mutables.