# K-moyenne, MDS & SOM {#k-moyenne-mds-som}




##### Objectifs {-}

- Maîtriser la technique de classification par les k-moyennes comme alternative à la CAH pour les gros jeux de données.
- Comprendre la représentation d'une matrice de distances sur un carte (ordination) et la réduction de dimensions via le positionnement multidimensionnel MDS.
- Être capable de créer des cartes auto-adaptatives ou SOM, de les interpréter et de les utiliser comme autre technique de classification.

##### Prérequis {-}

Ces techniques étant basées sur des matrices de distances et complémentaires à la classification ascendante hiérarchique, le module \@ref(hierarchique) doit être assimilé avant de s'attaquer au présent module.


## K-moyennes

Les k-moyennes (ou "k-means" en anglais) représentent une autre façon de regrouper les individus d'un tableau multivarié. Par rapport à la CAH, cette technique est généralement moins efficace, mais elle a l'avantage de permettre le regroupement d'un très grand nombre d'individus (gros jeu de données), là où la CAH nécessiterait trop de temps de calcul et de mémoire vive. Il est donc utile de connaitre cette seconde technique à utiliser comme solution de secours lorsque le dendrogramme de la CAH devient illisible sur de très gros jeux de données.

Le principe des k-moyennes est très simple^[En pratique, différents algorithmes avec diverses optimisations existent. Le plus récent et le plus sophistiqué est celui de Hartigan-Wong. Il est utilisé par défaut par la fonction `kmeans()`. En pratique, il y a peu de raison d'en changer.]\ :

- L'utilisateur choisi le nombre de groupes *k* qu'il veut obtenir à l'avance.
- La position des *k* centres est choisie au hasard au début.
- Les individus sont attribués aux *k* groupes en fonction de leurs distances aux centres (attribution au groupe de centre le plus proche).
- Les *k* centres sont replacés au centre de gravité des groupes ainsi obtenus.
- Les individus sont réaffectés en fonction de leurs distances à ces nouveaux centres.
- Si au moins un individu a changé de groupe, le calcul est réitéré. Sinon, nous considérons avoir atteint la configuration finale.

La technique est superbement expliquée et illustrée dans la vidéo suivante\ :

<!--html_preserve--><iframe src="https://www.youtube.com/embed/Aic2gHm9lt0" width="770" height="433" frameborder="0" allowfullscreen=""></iframe><!--/html_preserve-->

Essayez par vous même via l'application ci-dessous qui utilise le célèbre jeu de données `iris`. Notez que vous devez utiliser des variables **numériques**. Par exemple, `Species` étant une variable qualitative, vous verrez que cela ne fonctionne pas dans ce cas.

<iframe src="https://jjallaire.shinyapps.io/shiny-kmeans/?showcase=0" width="100%" height="600px" style="border: none;"></iframe>

### Exemple simple

Afin de comparer la classification par k-moyennes à celle par CAH, nous reprendrons ici le même jeu de données `zooplankton`.


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

Commençons par l'exemple simplissime de la réalisation de deux groupes à partir de six individus issus de ce jeu de données, comme nous l'avons fait avec la CAH\ :





```r
zoo %>.%
  select(., -class) %>.% # Elimination de la colonne class
  slice(., 13:18) -> zoo6      # Récupération des lignes 13 à 18

zoo6_kmeans <- kmeans(zoo6, centers = 2)
zoo6_kmeans
```

```
# K-means clustering with 2 clusters of sizes 3, 3
# 
# Cluster means:
#         ecd      area perimeter    feret    major     minor      mean
# 1 1.1926500 1.1279667 10.346667 2.201133 1.677067 0.8596333 0.3217333
# 2 0.6292647 0.3188667  3.224133 1.159200 1.096433 0.4023333 0.1871667
#        mode        min       max   std_dev     range      size    aspect
# 1 0.3533333 0.00400000 0.8986667 0.2620000 0.8946667 1.2683500 0.5149422
# 2 0.1026667 0.01066667 0.5400000 0.1166667 0.5293333 0.7493833 0.4753843
#   elongation compactness transparency circularity    density
# 1  23.046713    7.987806   0.06173831   0.1357000 0.37630000
# 2   6.333315    2.727708   0.14732060   0.4900333 0.06943333
# 
# Clustering vector:
# [1] 2 1 2 1 1 2
# 
# Within cluster sum of squares by cluster:
# [1] 200.03837  54.18647
#  (between_SS / total_SS =  68.1 %)
# 
# Available components:
# 
# [1] "cluster"      "centers"      "totss"        "withinss"    
# [5] "tot.withinss" "betweenss"    "size"         "iter"        
# [9] "ifault"
```

Nous voyons que la fonction `kmeans()` effectue notre classification. Nous lui fournissons le tableau de départ et spécifions le nombre *k* de groupes souhaités via l'argument `centers =`. Ne pas oublier d'assigner le résultat du calcul à une nouvelle variable, ici `zoo6_kmeans`, pour pouvoir l'inspecter et l'utiliser par la suite. L'impression du contenu de l'objet nous donne plein d'information dont\ :

- le nombre d'individus dans chaque groupe (ici 3 et 3),
- la position des centres pour les *k* groupes dans `Cluster means`,
- l'appartenance aux groupes dans `Cluster vectors` (dans le même ordre que les lignes du tableau de départ),
- la sommes des carrés des distances entre les individus et la moyenne au sein de chaque groupe dans `Within cluster sum of squares`\ ; le calcul `between_SS / total_SS` est à mettre en parallèle avec le $R^2$ de la régression linéaire\ : c'est une mesure de la qualité de regroupement des données (plus la valeur est proche de 100% mieux c'est, mais attention que cette valeur augmente d'office en même temps que *k*),
- et enfin, la liste des composants accessibles via l'opérateur `$`\ ; par exemple, pour obtenir les groupes (opération similaire à `cutree()` pour la CAH), nous ferons\ :


```r
zoo6_kmeans$cluster
```

```
# [1] 2 1 2 1 1 2
```

Le package `broom` contient trois fonctions complémentaires qui nous seront utiles\ : `tidy()`, `augment()` et `glance()`. `broom::glance()` retourne un `data.frame` avec les statistiques permettant d'évaluer la qualité de la classification obtenue\ :


```r
broom::glance(zoo6_kmeans)
```

```
# # A tibble: 1 x 4
#   totss tot.withinss betweenss  iter
#   <dbl>        <dbl>     <dbl> <int>
# 1  796.         254.      542.     1
```

De plus, le package `factoextra` propose une fonction `fviz_nbclust()` qui réalise un graphique pour aider au choix optimal de *k*\ :


```r
factoextra::fviz_nbclust(zoo6, kmeans, method = "wss", k.max = 5)
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-8-1.png" width="672" style="display: block; margin: auto;" />

Le graphique obtenu montre la décroissance de la somme des carrés des distances intra-groupes en fonction de *k*. Avec *k* = 1, nous considérons toutes les données dans leur ensemble et nous avons simplement la somme des carrés des distances euclidiennes entre tous les individus et le centre de gravité du nuage de points dont les coordonnées sont les moyennes de chaque variable. C'est le point de départ qui nous indique de combien les données sont dispersées (la valeur absolue de ce nombre n'est pas importante).

Ensuite, avec *k* croissant, notre objectif est de faire des regroupement qui diminuent la variance intra-groupe autant que possible, ce que nous notons par la diminution de la somme des carrés intra-groupes (la variance du groupe est, en effet, la somme des carrés des distances enclidiennes entre les points et le centre du groupe, divisée par les degrés de liberté).

Nous recherchons ici des sauts importants dans la décroissance de la somme des carrés, tout comme dans le dendrogramme obtenu par la CAH nous recherchions des sauts importants dans les regroupements (hauteur des barres verticales du dendrogramme). Nous observons ici un saut important pour *k* = 2, puis une diminution moins forte de *k* = 3 à *k* = 5. Ceci *suggère* que nous pourrions considérer deux groupes.

\BeginKnitrBlock{note}<div class="note">Le nombre de groupes proposé par `factoextra::fviz_nbclust()` n'est qu'indicatif\ ! Si vous avez par ailleurs d'autres informations qui vous suggèrent un regroupement différent, ou si vous voulez essayer un regroupement plus ou moins détaillé par rapport à ce qui est proposé, c'est tout aussi correct.

La fonction `factoextra::fviz_nbclust()` propose d'ailleurs deux autres méthodes pour déterminer le nombre optimal de groupes *k*, avec `method = "silhouette"` ou `method = "gap_stat"`. Voyez l'aide en ligne de cette fonction `?factoextra::fviz_nbclust`. Ces différentes méthodes peuvent d'ailleurs suggérer des regroupements différents pour les mêmes données... preuve qu'il n'y a pas *une et une seule* solution optimale\ !</div>\EndKnitrBlock{note}

A ce stade, nous pouvons collecter les groupes et les ajouter à notre tableau de données. Pour la CAH, vous avez déjà remarqué que rajouter ces groupes dans le *tableau de départ* peut mener à des effets surprenants si nous relançons ensuite l'analyse sur le tableau ainsi complété^[Nous vous avons proposé exprès de rajouter les groupes dans le tableau de départ pour que vous soyez confronté à ce problème. Ici, nous proposons donc une autre façon de travailler qui l'évite en assignant le résultat dans une *autre* variable.]. Donc, nous prendrons soin de placer les données ainsi complétées de la colonne `cluster` dans un tableau différent nommé `zoo6b`. Pour se faire, nous pouvons utiliser `broom::augment()`.


```r
broom::augment(zoo6_kmeans, zoo6) %>.%
  rename(., cluster = .cluster) -> zoo6b
names(zoo6b)
```

```
#  [1] "ecd"          "area"         "perimeter"    "feret"       
#  [5] "major"        "minor"        "mean"         "mode"        
#  [9] "min"          "max"          "std_dev"      "range"       
# [13] "size"         "aspect"       "elongation"   "compactness" 
# [17] "transparency" "circularity"  "density"      "cluster"
```

Comme vous pouvez le constater, une nouvelle colonne nommée `.cluster` a été ajoutée au tableau en dernière position, que nous avons renommée immédiatement en `cluster` ensuite (c'est important pour le graphique plus loin). Elle contient ceci\ :


```r
zoo6b$cluster
```

```
# [1] 2 1 2 1 1 2
# Levels: 1 2
```

C'est le contenu de `zoo6_kmeans$cluster`, mais transformé en variable `factor`.


```r
class(zoo6b$cluster)
```

```
# [1] "factor"
```

Nous pouvons enfin utiliser `broom::tidy()` pour obtenir un tableau avec les coordonnées des *k* centres. Nous l'enregistrerons dans la variable `zoo6_centers`, en ayant bien pris soin de nommer les variables du même nom que dans le tableau original `zoo6` (argument `col.names = names(zoo6)`, cela sera important pour le graphique ci-dessous)\ :


```r
zoo6_centers <- broom::tidy(zoo6_kmeans, col.names = names(zoo6))
zoo6_centers
```

```
# # A tibble: 2 x 21
#     ecd  area perimeter feret major minor  mean  mode    min   max std_dev
#   <dbl> <dbl>     <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>  <dbl> <dbl>   <dbl>
# 1 1.19  1.13      10.3   2.20  1.68 0.860 0.322 0.353 0.004  0.899   0.262
# 2 0.629 0.319      3.22  1.16  1.10 0.402 0.187 0.103 0.0107 0.540   0.117
# # … with 10 more variables: range <dbl>, size <int>, aspect <dbl>,
# #   elongation <dbl>, compactness <dbl>, transparency <dbl>,
# #   circularity <dbl>, density <dbl>, withinss <dbl>, cluster <fct>
```

La dernière colonne de ce tableau est également nommée `cluster`. C'est le lien entre le tableau `zoo6b` augmenté et `zoo6_centers`. Nous avons maintenant tout ce qu'il faut pour représenter graphiquement les regroupements effectués par les k-moyennes en colorant les points en fonction de la nouvelle variable `cluster`.


```r
chart(data = zoo6b, area ~ circularity %col=% cluster) +
  geom_point() + # Affiche les points représentant les individus
  geom_point(data = zoo6_centers, size = 5, shape = 17) # Ajoute les centres
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-14-1.png" width="672" style="display: block; margin: auto;" />

Comparez avec le graphique équivalent au module précédent consacré à la CAH. Outre que l'ordre des groupes est inversé et que les données n'ont pas été standardisées ici, un point est classé dans un groupe différent par les deux méthodes. Il s'agit du point ayant environ 0.25 de circularité et 0.5 de surface. Comme nous connaissons par ailleurs la classe à laquelle appartient chaque individu, nous pouvons la récupérer comme colonne supplémentaire du tableau `zoo6b` et ajouter cette information sur notre graphique.


```r
zoo6b$class <- zoo$class[13:18]
zoo6_centers$class <- "" # Ceci est nécessaire pour éviter le label des centres
chart(data = zoo6b, area ~ circularity %col=% cluster %label=% class) +
  geom_point() +
  ggrepel::geom_text_repel() + # Ajoute les labels intelligemment
  geom_point(data = zoo6_centers, size = 5, shape = 17)
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-15-1.png" width="672" style="display: block; margin: auto;" />

Nous constatons que le point classé différemment est un "Poecilostomatoïd". Or, l'autre groupe des k-moyennes contient aussi un individu de la même classe. Donc, CAH a mieux classé notre plancton que les k-moyennes dans le cas présent. Ce n'est pas forcément toujours le cas, mais souvent.

Un dernier point est important à mentionner. Comme les k-moyennes partent d'une position aléatoire des *k* centres, le résultat final peut varier et n'est pas forcément optimal. Pour éviter cela, nous pouvons indiquer à `kmeans()` d'essayer différentes situations de départ via l'argument `nstart =`. Par défaut, nous prenons une seule situation aléatoire de départ `nstart = 1`, mais en indiquant une valeur plus élevée pour cet argument, il est possible d'essayer plusieurs situations de départ et ne garder que le meilleur résultat final. Cela donne une analyse plus robuste et plus reproductible... mais le calcul est naturellement plus long.





```r
kmeans(zoo6, centers = 2, nstart = 50) # 50 positions de départ différentes
```

```
# K-means clustering with 2 clusters of sizes 3, 3
# 
# Cluster means:
#         ecd      area perimeter    feret    major     minor      mean
# 1 0.6292647 0.3188667  3.224133 1.159200 1.096433 0.4023333 0.1871667
# 2 1.1926500 1.1279667 10.346667 2.201133 1.677067 0.8596333 0.3217333
#        mode        min       max   std_dev     range      size    aspect
# 1 0.1026667 0.01066667 0.5400000 0.1166667 0.5293333 0.7493833 0.4753843
# 2 0.3533333 0.00400000 0.8986667 0.2620000 0.8946667 1.2683500 0.5149422
#   elongation compactness transparency circularity    density
# 1   6.333315    2.727708   0.14732060   0.4900333 0.06943333
# 2  23.046713    7.987806   0.06173831   0.1357000 0.37630000
# 
# Clustering vector:
# [1] 1 2 1 2 2 1
# 
# Within cluster sum of squares by cluster:
# [1]  54.18647 200.03837
#  (between_SS / total_SS =  68.1 %)
# 
# Available components:
# 
# [1] "cluster"      "centers"      "totss"        "withinss"    
# [5] "tot.withinss" "betweenss"    "size"         "iter"        
# [9] "ifault"
```

Dans ce cas simple, cela ne change pas grand chose. Mais avec un plus gros jeu de données plus complexe, cela peut être important.


### Classification du zooplancton

Maintenant que nous savons utiliser `kmeans()` et les fonctions annexes, nous pouvons classer le jeu de données `zoo` tout entier.


```r
zoo %>.%
  select(., -class) %>.%
  factoextra::fviz_nbclust(., kmeans, method = "wss", k.max = 10)
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-18-1.png" width="672" style="display: block; margin: auto;" />

Nous observons un saut maximal pour *k* = 2, mais le saut pour *k* = 3 est encore conséquent. Afin de comparer avec ce que nous avons fait par CAH, nous utiliserons donc *k* = 3. Enfin, comme un facteur aléatoire intervient, qui définira au final le numéro des groupes, nous utilisons `set.seed()` pour rendre l'analyse reproductible. Pensez à donner une valeur différente à cette fonction pour chaque utilisation\ ! Et pensez aussi à éliminer les colonnes non numériques à l'aide de `select()`.


```r
set.seed(562)
zoo_kmeans <- kmeans(select(zoo, -class), centers = 3, nstart = 50)
zoo_kmeans
```

```
# K-means clustering with 3 clusters of sizes 786, 91, 385
# 
# Cluster means:
#         ecd     area perimeter    feret     major     minor      mean
# 1 0.6664955 0.431915  3.575374 1.134705 0.9744768 0.4780780 0.2388065
# 2 1.3774670 1.998097 19.653860 4.063837 2.1465758 0.9602846 0.1488495
# 3 0.9715857 1.009902  9.197299 2.668022 1.8468984 0.6194652 0.1723774
#         mode         min       max   std_dev     range      size    aspect
# 1 0.09256997 0.007094148 0.7269109 0.1842660 0.7198168 0.7262774 0.5372808
# 2 0.02470330 0.004000000 0.7013187 0.1472286 0.6973187 1.5534302 0.5362249
# 3 0.04455065 0.004207792 0.6315844 0.1512922 0.6273766 1.2331818 0.5349924
#   elongation compactness transparency circularity    density
# 1   7.184451    3.002093   0.07385014  0.42917214 0.09349338
# 2  61.837019   20.325398   0.09737903  0.05186813 0.31140879
# 3  27.898079    9.529156   0.11719954  0.11197351 0.16938468
# 
# Clustering vector:
#    [1] 1 1 1 1 1 1 2 1 2 1 1 3 1 3 1 3 3 1 2 1 1 1 1 3 3 1 1 3 1 1 1 1 1 1
#   [35] 3 1 3 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 3 1 1 3 1 1 3 1 3 3 1
#   [69] 3 1 1 1 1 2 1 1 1 1 1 3 1 1 1 1 1 2 1 1 3 1 1 3 1 2 1 1 1 1 1 1 1 1
#  [103] 1 1 3 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 3 1 1 1 1 1 1 1
#  [137] 1 1 1 1 1 3 1 1 2 3 2 3 1 1 1 1 1 3 3 3 3 3 1 2 1 1 1 1 1 1 1 3 1 1
#  [171] 1 1 1 1 1 1 1 1 1 1 1 1 1 3 1 1 1 1 1 1 1 1 3 1 1 1 1 1 1 1 1 1 1 1
#  [205] 1 3 1 1 1 1 1 1 1 1 1 3 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
#  [239] 1 1 1 1 1 3 1 1 1 1 1 1 1 3 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 3
#  [273] 3 2 1 1 3 1 3 3 1 3 1 1 2 1 1 2 3 2 3 3 1 1 1 1 3 3 2 1 3 1 3 3 1 3
#  [307] 3 3 2 1 3 3 2 1 3 2 2 1 2 3 3 3 1 1 3 1 1 3 1 1 1 1 1 3 1 2 1 1 1 3
#  [341] 1 1 1 1 1 1 3 3 3 1 3 3 1 1 1 1 1 1 3 1 1 1 1 1 1 1 1 3 1 3 1 1 1 1
#  [375] 3 1 1 1 1 1 1 3 3 1 1 1 1 1 3 3 1 1 3 1 3 3 1 3 3 1 1 1 1 1 1 1 1 3
#  [409] 1 1 1 3 3 1 1 1 1 1 1 1 3 3 3 2 1 3 3 1 1 1 3 3 3 1 1 1 1 2 1 1 3 3
#  [443] 2 1 3 1 3 1 3 1 3 1 1 3 1 3 1 3 3 1 1 1 1 1 3 1 1 3 1 1 1 3 1 1 3 1
#  [477] 1 1 3 1 1 1 1 1 1 1 1 3 1 3 3 3 1 3 1 3 1 3 3 3 1 1 1 1 1 1 1 1 3 3
#  [511] 1 3 2 3 1 2 1 1 3 3 3 1 3 1 1 1 2 1 2 2 3 1 3 1 1 1 3 1 1 1 1 3 3 1
#  [545] 1 1 3 3 1 1 3 1 3 1 1 3 2 3 1 1 1 1 2 1 3 1 1 1 1 1 1 1 1 1 1 1 1 1
#  [579] 1 2 3 3 3 2 1 2 3 3 2 1 3 3 3 1 1 3 3 2 1 1 1 1 1 1 1 1 1 3 1 3 3 1
#  [613] 1 1 1 1 1 1 2 1 1 1 1 1 3 3 2 3 1 3 2 3 1 3 1 3 3 1 1 1 1 3 3 3 3 3
#  [647] 3 3 1 1 3 1 1 1 1 1 1 1 3 3 3 3 3 3 1 2 3 3 3 3 1 1 3 3 1 1 1 1 3 3
#  [681] 3 1 1 1 1 1 1 3 1 1 1 3 1 1 3 3 3 3 3 1 1 2 1 1 2 3 1 3 1 1 3 2 2 3
#  [715] 1 1 2 3 1 2 3 3 3 2 3 1 3 2 1 3 1 1 2 3 1 2 1 2 3 3 3 1 2 1 3 3 2 3
#  [749] 3 1 1 3 1 3 3 1 1 1 1 1 3 3 3 1 1 1 1 3 3 3 1 1 1 1 3 1 1 1 3 1 1 3
#  [783] 1 3 1 1 1 3 1 1 3 1 1 3 1 1 3 1 1 1 3 1 3 2 3 1 1 3 3 2 1 3 1 1 3 3
#  [817] 3 1 1 1 3 1 1 1 1 1 1 1 1 1 3 1 3 3 1 1 1 3 1 3 3 1 3 1 1 1 1 1 1 1
#  [851] 3 3 3 1 1 3 3 3 1 1 1 1 1 1 3 1 3 2 3 1 3 1 3 1 1 2 1 2 1 3 3 3 3 3
#  [885] 3 3 1 1 3 1 1 1 3 2 1 3 1 1 1 1 3 1 3 1 1 1 3 3 1 3 1 1 3 1 1 3 1 2
#  [919] 1 3 3 1 1 3 3 1 1 1 2 3 2 3 2 3 3 3 2 1 3 2 3 3 3 1 3 3 3 3 3 3 2 3
#  [953] 3 1 3 1 1 1 1 1 1 3 1 1 1 1 1 1 3 1 1 2 1 2 1 1 1 1 3 3 3 3 1 1 1 2
#  [987] 2 1 3 3 1 2 1 3 3 2 3 3 3 3 1 3 2 3 2 3 2 1 3 1 1 1 1 3 3 1 3 1 2 3
# [1021] 1 1 3 3 3 3 3 1 3 3 3 1 1 3 3 1 2 3 3 3 2 1 3 2 1 3 3 2 1 1 3 1 1 1
# [1055] 3 1 1 1 1 1 1 1 1 3 3 3 3 3 2 3 3 3 3 1 3 3 3 1 3 2 1 3 3 3 2 3 1 1
# [1089] 1 1 1 2 1 3 3 1 2 2 3 1 1 3 3 1 1 2 3 3 3 3 3 1 1 1 2 1 1 1 1 1 1 1
# [1123] 3 1 1 1 3 1 3 3 1 1 3 1 1 1 1 1 1 3 3 1 3 3 3 3 2 1 1 1 1 3 1 1 1 1
# [1157] 1 1 1 1 1 1 1 1 1 1 1 3 3 3 3 3 1 1 3 3 1 1 1 3 1 1 1 1 3 1 1 1 1 1
# [1191] 1 1 1 1 1 1 1 1 1 1 3 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 3 1 1 1 1 1 1 1
# [1225] 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 3 1 1 1 1 1 1 1 1 1 1 1 1 1 1 3
# [1259] 3 3 3 3
# 
# Within cluster sum of squares by cluster:
# [1] 24356.96 43792.25 39813.31
#  (between_SS / total_SS =  77.0 %)
# 
# Available components:
# 
# [1] "cluster"      "centers"      "totss"        "withinss"    
# [5] "tot.withinss" "betweenss"    "size"         "iter"        
# [9] "ifault"
```

Récupérons les clusters dans `zoob`


```r
broom::augment(zoo_kmeans, zoo) %>.%
  rename(., cluster = .cluster) -> zoob
```

Et enfin, effectuons un graphique similaire à celui réalisé pour la CAH au module précédent. À noter que nous pouvons ici choisir n'importe quelle paire de variables quantitatives pour représenter le nuage de points. Nous ajoutons des ellipses pour matérialiser les groupes à l'aide de `stat_ellipse()`. Elles contiennent 95% des points du groupe à l'exclusion des extrêmes. Enfin, comme il y a beaucoup de points, nous choisissons de les rendre semi-transparents avec l'argument `alpha = 0.2` pour plus de lisibilité du graphique.


```r
chart(data = zoob, compactness ~ ecd %col=% cluster) +
  geom_point(alpha = 0.2) +
  stat_ellipse() +
  geom_point(data = broom::tidy(zoo_kmeans, col.names = names(zoo6)), size = 5, shape = 17)
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-21-1.png" width="672" style="display: block; margin: auto;" />

Nous observons ici un regroupement beaucoup plus simple qu'avec la CAH, essentiellement stratifié de bas en haut en fonction de la compacité des points (`Compactness`). La tabulation des clusters en fonction des classes connues par ailleurs montre aussi que les k-moyennes les séparent moins bien que ce qu'a pu faire la CAH\ :


```r
table(zoob$class, zoob$cluster)
```

```
#                   
#                      1   2   3
#   Annelid           38   6   6
#   Appendicularian   21   0  15
#   Calanoid          82  41 165
#   Chaetognath        6   0  45
#   Cirriped          14   0   8
#   Cladoceran        50   0   0
#   Cnidarian         13   3   6
#   Cyclopoid          5   5  40
#   Decapod          117   0   9
#   Egg_elongated     50   0   0
#   Egg_round         49   0   0
#   Fish              50   0   0
#   Gastropod         50   0   0
#   Harpacticoid       1   9  29
#   Malacostracan     54  26  41
#   Poecilostomatoid 143   1  14
#   Protist           43   0   7
```

Le cluster numéro 2 n'est pas vraiment défini en terme des classes de plancton car aucune classe ne s'y trouve de manière majoritaire. Le groupe numéro 1 contient la majorité des items de diverses classes, alors que le groupe 3 a une majorité de calanoïdes et d'harpacticoïdes (différents copépodes). Globalement, le classement a un sens, mais est moins bien corrélé avec les classes de plancton que ce que la CAH nous a fourni. Notez que, si nous avions standardisé les données avant d'effectuer les k-moyennes comme nous l'avons fait pour la CAH, nous aurions obtenu d'autres résultats. **La transformation des variables préalablement à l'analyse reste une approche intéressante pour moduler l'importance des différentes variables entre elles dans leur impact sur le calcul des distances, et donc, des regroupements réalisés**. Nous vous laissons réaliser les k-moyennes sur les données `zoo standardisées à l'aide de la fonction `scale()` comme pour la CAH comme exercice.


##### A vous de jouer ! {-}

- Réalisez le tutoriel afin de vérifier votre bonne compréhension de la méthode des k-moyennes.

\BeginKnitrBlock{bdd}<div class="bdd">Démarrez la SciViews Box et RStudio. Dans la fenêtre **Console** de RStudio, entrez l'instruction suivante suivie de la touche `Entrée` pour ouvrir le tutoriel concernant les bases de R\ :

    BioDataScience2::run("06a_kmeans")

N’oubliez pas d’appuyer sur la touche `ESC` pour reprendre la main dans R à la fin d’un tutoriel dans la console R.</div>\EndKnitrBlock{bdd}

- Complétez votre carnet de note par binôme sur le transect entre Nice et Calvi débuté lors du module 5. Lisez attentivement le README (Ce dernier a été mis à jour).

\BeginKnitrBlock{bdd}<div class="bdd">
Completez votre projet. Lisez attentivement le README.

La dernière version du README est disponible via le lien suivant\ :
  
- <https://github.com/BioDataScience-Course/spatial_distribution_zooplankton_ligurian_sea></div>\EndKnitrBlock{bdd}


##### Pour en savoir plus {-}

Il existe une approche mixte qui mèle la CAH et les k-moyennes. Cette approche est intéressante pour les gros jeux de données. Le problématique est expliquée [ici](https://lovelyanalytics.com/2017/11/18/cah-methode-mixte/), et l'implémentation dans la fonction `factoextra::hkmeans()` est détaillée [ici (en anglais)](https://www.datanovia.com/en/lessons/hierarchical-k-means-clustering-optimize-clusters/).

Cet [article](https://www.r-bloggers.com/the-complete-guide-to-clustering-analysis-k-means-and-hierarchical-clustering-by-hand-and-in-r/) explique dans le détail `kmeans()` et `hclust()` dans R, et montre aussi comment on peut calculer les k-moyennes à la main pour bien en comprendre la logique (en anglais).


## Positionnement multidimensionnel (MDS)

Le positionnement multidimensionnel, ou "multidimensional scaling" en anglais, d'où son acronyme fréquemment utilisé en français également\ : le MDS, est une autre façon de représenter clairement l'information contenue dans une matrice de distances. Ici, l'objectif n'est pas de **regrouper** ou de **classifier** les individus du tableau, mais de les **ordonner** sur un graphique en nuage de points en deux ou trois dimensions. Ce graphique s'appelle une "carte", et la technique qui la réalise est une **méthode d'ordination**.

Au départ, nous avons *p* colonnes et *n* lignes dans le tableau cas par variables, c'est-à-dire, *p* variables quantitatives mesurées sur *n* individus distincts. Nous voulons déterminer les similitudes ou différences de ces *n* individus en les visualisant sur une carte où la distance d'un individu à l'autre représente cette similitude. Plus deux individus sont proches, plus ils sont semblables. Plus les individus sont éloignés, plus ils diffèrent. Ces distances entre paires d'individus, nous les avons déjà calculées dans la matrice de distances. Mais comment les représenter\ ? En effet, une représentation exacte ne peut se faire que dans un espace à *p* dimensions (même nombre de dimensions que de variables initiales). Donc, afin de réduire les dimensions à seulement 2 ou 3, nous allons devoir "tordre" les données et accepter de perdre un peu d'information. Ce que nous allons faire avec la MDS correspond exactement à cela\ : nous allons littéralement "écraser" les données dans un plan (deux dimensions) ou dans un espace à trois dimensions. C'est donc ce qu'on appelle une technique de **réduction de dimensions**.

![](images/sdd2_06/tenor.gif)

Il existe, en réalité, plusieurs techniques de MDS. Elle répondent toutes au schéma suivant\ :

- A partir d'un tableau multivarié de *n* lignes et *p* colonnes, nous calculons une matrice de distances (le choix de la transformation initiale éventuelle et de la métrique de distance utilisée sont totalement libres ici^[Chaque métrique de distance offre un éclairage différent sur les données. Elles agissent comme autant de filtres différents à votre disposition pour explorer vos données multivariées.]).
- Nous souhaitons représenter une carte (nuage de points) à *k* dimensions (*k* = 2, éventuellement *k* = 3) où les *n* individus seront placés de telle façon que les proximités exprimées par des valeurs faibles dans la matrice de dissimilarité soient respectées *autant que possible* entre tous les points.
- Pour y arriver les points sont placés successivement sur la carte et réajustés afin de minimiser une **fonction de coût**, encore appelée **fonction de stress** qui quantifie de combien nous avons dû "tordre" le réseau à *p* dimensions initial représentant les distances entre toutes les paires. C'est en adoptant différentes fonctions de stress que nous aboutissons aux différentes variantes de MDS. La fonction de stress est représentée graphiquement (voir ci-dessous) pour diagnostiquer le traitement réaliser et décider si la représentation est utilisable (pas trop tordue) ou non.
- Le positionnement des points faisant intervenir un facteur aléatoire (choix des points à placer en premier, réorganisation ensuite pour minimiser la fonction de stress), le résutat final peut varier d'une fois à l'autre sur les mêmes données, voir ne pas converger vers une solution stable. Il faut en être conscient.

Nous vous épargnons ici les développements mathématiques qui mènent à la définition de la fonction de stress. Nous nous concentrerons sur les principales techniques et sur leurs propriétés utiles en pratique.


### MDS simplifiée sous SciViews::R

Dans R, il existe plus d'une dizaine de fonctions différentes pour réaliser le MDS. Afin de vous simplifier le travail et de pouvoir traiter votre MDS comme d'autres analyses similaires nous vous propoons les fonctions supplémentaites suivante. **Ces fonctions sont à copier-coller en haut de vos scripts R, ou dans un chunk de "setup" à l'intérieur de vos documents R Markdown/Notebook.**


```r
SciViews::R()
library(broom)

# function mds for several multidimensionnal scaling functions ------
mds <- function(dist, k = 2, type = c("metric", "nonmetric", "cmdscale",
"wcmdscale", "sammon", "isoMDS", "monoMDS", "metaMDS"), p = 2, ...) {
  type <- match.arg(type)
  res <- switch(type,
    metric = ,
    wcmdscale = structure(vegan::wcmdscale(d = dist, k = k, eig = TRUE, ...),
      class = c("wcmdscale", "mds", "list")),
    cmdscale = structure(stats::cmdscale(d = dist, k = k, eig = TRUE, ...),
      class = c("cmdscale", "mds", "list")),
    nonmetric = ,
    metaMDS = structure(vegan::metaMDS(comm = dist, k = k, ...),
      class = c("metaMDS", "monoMDS", "mds", "list")),
    isoMDS = structure(MASS::isoMDS(d = dist, k = k, ...),
      class = c("isoMDS", "mds", "list")),
    monoMDS = structure(vegan::monoMDS(dist = dist, k = k, ...),
      class = c("monoMDS", "mds", "list")),
    sammon = structure(MASS::sammon(d = dist, k = k, ...),
      class = c("sammon", "mds", "list")),
    stop("Unknown 'mds' type ", type)
  )
  # For non-metric MDS, we add also data required for the Shepard plot
  if (type %in% c("nonmetric", "sammon", "isoMDS", "monoMDS", "metaMDS"))
    res$Shepard <- MASS::Shepard(d = dist, x = res$points, p = p)
    res
}
class(mds) <- c("function", "subsettable_type")

# plot.mds : MDS2 ~ MDS1 --------------------------------
plot.mds <- function(x, y, ...) {
  points <- tibble::as_tibble(x$points, .name_repair = "minimal")
  colnames(points) <- paste0("mds", 1:ncol(points))
  
  plot(data = points, mds2 ~ mds1,...)
}

autoplot.mds <- function(object, labels, ...) {
  points <- tibble::as_tibble(object$points, .name_repair = "minimal")
  colnames(points) <- paste0("mds", 1:ncol(points))
  
  if (!missing(labels)) {
    if (length(labels) != nrow(points))
      stop("You must provide a character vector of length ", nrow(points),
        " for 'labels'")
    points$.labels <- labels
    chart::chart(points, mds2 ~ mds1 %label=% .labels, ...) +
      geom_point() +
      ggrepel::geom_text_repel() +
      coord_fixed(ratio = 1)
  } else {# Plot without labels
    chart::chart(points, mds2 ~ mds1, ...) +
      geom_point() +
      coord_fixed(ratio = 1)
  }
}

shepard <- function(dist, mds, p = 2)
  structure(MASS::Shepard(d = dist, x = mds$points, p = p),
    class = c("shepard", "list"))

plot.shepard <- function(x, y, l.col = "red", l.lwd = 1,
xlab = "Observed Dissimilarity", ylab = "Ordination Distance", ...) {
  she <- tibble::as_tibble(x, .name_repair = "minimal")
  
  plot(data = she, y ~ x, xlab = xlab, ylab = ylab, ...)
  lines(data = she, yf ~ x, type = "S", col = l.col, lwd = l.lwd)
}

autoplot.shepard <- function(object,  alpha = 0.5, l.col = "red", l.lwd = 1,
xlab = "Observed Dissimilarity", ylab = "Ordination Distance", ...) {
  she <- tibble::as_tibble(object)
  
  chart(data = she, y ~ x) +
    geom_point(alpha = alpha) +
    geom_step(chart::f_aes(yf ~ x), direction = "vh", col = l.col, lwd = l.lwd) +
    labs(x = xlab, y = ylab)
}

# augment.mds -------------------------------------------
augment.mds <- function(x, data, ...){
  points <- as_tibble(x$points)
  colnames(points) <- paste0(".mds", 1:ncol(points))
  bind_cols(data, points)
}

# glance.mds -------------------------------------------
glance.mds <- function(x, ...){
  if ("GOF" %in% names(x)) {# Probably cmdscale() or wcmdscale() => metric MDS
    tibble::tibble(GOF1 = x$GOF[1], GOF2 = x$GOF[2])
  } else {# Non metric MDS
    # Calculate linear and non linear R^2 from the Shepard (stress) plot
    tibble::tibble(
      linear_R2 = cor(x$Shepard$y, x$Shepard$yf)^2,
      nonmetric_R2 = 1 - sum(vegan::goodness(x)^2)
    )
  }
}
```


### MDS métrique ou PCoA

La forme classique, aussi appelée **MDS métrique** ou **analyse en coordonnées principales** (Principal Coordinates Analysis en anglais ou PCoA), va *projetter* le nuage de points à *p* dimensions dans un espace réduit à *k* = 2 dimensions (voire éventuellement à 3 dimensions). Cette projection se fait de manière similaire à une ombre chinoise projettée d'un objet tridimensionnel sur une surface plane en deux dimensions.

![Ombre chinoise\ : un placement astucieux des mains dans le faisceau lumineux permet de projetter l'ombre d'un animal ou d'un objet sur une surface plane. La PCoA fait de même avec vos données.](images/sdd2_06/shadow.jpg)

Considérons un relevé de couverture végétale en 24 stations concernant 44 plantes répertoriées sur le site de l'étude, par exemple, `Callvulg` est *[Calluna vulgaris](https://www.tela-botanica.org/bdtfx-nn-12262-synthese)*, `Empenigr` est *[Empetrum nigrum](https://www.tela-botanica.org/bdtfx-nn-23935-synthese)*, etc. Les valeurs sont les couvertures végétales observées pour chaque plante sur le site, expérimées en pourcents. La première colonne nommée `rownames` à l'importation contient les identifiants des stations (chaînes de caractères). Nous la renommons donc pour un intitulé plus explicite\ : `station`.


```r
read("varespec", package = "vegan") %>.%
  rename(., station = rownames) -> veg
veg
```

```
# # A tibble: 24 x 45
#    station Callvulg Empenigr Rhodtome Vaccmyrt Vaccviti Pinusylv Descflex
#    <chr>      <dbl>    <dbl>    <dbl>    <dbl>    <dbl>    <dbl>    <dbl>
#  1 18          0.55    11.1      0        0       17.8      0.07     0   
#  2 15          0.67     0.17     0        0.35    12.1      0.12     0   
#  3 24          0.1      1.55     0        0       13.5      0.25     0   
#  4 27          0       15.1      2.42     5.92    16.0      0        3.7 
#  5 23          0       12.7      0        0       23.7      0.03     0   
#  6 19          0        8.92     0        2.42    10.3      0.12     0.02
#  7 22          4.73     5.12     1.55     6.05    12.4      0.1      0.78
#  8 16          4.47     7.33     0        2.15     4.33     0.1      0   
#  9 28          0        1.63     0.35    18.3      7.13     0.05     0.4 
# 10 13         24.1      1.9      0.07     0.22     5.3      0.12     0   
# # … with 14 more rows, and 37 more variables: Betupube <dbl>,
# #   Vacculig <dbl>, Diphcomp <dbl>, Dicrsp <dbl>, Dicrfusc <dbl>,
# #   Dicrpoly <dbl>, Hylosple <dbl>, Pleuschr <dbl>, Polypili <dbl>,
# #   Polyjuni <dbl>, Polycomm <dbl>, Pohlnuta <dbl>, Ptilcili <dbl>,
# #   Barbhatc <dbl>, Cladarbu <dbl>, Cladrang <dbl>, Cladstel <dbl>,
# #   Cladunci <dbl>, Cladcocc <dbl>, Cladcorn <dbl>, Cladgrac <dbl>,
# #   Cladfimb <dbl>, Cladcris <dbl>, Cladchlo <dbl>, Cladbotr <dbl>,
# #   Cladamau <dbl>, Cladsp <dbl>, Cetreric <dbl>, Cetrisla <dbl>,
# #   Flavniva <dbl>, Nepharct <dbl>, Stersp <dbl>, Peltapht <dbl>,
# #   Icmaeric <dbl>, Cladcerv <dbl>, Claddefo <dbl>, Cladphyl <dbl>
```

Typiquement ce genre de données ne contient pas d'information constructive lorsqu'une plante est simultanément absente de deux stations (double zéros). Donc, les métriques de type euclidienne ou Manhattan ne conviennent pas ici. Nous devons choisir entre distance de Bray-Curtis ou Canberra en fonction de l'importance que nous souhaitons donner aux plantes les plus rares (avec couverture végétale faible et/ou absentes de la majorité des stations).

Afin de décider quelle métrique utiliser, visualisons à présent l'abondance ou la rareté des différentes plantes\ :



```r
veg %>.%
  select(., -station) %>.% # Colonne 'station' pas utile ici
  gather(., key = "espèce", value = "couverture") %>.% # Tableau en format long
  chart(., couverture ~ espèce) +
    geom_boxplot() + # Boites de dispersion
    labs(x = "Espèce", y = "Couverture [%]") +
    coord_flip() # Labels plus lisibles si sur l'axe Y
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-27-1.png" width="672" style="display: block; margin: auto;" />

Comme nous pouvions nous y attendre, sept ou huit espèces dominent la couverture végétales et les autres données sont complètement écrasées à zéro sur l'axe pour la majorité des stations. Si nous utilisons la distance de Bray-Curtis, l'analyse sera pratiquement réalisée sur seulement ces quelques espèces dominantes. Avec Canberra, nous risquons par contre de donner beaucoup trop d'importance aux espèces extrêmement rares (toutes les espèces ont une importance égale avec cette métrique). Une solution intermédiaire est de transformer les données pour réduire l'écart d'importance entre les espèces abondantes et les rares, soit avec $log(x + 1)$, soit avec $\sqrt{\sqrt{x}}$. Voyons ce que donne la transformation logarithmique ici en utilisant la fonction `log1p()` dans R.



```r
veg %>.%
  select(., -station) %>.%
  gather(., key = "espèce", value = "couverture") %>.%
  chart(., log1p(couverture) ~ espèce) + # Transformation log(couverture + 1)
    geom_boxplot() +
    labs(x = "Espèce", y = "Couverture [%]") +
    coord_flip()
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-28-1.png" width="672" style="display: block; margin: auto;" />

C'est nettement mieux car les données concernant les espèces rares ne sont plus totalement écrasées vers zéro sur l'axe horizontal\ ! La matrice de distances de Bray-Curtis sur nos données transformées log est la **première étape** de l'analyse\ :


```r
veg %>.%
  select(., -station) %>.%
  log1p(.) %>.%
  vegan::vegdist(., method = "bray") -> veg_dist
```

N'imprimez pas le contenu de `veg_dist`\ ! C'est de toutes façons illisible. La PCoA va visualiser son contenu de manière bien plus utile. La **seconde étape** consiste à calculer notre MDS métrique en utilisant `mds$metric()`





```r
veg_mds <- mds$metric(veg_dist)
```

Ensuite, **troisième étape**, le but étant de visualiser les distances nous effectons immédiatement un graphique comme suit\ :


```r
autoplot(veg_mds, labels = veg$station)
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-32-1.png" width="672" style="display: block; margin: auto;" />

Ce graphique s'interprète comme suit\ :

- Des stations proches l'une de l'autre sur la carte ont des indices de dissimilarité faibles. Ces stations sont semblables du point de vue de la couverture végétale.
- Plus les stations sont éloignées les unes des autres, plus elles sont dissemblables.
- Si des regroupements apparaissent sur la carte, il se peut que ce soit des biotopes semblables, et qui diffèrent des autres regroupements. Par exemple ici, les stations 14--15, 20 et 22--25 forment un groupe relativement homogène en haut à gauche du graphique qui s'individualise du reste. Au contraire, les stations 5, 21, ou encore 27 ou 28 sont relativement isolées et constituent donc des assemblages végétaux uniques.
- Les stations aux extrémités sont des configurations extrêmes\ ; celles au centre sont des configurations plus courantes.
- Par contre, ni l'orientation des axes, ni les valeurs absolues sur ces axes n'ont de significations particulières ici. N'en tenez pas compte.

\BeginKnitrBlock{warning}<div class="warning">
Attention\ : rien ne garantit que notre MDS métrique projettée en deux dimensions soit suffisamment représentative des données dans leur ensemble. Si la méthode n'a pas réussi à représenter fidèlement les données, c'est que ces dernières sont trop complexes et ne s'y prêtent pas. Contrôlez donc toujours les indicateurs que sont les valeurs de "Goodness-of-fit" (GOF, qualité d'ajustement).
</div>\EndKnitrBlock{warning}

Les indicateurs "GOF" sont obtenus via la fonction `glance()`\ :


```r
glance(veg_mds)
```

```
# # A tibble: 1 x 2
#    GOF1  GOF2
#   <dbl> <dbl>
# 1 0.527 0.554
```

Ici `GOF1` est la somme des valeurs propres obtenues lors du calcul (ces valeurs propres vous seront expliquées dans le module suivant consacré à l'ACP). Retenez simplement que c'est une mesure de la part de variance du jeu de données initial qui a pu être représentée sur la carte. Plus la valeur se rapproche de 1, mieux c'est, avec des valeurs > 0.7 ou 0.8 qui restent acceptables. Le second indicateur, `GOF2` est la somme uniquement des valeurs propres positives. Certains préfèrent ce dernier indicateur. En principe, les deux sont proches ou égaux. Donc, le choix de l'un ou de l'autre ne devrait pas fondamentalement modifier vos conclusions.

**Ici, avec des valeurs de goodness-of-fit à peine supérieures à 50% nous pouvons considérer que la carte n'est pas suffisamment représentative.** Soit nous tentons de la représenter en trois dimensions (mais c'est rarement plus lisible car il faut quand même se résigner à présenter ce graphique 3D dans un plan à deux dimensions -l'écran de l'ordinateur, ou une feuille de papier- au final). Une autre solution lorsque la MDS métrique ne donne pas satisfaction est de se tourner vers la MDs non métrique. Ce que nous allons faire ci-dessous.

\BeginKnitrBlock{note}<div class="note">
A noter que la PCoA sur matrice euclidienne après standardisation ou non des données est équivalente à une **Analyse en Composantes Principales** (ACP) que nous étudierons dans le module suivant, ... mais avec un calcul nettement moins efficace. Dans ce contexte, la PCoA n'a donc pas grand intérêt. Elle est surtout utile lorsque vous voulez représenter des métriques de distances *différentes* de la distance euclidienne comme c'est le cas ici avec un choix de distances de Bray-Curtis.
</div>\EndKnitrBlock{note}


### MDS non métrique

La version non métrique de la MDS vise à réaliser une carte sur base de la matrice de distances, mais en autorisant des écarts plus flexibles entre les individus... pour autant que des individus similaires restent plus proches les uns des autres que des individus plus différents, et ce, partout sur la carte. Donc, une dissimilarité donnée pourra être "compressée" ou "dilatée", pour autant que la distortion garde l'ordre des points intacts. Cela signifie que la distortion se fera via une fonction monotone croissante (une dissimilarité plus grande ne pouvant pas être représentée par une distance plus petite sur la carte).

La distortion ainsi introduite est appelée un **stress**. C'est un peu comme si vous écrasiez par la force un objet 3D sur une surface plane, au lieu de juste en projeter l'ombre. Comme il existe différentes fonctions de stress, il existe donc différentes versions de MDS non métriques. Ici, nous nous attacherons à maitriser une version implémentée dans `mds$nonmetric()`. Il s'agit de l'une des premières formes de MDS non métriques qui a été proposée par le statisticien Joseph Kruskal (on parle aussi du positionnement multidimensionnel de Kruskal).

La logique est la même que pour la MDS métrique\ :

- étape 1\ : construction d'une matrice de distances,
- étape 2\ : calcul du positionnement des points,
- étape 3\ : réalisation de la carte et vérification de sa validité.

Repartons de la même matrice de distances déjà réalisée pour la MDS métrique qui se nomme `veg_dist`. Le calcul est itératif. Comme il n'est pas garanti de converger, ni de donner la meilleure réponse, nous utilisons ici une fonction "intelligente" qui va effectuer une recherche plus poussée de la solution optimale, notamment en partant de différentes configurations au départ. Pour les détails et les paramètres de cet algorithme, voyez l'aide en ligne de la fonction `?vegan::metaMDS`. Dans le cadre de ce cours, nous ferons confiance au travail réalisé et vérifierons juste qu'une solution est trouvée (indication `*** Solution reached` à la fin). Notez toutefois que le stress est quantifié. Il tourne ici autour de 0,126. Plus la valeur de stress est basse, mieux c'est naturellement.




```r
veg_nmds <- mds$nonmetric(veg_dist) # Calcul
```

```
# Run 0 stress 0.1256617 
# Run 1 stress 0.1262346 
# Run 2 stress 0.1262346 
# Run 3 stress 0.1262346 
# Run 4 stress 0.1256617 
# ... Procrustes: rmse 1.056302e-05  max resid 3.244521e-05 
# ... Similar to previous best
# Run 5 stress 0.1256617 
# ... Procrustes: rmse 6.34123e-06  max resid 1.783762e-05 
# ... Similar to previous best
# Run 6 stress 0.1256617 
# ... Procrustes: rmse 1.768348e-05  max resid 4.243365e-05 
# ... Similar to previous best
# Run 7 stress 0.1262346 
# Run 8 stress 0.1262346 
# Run 9 stress 0.1256617 
# ... Procrustes: rmse 1.668964e-05  max resid 5.282751e-05 
# ... Similar to previous best
# Run 10 stress 0.1262347 
# Run 11 stress 0.1912667 
# Run 12 stress 0.1262346 
# Run 13 stress 0.1256617 
# ... Procrustes: rmse 1.797304e-05  max resid 5.269659e-05 
# ... Similar to previous best
# Run 14 stress 0.1256617 
# ... Procrustes: rmse 1.209131e-05  max resid 3.718001e-05 
# ... Similar to previous best
# Run 15 stress 0.2004491 
# Run 16 stress 0.1256617 
# ... New best solution
# ... Procrustes: rmse 9.079683e-06  max resid 3.527497e-05 
# ... Similar to previous best
# Run 17 stress 0.1262347 
# Run 18 stress 0.1262346 
# Run 19 stress 0.2250581 
# Run 20 stress 0.2105936 
# *** Solution reached
```

A présent, nous pouvons représenter la carte.


```r
autoplot(veg_nmds, labels = veg$station)
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-38-1.png" width="672" style="display: block; margin: auto;" />

Nous avons une représentation assez différente de celle de la MDS métrique. Les stations 5, 21, 27 et 28 sont toujours isolées, mais le reste est regroupé de manière plus homogène. Comment savoir si cette représentation est meilleure que la version métrique qui avait une "goodness-of-fit" décevante\ ? En visualisant les indicateurs de qualité d'ajustement, ainsi que la fonction de stress sur un graphique dit **graphique de Shepard**. Comme d'habitude, `glance()` nous donne les statistiques voulues.


```r
glance(veg_nmds)
```

```
# # A tibble: 1 x 2
#   linear_R2 nonmetric_R2
#       <dbl>        <dbl>
# 1     0.919        0.984
```

Le premier indicateur (R^2^ linéaire) est le coefficient de corrélation linéaire de Pearson entre les distances ajustées et les distances sur la carte au carré. Plus cette valeur est proche de un, moins les distances sont tordues. Le second indicateur, le R^2^ non métrique est calculé comme  1 - S^2^ où S est le stress (tel que quantifié plus haut lors de l'appel à la fonction `mds$nonmetric()`). Cette dernière statistique indique si l'*ordre* des points respecte l'*ordre* des distances partout sur le graphique. Avec 0,98, la valeur est excellente ici. Ensuite le R^2^ linéaire nous indique de combien les différentes distances sont éventuellement distordues. Avec une valeur de 0,92, la distortion n'est pas trop forte ici.

Le diagramme de Shepard permet de visualiser dans le détail la distortion introduite pour parvenir à réaliser la carte en deux dimensions.


```r
veg_sh <- shepard(veg_dist, veg_nmds)
autoplot(veg_sh)
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-40-1.png" width="672" style="display: block; margin: auto;" />

Sur l'axe des abscisses, nous avons les valeurs de dissimilarité présentes dans la matrice de distances. Sur l'axe des ordonnées, le graphique représente les distances de l'ordination, c'est-à-dire, les distances entre les paires de points sur la carte. Chaque point correspond à la dissimilarité d'une paire d'individus sur X, et à la distance entre cette paire sur la carte en Y. Enfin, le trait en escalier rouge matérialise la fonction monotone croissante choisie pour distordre les distances. C'est la **fonction de stress**.

Ce diagramme se lit comme suit\ :

- Plus les points sont proches de la fonction de stress, mieux c'est. Le R^2^ non métrique sera également d'autant plus élevé que les points sont proches de la fonction.
- Plus la fonction de stress est linéaire, plus les *distances* respectent les valeurs de *dissimilarités*. Le R^2^ linéaire est lié à la plus ou moins bonne linéarité de la fonction de stress.

Vous pouvez très bien décider que seul l'*ordre* des individus sur la carte compte. Dans ce cas, la forme de la fonction de stress et la valeur du R^2^ linéaire importent peu. Seul compte la proximité la plus forte possible des points par rapport à la fonction de stress sur le diagramme de Shepard, ainsi donc que la valeur du R^2^ non métrique.

Si par contre, vous voulez être plus contraignant, alors, les distances seront considérées également comme importantes. Vous rechercherez alors une fonction de stress pas trop éloignée d'une droite, ainsi qu'un R^2^ linéaire élevé. Dans ce cas, nous nous rapprochons des exigences de la MDS métrique.

Ici, nous pouvons constater que les deux critères sont bons. Nous pouvons donc nous fier à la carte obtenue à l'aide de la MDs non métrique de Kruskal.


\BeginKnitrBlock{warning}<div class="warning">
Restez toujours attentif à la taille du jeu de données que vous utilisez pour réaliser une MDS, en particuliers une MDS non métrique. Quelques centaines de lignes, ça dois passer, plusieurs dizaines de milliers ou plus, ça ne passera pas\ ! La limite dépend bien sûr de la puissance de votre ordinateur et de la quantité de mémoire vive disponible. Retenez toutefois que la quantité de calculs augmente drastiquement avec la taille du jeu de données à traiter.
</div>\EndKnitrBlock{warning}


##### Pour en savoir plus {-}

- La fonction `mds()` donne accès à d'autres versions de MDS non métriques également. Ainsi, `mds$isoMDS()` ou `mds$monoMDS()` correspondent toutes deux à la version de Kruskal mais en utilisant une seule configuration de départ (donc, moins robustes mais plus rapides à calculer). La `mds$sammon()` est une autre forme de MDS non métrique décrite dans l'aide en ligne de `?MASS::sammon`.

- Des techniques existent pour déterminer la dimension *k* idéale de la carte. Le **graphique des éboulis** (screeplot en anglais) sera abordé au module suivante dans le cadre de l'ACP. Il en existe une version pour le MDS, voyez [ici](https://rpubs.com/YaPi/393252) (en anglais).


##### A vous de jouer ! {-}

- Réalisez le tutoriel afin de vérifier votre bonne compréhension de la mds.

\BeginKnitrBlock{bdd}<div class="bdd">Démarrez la SciViews Box et RStudio. Dans la fenêtre **Console** de RStudio, entrez l'instruction suivante suivie de la touche `Entrée` pour ouvrir le tutoriel concernant les bases de R\ :

    BioDataScience2::run("06b_mds")

N’oubliez pas d’appuyer sur la touche `ESC` pour reprendre la main dans R à la fin d’un tutoriel dans la console R.</div>\EndKnitrBlock{bdd}

- Complétez votre carnet de note par binôme sur le transect entre Nice et Calvi débuté lors du module 5. Lisez attentivement le README (Ce dernier a été mis à jour).

\BeginKnitrBlock{bdd}<div class="bdd">
Completez votre projet. Lisez attentivement le README.

La dernière version du README est disponible via le lien suivant\ :
  
- <https://github.com/BioDataScience-Course/spatial_distribution_zooplankton_ligurian_sea></div>\EndKnitrBlock{bdd}


## Cartes auto-adaptatives (SOM)

Le positionnement multidimensionnel faisant appel à une matrice de distances entre tous les individus, les calculs deviennent vite pénalisants au fur et à mesure que le jeu de données augmente en taille. En général, les calculs sont assez lents. Nous verrons au module suivant que l'**analyse en composantes principales** apporte une réponse intéressante à ce problème, mais nous contraint à étudier des corrélations linéaires et des distances de type euclidiennes.

Une approche radicalement différente, qui reste plus générale car non linéaire, est la méthode des **cartes auto-adaptatives**, ou encore, **cartes de Kohonen** du nom de son auteur se désigne par "self-organizing map" en anglais. L'acronyme **SOM** est fréquemment utilisé, même en français. Cette technique va encore une fois exploiter une matrice de distances dans le but de représenter les individus sur une carte. Cette fois-ci, la carte contient un certain nombre de cellules qui forment une grille, ou mieux, une disposition en nid d'abeille (nous verrons plus loin pourquoi cette disposition particulière est intéressante). De manière similaire au MDS, nous allons faire en sorte que des individus similaires soient proches sur la carte, et des individus différents soient éloignés. La division de la carte en différentes cellules permet de regrouper les individus. Ceci permet une classification comme pour la CAH ou les k-moyennes. Les SOM apparaissent donc comme une technique hybride entre **ordination** (représentation sur des cartes) et **classification** (regroupement des individus).

La théorie et les calculs derrière les SOM sont très complexes. Elles font appel aux **réseaux de neurones adaptatifs** et leur fonctionnement est inspiré de celui du cerveau humain. Tout comme notre cerveau, les SOM vont utiliser l'information en entrée pour aller assigner une zone de traitement de l'information (pour notre cerveau) ou une cellule dans la carte (pour les SOM). Étant donné la complexité du calcul, les développement mathématiques n'ont pas leur place dans ce cours. Ce qui importe, c'est de comprendre le concept, et d'être ensuite capable d'utiliser les SOM à bon escient. Uniquement pour ceux d'entre vous qui désirent comprendre les détails du calcul, vous pouvez lire [ici](https://towardsdatascience.com/kohonen-self-organizing-maps-a29040d688da) ou visionner la vidéo suivante **(facultative et en anglais)**\ :

<!--html_preserve--><iframe src="https://www.youtube.com/embed/0qtvb_Nx2tA?end=266" width="770" height="433" frameborder="0" allowfullscreen=""></iframe><!--/html_preserve-->

Plutôt que de détailler les calculs, nous vous montrons ici comment un ensemble de pixels de couleurs différentes est organisé sur une carte SOM de Kohonen en un arrangement infiniment plus cohérent... automatiquement (cet exemple est proposé par [Frédéric De Lène Mirouze](https://amethyste16.wordpress.com/about/) dans [son blog](https://amethyste16.wordpress.com/2015/10/24/reseau-de-neurones-les-cartes-auto-adaptatives/)).

![Image créée artificiellement avec disposition aléatoire des pixels.](images/sdd2_06/pixels_aleatoires.png)

![Carte SOM obtenue à partir de l'image précédente\ : les pixels sont automatiquement triés par couleur sur la carte.](images/sdd2_06/pixels_som.png)


Ce qui est évident sur un exemple aussi visuel que celui-ci fonctionne aussi très bien pour ranger les individus dans un tableau multivarié *a priori* cahotique comme ceux que nous rencontrons régulièrement en statistiques multivariées en biologie.



### SOM sur le zooplancton

Reprenons notre exemple du zooplankton. 


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

Les 19 premières colonnes représentent des mesures réalisées sur notre plancton et la vingtième est la classe. Nous nous débarasserons de la colonne classe et transformons les données numériques en **matrice** après avoir standardisé les données (étapes *obligatoires*) pour stocker le résultat dans `zoo_mat`.


```r
zoo %>.%
  select(., -class) %>.%
  scale(.) %>.%
  as.matrix(.) -> zoo_mat
```

Avant de pouvoir réaliser notre analyse, nous devons décider d'avance la topologie de la carte, c'est-à-dire, l'arrangement des cellules ainsi que le nombre de lignes et de colonnes. Le nombre de cellules totales choisies dépend à la fois du niveau de détails souhaité, et du nombre d'individus dans votre jeu de données (il faut naturellement plus de données que de cellules, disons, au moins 5 à 10 fois plus). Pour l'instant, considérons les deux topologies les plus fréquentes\ : la **grille rectangulaire** et la **grille hexagonale**. Plus le nombre de cellules est important, plus la carte sera détaillée, mais plus il nous faudra de données pour la calculer et la "peupler". Considérons par exemple une grille 7 par 7 qui contient donc 49 cellules au total. Sachant que nous avons plus de 1200 particules de plancton mesurées dans `zoo`, le niveau de détail choisi est loin d'être trop ambitieux.

La grille rectangulaire est celle qui vous vient probablement immédiatement à l'esprit. Il s'agit d'arranger les cellules en lignes horizontales et colonnes verticales. La fonction `somgrid()` du package `kohonen` permet de créer une telle grille.


```r
library(kohonen) # Charge le package kohonen
```

```
# 
# Attaching package: 'kohonen'
```

```
# The following object is masked from 'package:purrr':
# 
#     map
```

```r
rect_grid_7_7 <- somgrid(7, 7, topo = "rectangular") # Crée la grille
```

Il n'y a pas de graphique `chart` ou `ggplot2` dans le package `kohonen`. Nous utiliserons ici les graphiques de base de R. Pour visualiser la grille, il faut la transformer en un objet `kohonen`. Nous pouvons ajouter plein d'information sur la grille. Ici, nous rajoutons une propriété calculée à l'aide de `unit.distances()` qui est la distance des cellules de la carte par référence à la cellule centrale. Les cellules sont numérotées de 1 à *n* en partant en bas à gauche, en progressant le long de la ligne du bas vers la droite, et en reprenant à gauche à la ligne au dessus. Donc, la ligne du bas contient de gauche à droite les cellules n°1 à 7. La ligne au dessus contient les cellules n°8 à 14, et ainsi de suite. La cellule du centre de la grille en en 4^ème^ ligne en partant du bas et en position 4 sur cette ligne, soit trois lignes complètes plus quatre ($3*7+4=25$). C'est la cellule n°25.


```r
rect_grid_7_7 %>.%
  # Transformation en un objet de classe kohonen qui est une liste
  structure(list(grid = .), class = "kohonen") %>.% # Objet de classe kohonen
  plot(., type = "property", # Graphique de propriété
    property = unit.distances(rect_grid_7_7)[25, ], # distance à la cellule 25
    main = "Distance depuis la cellule centrale") # Titre du graphique
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-48-1.png" width="672" style="display: block; margin: auto;" />

Les cellules de la grille ne sont pas disposées au hasard dans la carte SOM. Des relations de voisinage sont utilisées pour placer les individus à représenter dans des cellules adjacentes s'ils se ressemblent. Avec une grille rectangulaire, nous avons donc deux modalités de variation\ : en horizontal et en vertival, ce qui donne deux gradients possibles qui, combinés donnent des extrêmes dans les coins opposés. Une cellule possède huits voisins directs.

L'autre topologie possible est la grille hexagonale. Voyons ce que cela donne\ :


```r
hex_grid_7_7 <- somgrid(7, 7, topo = "hexagonal")

hex_grid_7_7 %>.%
  # Transformation en un objet de classe kohonen qui est une liste
  structure(list(grid = .), class = "kohonen") %>.% # Objet de classe kohonen
  plot(., type = "property", # Graphique de propriété
    property = unit.distances(hex_grid_7_7)[25, ], # distance à la cellule 25
    main = "Distance depuis la cellule centrale") # Titre du graphique
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-49-1.png" width="672" style="display: block; margin: auto;" />

Ici, nous n'avons que six voisins directs, mais trois directions dans lesquelles les gradients peuvent varier\ : en horizontal, en diagonale vers la gauche et en diagonale vers la droite. Cela offre plus de possibilités pour l'agencement des individus. Nous voyons aussi plus de nuances dans les distances (il y a plus de couleurs différentes), pour une grille de même taille 7 par 7 que dans le cas de la grille rectangulaire. **Nous utiliserons donc préférentiellement la grille hexagonale.**

Effectuons maintenant le calcul de notre SOM à l'aide de la fonction `som()` du package `kohonen`. Comme l'analyse fait intervenir le générateur pseudo-aléatoire, nous pouvons utiliser de manière optionnelle `set.seed()` avec un nombre choisi au hasard (et toujours différent à chaque utilisation) pour que cette analyse particulière-là soit reproductible. Sinon, à chaque exécution, nous obtiendrons un résultat légèrement différent.


```r
set.seed(8657)
zoo_som <- som(zoo_mat, grid = somgrid(7, 7, topo = "hexagonal"))
summary(zoo_som)
```

```
# SOM of size 7x7 with a hexagonal topology and a bubble neighbourhood function.
# The number of data layers is 1.
# Distance measure(s) used: sumofsquares.
# Training data included: 1262 objects.
# Mean distance to the closest unit in the map: 2.519.
```

Le résumé de l'objet ne nous donne pas beaucoup d'info. C'est normal. La technique étant visuelle, ce qui est important, c'est de représenter graphiquement la carte. Avec les graphiques R de base, la fonction utilisée est `plot()`. Nous avons plusieurs types disponibles et une large palette d'options. Voyez l'aide en ligne de`?plot.kohonen`. Le premier graphique (`type = "changes"`) montre l'évolution de l'apprentissage au fil des itérations. L'objectif est de descendre le plus possible sur l'axe des ordonnées pour réduire au maximum la distance des individus par rapport aux cellules (unit en anglais) où ils devraient se placer. Idéalement, nous souhaitons tendre vers zéro. En pratique, nous pourrons arrêter les itérations lorsque la courbe ne diminue plus de manière significative.


```r
plot(zoo_som, type = "changes")
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-51-1.png" width="672" style="display: block; margin: auto;" />

Ici, il semble que nous ne diminuons plus vraiment à partir de la 85^ème^ itération environ. Nous pouvons nous en convaincre en relançant l'analyse avec un plus grand nombre d'itérations (avec l'argument `rlen =` de `som()`).


```r
set.seed(954)
zoo_som <- som(zoo_mat, grid = somgrid(7, 7, topo = "hexagonal"), rlen = 200)
plot(zoo_som, type = "changes")
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-52-1.png" width="672" style="display: block; margin: auto;" />

Vous serez sans doute surpris de constater que la diminution de la courbe se fait plus lentement maintenant. En fait `som()` va adapter son taux d'apprentissage en fonction du nombre d'itérations qu'on lui donne et va alors "peaufiner le travail" d'autant plus. Au final, la valeur n'est pas plus basse pour autant. Donc, nous avons aboutit probablement à une solution. 

Le second graphique que nous pouvons réaliser consiste à placer les individus dans la carte, en utilisant éventuellement une couleur différente en fonction d'une caractéristique de ces individus (ici, leur `class`e). Ce graphique est obtenu avec `type = "mapping"`. Si vous ne voulez pas représenter la grille hexagonale à l'aide de cercles, vous pouvez spécifier `shape = "straight"`. Nous avons 17 classes de zooplancton et il est difficile de représenter plus de 10-12 couleurs distinctes, mais [ce site](https://sashat.me/2017/01/11/list-of-20-simple-distinct-colors/) propose une palette de 20 couleurs distinctes. Nous en utiliserons les 17 premières...


```r
colors17 <- c("#e6194B", "#3cb44b", "#ffe119", "#4363d8", "#f58231", "#911eb4",
  "#42d4f4", "#f032e6", "#bfef45", "#fabebe", "#469990", "#e6beff", "#9A6324",
  "#fffac8", "#800000", "#aaffc3", "#808000", "#ffd8b1")
plot(zoo_som, type = "mapping", shape = "straight", col = colors17[zoo$class])
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-53-1.png" width="672" style="display: block; margin: auto;" />

Nous n'avons pas ajouté de légende qui indique à quelle classe correspond quelle couleur. Ce que nous voulons voir, c'est si les cellules arrivent à séparer les classes. Nous voyons que la séparation est imparfaite, mais des tendances apparaissent avec certaines couleurs qui se retrouvent plutôt dans une région de la carte.

Nous voyons donc ici que, malgré que l'information contenue dans `class` n'ait pas été utilisées. Les différents individus de zooplancton ne se répartissent pas au hasard en fonction de ce critère. Nous pouvons également voir les cellules qui contiennent plus ou moins d'individus, mais si l'objectif est de visionner *uniquement* le remplissage des cellules, le `type = "counts"` est plus adapté. 


```r
plot(zoo_som, type = "counts", shape = "straight")
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-54-1.png" width="672" style="display: block; margin: auto;" />

Nous pouvons obtenir la cellule dans laquelle chaque individu est mappé comme suit\ :


```r
zoo_som$unit.classif
```

```
#    [1] 19  6 25 17 15 19 40 17 40 19 41 39  6 32 17 14 32 22 40 29 26 24 26
#   [24] 20 38 19 20 39 19  5 25  2 19 19 43 20 29 23 36 10 23  5 30 16 17 18
#   [47] 23 17 12 18 11  1 16 17 12 16 10 45 16 17 39  2 19 32  6 45 32  5 32
#   [70] 16  6  2  9 46  4 14 14  6 17 32 19  4  6  4  6 40  9 12 45 16  6 45
#   [93]  7 49  5 10 10 16 10  1 10 17 10 10 36 10 16 36 25  9 31  2 11 12 10
#  [116] 23 15 10 10 25  1  1 10 16 13 16 17 17 38 10 16 25 10  2 10 16 25 11
#  [139] 18 10  1 36 14  7 40 41 48 40 24  6  4 31 12 32 19 35 39 45 19 40 25
#  [162] 24  1 30  4  7 18 43 18 12 30 30 17 17 19 31 36 30 36 30 11 11 16 26
#  [185] 16 11 11 25 18 11 11 19 29 11 30 36 14 24 14 18 14 18 25 26  4 39 25
#  [208] 10 19 11 18 10 15 19 11 32 20 30  6 36  6 12 14 18 10  1 11 18 12 18
#  [231]  1  7 30 30 30 18 23 23 11  3 30 16  1 38 30  1  1 30 11 30 25 36  1
#  [254] 30 11 11 36  1 30  1 36 30 30  1  5 18 30 30 34 34 47 20 20 47 25 38
#  [277] 27 26 39 20 19 34 19 17 34 11 18 47 27 34 39 27 26 19 19 10 32 32 34
#  [300] 31 31 19 33 33 20 36 27 31 47 20 32 39 46 11 33 34 47 19 34 27 30 38
#  [323] 29 25 31 36 25 32  1 29  1 11 11 32 31 28 19 30 31 32 36 12 17 36 18
#  [346] 17 39 38 38 25 36 36 19 10 37 30 18 11 19 18 19 19 30 17 18 31  6 33
#  [369] 12 20 18 11 18 20 20 18 11 23 20 19 11 27 45 19  5 20 19 14 20 20 20
#  [392]  5 29 11 26 20 18 39 20 18  4 23 18 25 10 11 11 38 11 18 17 38 43 18
#  [415] 18 11 18  5 26 24 45 43 32 45  7 38 39 18  3 25 45 39 41 17 19 15  3
#  [438] 46 10 26 45 33 28 22 39  8 30  3 43 20 33  7 41 39 16 39 22 30 38  7
#  [461]  3 25 30  3 38  3 17 37  3  3 18 37 38 15 39 22 15  5 39  3 16 16 16
#  [484] 30 23  3  3 22 31 39 38 45 15 28  3 28 15 43 39 38  3 29 23  3 23 29
#  [507] 18 16 42 42 24 42 40 35  6 44 23  3 42 26 45 35 42 26 18  8 44  3 44
#  [530] 44 49 15 28 16  3  5 43 10 29 10  8 26 43 16 23 14 42 33  3 12 35 41
#  [553] 33 22 32 35 28 42  3 31 18 24 44 24 49 16 22 25 15  7  8 23 23 29 37
#  [576]  1 23 15  3 34 44 44 37 40 29 46 43 43 44 41 20 42 43 24  4 28 35 49
#  [599]  3  3 23 15  3 15 15 23 17 28 15 43 43 23  3 23  3  3  3  3 28  3 17
#  [622]  3 17 15 28 42 28 39  3 28 44 32 33 28  9 33 39 41 22 22  9 38 28 28
#  [645] 42 28 28 43  2 30 38  1 36  9 23 17 17 25 28 39 39 28 28 30  3 28 30
#  [668] 32 26 37 30 22 39 28 22 14 30 30 46 35 28  3  3  3 22 27 30 43  3  3
#  [691] 15 29 25  3 37 29 37 29 29 23  3 34 10 24 34 27 17 24  9  8 33 47 40
#  [714] 32  2  2 34 33 20 34 33 38 33 47 26  9 33 34  9 39  2 32 34 27  8 47
#  [737] 26 34 27 33 28  8 40  2 45 24 34 39 43 17 31 32 23 37 27  9  9 17  9
#  [760] 15 45 37 37 31 17  8 17 45 28 28 19 29 25  7 39 19  9  9 43 41 24 40
#  [783]  9 29  8 24  2 42  8 24 43  8  2 48  8  8 14 24 20 17 28  8 37 40 45
#  [806]  7  7 37 32 46 21 37  7 41 45 40 39  9 17 23 37  7 10 16 16 17 23 30
#  [829] 16  9 38 15 43 38 15 16 15 38 23 36 37  7 29  9 23  9 17 17 17 17 37
#  [852] 39 24 19 32 35 35 44 20 19 23 20 19 17 44 42 45 40 20 24 44 33 45 19
#  [875] 33 46 19 44 33 39 32 39 26 39 38 30 23 30 37 23 20 17 38 39 31 31 29
#  [898] 19 12 23 37 30 38 25 30 16 38 37 12 45 16 23 38 31  7 39 25 46 26 44
#  [921] 35 14 19 39 42 19 19 38 40 14 44 45 40 24 35 39 28 21 48 46 45 32 32
#  [944] 16 44 22 39 43 38 39 46 32 32 25 38  7 23 23 12 23 30 43 22 30 29 23
#  [967] 16 23 38 37 37 40 24 40 26 19 24 22 37 14 28 46  6 26 27 44 44 24 44
#  [990] 45 24 46 26 32 24 45 44 37 39 32 24 42 40 30 40 40 46 23 33 15  5 23
# [1013] 23 37 44 12 43 23 44 42 16 26 44 35 38 42 45 24 35 43 26 20 23 42 43
# [1036] 33 40 44 45 45 44 24 43 46 25 32 42 46  4 24 32  7 23 25 37 17  7 22
# [1059] 23 29 23 15 10 29 38 37 37 35 40 42 39 45 42 24 42 42 44 26 35 46 35
# [1082] 39 42 20 46 42 26 26 14  5 19 46 24 42 35 26 40 40 33 26 24 42 35 24
# [1105] 12 46 42 45 42 42 42 19 24 11 46  5 13  8 13 12 10 17 32 10 15  7 28
# [1128] 11 39 20 10  7 28 32 18  4 11 18 12 45 28 18 45 33 26 28 28  5 11  7
# [1151] 18 28  7  5  5  7  7 18 18 18 18  7 16 18  5  5 16 28 43 32 45 27  5
# [1174] 22 29 29  7 36  6 29  5  5  7  5 29 11 16  5  7 11  7  7  7  7 31  2
# [1197]  8  4  9  8 28  6  2 30  9  8  4 10  8  8  4  9 31 20 11  4 45  2  4
# [1220]  8  1  2  1 31  1 11 10 17  5  8  8 25  9  8  1  1 10  1  1  1  1 23
# [1243] 36 25 10  1  1  1 10 10  1 36  1 25  6 36  2 36 37 43 45 38
```

Par conséquent, nous pouvons créer un tableau de contingence qui répertorie le nombre d'iundividus mappés dans chaque cellule à l'aide de `table()`. Nous l'enregistrons dans `zoo_som_nb` car nous la réutiliserons plus tard.


```r
zoo_som_nb <- table(zoo_som$unit.classif)
zoo_som_nb
```

```
# 
#  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 
# 33 17 40 15 24 17 32 24 23 38 38 18  3 16 24 35 42 43 44 31  2 17 46 33 30 
# 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 
# 28 13 37 26 45 20 35 23 17 18 24 29 33 42 26  9 30 27 26 34 20  8  3  4
```


### Interprétation d'un SOM

De nombreuses autres présentations graphiques sont possibles sur cette base. Nous allons explorer deux aspects complémentaires\ : (1) représentation des variables, et (2) réalisation et représentation de regroupements.


#### Représentation des variables

La carte SOM est orientée. C'est-à-dire que les cellules représentent des formes différentes de plancton telles qu'exprimées à travers les 19 variables utilisées ici (quantification de la taille, de la forme, de la transparence, ...). Le graphique `type = "codes"` permet de visualiser ces différences de manière générale\ :


```r
plot(zoo_som, type = "codes", codeRendering = "segments")
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-57-1.png" width="672" style="display: block; margin: auto;" />

Ce graphique est riche en informations. Nous voyons que\ :

- les très grands individus (`ecd`, `area`, `perimeter`, etc.), soit les segments verts sont en haut à droite de la carte et les petits sont à gauche,
- les individus opaques (variables `mean`, `mode`, `max`, etc.^[Attention\ : la variable `transparency`, contrairement à ce que son nom pourrait suggérer n'est pas une mesure de la transparence de l'objet, mais de l'aspect plus ou moins régulier et lisse de sa silhouette.]), soit des segments dans les tons jaunes sont en bas à droite. Les organismes plus transparents sont en haut à gauche,
- au delà de ces deux principaux critères qui se dégagement prioritairement, les aspects de forme (segments rose-rouges) se retrouvent exprimés moins nettement le long de gradients. La `circularity` mesure la silhouette plus ou moins arrondie des items (sa valeur est d'autant plus élevée que la forme se rapproche d'un cercle). Les organismes circulaires se retrouvent dans le bas de la carte. L'`elongation` et l'`aspect` mesurent l'allongement de la particule et se retrouvent plutôt exprimés positivement vers le haut de la carte.

Nous pouvons donc **orienter** notre carte SOM en indiquant l'information relative aux variables. Lorsque le nombre de variables est élevé ou relativement élevé comme ici, cela devient néanmoins difficile à lire. Il est aussi possible de colorer les cartes en fonction d'une et une seule variable pour en faciliter la lecture à l'aide de `type = "property"`. Voici quelques exemples (notez la façon de diviser une page graphique en lignes et colonnes à l'aide de `par(mfrow = ))` en graphiques R de base, ensuite une boucle `for` réalise les six graphiques l'un après l'autre)\ :


```r
par(mfrow = c(2, 3))
for (var in c("size", "mode", "range", "aspect", "elongation", "circularity"))
  plot(zoo_som, type = "property", property = zoo_som$codes[[1]][, var],
    main = var, palette.name = viridis::inferno)
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-58-1.png" width="672" style="display: block; margin: auto;" />

Nous pouvons plus facilement inspecter les zones d'influence de différentes variables ciblées. Ici, `size` est une mesure de la taille des particules, `mode` est le niveau d'opacité moyen, `range` est la variation d'opacité (un `range` important indique que la particule a des parties très transparentes et d'autres très opaques), `aspect` est le rapport longueur/largeur, `elongation` est une indication de la complexité du périmètre de la particule, et `circularity` est sa forme plus ou moins circulaire. Pour une explication détaillée des 19 variables, faites `?zooplankton`.


#### Regroupements

Lorsque nous avons réalisé une CAH sur le jeu de données `zooplankton`, nous étions obligés de choisir deux variables parmi les 19 pour visualiser le regroupement sur un graphique nuage de points. C'est peu, et cela ne permet pas d'avoir une vision synthétique sur l'ensemble de l'information. Les méthodes d'ordination permettent de visualiser plus d'information sur un petit nombre de dimensions grâce aux techniques de réduction des dimensions qu'elles implémentent. Les cartes SOM offrent encore un niveau supplémentaire de raffinement. Nous pouvons considérer que chaque cellule est un premier résumé des données et nous pouvons effectuer ensuite une CAH sur ces cellules afin de dégager un regroupement et le visualiser sur la carte SOM. L'intérêt est que l'on réduit un jeu de données potentiellement très volumineux à un nombre plus restreint de cellules (ici 7x7 = 49), ce qui est plus "digeste" pour la CAH. Voici comment ça fonctionne\ :


```r
zoo_som_dist <- dist(zoo_som$codes[[1]]) # Distance euclidienne entre cellules
zoo_som_cah <- hclust(zoo_som_dist, method = "ward.D2", members = zoo_som_nb)
```

Notre CAH a été réalisée ici avec la méthode D2 de Ward. L'argument `members =` est important. Il permet de pondérer chaque cellule en fonction du nombre d'individus qui y sont mappés. Toutes les cellules n'ont pas un même nombre d'individus, et nous souhaitons mettre plus de poids dans l'analyse aux cellules les plus remplies.

Voici le dendrogramme\ :


```r
plot(zoo_som_cah, hang = -1)
abline(h = 11.5, col = "red") # Niveau de coupure proposé
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-60-1.png" width="672" style="display: block; margin: auto;" />

Les V1 à V49 sont les numéros de cellules. Nous pouvons couper à différents endroits dans ce dendrogramme, mais si nous décidons de distringuer les cinq groupes correspondants au niveau de coupure à une hauteur de 11,5 (comme sur le graphique), voici ce que cela donne\ :


```r
groupes <- cutree(zoo_som_cah, h = 11.5)
groupes
```

```
#  V1  V2  V3  V4  V5  V6  V7  V8  V9 V10 V11 V12 V13 V14 V15 V16 V17 V18 
#   1   1   1   1   2   2   2   1   1   1   2   2   2   2   1   1   1   2 
# V19 V20 V21 V22 V23 V24 V25 V26 V27 V28 V29 V30 V31 V32 V33 V34 V35 V36 
#   2   2   2   1   1   1   1   2   3   3   1   1   1   3   3   3   4   1 
# V37 V38 V39 V40 V41 V42 V43 V44 V45 V46 V47 V48 V49 
#   3   3   3   3   4   4   3   3   3   3   4   4   5
```

Visualisons ce découpage sur la carte SOM (l'argument `bgcol = ` colorie le fond des cellules en fonction des groupes^[Nous avons choisi ici encore une autre palette de couleurs provenant du package `RColorBrewer`, voir [ici](http://www.sthda.com/french/wiki/couleurs-dans-r).], et `add.cluster.boudaries()` individualise des zones sur la carte en fonction du regroupement choisi).


```r
plot(zoo_som, type = "mapping", pch = ".", main = "SOM zoo, 5 groupes",
  bgcol =  RColorBrewer::brewer.pal(5, "Set2")[groupes])
add.cluster.boundaries(zoo_som, clustering = groupes)
```

<img src="06-k-moyenne-som_files/figure-html/unnamed-chunk-62-1.png" width="672" style="display: block; margin: auto;" />

Grâce à la topographie des variables que nous avons réalisée plus haut, nous savons que\ :

- le groupe vert bouteille en bas à gauche reprend les petites particules plutôt transparentes,
- le groupe orange en bas à droite est constituée de particules très contrastées avec des parties opaques et d'autres transparentes (`range` important),
- le groupe du dessus en bleu est constitué de particules petites à moyennes ayant une forme complexe (variable `elongation`),
- le groupe rose est constitué des particules moyennes à grandes,
- le groupe vert clair d'une seule cellule en haut à droite reprend les toutes grandes particules.

Nous n'avons fait qu'effleurer les nombreuses possibilités des cartes auto-adaptatives SOM... Il est par exemple possible d'aller mapper des nouveaux individus dans cette carte (données supplémentaires), ou même de faire une classification sur base d'exemples (classification supervisée) que nous verrons au cours de Science des Données Biologiques III. Nous espérons que cela vous donnera l'envie et la curiosité de tester cette méthode sur vos données et d'explorer plus avant ses nombreuses possibilités.


##### Pour en savoir plus {-}

- Une [explication très détaillée en français](https://meritis.fr/ia/cartes-topologiques-de-kohonen/) accompagnée de la résolution d'un exemple fictif dans R.

- Une [autre explication détaillée en français](http://eric.univ-lyon2.fr/~ricco/tanagra/fichiers/fr_Tanagra_Kohonen_SOM_R.pdf) avec exemple dans R.

- Si vous êtes aventureux, vous pouvez vous lancer dans la réimplémentation des graphiques du package `kohonen` en `chart`ou `ggplot2`. Voici [un bon point de départ](http://blog.schochastics.net/post/soms-and-ggplot/) (en anglais).

##### A vous de jouer ! {-}

- Réalisez le tutoriel afin de vérifier votre bonne compréhension de la som.

\BeginKnitrBlock{bdd}<div class="bdd">Démarrez la SciViews Box et RStudio. Dans la fenêtre **Console** de RStudio, entrez l'instruction suivante suivie de la touche `Entrée` pour ouvrir le tutoriel concernant les bases de R\ :

    BioDataScience2::run("06c_som")

N’oubliez pas d’appuyer sur la touche `ESC` pour reprendre la main dans R à la fin d’un tutoriel dans la console R.</div>\EndKnitrBlock{bdd}

- Complétez votre carnet de note par binôme sur le transect entre Nice et Calvi débuté lors du module 5. Lisez attentivement le README (Ce dernier a été mis à jour).

\BeginKnitrBlock{bdd}<div class="bdd">
Completez votre projet. Lisez attentivement le README.

La dernière version du README est disponible via le lien suivant\ :
  
- <https://github.com/BioDataScience-Course/spatial_distribution_zooplankton_ligurian_sea></div>\EndKnitrBlock{bdd}

