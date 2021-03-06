# (PART) Cours II: analyse {-}

# Classification hiérarchique {#hierarchique}




##### Objectifs {-}

- Comprendre la notion de distance et la matrice de distance.

- Appréhender la classification hiérarchique et le dendrogramme.

- Être capable d'effectuer un regroupement pertinent des individus d'un jeu de données multivarié à l'aide de ces techniques.


##### Prérequis {-}

Vous devez être à l'aise avec l'utilisation de R et Rstudio, en particulier pour l'importation, le remaniement et la visualisation de données multivariées. Ceci correspond au cours SDD I. Il n'est pas nécessaire d'avoir acquis toutes les notions vue dans la partie **Cours II :modélisation** pour pouvoir comprendre cette seconde partie du cours. Si vous ne vous sentez pas assez à l'aise avec R et RStudio, c'est peut-être le bon moment pour refaire le premier "learnr" du package `BioDataScience2`\ :

\BeginKnitrBlock{bdd}<div class="bdd">Démarrez la SciViews Box et RStudio. Dans la fenêtre **Console** de RStudio, entrez l'instruction suivante suivie de la touche `Entrée` pour ouvrir le tutoriel concernant les bases de R\ :

    BioDataScience2::run("01a_rappel")

N’oubliez pas d’appuyer sur la touche `ESC` pour reprendre la main dans R à la fin d’un tutoriel dans la console R.</div>\EndKnitrBlock{bdd}


## Analyse de données

L'analyse de données (on dit aussi *analyse exploratoire des données*, EAD ou **statistiques exploratoires**) mets en œuvre des méthodes statistiques multivariées visant à découvrir de l'information pertinente dans un gros jeu de données via des approches multidimensionnelles et essentiellement descriptives. Ces méthodes se regroupent en deux grandes familles\ :

- Celles visant à **réduire la dimensionnalité** (travailler avec des tableaux ayant moins de colonnes). Elles permettent ensuite de présenter les données de manière synthétique pour observer des relations entre les variables ou les individus via des **représentations graphiques**. Nous aborderons ces techniques dans les modules suivants.

- Celles cherchant à **classifier** (ou regrouper) les individus. Il s'agit ici de synthétiser le gros tableau de données dans l'autre sens, selon les lignes. L'approche via la *classification hiérarchique* sera détaillée ici.

La vidéo suivante introduit l'EAD (jusqu'à 2:11)\ :

<!--html_preserve--><iframe src="https://www.youtube.com/embed/q-IVQoh1nxA?end=131" width="770" height="433" frameborder="0" allowfullscreen=""></iframe><!--/html_preserve-->


## Distance entre individus

Vous êtes en train d'analyser des données concernant les échantillons de plancton que vous avez prélevé sur votre lieu de recherche. Ce plancton a été numérisé (photo de chaque organisme) et les images ont été traitée avec un logiciel qui mesure automatiquement une vingtaine de variables telles que la surface de l'objet sur l'image, son périmètre, sa longueur, ... Vous vous trouvez donc face à un jeu de données qui a une taille non négligeable\ : 20 colonnes par 1262 lignes, soit le nombre d'individus mesurés dans vos échantillons.


```r
zoo <- read("zooplankton", package = "data.io")
zoo
```

```
# # A tibble: 1,262 x 20
#      ecd  area perimeter feret major minor  mean  mode   min   max std_dev
#    <dbl> <dbl>     <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>   <dbl>
#  1 0.770 0.465      4.45 1.32  1.16  0.509 0.363 0.036 0.004 0.908   0.231
#  2 0.700 0.385      2.32 0.728 0.713 0.688 0.361 0.492 0.024 0.676   0.183
#  3 0.815 0.521      4.15 1.33  1.11  0.598 0.308 0.032 0.008 0.696   0.204
#  4 0.785 0.484      4.44 1.78  1.56  0.394 0.332 0.036 0.004 0.728   0.218
#  5 0.361 0.103      1.71 0.739 0.694 0.188 0.153 0.016 0.008 0.452   0.110
#  6 0.832 0.544      5.27 1.66  1.36  0.511 0.371 0.02  0.004 0.844   0.268
#  7 1.23  1.20      15.7  3.92  1.37  1.11  0.217 0.012 0.004 0.784   0.214
#  8 0.620 0.302      3.98 1.19  1.04  0.370 0.316 0.012 0.004 0.756   0.246
#  9 1.19  1.12      15.3  3.85  1.34  1.06  0.176 0.012 0.004 0.728   0.172
# 10 1.04  0.856      7.60 1.89  1.66  0.656 0.404 0.044 0.004 0.88    0.264
# # … with 1,252 more rows, and 9 more variables: range <dbl>, size <dbl>,
# #   aspect <dbl>, elongation <dbl>, compactness <dbl>, transparency <dbl>,
# #   circularity <dbl>, density <dbl>, class <fct>
```

Vous voulez regrouper votre plancton en fonction de la ressemblance entre les organismes, c'est-à-dire, en fonction des écarts entre les mesures effectuées pour les 19 variables quantitatives, à l'exclusion de la vingtième colonne `class` qui est une variable `factor`). En raison de la taille du tableau, il est évident que cela ne pourra pas se faire de manière manuelle. Nous pouvons raisonnablement considérer que plus les mesures sont similaires entre deux individus, plus ils ont des chances d'être semblables, c'est-à-dire, d'appartenir au même groupe taxonomique. **Mais comment faire pour synthétiser l'information de similarité ou différence contenue dans 19 paires de valeurs** (une paire par variable) \ ? Nous avons besoin d'une **mesure de distance** qui quantifie la similarité (ou à l'inverse la dissimilarité) en un seul nombre. Celle qui vient naturellement à l'esprit est la **distance euclidienne**. Prenons un cas simplifié. Quelle est la distance qui sépare deux individus *A* et *B* par rapport à trois variables *x*, *y*, *z*\ ? Ici, nous pouvons représenter l'information graphiquement dans un espace à trois dimensions. La distance qui nous intéresse est la distance linéaire entre les deux points dans l'espace. Autrement dit, c'est la longueur du segment de droite qui relie les deux points dans l'espace. Cette distance, nous pouvons la calculer à l'aide de la formule suivante (problème de niveau lycée, voir par exemple [ici](http://www.mathematiques-lycee.com/geometrie/2nde-01-longueur-segment.html) pour une résolution dans le plan)\ :

$$\mathrm{D_{Euclidean}}_{A, B} = \sqrt{(x_A - x_B)^2 + (y_A - y_B)^2 + (z_A - z_B)^2}$$

Notez que cette formule se généralise à *n* dimensions et s'écrit alors, pour n'importe quelle paire d'individus indicés *j* et *k* dans notre tableau et pour les différentes mesures de 1 à *i* notées *y~i~*\ :

$$\mathrm{D_{Euclidean}}_{j, k} = \sqrt{\sum_{i=1}^{n}(y_{ij}-y_{ik})^2}$$

C'est la racine carré de la somme dans les *n* dimensions des écarts entre les valeurs au carré pour toutes les variables *y~i~*. Plus sa valeur est grande, plus les individus sont éloignés (différents). Pour cette raison, nous appelerons cette distance, une mesure de **dissimilarité**.


### Matrice de distances

Nous avons maintenant la possibilité de quantifier la similitude entre nos organismes plactoniques... mais nous en avons un grand nombre. Cela va être impossible à gérer autant de mesures qu'il y a de paires possibles parmi 1262 individus^[Le nombre de paires uniques et distinctes (pas *j*, *j* ou *k*, *k*) possibles parmi *n* items est $n(n-1)/2$, soit ici pour 1262 éléments nous avons 795.691 paires.]. La **matrice de distance** est une matrice ici 1262 par 1262 qui rassemble toutes les valeurs possibles. Notez que sur la diagonale, nous comparons chaque individu avec lui-même. La distance euclidienne vaut donc systématiquement zéro sur la diagonale.

$$\mathrm{D_{Euclidean}}_{j, j} = 0$$

De plus, de part et d'autre de cette diagonale, nous trouvons les paires complémentaires (*j versus k* d'un côté et *k versus j* de l'autre). Or qu'elle soit mesurée dans un sens ou dans l'autre, la distance du segment de droite qui relie deux points dans l'espace est toujours la même.

$$\mathrm{D_{Euclidean}}_{j, k} = \mathrm{D_{Euclidean}}_{k, j}$$

Par conséquent, seulement une portion (soit le triangle inférieur, soit le triangle supérieur hors diagonale) est informative. La diagonale ne porte aucune informartion utile, et l'autre triangle est redondant. Nous avons donc pour habitude de ne calculer et représenter que le triangle inférieur de cette matrice. Maintenant que cela est clair, nous pouvons créer un objet `dist` qui contiendra notre matrice de distances enclidiennes. Il suffit d'utiliser la fonction `dist()`, ou mieux `vegan::vegdist()` qui offre plus de possibilités^[La fonction `dist()` ne propose par exemple pas la méthode de Bray-Curtis qui est fréquemment utilisée en biologie et en écologie, contrairement à `vegan::vegdist(method = "bray")`.]. Comme cela prendrait trop de place d'imprimer la matrice complète, nous allons réaliser le travail sur seulement les six premiers individus de notre tableau (et nous devons aussi éliminer la colonne `class` qui ne contient pas de données numériques et qui ne nous intéresse pas pour le moment)\ :


```r
zoo %>.%
  select(., -class) %>.% # Elimination de la colonne class
  head(., n = 6) -> zoo6 # Récupération des 6 premiers individus
zoo6_dist <- vegan::vegdist(zoo6, method = "euclidean")
zoo6_dist # En pratique, on n'imprime généralement ce genre d'objet !
```

```
#            1          2          3          4          5
# 2  8.2185826                                            
# 3  2.5649705  5.7320911                                 
# 4  0.8582142  7.8813537  2.2220079                      
# 5  4.8478629  4.2950067  3.0119468  4.6148173           
# 6  2.4269520 10.6197317  4.9228255  2.8477882  7.1766853
```

Nous voyons bien ici que R n'imprime que le *triangle inférieur* de notre matrice 6 par 6. Notez aussi que les objets `dist` de tailles plus réalistes que vous génèrerez dans vos analyses ne sont **prévue** pour être imprimées et visualisées telles quelles. Il s'agit seulement de la *première étape* vers une représentation utile qui sera réalisée à la page suivante, à l'aide de la classification hiérarchisée.

**Félicitations\ ! Vous venez de calculer votre première matrice de distances.** Nous verrons à la page suivante comment nous pouvons utiliser l'information qu'elle contient pour regrouper les individus de manière pertinente. Mais avant cela, nous avons besoin d'un peu de théorie pour bien comprendre quelle **métrique** choisir pour calculer nos distances et pourquoi. On parle aussi d'**indices** de **similarité** ou **dissimilarité**.

\BeginKnitrBlock{warning}<div class="warning">
Attention\ : nous n'avons pas considéré ici les unités respectives de nos variables. Une surface (mm^2^) ou une longeur (mm) ne sont pas mesurées dans les mêmes unités. Nous risquons alors de donner plus de poids aux valeurs élevées. Nous aurions le même effet si nous décidions par exemple d'exprimer une mesure longitudinale en µm au leiu de l'exprimer en mm. Dans ce cas, il vaut mieux standardiser d'abord le tableau (moyenne de zéro et écart type de un) selon les colonnes avant d'effectuer le calcul. Ceci sera fait à la page suivante.
</div>\EndKnitrBlock{warning}

### Indices de (dis)similarité

Un indice de similarité (*similarity index* en anglais) est une descripteur statistique (nombre unique) de la similitude de deux échantillons ou individus représentés par plusieurs variables dans un échantillon multivarié. Un indice de similarité prend une valeur comprise entre 0 (différence totale) et 1 ou 100% (similitude totale). Un indice de dissimilarité} est le complément d’un indice de similarité (dis = 1 – sim); sa valeur est comprise entre 100% (différence totale) et 0 (similitude totale).

\BeginKnitrBlock{warning}<div class="warning">
Attention\ : dans certains cas, un indice de dissimilarité peut varier de 0 à +$\infty$**. Il n'existe alors pas d'in,dice de similarité complémentaire. C'est le cas précisément de la distance euclidienne que nous avons exploré jusqu'ici.
</div>\EndKnitrBlock{warning}

Tous les indices de similarité / dissimilarité peuvent servir à construire des matrices de distances.


#### Indice de Bray-Curtis

L'indice de dissimilarité de Bary-Curtis, aussi appelé coefficient de Czecanowski est calculé comme suit\ :

$$\mathrm{D_{Bray-Curtis}}_{j,k}=\frac{\sum_{i=1}^{n}\left|y_{ij}-y_{ik}\right|}{\sum_{i=1}^{n}(y_{ij}+y_{ik})}$$

Dans R nous utiliserons `vegan::vegdist(DF, method = "bray")`.Il s’utilise pour mesurer la similitude entre échantillon sur base du **dénombrement d’espèces**. Si le nombre d’individus est très variable (espèces dominantes _versus_ espèces rares), nous devons transformer les données pour éviter de donner trop de poids aux espèces les plus abondantes (ex: $log(x+1)$, double racine carrée, ...).

Une caractéristique essentielle de cet indice (contrairement à la distance euclidienne) est que toute double absence n'est pas prise en compte dans le calcul. C'est souvent pertinent dans le cadre de son utilisation comme le dénombrement d'espèces. En effet, quelle information utile retire-t-on de doubles zéros dans un tableau répertoriant la faune belge pour le crocodile du Nil et le tigre de Sibérie par exemple\ ? Aucune\ ! Ils sont tous deux systématiquement absents des dénombrements, mais cette double absence n'apporte aucune information utile pour caractériser la faune belge par ailleurs.

L'indices de similarité de Bray-Curtis (*sim*) est complémentaire à l'indices de dssimilarité correspondant (*dis* tel que calculé ci-dessus)\ : 

$$sim = 1 – dis$$



#### Indice de Canberra

L'indice de dissimilarité de Canberra est similaire à l'indice de Bray-Curtis mais il pondère les espèces en fonction du nombre d’occurrences afin de donner le *même* poids à chacune dans le calcul. Il se calcule comme suit\ :

$$\mathrm{D_{Canberra}}_{j,k}=\frac{1}{nz}\sum_{i'=1}^{nz}\frac{\left|y_{i'j}-y_{i'k}\right|}{\left|y_{i'j}\right|+\left|y_{i'k}\right|}$$

où $nz$ est le nombre de valeurs non nulles simultanément dans le tableau de départ. Toutes les espèces contribuent ici de manière égale. C'est un point positif, mais il faut faire attention à ce que cet indice a souvent tendnace à donner une surimportance aux espècces très rares observées une seule fois ou un petit nombre de fois\ ! Dans R, nous utiliserons `vegan::vegdist(DF, method = "canberra")`.

Toute double absence n’est pas prise en compte ici aussi. Seuls les indices ne dépendant pas des doubles zéros sont utilisables pour des dénombrements d’espèces ou des présence-absence. Ainsi pour ce type de données, notre choix se portera sur\ :

- Bray-Curtis si l'on souhaite que le résultat soit dominé par les espèces les plus abondantes.

- Canberra si notre souhait est de donner la même importance à toutes les espèces, mais avec un risque de domination des espèces rares. 

- Bray-Curtis sur données transformées ($log(x+1)$ ou double racine carrée) pour un compromis entre les deux avec prise en compte de toutes les espèces, mais domination partielle des espèces les plus abondantes. C'est souvent un bon compromis.

\BeginKnitrBlock{warning}<div class="warning">
Attention\ : Si les volumes échantillonnés entre stations ne sont pas comparables, il faut standardiser (moyenne nulle et écart type un) les données selon les échantillons avant de faire les calculs de distances.
</div>\EndKnitrBlock{warning}

De même que pour Bray-Curtis, l'indice de similarité *sim* se calcule à partir de l'indice de dissimilarité *dis* tel que ci-dessus comme $sim = 1 - dis$.


#### Distance Euclidienne

Nous savons déjà que c'est la distance géométrique entre les points dans un espace à *n* dimensions\ :

$$\mathrm{D_{Euclidean}}_{j,k}=\sqrt{\sum_{i=1}^{n}(y_{ij}-y_{ik})^2}$$

Dans R, cette distance peut être calculée avec `dist(DF)` ou `vegan::vegdist(DF, method = "euclidean")`. Cet indice de dissimilarité est utile pour des mesures quantitatives, pour des données environnmentales, etc. Il faut que les mesures soient toutes effectuées dans les mêmes unités. Si ce n'est pas le cas, penser alors à standardiser les mesures avant le calcul comme nous l'avons fait plus dans l'exemple sur le zooplancton. Il n'existe pas d'indice de similarité complémentaire.


#### Distance de Manhattan

La distance de Manhattan, encore appelée "city-block distance" est un indice de dissimilarité qui, contrairement à la distance euclidienne ne mesure pas la distance géométrique entre les points en ligne droite, mais via un trajet qui suit les parallèmes aux axes. C'est comme si la distance euclidenne reliait les points à vol d'oiseau, alors qu'avec la distance de Manhattan, on devait contourner les blocs de maisons du quartier pour aller d'un point A à un point B (d'où le nom de cette métrique). Elle se calcule comme suit\ :

$$\mathrm{D_{Manhattan}}_{j,k}=\sum_{i=1}^{n}|y_{ij}-y_{ik}|$$

Dans R, nous utiliserons `vegan::vegdist(DF, method = "manhattan")`. Ici aussi, seul l'indice de dissimilarité est défini. L'indice de similarité complémentaire n'existe pas car la valeur de l'indice de dissimlarité n'est pas borné à droite et peut varier de zéro (dissimilarité nulle, les deux individus soint identiques) à l'infini pour une différence maximale.


### Utilisation des indices

- Les distances euclidienne ou de Manhattan sont à préférer pour les **mesures environnementales** ou de manière générale pour les variables quantitatives continues.

- Les distances de Bray-Curtis ou Canberra sont meilleure pour les **dénombrements d’espèces** (nombreux double zéro), ou de manière générale, pour les variables quantitatives discrètes prenant des valeurs nulles ou positives.


### Propriétés des indices

Les indices varient en 0 et 1 (0 et 100%), mais les distances sont utilisées aussi comme indices de dissimilarité et varient entre 0 et $+\infty$.

Un indice est dit **métrique** si\ :

- **Minimum 0** : $I_{j, k} = 0$ si $j = k$

- **Positif** : $I_{j, k}>0$ si $j \neq k$

- **Symétrique** : $I_{j, k}=I_{k, j}$

- **Inégalité triangulaire** : $I_{j, k} + I_{k, l} >= I_{j, l}$

La dernière propriété d'inégalité triangulaire est la plus difficile à obtenir, et n'est pas toujours nécessaire. Nous pouvons montrer que certains indices qui ne respectent pas cette dernière propriété sont pourtant utiles dans le contexte. Nous dirons alors d'un indice que c'est une **semi-métrique** s’il répond à toutes les conditions sauf la quatrième. Enfin, un indice est dit **non métrique** dans tous les autres cas. Le tableau suivant reprend les métriques que nous avons vu jusqu'ici, et rajoute d'autres candidats potentiels (la distance Chi carré, l'indice de correlation ou de variance/covariance) en indiquant leur type\ :

| **Distance**           | **Type**
|------------------------|---------------
| Bray-Curtis            | semi-métrique
| Canberra               | métrique
| Euclidienne            | métrique
| Manhattan              | métrique
| Chi carré              | métrique
| (correlation)          | (non métrique)
| (variance/covariance)  | (non métrique)


##### Pour en savoir plus {-}

- Vous pouver aussi transposer le tableau pour calculer la distance entre les variables en utilisant la fonction `t()` dans R (`dist(t(DF))`). Par exemple, dans le cas d'un tableau "espèces - station" (dénombrement d'espèces en différentes stations), nous pouvons souhaiter comparer les stations du point de vue de la composition en espèces, mais nous pouvons aussi comparer les espèces du point de vue de leur répartition entre les stations. Pour passer d'un calcul à l'autre, nous transposerons donc le tableau (les colonnes deviennent les lignes et inversément) avant d'utiliser `dist()` ou `vegan::vegdist()`.

- Pour bien comprendre la logique derrière les indices, il est utile de comprendre les équations correspondantes. Si ces équations sont pour vous trop abstraites, une façon efficace de comprendre consiste à faire le calcul à la main. Par exemple dans le cas de l'indice de Canberra, la notion de nombre de données non nulles $nz$ n'est pas évident. Effectuons un calcul à la main détaillé sur le tableau fictif suivant concernant trois espèces A, B, et C dénombrées en trois stations Sta1, Sta2 et Sta3 dans le tableau nommé `ex1`\ :


         A    B    C
-----  ---  ---  ---
Sta1     4    0    2
Sta2     3    0   10
Sta3     1    8    0

Pour rappel, la dissimilarité de Canberra se calcule comme suit:

$$\mathrm{D_{Canberra}}_{j,k}=\frac{1}{nz}\sum_{i'=1}^{nz}\frac{\left|y_{i'j}-y_{i'k}\right|}{y_{i'j}+y_{i'k}}$$

où:

- _nz_ est le nombre d'observations non nulles simultanément dans les deux vecteurs comparés (les doubles zéros ne sont pas pris en compte)
- _i'_ est l'itérateur sur toutes les valeurs non double zéros

Voici le détail du calcul (notez bien comment le double zéro pour l'espèce B entre les stations 1 et 2 est pris en compte dans le calcul)\ :


```r
Sta1_2 <- (1/2) * ((abs(4 - 3)) / (4 + 3) + (abs(2 - 10)) / (2 + 10))
round(Sta1_2, 2)
```

```
# [1] 0.4
```

```r
Sta1_3 <- (1/3) * (abs(4 - 1) / (4 + 1) + abs(0 - 8) / (0 + 8) +
  abs(2 - 0) / (2 + 0))
round(Sta1_3, 2)
```

```
# [1] 0.87
```

```r
Sta2_3 <- (1/3) * (abs(3 - 1) / (3 + 1) + abs(0 - 8) / (0 + 8) +
  abs(10 - 0) / (10 + 0))
round(Sta2_3, 2)
```

```
# [1] 0.83
```

La matrice finale est la suivante\ :


```
#      Sta1 Sta2
# Sta2 0.40     
# Sta3 0.87 0.83
```

Vérifions en laissant R faire le calcul\ :


```r
vegan::vegdist(ex1, method = "canberra") %>.%
  round(., 2)
```

```
#      Sta1 Sta2
# Sta2 0.40     
# Sta3 0.87 0.83
```

Attention\ ! `dist(, method = "canberra")` ne multiplie pas la somme par $1/nz$, mais par $n/nz$ (variante). Ainsi, nous obtenons des distances trois fois plus grandes ici avec `dist()` qu'avec `vegan::vegdist()`, mais les deux calculs restent valables car les écarts relatifs au sein de la matrice de distance sont les mêmes.


```r
dist(ex1, method = "canberra") %>.%
  round(., 2)
```

```
#      Sta1 Sta2
# Sta2 1.21     
# Sta3 2.60 2.50
```


##### A vous de jouer ! {-}

- Réalisez le tutoriel afin de vérifier votre bonne compréhension des matrices de distance.

\BeginKnitrBlock{bdd}<div class="bdd">Démarrez la SciViews Box et RStudio. Dans la fenêtre **Console** de RStudio, entrez l'instruction suivante suivie de la touche `Entrée` pour ouvrir le tutoriel concernant les bases de R\ :

    BioDataScience2::run("05a_distance_matrix")

N’oubliez pas d’appuyer sur la touche `ESC` pour reprendre la main dans R à la fin d’un tutoriel dans la console R.</div>\EndKnitrBlock{bdd}

- Réalisez un carnet de note par binôme sur le transect entre Nice et Calvi. Lisez attentivement le README

\BeginKnitrBlock{bdd}<div class="bdd">
Vous avez à votre disposition une assignation GitHub Classroom.

Pour l'année académique 2019-2020, les URLs à utiliser pour accéder à votre tâche sont les suivants : 

* Cours de Sciences des données II à Mons : <https://classroom.github.com/g/R45egn-p>

* Pour les autres utilisateurs de ce livre, veuillez faire un "fork" du dépôt [spatial_distribution_zooplankton_ligurian_sea](https://github.com/BioDataScience-Course/spatial_distribution_zooplankton_ligurian_sea). Si vous souhaitez accéder à une version précédente particulière de l'exercice, sélectionnez la release correspondante à l'année que vous recherchez.  

Lisez le README afin de prendre connaissance de l'exercice.
</div>\EndKnitrBlock{bdd}


## Regroupement avec CAH

A partir du moment où nous avons une matrice de distances entre toutes les paires d'individus dans notre jeu de données, nous pouvons les regrouper en fonction de leur ressemblance. Deux approches radicalement différentes existent. Soit nous pouvons diviser l'échantillon progressivement jusqu'à obtenir des groupes homogènes (**méthodes divisives, telle que les K-moyennes** que nous aborderons dans le module suivant), soit nous pouvons regrouper les items semblables petit à petit jusqu'à avoir traité tout l'échantillon (**méthodes agglomératives, telle que la classification ascendante hiérarchique** étudiée ici).


### Dendrogramme

Le **dendrogramme** est une manière très utile de représenter graphiquement un regroupement. Il s'agit de réaliser un arbre dichotomique ressemblant un peu à un arbre phylogénétique qui est familier aux biologistes (par exemple [ici](https://evolution.frama.wiki/la_theorie_de_l_evolution_ne_fait_pas_de_predictions))\ :

![](images/sdd2_05/arbre_ornithorynque1.png)

Dans l'arbre, nous représentons des divisions dichotomiques (division d'une branche en deux) matérialisant les divergences au cours de l'évolution. Dans l'arbre présenté ici, l'axe des ordonnées est utilisé pour représenter le temps et les branchements sont placées à une hauteur correspondante en face de l'axe. Le dendrogramme est une représentation similaire de la CAH avec l'axe des ordonnées indiquant *à quelle distance* le rassemblement se fait.

Dans R, la réalisation du dendrogramme se fait en deux étapes\ :

1) Son calcul par CAH à l'aide de la fonction `hclust()`

2) Sa représentation sur un graphique en utilisant `plot()` (pour un graphique de base) ou `ggdendro::ggdendrogram()` (pour un graphique ggplot2).

Partons, pour étayer notre raisonnement, d'une matrice de distances euclidiennes sur les données de zooplancton. Au passage, abordons quelques fonctions de R utiles dans le contexte pour préparer correctement nos données.

- A la section précédente, nous avons suggéré qu'il peut être utile de *standardiser* nos données préalablement si une distance de type euclidienne ou manhattan est ensuite calculée et si les données numériques sont mesurées dans des unités différentes, comme c'est le cas ici. La fonction `scale()` se charge de cette standardisation colonne par colonne dans un tableau. Comme elle renvoie une matrice, nous devons ensuite retransformer le résultat en `data.frame` ou `tibble`. Nous choisissons ici d'utiliser la fonction `as_tibble()`.
- Limitons, pour l'instant notre ambition à la comparaison de six individus. Afin d'observer tous les cas possibles dans le dendrogramme, nous ne prendrons pas les six premières lignes du tableau, mais les lignes 13 à 18. Cela peut se faire à l'aide de la fonction `slice()` que nous n'avons pas encore beaucoup utilisée jusqu'ici. Cette fonction permet de spécifier explicitement les numéros de lignes à conserver, contrairement à `filter()` qui applique un test de condition pour décider qulles ligne(s) converser.

Voici donc notre matrice de distances euclidiennes sur les données ainsi traitées. Les individus initiaux 13 à 18 sont renumérotés 1 à 6. Nous n'imprimons plus ici la matrice de distance obtenue car ce n'est que la première étape du travail vers une représentation plus utile (le dendrogramme).


```r
zoo %>.%
  select(., -class) %>.% # Elimination de la colonne class
  scale(.) %>.% # Standardisation des 19 colonnes
  as_tibble(.) %>.% # Conversion de la matrice en data.frame +tibble
  slice(., 13:18) -> zoo6      # Récupération des lignes 13 à 18

zoo6 %>.%
  vegan::vegdist(., method = "euclidean") -> zoo6std_dist
```

Notre objet `zoo6std_dist` est ensuite utilisé pour calculer notre CAH à l'aide de `hclust()`. Enfin, `plot()` (ou `ggdendro::ggdendrogram()`) se charge de tracer le dendrogramme.


```r
zoo6std_dist %>.%
  hclust(.) -> zoo6std_clust # Calcul du dendrogramme
  plot(zoo6std_clust) # ggdendro::ggdendrogram() peut aussi être utilisé ici
```

<img src="05-classi-hierarchique_files/figure-html/unnamed-chunk-16-1.png" width="672" style="display: block; margin: auto;" />

Voici comment interpréter ce graphique. Les deux individus les plus semblables sont le 2 et le 5 (regroupepent effectué le plus bas, donc, avec la valeur de l'indice de dissimilarité le plus faible), également tous deux similaires au 3. Le 4 se rattache à ce groupe, mais bien plus haut sur l'axe, indiquant ainsi qu'il s'en différencie un peu plus, enfin, le 1 et le 6 sont rassemblés à un niveau à peu près équivalent. Finalement, le groupe de droite constitué des individus 2, 3, 4 et 5 et celui de gauche contenant le 1 et le 6 sont reliés encore plus haut suggérant ainsi la dissimilitude la plus forte entre ces deux groupes.

\BeginKnitrBlock{warning}<div class="warning">
Attention\ : La position des individus selon l'axe horizontal n'est pas importante. Il faut voir le dendrogramme comme un mobile qui peut tourner librement. C'est-à-dire que le groupe constitué de 2, 3, 4 et 5 aurait très bien pu être placé à la gauche de celui constitué de 1 et 6. De même, le sous-groupe 2, 3 et 5 aurait très bien pu être à la gauche du regroupement avec 4 à la droite, etc.
</div>\EndKnitrBlock{warning}

Notez que nous n'avons *pas* utilisé l'information concernant les classes taxonomiques auxquelles les individus appartiennent (nous avons éliminé la variable `class` en tout début d'analyse). Pour cette raison, ce type de regroupement s'appelle une **classification non supervisée** parce que nous n'imposons pas les groupes que nous souhaitons réaliser^[Les techniques complémentaires de classification supervisées, recommandées dans le cas du jeu de données zooplancton seront abordées dans le cours de Science des Données Biologiques III l'an prochain.]. Nous pouvons néanmoins révéler ces classes maintenant pour vérifier si le dendrogramme réalisé est logique (étape bien sûr facultative et souvent pas possible en pratique lorsque cette information *n'est pas* disponible)\ :


```r
zoo$class[13:18]
```

```
# [1] Egg_round        Poecilostomatoid Poecilostomatoid Decapod         
# [5] Calanoid         Appendicularian 
# 17 Levels: Annelid Appendicularian Calanoid Chaetognath ... Protist
```

Les individus 2, 3 et 5 ("Poecilostomatoid" ou "Calanoid") sont des [copépodes](http://www.obs-vlfr.fr/~gaspari/copepodes/cop_doc.htm), des crustacés particulièrement abondants dans le zooplancton. Leur forme est similaire. Leur regroupement est logique. L'individu 4 est une [larve de décapode](https://news.ecoledelamer.fr/plancton/observation/fiche/zoe-1), un autre crustacé. Ainsi le regroupement de 2, 3 et 5 avec 4 correspond aux crustacés ici. Enfin, les individus 1 et 6 sont très différents puisqu'il s'agit respectivement d'un [œuf](http://grainesdexplorateurs.ens-lyon.fr/archives-1/2012-2013/projets/tara-2013/pouldreuzic-2013/plancton-de-beg-meil/copy_of_embryon-de-poisson/view) rond probablement de poisson et d'un [appendiculaire](https://scripps.ucsd.edu/zooplanktonguide/species/oikopleura-dioica).


### Séparer les groupes

Si notre dendrogramme est satisfaisant (nous le déterminons en l'étudiant et en vérifiant que le regroupement obtenu a un sens biologique  par rapport à l'objectif de notre étude), nous concrétisons le regroupement en coupant l'arbre à une certaine hauteur à l'aide de la fonction `cutree()`. Nous pouvons matérialiser ce niveau de coupure en traçant un trait horizontal rouge avec `abline(h = XXX, col = "red")` pour le graphe de base, ou en ajoutant `+ geom_hline(yintercept = XXX, col = "red")` au graphique ggplot2, avec 'XXX' la hauteur de coupure souhaitée. Par exemple, si nous voulons réaliser deux groupes, nous ferons\ :


```r
plot(zoo6std_clust)
abline(h = 8, col = "red")
```

<img src="05-classi-hierarchique_files/figure-html/unnamed-chunk-19-1.png" width="672" style="display: block; margin: auto;" />

Pour réaliser trois groupes, nous couperons plus bas (ici version ggplot2)\ :


```r
ggdendro::ggdendrogram(zoo6std_clust) +
  geom_hline(yintercept = 6.5, col = "red")
```

<img src="05-classi-hierarchique_files/figure-html/unnamed-chunk-20-1.png" width="672" style="display: block; margin: auto;" />

Et ainsi de suite. A mesure que la hauteur de coupure descend, nous réaliserons quatre, puis cinq groupes. Quel est le meilleur niveau de coupure\ ? Il n'y en a pas un seul forcément car les différents niveaux correspondent à un point de vue de plus en plus détaillé du regroupement. Néanmoins, lorsque le saut sur l'axe d'un regroupement à l'autre est fort, nous pouvons considérer une séparation des groupes d'autant meilleure. Cela crédibilise d'autant le regroupement choisi. Ainsi la séparation en deux groupes apparait forte (entre 10,2 et 7,4, sur un intervalle de 2,8 unités donc). De même, la séparation de l'individu 4 par rapport au groupe constitué de 2, 3 et 5 se fait sur un intervalle de 4,7 unités (entre 7,4 et 2,7). En comparaison, la séparation de l'individu 3 du groupe 2 et 5 est nettement moins nette puisqu'elle apparait aux hauteurs entre 2,7 à 2.4, soit seulement sur un intervalle de 0,3 unités. Ces mesures d'intervalles n'ont aucune valeur absolue. Il faut les considérer uniquement de manière relatives les unes par rapport aux autres, et seulement au sein d'un même dendrogramme.

Ici, deux niveaux de coupure se détachent. Nous utilisons aussi la fonction `rect.hclust()` pour matérialiser ces groupes sur le dendrogramme, et `cutree()` pour obtenir une nouvelle variable que nous pourrons ajouter dans notre tableau de données qui concrétise le regroupement choisi\ :

- en deux groupes, nous avons les crustacés séparés des autres.


```r
plot(zoo6std_clust)
abline(h = 8, col = "red")
rect.hclust(zoo6std_clust, h = 8, border = c("blue", "red"))
```

<img src="05-classi-hierarchique_files/figure-html/unnamed-chunk-21-1.png" width="672" style="display: block; margin: auto;" />


```r
(group2 <- cutree(zoo6std_clust, h = 8))
```

```
# [1] 1 2 2 2 2 1
```

Les individus 1 et 6 sont dans le groupe 1, et les autres dans le groupe 2.

- en quatre groupes, nous avons les copépodes séparés des trois autres items chacun dans un groupe séparé.


```r
plot(zoo6std_clust, hang = -1) # Prolonger les tiges jusqu'en bas
abline(h = 5, col = "red")
rect.hclust(zoo6std_clust, h = 5, border = c("blue", "blue", "blue", "red"))
```

<img src="05-classi-hierarchique_files/figure-html/unnamed-chunk-23-1.png" width="672" style="display: block; margin: auto;" />

Le groupe des individus 2, 3 et 5 est encadré en rouge. Les trois autres groupes des individus uniques 1, 6 et 4 respectivement sont ici encadrés en bleu.


```r
(group4 <- cutree(zoo6std_clust, h = 5))
```

```
# [1] 1 2 2 3 2 4
```

L'individu 1 est dans le groupe 1, les individus 2, 3 et 5 sont dans le groupe 2, l'individu 4 est dans le groupe 3 et enfin l'individu 6 est dans le groupe 4. Donc, `cutree()` numérote ses groupes en fonction du premier item présent dedans dans l'ordre du tableau de données.

Nous pouvons rajouter ces regroupements dans le tableau de données, et utiliser cette information pour en faire d'autres choses utiles. Par exemple, nous représentons un nuage de point des données et colorons nos points en fonction du premier regroupement effectué comme suit\ :


```r
zoo6$group2 <- as.factor(group2) # Utiliser une variable facteur ici
chart(zoo6, area ~ circularity %col=% group2) +
  geom_point()
```

<img src="05-classi-hierarchique_files/figure-html/unnamed-chunk-25-1.png" width="672" style="display: block; margin: auto;" />

Nous voyons que les copépodes (groupe 2 en turquoise) sont plus grands (`area`) et moins circulaires (`circularity`) que les deux autres particules du groupe 1 en rouge. Nous pouvons ainsi réexplorer nos données en fonction du regroupement pour mieux le comprendre.


#### Méthodes de CAH

Tant que l'on compare des individus isolés entre eux, il n'y a pas d'ambiguïté. Par contre, dès que nous comparons un groupe avec un individu isolé, ou deux groupes entre eux, nous avons plusieurs stratégies possible pour calculer leurs distances (argument `method =` de `hclust()`)\ :

- **Liens simples** (single linkages en anglais) `hclust(DIST, method = "single")`\ : la distance entre les plus proches voisins au sein des groupes est utilisée.

![](images/sdd2_05/link_single.png)


```r
zoo6std_dist %>.%
  hclust(., method = "single") %>.%
  plot(.)
```

<img src="05-classi-hierarchique_files/figure-html/unnamed-chunk-26-1.png" width="672" style="display: block; margin: auto;" />

Les données tendent à être aggrégées de proche en proche en montrant une gradation d'une extrême à l'autre. Dans le cas d'une variation progressive, par exemple lors d'un passage graduel d'une situation A vers une situation B le long d'un transect, le dendrogramme obtenu à l'aide de cette méthode représentera la situation au mieux.

- **Liens complets** (complete linkages) `hclust(DIST, method = "complete")`, méthode utilisée par défault si non précisée\ : la distance entre les plus lointains voisins est considérée. C'est le dendrogramme que nous avons obtenu au début.

![](images/sdd2_05/link_complete.png)

Le dendrogramme a tendance à effectuer des groupes séparés plus nettement les uns des autres qu'avec les liens simples.

- **Liens moyens** (group-average) `hclust(DIST, method = "average")` encore appelée méthode UPGMA\ : moyenne des liens entre toutes les paires possibles intergroupes.

![](images/sdd2_05/link_average.png)

Nous obtenons une situation intermédiaire entre liens simples et liens moyens.


```r
zoo6std_dist %>.%
  hclust(., method = "average") %>.%
  plot(.)
```

<img src="05-classi-hierarchique_files/figure-html/unnamed-chunk-27-1.png" width="672" style="display: block; margin: auto;" />

Dans ce cas-ci, le résultat est similaire aux liens simples, mais ce n'est pas forcément le cas à chaque fois.

- **Méthode de Ward** `hclust(DIST, method = "ward.D2")`. Considérant le partitionnement de la variance totale du nuage de points (on parle aussi de l'*inertie* du nuage de points) entre variance interclasse et variance intraclasse, la méthode vise à maximiser la variance interclasse et minimiser la variance intraclasse, ce qui revient d'ailleurs au même. Cette technique fonctionne souvent très bien pour obtenir des groupes bien individualisés.


```r
zoo6std_dist %>.%
  hclust(., method = "ward.D2") %>.%
  plot(.)
```

<img src="05-classi-hierarchique_files/figure-html/unnamed-chunk-28-1.png" width="672" style="display: block; margin: auto;" />

Ici nous obtenos un dendrogramme très proche des liens complets. Encore une fois, ce n'est pas forcément le cas dans toutes les situations.

- Liens **médians**, **centroïdes**, ..., constituent encore d'autre méthodes possibles, mais moins utilisées. Elles ont l'inconvénient de produire parfois des *inversions* dans le dendrogramme, c'est-à-dire qu'un regroupement plus avant se fait parfois à une hauteur plus basse sur l'axe des ordonnées, ce qui rend le dendrogramme peu lisible et beaucoup moins esthétique. Dans notre exemple, la méthode centroïde crée une telle inversion à la hauteur de 5 environ sur l'axe des ordonnées entre l'individu 6 et l'individu 1. Alors, qui est regroupé en premier\ ? Pas facile à déterminer dans ce cas\ !


```r
zoo6std_dist %>.%
  hclust(., method = "centroid") %>.%
  plot(.)
```

<img src="05-classi-hierarchique_files/figure-html/unnamed-chunk-29-1.png" width="672" style="display: block; margin: auto;" />


### Étude complète

Voici ce que cela donne si nous effectuons une CAH sur le jeu zooplancton complet avec ses 1262 lignes.



```r
zoo %>.%
  select(., -class) %>.% # Elimination de la colonne class
  scale(.) %>.% # Standardisation des 19 colonnes
  as_tibble(.) %>.% # Transformation en data.frame + tibble
  vegan::vegdist(., method = "euclidean") %>.% # Matrice de distances
  hclust(., method = "ward.D2") -> zoo_clust # CAH avec Ward
  
  plot(zoo_clust, labels = FALSE) # Dendrogramme sans labels des individus
  abline(h = 70, col = "red") # Séparation en 3 groupes
```

<img src="05-classi-hierarchique_files/figure-html/unnamed-chunk-30-1.png" width="672" style="display: block; margin: auto;" />

Naturellement, avec 1262 branches, notre dendrogramme est très encombré\ ! Cependant, il reste analysable tant que nous nous intéressons aux regroupements de plus haut niveau (vers le haut du dendrogramme). Notez comme les vlaeurs sur l'axe des ordonnées ont changé par rapport à nos cas simple à six items (ne jamais comparer les hauteurs *entre* dendrogrammes différents).

Notre CAH est terminée, mais nous pouvons matérialiser le regroupement sous forme d'une variable supplémentaire dans le tableau et l'utiliser ensuite.


```r
  zoo_clust %>.%
    cutree(., h = 70) %>.% # Matérialisation des groupes
    as.factor(.) -> # Conversion en variable facteur
    zoo$Groupes # Ajout au tableau comme variable Groupes
  
  chart(zoo, compactness ~ ecd %col=% Groupes) +
    geom_point() + # Exploration visuelle des groupes (exemple)
    coord_trans(x = "log10", y = "log10") # Axes en log10
```

<img src="05-classi-hierarchique_files/figure-html/unnamed-chunk-31-1.png" width="672" style="display: block; margin: auto;" />

Et de manière optionnelle, si un classement est disponible par ailleurs (c'est le cas ici avec la variable `class`), nous pouvons réaliser un tableau de contingence à double entrées entre le regroupement obtenu et le classement pour comparaison.


```r
table(zoo$class, zoo$Groupes)
```

```
#                   
#                      1   2   3
#   Annelid           31  13   6
#   Appendicularian    0  36   0
#   Calanoid           9 237  42
#   Chaetognath        0  17  34
#   Cirriped           1  21   0
#   Cladoceran        47   3   0
#   Cnidarian          3   5  14
#   Cyclopoid          0  50   0
#   Decapod          121   5   0
#   Egg_elongated      2  48   0
#   Egg_round         44   0   5
#   Fish               4  46   0
#   Gastropod         48   2   0
#   Harpacticoid       0  39   0
#   Malacostracan     27  66  28
#   Poecilostomatoid  20 136   2
#   Protist            0  50   0
```

Ainsi, nous pouvons constater que le groupe 1 contient une fraction importante des annélides, des cladocères,des décapodes, des œufs ronds et des gastéropodes. Le groupe 2 contient un maximum des copépodes représentés par les calanoïdes, les cyclopoïdes, les harpacticoïdes et les poecilostomatoïdes. Il contient aussi tous les appendiculaires, tous les protistes, presque tous les œufs allongés, les poissons, et une majorité des malacostracés. Enfin, le groupe 3 contient une majorité des chaetognathes et des cnidaires. Le regroupement a été réalisé uniquement en fonction de mesures effectuées sur les images. Il n'est pas parfait, mais des tendances se dégagent tout de même.


##### Pour en savoir plus {-}

- La vidéo suivante présente la matière que nous venons d'étudier de manière légèrement différente. Plus d'explications sont également apportées concernant la méthode de Ward.

<!--html_preserve--><iframe src="https://www.youtube.com/embed/SE_4dLh5vXY?start=192&amp;end=1004" width="770" height="433" frameborder="0" allowfullscreen=""></iframe><!--/html_preserve-->

- Une autre explication encore [ici](http://larmarange.github.io/analyse-R/classification-ascendante-hierarchique.html).

- Pour bien comprendre la façon dont une CAH est réalisée, il est utile de détailler le calcul étape par étape sur un exemple simple. Voici une matrice de distances euclidiennes fictive entre 6 stations \ :


     A       B       C      D      E    
---  ------  ------  -----  -----  -----
B    15                                 
C    6.4     10.86                      
D    5.2     13.04   5.48               
E    5.1     12.37   7.28   7.81        
F    10.39   7.42    5.57   9.64   9.49 

Effectuons une CAH manuellement par liens complets et traçons le dendrogramme correspondant.

**Etape 1\ :** Nous repérons dans la matrice de distance la paire qui a l'indice de dissimilarité le plus petit et effectuons un premier regroupement. A_E = groupe I à la distance 5,1. La matrice de distances est simplifiées par rapport à ce groupe I en considérant la règle utilisée (ici, liens complets, donc on garde la plus grande distance entre toutes les paires possibles lorsqu'il y a des groupes).

    * Distance entre B et I = B_A = 15
    * Distance entre C et I = C_E = 7,28
    * Distance entre D et I = D_E = 7,81
    * Distance entre F et I = F_A = 10,39

Matrice de distance recalculée\ :


     I       B       C      D    
---  ------  ------  -----  -----
B    15.00                       
C    7.28    10.86               
D    7.81    13.04   5.48        
F    10.39   7.42    5.57   9.64 

**Etape 2\ :** on répète le processus. C_D = groupe II à la distance 5,48.

    * Distance entre I et II = I_D = 7,81
    * Distance entre B et II = B_D = 13,04
    * Distance entre F et II = F_D = 9,64

Matrice de distance recalculée\ :


     I       B       II   
---  ------  ------  -----
B    15.00                
II   7.81    13.04        
F    10.39   7.42    9.64 

**Etape 3\ :** B_F = groupe III à la distance 7,42

    * Distance entre I et III = I_B = 15
    * Distance entre II et III = II_B = 13,04

Matrice de distance recalculée \ :


      I       III   
----  ------  ------
III   15.00         
II    7.81    13.04 

**Etape 4\ :** I_II = groupe IV à la distance 7,81

     * Distance entre III et IV = III_I = 15

Matrice de distance recalculée\ :


      III
---  ----
IV     15

**Etape 5\ :** III - IV = groupe V à la distance 15.

**fini\ !**

Voici le dendrogramme résultant\ :

<img src="05-classi-hierarchique_files/figure-html/unnamed-chunk-39-1.png" width="672" style="display: block; margin: auto;" />
