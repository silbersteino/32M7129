Cours _Distant Reading_ : Visualisation

# 8. Analyse de réseau avec `R`

Simon Gabay

<a style="float:right; width: 20%;" rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Licence Creative Commons" src="https://i.creativecommons.org/l/by/4.0/88x31.png"/></a>

---


# 0. Préparatifs

```r
setwd("~/GitHub/UNIGE/32M7129/Cours_08")
#je charge les données que l'enseignant a préparé pour éviter les problèmes
#load("Cours_Geneve_8.RData")
```

```r
if(!require("devtools")){
  install.packages("devtools")
  library(devtools)
}
if(!require("igraph")){
  install.packages("igraph")
  library(igraph)
}
if(!require("tidyverse")){
  install.packages("tidyverse")
  library(tidyverse)
}
if(!require("tidygraph")){
  install.packages("tidygraph")
  library(tidygraph)
}
if(!require("ggraph")){
  install.packages("ggraph")
  library(ggraph)
}
if(!require("qgraph")){
  install.packages("qgraph")
  library(qgraph)
}
if(!require("corrr")){
  install.packages("corrr")
  library(corrr)
}
if(!require("visNetwork")){
  install.packages("visNetwork")
  library(visNetwork)
}
if(!require("networkD3")){
  install.packages("networkD3")
  library(networkD3)
}
if(!require("ForceAtlas2")){
  devtools::install_github("analyxcompany/ForceAtlas2")
  library(ForceAtlas2)
}
if(!require("rgl")){
  install.packages("rgl")
  library(rgl)
}
if(!require("igraphdata")){
  install.packages("igraphdata")
  library(igraphdata)
}
if(!require("netrankr")){
  install.packages("netrankr")
  library(netrankr)
}
if(!require("popgraph")){
  devtools::install_github("dyerlab/popgraph")
  library(popgraph)
}
if(!require("ggmap")){
  install.packages("ggmap")
  library(ggmap)
}
```

Je charge mes deux fichiers: celui avec les nœuds (`nodes.csv`) et celui avec les arêtes (`edges.csv`).

```r
nodes <- as.data.frame(read.csv(file="data/basic/nodes.csv", sep = "\t", header = FALSE, fileEncoding="UTF-8"))
edges <- as.data.frame(read.csv(file="data/basic/edges.csv", sep = "\t", header = FALSE, fileEncoding="UTF-8"))
#Je donne un nom aux colonnes de chaque data.frame
colnames(nodes) <- c("id", "label","type")
colnames(edges) <- c("from", "to")
#Je contrôle que tout est en ordre
```

J'affiche les premiers éléments de chaque fichier, et je compte les rangs pour me faire une idée de ce qui se trouve dans mes données

```r
head(nodes)
head(edges)
nrow(nodes); length(unique(nodes$id))
nrow(edges); nrow(unique(edges[,c("from", "to")]))
```

Je transforme ces deux objets en données `igraph`, qui vont me permettre de faire mes analyses de réseau par la suite.

```r
data <- graph_from_data_frame(d=edges, vertices=nodes, directed=F)
class(data)
```

Mes données se présentent sous cette forme:

```r
data
```

Je peux désormais afficher les nœuds, les arêtes de cette manière. Je peux aussi sélectionner certaines colonnes pour chaque fichier d'une manière particulière aux objets `igraph`

```r
#edges
E(data)
#nodes
V(data)
#nodes labels
V(data)$label
#nodes types
V(data)$type
```

# 1. Mon premier graphe

Je peux désormais fabriquer mon réseau avec la fonction `plot`:

```r
plot(data)
```

Il existe une fonction alternative `plot.igraph`, qui fait la même chose

```r
plot.igraph(data)
```

Il existe enfin une fonction `tkplot` qui est un prototype d'interface utilisateur:

```{r,include=TRUE}
#tkplot(data)
```

Je dispose d'un grand nombre de paramètres pour modifier l'apparence de mon graph et le rendre (si on en a le talent) esthétique:

```r
plot(data,
     #courbure de l'arête
     edge.curved=0.1,
     #couleur de l'arête
     edge.color="orange",
     #couleur du nœud
     vertex.color="green",
     #couleur du contour du nœud
     vertex.frame.color="#555555",
     #couleur de l'étiquette du nœud
     vertex.label.color="darkred",
     #contenu de l'étiquette du nœud
     vertex.label=V(data)$type,
     #taille de la police
     vertex.label.cex=1)
```

On peut customiser encore plus la décoration en intervenant plus lourdement sur la mise en page. Une manière de faire va être de créer des vecteurs à partir des données, en substituant la valeur qui nous intéresse par la forme que l'on souhaite lui donner. Par exemple, si je veux changer la couleur du nœud en fonction du label

```r
#J'ai une colonne de mon objet igraph avec les labels
V(data)$type
#je copie le contenu de cette colonne dans un nouvel objet
to_colors<-V(data)$type
#Je substitue toute les cellules avec l'information `libraire` par la couleur souhaitée
to_colors<-replace(to_colors,to_colors=="Libraire","orange")
to_colors
#Je continue avec les autres valeurs
to_colors<-replace(to_colors,to_colors=="Auteur","green")
to_colors<-replace(to_colors,to_colors=="Imprimeur","red")
#J'obtiens un nouvel objet avec des couleurs à la place des labels
to_colors
```

Je peux désormais appliquer cette couleur à chaque nœud

```r
#J'ai une colonne couleur qui est vide, que je vais remplir avec l'objet `to_colors` que je viens de créer
V(data)$color
#Je remplace la couleur du graphe avec celle que je viens de définir
V(data)$color <- to_colors
#Le graphe change de couleur
plot(data)
```

Je peux faire la même opération avec la forme des nœuds, et améliorer encore le rendu.

**ATTENTION**: le rendu dans RStudio n'est pas forcément optimum: pensez l'ouvrir dans une nouvelle fenêtre pour voir le résultat.

```r
#On change la forme du nœud en fonction du label
to_shape<-V(data)$type
to_shape<-replace(to_shape,to_shape=="Auteur","square")
to_shape<-replace(to_shape,to_shape=="Imprimeur","circle")
to_shape<-replace(to_shape,to_shape=="Libraire","sphere")

#Je peux à nouveau changer l'objet igraph
V(data)$color <- to_colors
V(data)$label.color <- "black"
#E(data)$edge.color <- "gray80"

#ou bien intervenir directement dans les paramètres du graphe
plot(data,
     vertex.label.degree=2,
     #on injecte le vecteur avec les formes que nous venons de créer
     vertex.shape=to_shape,
     #taille des nœuds
     vertex.size = 10,
     #taille de la police
     vertex.label.cex=0.7
     )

title("mon graphe", sub="premier test")

#J'ajoute une petite légende
legend(x=-1.5, y=-1.1, c("Auteur","Libraire", "Imprimeur"), pch=21,
       col="#777777", pt.bg=c("green", "orange", "red"), pt.cex=2, cex=.8, bty="n", ncol=1)
```

Nous avons encore un problème: les relations multiples sont toutes dessinées, car elles sont restées dans le tableau. Quelles sont-elles?

Deux problèmes sont possibles:
* Une boucle (_loop_) est un nœud relié par une arête à lui-même
* Un multiple (_multiple_) sont deux nœuds reliés plusieurs fois ensemble. _N.B._ si le graph est dirigé `2->1` n'est pas un multiple de `1->2`, mais s'il est dirigé oui.

```r
which_loop(data)
which_multiple(data)
```

Comme j'ai de nombreux multiples, je vais les transformer en poids pour chaque arête

```r
#Je compte les multiples pour chaque arête
count_multiple(data)
#Je fais une copie pour travailler dessus (si j'ai besoin des données originales plus tard)
data_simplified <- data
#Je copie-colle le nombre de multiple par arête dans la colonne des poids
E(data_simplified)$weight <- count_multiple(data_simplified)
#je retire les doublons (les multiples) avec la fonction simplify()
data_simplified <- simplify(data_simplified)
```

J'affiche mon nouveau graph: la largeur de l'arête dépend désormais du poids

```r
E(data_simplified)$width <-E(data_simplified)$weight/2
plot(data_simplified,
     #pas de courbure de l'arête
     edge.curved=0,
     #distance entre le label et le nœud
     vertex.label.dist=1,
     #Choix de la police
     vertex.label.family="Times",
     #Choix de la forme
     vertex.shape=to_shape,
     # Taile du nœud
     vertex.size = 6,
     #taille de la police
     vertex.label.cex=0.6
     )
title("Et voilà!")
#J'ajoute une petite légende
legend(x=-2, y=-0.5, c("Auteur","Libraire", "Imprimeur"), pch=21,
       col="#777777", pt.bg=c("green", "orange", "red"), pt.cex=1, cex=.8, bty="n", ncol=1)
```

# 2. Tracé de graphe

Il existe différentes visualisation, algorithmes, etc. La logique est toujours la même: je pré-traite mon objet `igraph` avec une fonction spécifique au tracé choisi, et j'utilise le résultat de ce prétraitement comme valeur du paramètre `layout`. Ici, le _circular layout_:

```r
to_circle<-layout_in_circle(data)
plot(data, layout=to_circle, vertex.size=1)
```

Il existe une multitude de _layouts_. Je peux tous les afficher d'un coup, pour jeter un coup d'œil à la forme qu'ils prennent, et choisir celui qui m'intéresse le plus

```r
layouts <- grep("^layout_", ls("package:igraph"), value=TRUE)[-1]
# J'en retire certains si je veux
layouts <- layouts[!grepl("bipartite|merge|norm|sugiyama|tree", layouts)]
#je prépare la mise en page du résultat
par(mfrow=c(3,3), mar=c(1,1,1,1))
#Je fais un boucle: un graph par itération
for (layout in layouts) {
  print(layout)
  l <- do.call(layout, list(data))
  plot(data_simplified, edge.arrow.mode=0, layout=l, main=layout) }
```

S'il y en a un qui m'intéresse, je peux l'appliquer de la même manière que pour le cercle.

Evidemment, je ne dois choisir un graphe adapté…

```r
data("Koenigsberg")
plot(Koenigsberg)
```

Il semble que la forme en étoile soit bien adaptée, étant donné la centralité de Molière:

```r
to_star<-layout_as_star(data)
plot(data, layout=to_star, vertex.size=0.1)
```

## 2.1 Les algorithmes _forced base_

Les tracés les plus importants sont les _forced base_, qui nécessitent des algorithmes particuliers. Regrdons-les en détail.

### 2.1.1 _Fruchterman Reingold_

Fruchterman, Thomas M. J.; Reingold, Edward M. (1991), "Graph Drawing by Force-Directed Placement", _Software – Practice & Experience_, Wiley, 21 (11): 1129–1164, [doi:10.1002/spe.4380211102](https://doi.org/10.1002/spe.4380211102).

Plus d'informations [ici](https://github.com/gephi/gephi/wiki/Fruchterman-Reingold)

Le algorithme est assez comppliqué, et le temprs de calcul conséquent. On l'utilise peu avec des grandes bases de données (plus de 10 000 nœuds).

```r
layout_fr<-layout_with_fr(data_simplified)
E(data_simplified)$width <-E(data_simplified)$weight
plot(data_simplified, layout=layout_fr,
     #J'ajoute les poids
     weight=TRUE,
     #pas de courbure de l'arête
     edge.curved=0)
```

On peut jouer sur les paramètres et augmenter le nombre d'itération

```r
#Je passe sur 2 colonnes, 2 rangs
par(mfrow=c(2, 2))
#J'ajuste la marge pour le titre
par(oma=c(1,1,1,1))

## layout_with_fr
plot.igraph(data_simplified, layout = layout_with_fr(data_simplified, niter = 1), main = 1)
plot.igraph(data_simplified, layout = layout_with_fr(data_simplified, niter = 5), main = 5)
plot.igraph(data_simplified, layout = layout_with_fr(data_simplified, niter = 10), main = 10)
plot.igraph(data_simplified, layout = layout_with_fr(data_simplified, niter = 20), main = 20)
plot.igraph(data_simplified, layout = layout_with_fr(data_simplified, niter = 50), main = 50)
plot.igraph(data_simplified, layout = layout_with_fr(data_simplified, niter = 75), main = 75)
plot.igraph(data_simplified, layout = layout_with_fr(data_simplified, niter = 100), main = 100)
plot.igraph(data_simplified, layout = layout_with_fr(data_simplified, niter = 150), main = 150)
plot.igraph(data_simplified, layout = layout_with_fr(data_simplified, niter = 200), main = 200)
plot.igraph(data_simplified, layout = layout_with_fr(data_simplified, niter = 300), main = 300)
plot.igraph(data_simplified, layout = layout_with_fr(data_simplified, niter = 500), main = 500)
plot.igraph(data_simplified, layout = layout_with_fr(data_simplified, niter = 700), main = 700)
title("No. of Iterations (layout_with_fr)", outer = TRUE)
par(mfrow=c(1, 1))
par(oma=c(0,0,0,0))

```

Voyons le résultat apèrs 700 itérations:

```r
## J'épaissis l'arête en fonction du poids
E(data_simplified)$width <-E(data_simplified)$weight
plot.igraph(data_simplified,
            layout = layout_with_fr(data_simplified, niter = 700),
            main = 700,
            #pas de courbure de l'arête
            edge.curved=0,
            #taille de la police
            vertex.label.cex=0.6,
            # Taile du nœud
            vertex.size = 8)
title("700 itérations", outer = TRUE)
```

### 2.1.2 _DrL_

Le nom a changé plusieurs fois: d'abord rebaptisé _vxOrd_, on parle d'_OpenOrd_.

* Martin, S., Brown, W.M., Klavans, R., Boyack, K.W., DrL: Distributed Recursive (Graph) Layout. SAND Reports, 2008. 2936: p. 1-10.
* S. Martin, W. M. Brown, R. Klavans, and K. Boyack (to appear, 2011), "OpenOrd: An Open-Source Toolbox for Large Graph Layout," SPIE Conference on Visualization and Data Analysis (VDA).

Plus d'informations [ici](https://github.com/gephi/gephi/wiki/OpenOrd)

Il va tenter de faire ressortir au maximum des grands clusters très nets en coupant les liens les plus longs. Evidemment cet algorithme nécessite des graphes de grande taille, nous chargeons donc un gros jeu de données tiré du package `igraphdata`.

```r
data("USairports")
#On trace le graphe
layout_drl <- layout.drl(USairports)
plot(USairports, layout=layout_drl, main="DrL")
```

Nous pouvons comparer l'effet de ce découpage avec celui effectué par l'algorithme précédent, _Fruchterman Reingold_

```r
par(mfrow=c(1, 2))
par(oma=c(0,0,2,0))
plot(USairports, layout=layout_with_fr(USairports), weight=T, main="FR")
plot(USairports, layout=layout_drl, main="DrL")
```

## 2.2 le `Geolayout`

J'ai préparé un tout petit jeu de données avec des coordonnées géographiques

```r
nodes_geo <- as.data.frame(read.csv(file="data/geo/nodes.csv", sep = "\t", header = FALSE, fileEncoding="UTF-8"))
edges_geo <- as.data.frame(read.csv(file="data/geo/edges.csv", sep = "\t", header = FALSE, fileEncoding="UTF-8"))
#Je donne un nom aux colonnes de chaque data.frame
colnames(nodes_geo) <- c("id", "label","lat","long")
colnames(edges_geo) <- c("from", "to")
nodes_geo
edges_geo
```

Je transforme ces deux objets en données `igraph`, qui vont me permettre de faire mes analyses de réseau par la suite.

```r
data_geo <- graph_from_data_frame(d=edges_geo, vertices=nodes_geo, directed=F)
class(data_geo)
plot(data_geo)
```

Je projette ce réseau sur une carte, en plaçant les nœuds en fonction de leurs coordonnées géographiques

```r
#Je définis les lat et long de mon cadre
western_europe<-c(left = -12, bottom = 40, right = 20, top = 55)
#Je récupère mon fond de carte selon les dimensions prévues supra
map <- get_stamenmap(western_europe, zoom=6, source="stamen", maptype="toner-lite", filetype="png")
#Je crée une carte à partir de mon fond de carte
p<-ggmap(map)
#J'ajoute les arêtes
p = p + geom_edgeset(aes(x=long, y=lat), data_geo, colour=gray(0.1, 0.3), size=1)
#J'ajoute les nœuds
p = p + geom_nodeset(aes(x=long, y=lat), data_geo, size=3, colour="red")
#J'affiche le tout
p
```

# 3. La force des liens

## 3.1 Les mesures

Densité (_density_): la proportion de liens dans un réseau relativement au total des liens possibles.

```r
edge_density(data)
```

Centralité de proximité: Distance moyenne du nœud à tous les autres nœuds (_Closeness_)

```r
closeness.cent <- closeness(data)
closeness.cent
```

Pour rappel, les noms attachés sont accessibles ainsi:

```r
cbind(V(data)$label,closeness.cent)
```

Centralité d’intermédiarité: Nombre de fois que le nœud se trouve sur le plus court chemin entre deux autres nœuds (_Betweenness_)

```r
closeness.bet <- betweenness(data)
closeness.bet
```

Centralité de vecteurs propres: Score d’autorité attribué à un nœud en fonction du score de ses voisins. (_Eigenvector_).

```r
closeness.eig <- eigen_centrality(data)
closeness.eig$vector
```

Centralité de degré: Nombre de connexions du nœud (_Degree_)

```r
closeness.deg <- degree(data_simplified)
closeness.deg
```

On peut réutiliser ces données pour la visualisation, en ajustant la taille des nœuds à la centralité de degré

```r
V(data_simplified)$size <- (closeness.deg*0.3)
plot(data_simplified, layout=layout_fr, main="FR")
```

On peut ajuster cette taille avec d'autres mesures de centralité, comme la centralité de vecteur

```r
V(data_simplified)$size <- (closeness.eig$vector*30)
plot(data_simplified, layout=layout_fr, main="FR")
```

## 3.2 Un exemple d'analyse de la centralité

Tentons une approche plus pratique avec un réseau célèbre: celui de la Florence de la Renaissance (disponible dans le package `netrankr`).

```r
data("florentine_m")
#Une famille n'est pas reliée aux autres: la famille Pucci
degree(florentine_m)==0
#On la retire pour simplifier la visualisation
florence <- delete_vertices(florentine_m,which(degree(florentine_m)==0))
```

J'obtiens un joli graphe:

```r
plot.igraph(florence, layout = layout_with_fr(florence, niter = 500),
                      main = "Florence au XVème s.")
```

Je peux proportionner la taille des nœuds en fonction de la fortune de chaque famille:

```r
plot(florence,
     layout = layout_with_fr(florence, niter = 500),
     vertex.label.cex=V(florence)$wealth*0.012,
     vertex.size=V(florence)$wealth*0.2)
```

Mais est-ce que la richesse fait tout? Probablement pas… tentons d'évaluer la centralité des différentes familles

```r
#Je fais un data-frame à partir de différents calculs de centralité
cent.data_frame <- data.frame(
  degree = degree(florence),
  betweenness = betweenness(florence),
  closeness = closeness(florence),
  eigenvector = eigen_centrality(florence)$vector,
  subgraph = subgraph_centrality(florence))
#Je peux accéder au résultat sous la forme de tableau qui résume toutes les données
View(cent.data_frame)

# Je donne le nom de la famille la plus centrale pour chaque mesure
V(florence)$name[apply(cent.data_frame,2,which.max)]
```

# 4. Visualiser

Il est possible de proposer une foule de visualisation. Par exemple, il est possible de remplacer les labels par la distance qui sépare un nœud d'un autre – ici, Thomas Jolly.

```r
dist.thoJo <- distances(data, v=V(data)[label=="Thomas Jolly"], to=V(data), weights=NA)
# Set colors to plot the distances:

green.dark <- colorRampPalette(c("darkgreen", "lightgreen"))
col <- green.dark(max(dist.thoJo)+1)
col <- col[dist.thoJo+1]

plot(data, vertex.color=col, vertex.label=dist.thoJo)
```

On peut mettre en valeur le chemin le plus court entre deux poins

```r
mon_chemin <- shortest_paths(data,
                            from = V(data)[label=="Thomas Jolly"],
                             to  = V(data)[label=="Pierre Corneille"],
                            #je colorie le nœud et l'arête
                             output = "both")

# On génère une couleur pour les arêtes en fonction du chemin
couleur_arc <- rep("gray80", ecount(data))
couleur_arc[unlist(mon_chemin$epath)] <- "orange"
"Couleur des arcs"
couleur_arc #cf. 128 et 219

# On génère une largeur pour l'arête en fonction du chemin
largeur_arc <- rep(2, ecount(data))
largeur_arc[unlist(mon_chemin$epath)] <- 6
"Largeur des arcs"
largeur_arc #cf. 128 et 219

# Generate node color variable to plot the path:
coleur_noeud <- rep("gray40", vcount(data))
coleur_noeud[unlist(mon_chemin$vpath)] <- "gold"
"Les nœuds"
coleur_noeud #cf. 1 et 13

plot(data, vertex.color=coleur_noeud, edge.color=couleur_arc,
     edge.width=largeur_arc, edge.arrow.mode=0)
```

On peut aussi regrouper en cluster les données, que l'on représente en dendogramme, comme pour la stylométrie

```r
ceb <- cluster_edge_betweenness(data)
dendPlot(ceb, mode="hclust")
```

Je projette ensuite ma classification sur mon graph

```r
plot(ceb, data)
```

On peut l'afficher en 3d. Pour cela j'utilise le paramètre dim=3 pour mon layout.

```r
coords <- layout_with_fr(data_simplified, dim=3)
open3d()
rglplot(data, layout=coords)
```

Je peux faire une sauvegarde, notamment en HTML pour l'ouvrir dans le navigateur

```r
dirfolder=getwd()
#open3d plutôt que rgl.open() pour une sauvegarde
open3d()
rglplot(data_simplified, layout=coords)

#Je prépare l'angle
rgl.viewpoint(theta=0, phi=0)

#Sauvegarder un screenshot (en png)
rgl.snapshot(paste(dirfolder,"monGRaph3d.png",sep=""), fmt="png", top=TRUE)

#Sauvegarde en html
rglfolder=writeWebGL(dir = paste(dirfolder,"first_net3d",sep=""), width=900)

#J'ouvre le résultat dans le navigateur
browseURL(rglfolder)
```

Je peux produire une visualisation interactive directement dans `R`. Je prépare les données

```r
# je convertis mon igraphe en une liste  composée de deux data.frames (nodes et edges)
data_3d_vis <- toVisNetworkData(data)

# Pour le menu déroulant (cf infra)
names <- sort(data_3d_vis$nodes$label)
```

Et je lance une visualisation en 3D

```r
visNetwork(nodes = data_3d_vis$nodes,
           edges = data_3d_vis$edges,
           main = "Mon graphe interactif",
           submain = "Algorithme de Fruchterman–Reingold",
           footer = "Wow") %>%
  #Je trace le graphe
  visIgraphLayout(layout = "layout_with_fr",
                  smooth = FALSE,
                  #J'ajoute de la dynamique (cf. _infra_)
                  physics = TRUE
                )
```

Je rajoute des options de visualisation, comme une modification des nœuds s'ils sont sélectionnés, ou un selecteur sous la forme de liste déroulante

```r
visNetwork(nodes = data_3d_vis$nodes,
           edges = data_3d_vis$edges,
           main = "Mon graphe interactif",
           submain = "Algorithme de Fruchterman–Reingold",
           footer = "Wow") %>%
  #Je trace le graphe
  visIgraphLayout(layout = "layout_with_fr",
                  smooth = FALSE,
                  #J'ajoute de la dynamique (cf. _infra_)
                  physics = TRUE
                ) %>%
  #Je mets en valeur les nœuds liés
  visOptions(highlightNearest = list(enabled = TRUE,
                                     #séparés de 1 degré
                                     degree = 1,
                                     #il s'illuminent quand la souris passe sur le nœud
                                     hover = TRUE),
             #Je crée un sélecteur
             nodesIdSelection = list(enabled = TRUE,
                                     values = names))
```

Je vais avoir besoin "d'écarter" mon graphe, en ajoutant de la répulsion entre les nœuds (ce qui n'est pas tâche facile…)

```r
data_3d_vis_plot <- visNetwork(nodes = data_3d_vis$nodes,
                               edges = data_3d_vis$edges,
                               main = "Mon graphe interactif",
                               submain = "Algorithme de Fruchterman–Reingold",
                               footer = "Wow") %>%
                    #Je trace le graphe
                    visIgraphLayout(layout = "layout_with_fr",
                                    smooth = FALSE,
                                    #J'ajoute de la dynamique (cf. _infra_)
                                    physics = TRUE
                                    ) %>%
                    #Je mets en valeur les nœuds liés
                    visOptions(highlightNearest = list(enabled = TRUE,
                                                      #séparés de 1 degré
                                                      degree = 1,
                                                      #passage souris
                                                      hover = TRUE),
                              #Je crée un sélecteur
                              nodesIdSelection = list(enabled = TRUE,
                                                      values = names)
                              ) %>%
                    #taille des nœuds
                    visNodes(size = 50) %>%
                    #couleur des arêtes
                    visEdges(color = list(highlight = "lightgray")) %>%
                    #Je paramètre la répulsion
                    visPhysics(#Vélocité des nœuds
                               maxVelocity = 1,
                               #type de répulsion hiérarchique
                               solver = "forceAtlas2Based",
                               #paramètres du forceAtlas2Based
                               #la `gravitationalConstant` décrit la répulsion (l'écartement entre les nœuds), le chiffre est donc négatif, sinon oncrée de l'attraction
                               forceAtlas2Based = list(gravitationalConstant = -1000)
                     )

data_3d_vis_plot
```

# 5. Sauvegardes

On sauvegarde le résultat
```r
write_graph(data, "edgelist.txt", format="edgelist")
svg(file="monGraph.svg")
plot(data)
dev.off()

png(file="monGraph.png")
plot(data)
dev.off()
```

![100% center](monGraph.png)

![100% center](monGraph.svg)


