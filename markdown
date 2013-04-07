
Dunbar Coummunity Map Data
----------------------------------------------------------------------------------------------
```{r echo=FALSE, results='hide', echo=FALSE, message=FALSE, comment=FALSE}
{library(osmar)
library(rgeos)
library(rgdal)
library(knitr)
 library(sp)}
```
```{r, echo=FALSE}
src <- osmsource_api()
bb2 <- center_bbox(-2.515716, 56.00225, 500, 500)
ua3 <- get_osm(bb2, source = src)

hw_ids2 <- find(ua3, way(tags(k == "lit")))
hw_ids2 <- find_down(ua3, way(hw_ids2))
hw2 <- subset(ua3, ids = hw_ids2)

bg_ids2 <- find(ua3, way(tags(k == "building")))
bg_ids2 <- find_down(ua3, way(bg_ids2))
bg2 <- subset(ua3, ids = bg_ids2)

pg_ids2 <- find(ua3, way(tags(v == "playground")))
pg_ids2 <- find_down(ua3, way(pg_ids2))
pg2 <- subset(ua3, ids = pg_ids2)

bg_poly2 <- as_sp(bg2, "polygons")
hw_poly2 <- as_sp(hw2, "lines")
pg_poly2 <- as_sp(pg2, "polygon")

bgUTM <-spTransform(bg_poly2,CRS("+proj=utm")) 
pgUTM <-spTransform(pg_poly2,CRS("+proj=utm")) 

# spplot(bg_poly2, c("user"),main ="Dunbar building")
time <- Sys.time()
nodes <- ua3$nodes
ntags <- nodes$tags
nattrs <- nodes$attrs
dn <- nattrs[order(nattrs$timestamp),] 

```
*This is a test webpage showing some community assets within 1km of Dunbar's town center as mapped by volunteers using OpenStreetMap*

*Report generated from live data: `r date()`*

**Landuse Area**

<p>Buildings: `r gArea(bgUTM) / 10000` hectares<p>
<p>Playgrounds: `r gArea(pgUTM) / 10000` hectares</p>

**Transport assets**

```{r fig.width=7, fig.height=6, echo=FALSE}
barplot (c(sum(nodes$tags == "bus_stop"), sum(nodes$tags == "crossing"),
           sum(nodes$tags == "station"), sum(ntags$k == "bicycle_parking"),
         sum(nodes$tags == "car_sharing"), sum(nodes$tags == "charging_station"))
         , names.arg=c("bus stops", "crossings", "train stations", "cycle parking", "car share", "charging station"),
         main = "Dunbar transport assets", col = 'lightgreen', cex.names = 0.7)
```
**Highways tagged with street lighting (lit=)**


```{r fig.width=7, fig.height=6, echo=FALSE}
plot(hw_poly2)
rect(par("usr")[1],par("usr")[3],par("usr")[2],par("usr")[4],col = "black")
lines(hw_poly2, col = 'yellow')
```

```{r fig.width=7, fig.height=6, echo=FALSE}
plot(x = nattrs$timestamp, y = nattrs$user, col = c(dn$user), main = "number of contributors")
```
```{r fig.width=7, fig.height=6, echo=FALSE}
spplot(bg_poly2, c("timestamp"), main ="Dunbar")
```

