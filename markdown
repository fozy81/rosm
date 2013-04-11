
Dunbar Coummunity Map Data
----------------------------------------------------------------------------------------------
```{r echo=FALSE, results='hide', echo=FALSE, message=FALSE, comment=FALSE}
{library(osmar) # loads osmar libarary which downloads data from openstreetmap as puts it into a dataframe 
library(rgeos) # geometry library get area in metres squared
library(rgdal) # projection linrary to transform projection
library(knitr) # create html webpage from this code
 library(sp)} # library for spatial analysis - can plot 'graphs'/maps of sp objects
```
```{r, echo=FALSE} 
src <- osmsource_api() # sets default openstreetmap api - url from data is downloaded
bb2 <- center_bbox(-2.515714, 56.0028, 1000, 1000) # the center point co-ordinates of the bounding box for data download, 1000 by 1000 metres bounding box
ua3 <- get_osm(bb2, source = src) # get data and from osm api and called it 'ua3'

hw_ids2 <- find(ua3, way(tags(k == "lit"))) #find ways tagged for 'lit'
hw_ids2 <- find_down(ua3, way(hw_ids2)) #find nodes in the ways tagged 'lit'
hw2 <- subset(ua3, ids = hw_ids2) # extract nodes associated with lit tagged ways from ua3 and call it hw2

bg_ids2 <- find(ua3, way(tags(k == "building"))) # find ways tagged 'building' in ua3
bg_ids2 <- find_down(ua3, way(bg_ids2)) # find nodes in ways tagged 'building'
bg2 <- subset(ua3, ids = bg_ids2) # extract all the nodes associated with ways with 'building'

pg_ids2 <- find(ua3, way(tags(v == "playground"))) #find ways tagged 'playground' i.e. leisure=playground. v = value(playground), k= key(leisure) so I'm assuming all playground values are associated with leisure=
pg_ids2 <- find_down(ua3, way(pg_ids2)) #find nodes within playground ways
pg2 <- subset(ua3, ids = pg_ids2) #extract all nodes associated with playgrounds

pk_ids2 <- find(ua3, way(tags(v == "park")))
pk_ids2 <- find_down(ua3, way(pk_ids2))
pk <- subset(ua3, ids = pk_ids2)

pkg_ids2 <- find(ua3, way(tags(v == "parking")))
pkg_ids2 <- find_down(ua3, way(pkg_ids2))
pkg <- subset(ua3, ids = pkg_ids2)

bg_poly2 <- as_sp(bg2, "polygons") #create sp class objects for ploygons for buildings
hw_poly2 <- as_sp(hw2, "lines") #create sp class objects for lit= highways
pg_poly2 <- as_sp(pg2, "polygon") #create sp class objects for polygons for playgrounds
pk_poly2 <- as_sp(pk, "polygon")
pkg_poly2 <- as_sp(pkg, "polygon")

bgUTM <-spTransform(bg_poly2,CRS("+proj=utm")) #transforms building polygon into universal transverse mercator - so we can calculate area
pgUTM <-spTransform(pg_poly2,CRS("+proj=utm"))  #transforms playground polygon into universal transverse mercator - so we can calculate area
pkUTM <- spTransform(pk_poly2,CRS("+proj=utm"))
pkgUTM <- spTransform(pkg_poly2,CRS("+proj=utm"))

# spplot(bg_poly2, c("user"),main ="Dunbar building")
time <- Sys.time() # get system time - today's date
nodes <- ua3$nodes # get all the nodes from ua3 osm data
ntags <- nodes$tags # get the tags from all the nodes
nattrs <- nodes$attrs # get the attributes from all the nodes
dn <- nattrs[order(nattrs$timestamp),] # tried to reorder the attrs by timestamp but in the end didn't use this

```
*This is a test webpage showing some community assets within 1km of Dunbar's town center as mapped by volunteers using OpenStreetMap*

*Report generated from live data: `r date() #print today's date`* 

**Landuse Area**

<p>Buildings: `r gArea(bgUTM) / 10000` hectares or approx `r round((gArea(bgUTM) / 1000)*0.62, digits=0)` football pitches<p>
<p>Playgrounds: `r gArea(pgUTM) / 10000` hectares or approx `r round((gArea(pgUTM) / 1000)*0.62, digits=0)` football pitches</p>
<p>Parks: `r gArea(pkUTM) / 10000` hectares or approx `r round((gArea(pkUTM) / 1000)*0.62, digits=0)` football pitches</p>
<p>Car parks: `r gArea(pkgUTM) / 10000` hectares or approx `r round((gArea(pkgUTM) / 1000)*0.62, digits=0)` football pitches</p>

**Transport assets**

```{r fig.width=7, fig.height=6, echo=FALSE}
barplot (c(sum(nodes$tags == "bus_stop"), sum(nodes$tags == "crossing"),
           sum(nodes$tags == "station"), sum(ntags$k == "bicycle_parking"),
         sum(nodes$tags == "car_sharing"), sum(nodes$tags == "charging_station"))
         , names.arg=c("bus stops", "crossings", "train stations", "cycle parking", "car share", "charging station"),
         main = "Dunbar transport assets", col = 'lightgreen', cex.names = 0.7) # barplot of various tags, had to do something different with bicycle_parking because it can be used as a key or a value
```
**Highways tagged with street lighting (lit=)**


```{r fig.width=7, fig.height=6, echo=FALSE}
plot(hw_poly2)
rect(par("usr")[1],par("usr")[3],par("usr")[2],par("usr")[4],col = "black")
lines(hw_poly2, col = 'yellow') # highway could be lit=yes or lit=no - could work on this further
```

```{r fig.width=7, fig.height=6, echo=FALSE}
plot(x = nattrs$timestamp, y = nattrs$user, col = c(dn$user), main = "number of contributors") # simple plot of contributors - needs a bit more work. 
```
```{r fig.width=7, fig.height=6, echo=FALSE}
spplot(bg_poly2, c("timestamp"), main ="Dunbar") #spplot (uses sp library) to plot spatial object - this one is buildings, coloured by timestamp
```
```{r fig.width=7, fig.height=6, echo=FALSE}
plot(nattrs$user, col = "blue", cex.names = 0.3, main = "number of nodes per contributor") #plot contributors - probably table could be better
```

