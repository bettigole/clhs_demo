This script is designed to walk you through the process of creating a raster stack to pass to the clhs() function, optimizing sample design based on landscape condition

In this example, four raster bands are used, but this could be more or less depending on your preferences

```{r}
library(rgdal)
library(raster)
library(tidyverse)
library(sp)
library(clhs)
```

Read in your boundary shapefile for the property
```{r}
bound = readOGR('./data/lock_07_bounds_p.shp')
fieldCRS <- proj4string(bound)
```

Read in four raster layers and crop to the boundary
```{r}
Aspect <- raster('./data/lock_07_aspect_p.tif')%>%crop(bound)
NDVI <- raster('./data/lock_07_ndvi_p.tif')%>%crop(bound)
Slope <- raster('./data/lock_07_slope_p.tif')%>%crop(bound)
```

Just in case, align/project your rasters
```{r}
r2resampled <- projectRaster(NDVI,Aspect,method = 'ngb')
r3resampled <- projectRaster(Slope,Aspect,method = 'ngb')
```

Stack and then crop these raster layers
```{r}
r.m<-stack(c(Aspect,r2resampled, r3resampled))
r.clip<-mask(r.m,bound)
```

Run the clhs function on your cropped raster data. Reduce iterations to speed processing
```{r}
clhs_run<-clhs(r.clip, size = 50 ,iter = 1000, progress = TRUE, simple = FALSE)
```

Transform results and write to your desired location
```{r}
clhs_carbon<-spTransform(clhs_run$sampled_data,fieldCRS)
plot(bound)
plot(clhs_carbon,add=T)
```

Save the shapefile to your desired location and plug it into your favorite sampling app!
```{r}
writeOGR(clhs_carbon, ".", "filename", 
           driver = "ESRI Shapefile",
         overwrite_layer = TRUE)
```
