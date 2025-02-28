Cours _Distant Reading_ : Visualisation

# 11. Cartographie: premiers pas

Simon Gabay

<a style="float:right; width: 20%;" rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Licence Creative Commons" src="https://i.creativecommons.org/l/by/4.0/88x31.png"/></a>

---

# 0. Préparatifs

```r
setwd("~/GitHub/UNIGE/32M7129/Cours_11")
#je charge les données que l'enseignant a préparé pour éviter de potentiels problèmes
#load("Cours_Geneve_11.RData")
```

```r
if(!require("leaflet")){
  install.packages("leaflet")
  library(leaflet)
}
if(!require("sp")){
  install.packages("sp")
  library(sp)
}
if(!require("rgdal")){
  install.packages("rgdal")
  library(rgdal)
}
if(!require("RColorBrewer")){
  install.packages("RColorBrewer")
  library(RColorBrewer)
}
if(!require("htmlwidgets")){
  install.packages("htmlwidgets")
  library(htmlwidgets)
}
if(!require("htmltools")){
  install.packages("htmltools")
  library(htmltools)
}
if(!require("leaflet.extras")){
  install.packages("leaflet.extras")
  library(leaflet.extras)
}
if(!require("geojsonio")){
  install.packages("geojsonio")
  library(geojsonio)
}
if(!require("geojsonlint")){
  install.packages("geojsonlint")
  library(geojsonlint)
}
if(!require("rjson")){
  install.packages("rjson")
  library(rjson)
}
if(!require("leaftime")){
  install.packages("leaftime")
  library(leaftime)
}
```

# 1. Une première carte

Créer sa carte

```r
map <- leaflet() %>% addTiles()
#J'affiche la carte
map
```

Ajoutons Genève sur la carte. Pour cela, rappelons-nous les coordonnées de la ville:
* 46° 20′ 00″ nord, 6° 15′ 00″ est
* 46.2 et 6.15 en degrés décimaux

```r
map <- leaflet() %>% addTiles() %>% 
    #Je définis le point central de ma carte
      setView(lng = 6.15, lat = 46.20, zoom =13)
#J'affiche la carte
map
```

Le zoom permet de se focaliser sur un point précis: ici l'université.

```r
map <- leaflet() %>% addTiles() %>% 
      #Je définis le point central de ma carte et j'ajuste le zoom
      setView(lng = 6.145, lat = 46.199, zoom = 20)
#J'affiche la carte
map
```

Je peux ajouter des marqueurs sur ma carte

```r
map <- leaflet() %>% addTiles() %>% 
      #Je définis le point central de ma carte et j'ajuste le zoom
      setView(lng = 6.15, lat = 46.20, zoom =15) %>% 
      #J'ajoute mon marqueur en précisant sa longitude, sa latitude et son contenu
      addMarkers(lng = 6.145, lat = 46.199, popup = "UniGE")
#J'affiche la carte
map
```

Si je veux ajouter beaucoup d'informations, je peux créer un objet à part. La mise en page se fait avec des règles HTML de base: éléments `<b>` pour le gras, `<i>` pour l'italique, `<a>` et ses attributs pour les liens

```r
#Je crée un objet qui stocke le contenu de mon marqueur
myDesc <- paste(sep = "<br/>",
                     "<b>Université de Genève</b>",
                     "<i>Genève, Suisse</i>",
                     "<img src='https://upload.wikimedia.org/wikipedia/fr/thumb/d/d2/Universit%C3%A9_de_Gen%C3%A8ve_%28logo%29.svg/343px-Universit%C3%A9_de_Gen%C3%A8ve_%28logo%29.svg.png' height='40' width='150'>",
                     "<a href='https://www.unige.ch'>Voir le site</a>")

map <- leaflet() %>% addTiles() %>% 
      #Je définis le point central de ma carte et j'ajuste le zoom
      setView(lng = 6.15, lat = 46.20, zoom =15) %>% 
      #J'ajoute mon marqueur en précisant sa longitude, sa latitude et son contenu
      addMarkers(lng = 6.145, lat = 46.199, popup = myDesc) 
#J'affiche la carte
map
```

Je peux créer un marqueur dédié

```r
#Je crée une icone
myIcon <- makeIcon(iconUrl = "https://upload.wikimedia.org/wikipedia/fr/thumb/d/d2/Universit%C3%A9_de_Gen%C3%A8ve_%28logo%29.svg/343px-Universit%C3%A9_de_Gen%C3%A8ve_%28logo%29.svg.png",
                   #J'en précise la taille
                   iconWidth = 150,
                   iconHeight = 40)

#Je crée un objet qui stocke le contenu de mon marqueur
myDesc <- paste(sep = "<br/>",
                     "<b>Université de Genève</b>",
                     "<i>Genève, Suisse</i>",
                     "<a href='https://www.unige.ch'>Voir le site </a>")

map <- leaflet() %>% addTiles() %>% 
      #Je définis le point central de ma carte et j'ajuste le zoom
      setView(lng = 6.15, lat = 46.20, zoom =15) %>% 
      #J'ajoute mon marqueur en précisant sa longitude, sa latitude et son contenu
      addMarkers(lng = 6.145, lat = 46.199, popup = myDesc, icon = myIcon) #J'ajoute l'icone
#J'affiche la carte
map
```

# 2. Opération tuning

![100%](images/tuning.gif)

Il n'est évidemment pas possible de rentrer manuellement les points: il nous faut créer un _dataframe_.

```r
#Je crée un data frame contenant les longitudes et latitudes de trois villes
capitals <- data.frame(latitudes = c(48.8534, 52.520008, 51.509865),
                     longitudes = c(2.3488, 13.404954, -0.118092),
                     labels = c("Paris", "Berlin", "Londres"))
capitals
```

J'utilise ce _dataframe_ pour la construction de ma carte en l'ajoutant à la fonction `leaflet()`.

```r
#Je place mon dataframe comme contenu de la fonction leaflet()
map <- leaflet(capitals) %>%
  addTiles() %>%
  setView(lng = 5, lat = 50, zoom = 5)  %>%
  #Je récupère le contenu des colonnes avec le tilde ~ suivi du nom de la colonne
  addMarkers(lng = ~longitudes, lat = ~latitudes, popup = ~labels)
map
```

Je peux rajouter des informations pour chaque point

```r
capitals <- data.frame(latitudes = c(48.8534, 52.520008, 51.509865, 50.8466),
                     longitudes = c(2.3488, 13.404954, -0.118092, 4.3528),
                     #population en millions
                     population = c(2.148, 3.769,8.982, 1.209),
                     labels = c("Paris", "Berlin", "Londres", "Bruxelles"))
capitals
```

Je modifie l'apparence de ma carte, an ajoutant des cercles qui sont fonction de la population de chaque ville.

```r
#Je place mon dataframe comme contenu de la fonction leaflet()
map <- leaflet(capitals) %>%
  addTiles() %>%
  setView(lng = 5, lat = 50, zoom = 5)  %>%
  #Je remplace la fonction addMarkers() par la fonction addCircles()
  addCircles(lng = ~longitudes,
             lat = ~latitudes,
             #Je concatène des informations comme contenu de chaque cercle
             popup = ~paste(labels, ":", population),
             #taille du cercle (population multiplié par 5000 pour rendre le cercle visible)
             radius = ~sqrt(population) * 50000,
             #couleur du cercle
             color = "red",
             #opacité du cercle
             fillOpacity = 0.5
            )
map
```

Je peux modifier la couleur du cercle en fonction de sa taille avec la fonction `colorNumeric()`

```r
#Je crée un dégradé entre le jaune et le rouge foncé, le minimum et le maximum étant défini par le contenu de la colonne `population` de mon dataframe `capitals`
colors_function <- colorNumeric(c("yellow", "darkred"), capitals$population)

map <- leaflet(capitals) %>%
  addTiles() %>%
  setView(lng = 5, lat = 50, zoom = 5)  %>%
  #Je remplace la fonction addMarkers() par la fonction addCircles()
  addCircles(lng = ~longitudes,
             lat = ~latitudes,
             #Je concatène des informations comme contenu de chaque cercle
             popup = ~paste(labels, ":", population),
             #taille du cercle (population multiplié par 5000 pour rendre le cercle visible)
             radius = ~sqrt(population) * 50000,
             #couleur du cercle, fonction de la population
             color = ~colors_function(population),
             #opacité du cercle
             fillOpacity = 0.5
            )
map
```

Je dois donc désormais ajouter une légende pour rendre ma carte intelligible

```r
#Je crée un dégradé entre le jaune et le rouge foncé, le minimum et le maximum étant défini par le contenu de la colonne `population` de mon dataframe `capitals`
colors_function <- colorNumeric(c("yellow", "darkred"), capitals$population)

map <- leaflet(capitals) %>%
  addTiles() %>%
  setView(lng = 5, lat = 50, zoom = 5)  %>%
  #Je remplace la fonction addMarkers() par la fonction addCircles()
  addCircles(lng = ~longitudes,
             lat = ~latitudes,
             #Je concatène des informations comme contenu de chaque cercle
             popup = ~paste(labels, ":", population),
             #taille du cercle (population multiplié par 5000 pour rendre le cercle visible)
             radius = ~sqrt(population) * 50000,
             #couleur du cercle, fonction de la population
             color = ~colors_function(population),
             #opacité du cercle
             fillOpacity = 0.5
            ) %>%
            #J'ajoute ma lgende
            addLegend(pal = colors_function, values = ~population, opacity = 0.7)
map
```

Il est possible de changer le fond de carte (_basemap_) en remplaçant la fonction `addTiles()` par la fonction `addProviderTiles`. J'ai le choix entre un grand nombre d'options:
* `providers$OpenTopoMap`
* `providers$Esri.WorldTopoMap`
* `providers$Stamen.Toner`
* `providers$Esri.WorldImagery`
* `providers$Esri.WorldPhysical`
* Pour plus d'exemples, cf. https://leaflet-extras.github.io/leaflet-providers/preview/

```r
#Je crée un dégradé entre le jaune et le rouge foncé, le minimum et le maximum étant défini par le contenu de la colonne `population` de mon dataframe `capitals`
colors_function <- colorNumeric(c("yellow", "darkred"), capitals$population)

map <- leaflet(capitals) %>%
  # changement du fond de carte
  addProviderTiles(providers$Esri.WorldPhysical) %>%
  setView(lng = 5, lat = 50, zoom = 5)  %>%
  #Je remplace la fonction addMarkers() par la fonction addCircles()
  addCircles(lng = ~longitudes,
             lat = ~latitudes,
             #Je concatène des informations comme contenu de chaque cercle
             popup = ~paste(labels, ":", population),
             #taille du cercle (population multiplié par 5000 pour rendre le cercle visible)
             radius = ~sqrt(population) * 50000,
             #couleur du cercle, fonction de la population
             color = ~colors_function(population),
             #opacité du cercle
             fillOpacity = 0.5
            ) %>%
            #J'ajoute ma lgende
            addLegend(pal = colors_function, values = ~population, opacity = 0.7)
map
```

# 3. Travailler avec des csv

```{r, results='hide'}
#import de csv
boyer  <- read.csv("Data/boyer.csv")
```

Notre ficher `.csv` est riche en informations. On a, entre autres, pris soin de noter les longitudes et les latitudes

```r
View(boyer)
df <- data.frame(occ=boyer$occurrence,
                 lat=boyer$lat,
                 long=boyer$lng)
df
```


```r
leaflet()%>%
## On ajoute un fond de carte
addProviderTiles(providers$Esri.NatGeoWorldMap)%>% 
  
  # Je définis le point central de ma carte
  setView(lng = 45, 
          lat = 40, 
          zoom = 3 )%>%
  
  # J'ajoute les points issues des pièces de Boyer
  addCircleMarkers(lng = boyer$lng, 
                   lat = boyer$lat,
                   color = "brown",
                   radius = 14,
                   fillOpacity = 0.6)
```

# 4. Travailler avec GEOjson

Pour rappel un fichier GEOjson ressemble à cela:

```json
{
  "type": "Feature",
  "geometry": {
    "type": "Polygon",
    "coordinates": [
      [
        [100.0, 0.0], [101.0, 0.0], [101.0, 1.0],
        [100.0, 1.0], [100.0, 0.0]
      ]
    ]
  },
  "properties": {
    "Postal code": "2000",
    "Name": "Neuchâtel"
  }
}
```

Vous trouverez dans le dossier `Data` un fichier GEOjson avec les coordonnées de la Suisse.

```r
browseURL("Data/switzerland.geojson")
```

Il est possible d'importer le fichier GEOjson dans `R`.

```r
Switzerland <- rgdal::readOGR("Data/switzerland.geojson")
```

On peut ensuite le projeter dans `R`.

```r
#Je crée la carte
leaflet(Switzerland) %>%
  #Je change mon fond de carte
  addProviderTiles(providers$Stamen.Toner) %>%
  #Je projette mon polygone sur la carte
  addPolygons(stroke = FALSE, smoothFactor = 0.3, fillOpacity = 0.7, color = "orange")  %>%
      setView(lng = 8, lat = 46.5, zoom = 5)
```

Afin de simplifier le travail de création des fichiers GEOjson, vous pouvez aller à cette adresse: http://geojson.io.

![100%](images/geojson_1.png)

![100%](images/geojson_2.png)

```r
Switzerland <- rgdal::readOGR("Data/map.geojson")
#Je crée la carte
leaflet(Switzerland) %>%
  #Je change mon fond de carte
  addProviderTiles(providers$Stamen.Toner) %>%
  #Je projette mon polygone sur la carte
  addPolygons(stroke = FALSE, smoothFactor = 0.3, fillOpacity = 0.4, color = "red")  %>%
      setView(lng = 2.3, lat = 48.85, zoom = 10)
```

# 5. Travailler avec des _shapefiles_

```{r, results='hide'}
#import de shapefile
shp <- readOGR('Data/shapefile/area.shp')
```


On le rappelle, les fichiers `.shp` sont aussi des fichiers complexes, qui contiennent des informations en plus des simples formes géométriques. En l'occurrence, notre document contient

```r
df <- data.frame(area=shp$area,occurrences=shp$number)
df
```

On crée une palette, qui utilise une palette de couleur prédéfinie et qui est fonction du nombre d'occurrence par zone

```r
pal <- colorNumeric(palette = "YlOrRd", domain = shp$number)
```

```r
leaflet(shp)%>%
## On ajoute un fond de carte
addProviderTiles(providers$Esri.NatGeoWorldMap)%>% 
  
  # Je définis le point central de ma carte
  setView(lng = 13, 
          lat = 40, 
          zoom = 2.2 )%>%
  
  # J'ajoute mes grandes zones
    addPolygons(data = shp, 
                stroke = FALSE, 
                smoothFactor = 0.2, 
                color = ~pal(shp$number),
                group = "Zones",
                fillOpacity = 1)
```

Je peux évidemment mélanger les sources

```r
leaflet(shp)%>%
## On ajoute un fond de carte
addProviderTiles(providers$Esri.NatGeoWorldMap)%>% 
  
  # Je définis le point central de ma carte
  setView(lng = 13, 
          lat = 40, 
          zoom = 2.2 )%>%
  
  # J'ajoute mes grandes zones
    addPolygons(data = shp, 
                stroke = FALSE, 
                smoothFactor = 0.2, 
                color = ~pal(shp$number),
                group = "Zones",
                fillOpacity = 1)%>%
  
  # J'ajoute les points issues des pièces de Boyer, importées précédemment
  addCircleMarkers(lng = boyer$lng, 
                   lat = boyer$lat,
                   color = "brown",
                   group = "Boyer",
                   weight = 1,
                   radius = 14,
                   fillOpacity = 0.6)
  
```

Enfin, un petit coup de tuning

![100%](images/tuning.gif)


```r
map<-leaflet(shp)%>%
## On ajoute un fond de carte
addProviderTiles(providers$Esri.NatGeoWorldMap)%>% 
  
  # Je définis le point central de ma carte
  setView(lng = 13, 
          lat = 40, 
          zoom = 2.2 )%>%
  
  # J'ajoute mes grandes zones
    addPolygons(data = shp, 
                stroke = FALSE, 
                smoothFactor = 0.2, 
                color = ~pal(shp$number),
                group = "Zones",
                fillOpacity = 1)%>%
  
  # J'ajoute les points issues des pièces de Boyer, importées précédemment
  addCircleMarkers(lng = boyer$lng, 
                   lat = boyer$lat,
                   color = "brown",
                   group = "Boyer",
                   weight = 1,
                   radius = 14,
                   fillOpacity = 0.6)  %>%
  
  #J'ajoute une fonction "reset", qui permet de reactiver setView()
  addResetMapButton() %>%
    
  #J'ajoute un panneau de contrôle qui me permet de choisir ce que je sélectionne
  addLayersControl(baseGroups = c("Empty layer"),
                   
                   overlayGroups = c("Zones", "Boyer"),
                   options = layersControlOptions(collapsed = TRUE)) %>%
  
  #Je cache par défaut les points tirés des pièces de Boyer
  hideGroup(c("Boyer"))
map
```

Je peux sauvegarder ma carte en html et l'ouvrir dans le navigateur (cela peut prendre un certain donc le code a été commenté).

```r
#saveWidget(map, file="maCarte.html")
#browseURL("maCarte.html")
```

# 6. Timeline

Nous reprenons le même jeu de données que précédemment, avec nos quatre villes.

```r
sales <- data.frame(
                    "Latitude" = c(
                      48.8534, 52.520008, 51.509865, 50.8466
                    ),
                    "Longitude" = c(
                      2.3488, 13.404954, -0.118092, 4.3528
                    ),
                    "population" = c(
                      2.148, 3.769,8.982, 1.209 # en millions
                    ),
                    labels = c("Paris", "Berlin", "Londres", "Bruxelles")
  )
```

Pour créer une _Timeline_ il va logiquement falloir associer des dates: une de début, et une de fin.

```r
#j'ajoute une colonne à mon dataframe
sales$start <- c(
                    "start" = do.call(
                      #je précise que les données sont des dates
                      "as.Date",
                      #je crée une liste avec
                      list(
                        #les données
                        x = c("1-Jan-1918", "1-Jan-1919","1-Jan-1920", "1-Jan-1921"),
                        #le format des données
                        format = "%d-%b-%Y"
                      )
                    )
  )
#je recommence la même manipulation, cette fois avec la date de fin
sales$end <- c(
                    "end" = do.call(
                      "as.Date",
                      list(
                        x = c("30-Dec-1918", "30-Dec-1919","30-Dec-1920", "30-Dec-1921"),
                        format = "%d-%b-%Y"
                      )
                    )
  )
#Je pourrais copier-coller les dates de début si ce sont les mêmes que les dates de fin
#sales$end<-sales$start
```

J'obtiens donc un data frame, avec la population, les dates de début, les dates de fin

```r
sales
```

Je peux désormais convertir mon dataframe en fichier GEOjson:

```r
sales_timeline <- geojson_json(sales,lat="Latitude",lon="Longitude")
```

Le fichier GEOjson que nous avons fabriqué respecte les recommendations de `Leaflet.timeline`, qui est un type spécifique de fichier `GEOjson`. Le début est le suivant:

```json
{"type":
  "FeatureCollection",
  "features":
  [
    {"type":"Feature",
      "geometry": {
        "type":"Point",
        "coordinates":[2.3488,48.8534]
      },
      "properties":{
        "labels":"Paris",
        "population":"2.148",
        "start":"1918-01-01",
        "end":"1918-01-01"
      }
    },
    {"type":"Feature",
      "geometry":{
        "type":"Point",
        "coordinates":[13.404954,52.52001]
      },
      "properties":{
        "labels":"Berlin",
        "population":"3.769",
        "start":"1919-01-01",
        "end":"1919-01-01"
      }
    },
    …
```

Je peux sauvegarder le résultat au format `json` si je veux regarder en détail le code produit:

```r
data_JSON <- fromJSON(sales_timeline)
write(sales_timeline, "output.json") 
```

Je peux désormais créer ma première carte avec timeline.

```r
  leaflet(data_JSON) %>%
    addTiles() %>%
    setView(lng = 5, lat = 50, zoom = 5) %>%
   # il suffit d'ajouter la fonction addTimeline(), c'est tout
    addTimeline()
```

Je peux néanmoins faire mieux, et ajouter des options. Jouez un peu avec celles-ci pour apprendre à les maîtriser.

```r
  leaflet(data_JSON) %>%
    addTiles() %>%
    setView(lng = 5, lat = 50, zoom = 5) %>%
    addTimeline(
      #largeur du bouton de contrôle
      width = "50%",
      sliderOpts = sliderOptions(
        #position sur la carte de la timeline
        position = "topright", #bottomright…
        #nombre d'étapes lors du défilement de la timeline
        step = 20, 
        #rapidité du défilement
        duration = 4000 
      )
    )
```

Je peux aussi modifier la forme des marqueurs (_marker_) que j'utilise sur ma carte.

```r
  leaflet(data_JSON) %>%
    addTiles() %>%
    setView(lng = 5, lat = 50, zoom = 5) %>%
    addTimeline(
      #largeur du bouton de contrôle
      width = "50%",
      sliderOpts = sliderOptions(
        #position sur la carte de la timeline
        position = "topright", #bottomright…
        #nombre d'étapes lors du défilement de la timeline
        step = 20, 
        #rapidité du défilement
        duration = 4000 
      ),
      timelineOpts = timelineOptions(
        styleOptions = styleOptions(
          #rayon des marqueurs (qui seront donc des cercles)
          radius = 10,
          #couleur de la bordure des marqueurs
          color = "grey",
          #couleur de fond des marqueurs
          fillColor = "orange",
          #opacité du marqueur
          fillOpacity = 1
        )
      )
    )
```

Je crée une copie de mon data frame, et je crée une colonne `radius` (rayon) qui est fonction de la population de la ville

```r
sales_radius<-sales
sales_radius$radius <- sales_radius$population*3
```

J'utilise cette information pour gérer la taille des marqueurs

```r
  leaflet(geojson_json(sales_radius)) %>%
    addTiles() %>%
    setView(lng = 5, lat = 50, zoom = 5) %>%
    addTimeline(
      #largeur du bouton de contrôle
      width = "50%",
      sliderOpts = sliderOptions(
        #position sur la carte de la timeline
        position = "topright", #bottomright…
        #nombre d'étapes lors du défilement de la timeline
        step = 20, 
        #rapidité du défilement
        duration = 4000 
      ),
      timelineOpts = timelineOptions(
         # j'annule tous les styles (et j'évite que les paramètres par défaut ne soient utilisés)
        styleOptions = NULL,
        #Je vais mettre mes options de visualisation ici
        pointToLayer = htmlwidgets::JS(
          "
  function(data, latlng) {
  return L.circleMarker(
    latlng,
    {
      radius: +data.properties.radius,
      fillOpacity: 1
    }
  );
}
"
        )
      )
    )
```
