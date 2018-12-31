---
title: "La valeur d'un but (partie 1)"
slug: "/la-valeur-dun-but"
date: "2017-09-10T09:39:02.000Z"
image: "./images/football-1274661_640_ajf5ij.jpg"
featured: false
draft: false
tags: ["football","ligue1"]
---

Je me suis souvent demandé si certains buts valaient plus que d'autres et surtout comment calculer cela... Doit-on considérer la valeur au moment du but ou à la fin du match ?

#### Préambule - rapport entre buts et points, l'unité de valeur

Le **point**, celui des classements, sera l'unité de mesure de la valeur d'un but (on aurait pu se poser la question de penser en terme de probabilité de victoire).
Il y a un lien direct, bien que non linéaire, entre buts et points puisque les buts permettent dans certaines conditions de marquer des points.

Les règles de distribution des points pour un match de championnat sont simples :

* 0 point si l'adversaire marque plus de buts
* 1 point si l'adversaire marque autant de buts
* 3 points si l'adversaire marque moins de buts

Il s'agirait donc de *répartir* les points remportés après chaque match entre les différents buts ?
Mais alors suivant quels critères ?

#### Le déroulement du match

Itérons à partir de différentes situations pour tenter de comprendre comment nous allons procéder :

* 1 - 0 score final pour **l'équipe A** : dans ce cas, c'est facile, il y a un seul but, qui a rapporté 2 points supplémentaires à son équipe
* 2 - 0 score final pour **l'équipe A** : déjà ça se corse, les deux buts doivent-ils être notés pareil ? Je tendrais à dire que le premier but change plus la donne, le deuxième est utile en cas de remontée adverse mais le premier provoque un changement dans la répartition des points
* 5 - 0 score final ; il me semble que plus l'écart est important, moins la valeur d'un but est grande
* **L'équipe A** mène 2 - 0 mais **l'équipe B** marque un but dans les 10 dernières minutes, score final de 2 - 1. Est-ce que le but du joueur de **l'équipe B** ne vaut rien puisqu'il n'a rapporté aucun point ? C'est difficilement acceptable, car il permet à son équipe d'espérer un match nul dans les dernières minutes du jeu. De manière générale, je pense qu'un but ne devrait jamais valoir 0 points.
* **L'équipe A** mène 2 - 0, mais **l'équipe B** égalise en marquant 2 fois après. Là aussi, est-ce que les buts de **l'équipe A** doivent être dévalués ?
En quoi valent-ils moins que si **l'équipe B** n'avait pas marqué ?
* Plus tordu : 1 - 0 score final, mais la situation me semble différente si le but est marqué à la 2è plutôt qu'à la 90è. Dans ce dernier cas, il est pratiquement impossible que l'équipe adverse parvienne à égaliser.

Il y a beaucoup de points délicats, mais je vois émerger un pattern : 

On se concentrera sur la valeur attendue d'un but plutôt que sur sa valeur réelle, car plus *juste*.
Un but doit être valorisé suivant la différence de points qu'il apporte à l'équipe qui le marque, 
mais il faut également prendre en compte, non pas les buts suivants marqués au cours de la rencontre, mais plutôt l'éventail des possibles scores après ce but et voir à chaque fois comment notre but a influé sur le nombre de points attribués à l'équipe. Chaque possibilité sera pondérée suivant sa probabilité.

Ainsi on obtient la **Valeur Attendue d'un But** (en points) :

* `la somme` (pour tous les cas de figures dans l'évolution du score après le but B) de :
  * `la différence de points avec ou sans le but B`
  * `pondérée par la probabilité du cas de figure`

prenons l'exemple le plus simple qui représente à mon sens la valeur maximale d'un but : 
Le score est de 0 - 0, une équipe marque à la toute fin des arrêts de jeu

* Si le score en reste là
  * le nombre de points de l'équipe avec le but - le nombre de points de l'équipe sans le but = 3 - 1 = 2
  * Probabilité (que le score en reste là) ~ 1

→ `on obtient une valeur de ~ 2 points pour ce but`

En revanche si le premier but est marqué à la 5ème minute : 

* Si le score en reste là :
  * différence avec ou sans le but = 2
  * probabilité ~ 0.3
* Si l'équipe adverse égalise
  * différence de points avec ou sans le but (1 - 0)
  * probabilité  ~ 0.2
* Si l'équipe marque un autre but
  * différence de points avec ou sans le but 3 - 3 = 0
  * probabilité ~ 0.2
* Si les deux équipes marque 1 fois chacune
  * différence de points avec ou sans le but 3 - 1
  * probabilité ~ 0.1
* ...

→ `si on somme le tout, on obtient une valeur beaucoup moins importante pour ce but qu'avec l'exemple précédent, ce qui semble cohérent`.

Le tout est désormais d'obtenir des probabilités pour chaque possibilité. Pour cela nous nous baserons sur les précédentes saisons de ligue 1 pour voir en un temps restant donné, quelles sont les probabilités d'évolution du score et nous stockerons le tout dans une table.

Afin d'éviter le sur-apprentissage : 

* nous calculerons les probabilités sur des tranches de 5 minutes
* et nous nous bornerons à ne  calculer que les probabilités de nombre de buts dans les X dernières minutes, réparties ensuite entre les deux équipes... on pourrait penser à des modèles plus complexes (domicile / extérieur et en tenant compte du score au moment du but)

#### Bonus : le niveau de l'adversaire ?

Maintenant que nous avons un nombre de points, doit-on le pondérer en fonction du niveau de l'adversaire ?

Deux points de vue s'affrontent ici ; le premier, à priori rationnel, prévaut qu'un point vaut exactement la même chose  dans le classement final, qu'il soit remporté face à une équipe ou une autre.

Mais quelque chose en nous a aussi tendance à dire qu'un point face à un adversaire bien classé doit valoir plus.

Pour comprendre comment nous allons procéder, il faut partir du principe que dans un monde entièrement équitable, il n'y aurait que des matchs nuls. Mais de fait les matchs sont déséquilibrés. Une façon d'évaluer ce déséquilibre est d'utiliser les cotes des bookmakers, qui ne sont jamais qu'une enquête d'opinion et dont on peut tirer des points pondérés.

On peut ainsi obtenir un coefficient `E` : 
```math
E = points dans match parfaitement équitable / points attendus face à l'adversaire
Soit :
E = 1 / expected points
```

Qui nous permettra de pondérer la `Valeur Attendue d'un But` relativement au niveau des deux adversaires. 

Par exemple : 

* **A** rencontre **B**
* les cotes sont largement en faveur de **A** : 60% pour sa victoire, 30% pour un nul, et 10% pour sa défaite
* soit 2.1 expected points pour **A** (0.6*3pts + 0.3*1pt) et 0.6 expected points pour **B**
* `E(A) = 1/2.1 = 0.48 et E(B) = 1/0.6 = 1.67`
* les `Valeurs Attendues` de chaque but de **A** et **B** pourront donc être pondérés par leurs coefficients respectifs

---

Je ne suis pas certain de la validité ni de la pertinence de la pondération en fonction de l'adversaire mais je préfère en garder une trace.

Dans un prochain article, je calculerai la table de probabilité de l'évolution des scores et fournirai quelques exemples concrets de la `Valeur Attendue d'un But`.
