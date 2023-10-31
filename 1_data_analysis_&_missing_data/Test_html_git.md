---
title: "test_html_git"
author: "Med"
date: "2023-10-31"
output:
  html_document: 
    number_sections: yes
    fig_caption: yes
    toc: true
    toc_float: 
      collapsed: true
    theme: cerulean
    highlight: kate
    toc_depth: 5
    keep_md: yes
    df_print: paged
---

# Load libraries


```r
# load libraries
library(tictoc)
library(tidyverse)
```

```
## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
## ✔ dplyr     1.1.2     ✔ readr     2.1.4
## ✔ forcats   1.0.0     ✔ stringr   1.5.0
## ✔ ggplot2   3.4.2     ✔ tibble    3.2.1
## ✔ lubridate 1.9.2     ✔ tidyr     1.3.0
## ✔ purrr     1.0.1     
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
```

```r
library(terra)
```

```
## terra 1.7.29
## 
## Attaching package: 'terra'
## 
## The following object is masked from 'package:tidyr':
## 
##     extract
## 
## The following objects are masked from 'package:tictoc':
## 
##     shift, size
```

```r
library(sf)
```

```
## Linking to GEOS 3.11.0, GDAL 3.5.3, PROJ 9.1.0; sf_use_s2() is TRUE
```

```r
library(RColorBrewer)
library(rts)
```

```
## Loading required package: xts
## Loading required package: zoo
## 
## Attaching package: 'zoo'
## 
## The following object is masked from 'package:terra':
## 
##     time<-
## 
## The following objects are masked from 'package:base':
## 
##     as.Date, as.Date.numeric
## 
## 
## ######################### Warning from 'xts' package ##########################
## #                                                                             #
## # The dplyr lag() function breaks how base R's lag() function is supposed to  #
## # work, which breaks lag(my_xts). Calls to lag(my_xts) that you type or       #
## # source() into this session won't work correctly.                            #
## #                                                                             #
## # Use stats::lag() to make sure you're not using dplyr::lag(), or you can add #
## # conflictRules('dplyr', exclude = 'lag') to your .Rprofile to stop           #
## # dplyr from breaking base R's lag() function.                                #
## #                                                                             #
## # Code in packages is not affected. It's protected by R's namespace mechanism #
## # Set `options(xts.warn_dplyr_breaks_lag = FALSE)` to suppress this warning.  #
## #                                                                             #
## ###############################################################################
## 
## Attaching package: 'xts'
## 
## The following objects are masked from 'package:dplyr':
## 
##     first, last
## 
## rts 1.1-8 (2022-09-16)
```

```r
library(doParallel)
```

```
## Loading required package: foreach
## 
## Attaching package: 'foreach'
## 
## The following objects are masked from 'package:purrr':
## 
##     accumulate, when
## 
## Loading required package: iterators
## Loading required package: parallel
```

```r
library(parallel)
library(foreach)
library(data.table)
```

```
## 
## Attaching package: 'data.table'
## 
## The following objects are masked from 'package:xts':
## 
##     first, last
## 
## The following object is masked from 'package:terra':
## 
##     shift
## 
## The following objects are masked from 'package:lubridate':
## 
##     hour, isoweek, mday, minute, month, quarter, second, wday, week,
##     yday, year
## 
## The following objects are masked from 'package:dplyr':
## 
##     between, first, last
## 
## The following object is masked from 'package:purrr':
## 
##     transpose
## 
## The following object is masked from 'package:tictoc':
## 
##     shift
```

```r
library(htmltools)
library(firebase)
```


# Functions


```r
#---- Function to save layers ----
saveLayers = function(dataRast, shapeFile, pathFile, strRemove,
                      x_min=xmin(dataRast), x_max=xmax(dataRast), 
                      y_min=ymin(dataRast), y_max=ymax(dataRast)){
  
  # Iterate all the layers
  for (i in 1:nlyr(dataRast)){
    print(i)
    dataRast.i <- dataRast[[i]]
    # Crop data
    dataRast.basin.i <- mask(dataRast.i, mask = shapeFile) %>%
      crop(ext(x_min, x_max, y_min, y_max))
    # Path of the file
    path <- gsub(strRemove, "", names(dataRast.basin.i)) %>%
      paste0(pathFile, ., ".tif")
    # Save file
    writeRaster(dataRast.basin.i, path, overwrite=TRUE)
    # Clear memory
    gc()
  }
}

#---- Function to rename layers ----
renameLayers <- function(dataRast, fileStart, prefix){
  # index of file name in the path
  id.date <- unlist(gregexpr(fileStart, sources(dataRast)[[1]])) + nchar(fileStart)
  # rename layers
  names(dataRast) <- 
    substr(sources(dataRast), id.date, (id.date+50)) %>%
    gsub(".tif", "_1", .) %>%
    as.Date("%Y_%m_%d") %>%
    format(., '%Y_%m') %>%
    paste0(prefix, .)
  
  return(dataRast)
}

#---- Function to plot ----
myPlot <- function(
  rast, 
  title=NULL, 
  sub_title=NULL, 
  theme=2, 
  # na_value=NULL, 
  max_cell=1e8,
  x_angle=0,
  b_size=16,
  v_unit=c(2, 2, 2, 2) # c(t, r, b, l)
){
  # Prepare missing data
  # if (!is.null(na_value)) {
  #   min.rast <- minmax(rast)[1]
  #   rast.na <- rast
  #   rast.na[is.na(rast.na)] <- min.rast - 1e3
  #   rast.na[rast.na >= min.rast] <- NA
  #   # levels(rast.na) <- data.frame(id=c(min.rast - 1e3), val=c('1'))
  #   rast.na <- rast.na %>% mask(mask = amaz.basin.shp)
  #   # rast.na[rast.na < min.rast] <- -1e-09
  #   p1 <- ggplot() +
  #     geom_spatraster(data = rast.na, maxcell = max_cell)
  # }else{
  #   p1 <- ggplot()
  # }
  p1 <- ggplot()
  p1 <- switch(
    theme,
    p1,
    p1 + theme_bw(base_size=b_size),
    p1 + theme_linedraw(base_size=b_size),
    p1 + theme_light(base_size=b_size),
    p1 + theme_minimal(base_size=b_size),
    p1 + theme_classic(base_size=b_size),
    p1 + theme_gray(base_size=b_size),
    p1 + theme_dark(base_size=b_size)
  )
  p1 <- p1  +
    geom_spatraster(data = rast, maxcell = max_cell) +
    geom_spatvector(data = amaz.basin.shp$geometry, fill = NA, color = "gray40") +
    scale_x_continuous(labels = function(x) format(x, scientific = T)) +
    scale_y_continuous(labels = function(x) format(x, scientific = T)) +
    ggtitle(label=title, subtitle=sub_title) +
    coord_sf(datum = pull_crs(rast)) +
    theme(
      axis.text.x = element_text(angle = x_angle)
      , plot.margin = unit(v_unit, "pt")
    )
  if (theme == 1) {p1 <- p1 + theme_void(base_size=b_size)}
  return(p1)
}
```

# Load data


```r
my.path <- "~/Documents/"
my.path <- "/Users/abid.med/Documents/Data_ScienceTech_Institute/00.Intership/Project/Code_official_data"
path.png <- paste0(my.path, "/png")
# path.csv <- paste0(my.path, "/csv")
path.data <- paste0(my.path, "/Amazon_new_data")
# load(paste0(my.path,"R_data/data-amaz_na3.RData"))
```


# Land Cover data

## Import data


```r
# list of files
amaz.landCover.list <- list.files(paste0(path.data,"/2. Land Cover/03. Working Data"),
                                  full.names=TRUE,
                                  pattern = ".tif$")
# Import data with "Terra"
landCover.rast <- rast(amaz.landCover.list)
landCover.rast
```

```
## class       : SpatRaster 
## dimensions  : 5860, 7806, 240  (nrow, ncol, nlyr)
## resolution  : 500, 500  (x, y)
## extent      : -2156811, 1746189, 1625314, 4555314  (xmin, xmax, ymin, ymax)
## coord. ref. : South_America_Albers_Equal_Area_Conic 
## sources     : landcover_working_2001_1.tif  
##               landcover_working_2001_10.tif  
##               landcover_working_2001_11.tif  
##               ... and 237 more source(s)
## names       : lulc_~_proj, lulc_~_proj, lulc_~_proj, lulc_~_proj, lulc_~_proj, lulc_~_proj, ... 
## min values  :           0,           0,           0,           0,           0,           0, ... 
## max values  :          10,          10,          10,          10,          10,          10, ...
```

## Rename and order layers


```r
# Rename layers
landCover.rast <- renameLayers(landCover.rast, 'landcover_working_', '')
# Order layers
ordered.names <- seq(as.Date("2001-1-1"), as.Date("2020-12-1"), by = "month") %>% 
  format(., '%Y_%m')
landCover.rast <- landCover.rast[[ordered.names]]
landCover.rast
```

```
## class       : SpatRaster 
## dimensions  : 5860, 7806, 240  (nrow, ncol, nlyr)
## resolution  : 500, 500  (x, y)
## extent      : -2156811, 1746189, 1625314, 4555314  (xmin, xmax, ymin, ymax)
## coord. ref. : South_America_Albers_Equal_Area_Conic 
## sources     : landcover_working_2001_1.tif  
##               landcover_working_2001_2.tif  
##               landcover_working_2001_3.tif  
##               ... and 237 more source(s)
## names       : 2001_01, 2001_02, 2001_03, 2001_04, 2001_05, 2001_06, ... 
## min values  :       0,       0,       0,       0,       0,       0, ... 
## max values  :      10,      10,      10,      10,      10,      10, ...
```

## Verification of the values


```r
# Verification of the values
landCover.minmax <- minmax(landCover.rast) %>% t() %>% as.data.frame()
landCover.minmax
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["min"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["max"],"name":[2],"type":["dbl"],"align":["right"]}],"data":[{"1":"0","2":"10","_rn_":"2001_01"},{"1":"0","2":"10","_rn_":"2001_02"},{"1":"0","2":"10","_rn_":"2001_03"},{"1":"0","2":"10","_rn_":"2001_04"},{"1":"0","2":"10","_rn_":"2001_05"},{"1":"0","2":"10","_rn_":"2001_06"},{"1":"0","2":"10","_rn_":"2001_07"},{"1":"0","2":"10","_rn_":"2001_08"},{"1":"0","2":"10","_rn_":"2001_09"},{"1":"0","2":"10","_rn_":"2001_10"},{"1":"0","2":"10","_rn_":"2001_11"},{"1":"0","2":"10","_rn_":"2001_12"},{"1":"0","2":"10","_rn_":"2002_01"},{"1":"0","2":"10","_rn_":"2002_02"},{"1":"0","2":"10","_rn_":"2002_03"},{"1":"0","2":"10","_rn_":"2002_04"},{"1":"0","2":"10","_rn_":"2002_05"},{"1":"0","2":"10","_rn_":"2002_06"},{"1":"0","2":"10","_rn_":"2002_07"},{"1":"0","2":"10","_rn_":"2002_08"},{"1":"0","2":"10","_rn_":"2002_09"},{"1":"0","2":"10","_rn_":"2002_10"},{"1":"0","2":"10","_rn_":"2002_11"},{"1":"0","2":"10","_rn_":"2002_12"},{"1":"0","2":"10","_rn_":"2003_01"},{"1":"0","2":"10","_rn_":"2003_02"},{"1":"0","2":"10","_rn_":"2003_03"},{"1":"0","2":"10","_rn_":"2003_04"},{"1":"0","2":"10","_rn_":"2003_05"},{"1":"0","2":"10","_rn_":"2003_06"},{"1":"0","2":"10","_rn_":"2003_07"},{"1":"0","2":"10","_rn_":"2003_08"},{"1":"0","2":"10","_rn_":"2003_09"},{"1":"0","2":"10","_rn_":"2003_10"},{"1":"0","2":"10","_rn_":"2003_11"},{"1":"0","2":"10","_rn_":"2003_12"},{"1":"0","2":"10","_rn_":"2004_01"},{"1":"0","2":"10","_rn_":"2004_02"},{"1":"0","2":"10","_rn_":"2004_03"},{"1":"0","2":"10","_rn_":"2004_04"},{"1":"0","2":"10","_rn_":"2004_05"},{"1":"0","2":"10","_rn_":"2004_06"},{"1":"0","2":"10","_rn_":"2004_07"},{"1":"0","2":"10","_rn_":"2004_08"},{"1":"0","2":"10","_rn_":"2004_09"},{"1":"0","2":"10","_rn_":"2004_10"},{"1":"0","2":"10","_rn_":"2004_11"},{"1":"0","2":"10","_rn_":"2004_12"},{"1":"0","2":"10","_rn_":"2005_01"},{"1":"0","2":"10","_rn_":"2005_02"},{"1":"0","2":"10","_rn_":"2005_03"},{"1":"0","2":"10","_rn_":"2005_04"},{"1":"0","2":"10","_rn_":"2005_05"},{"1":"0","2":"10","_rn_":"2005_06"},{"1":"0","2":"10","_rn_":"2005_07"},{"1":"0","2":"10","_rn_":"2005_08"},{"1":"0","2":"10","_rn_":"2005_09"},{"1":"0","2":"10","_rn_":"2005_10"},{"1":"0","2":"10","_rn_":"2005_11"},{"1":"0","2":"10","_rn_":"2005_12"},{"1":"0","2":"10","_rn_":"2006_01"},{"1":"0","2":"10","_rn_":"2006_02"},{"1":"0","2":"10","_rn_":"2006_03"},{"1":"0","2":"10","_rn_":"2006_04"},{"1":"0","2":"10","_rn_":"2006_05"},{"1":"0","2":"10","_rn_":"2006_06"},{"1":"0","2":"10","_rn_":"2006_07"},{"1":"0","2":"10","_rn_":"2006_08"},{"1":"0","2":"10","_rn_":"2006_09"},{"1":"0","2":"10","_rn_":"2006_10"},{"1":"0","2":"10","_rn_":"2006_11"},{"1":"0","2":"10","_rn_":"2006_12"},{"1":"0","2":"10","_rn_":"2007_01"},{"1":"0","2":"10","_rn_":"2007_02"},{"1":"0","2":"10","_rn_":"2007_03"},{"1":"0","2":"10","_rn_":"2007_04"},{"1":"0","2":"10","_rn_":"2007_05"},{"1":"0","2":"10","_rn_":"2007_06"},{"1":"0","2":"10","_rn_":"2007_07"},{"1":"0","2":"10","_rn_":"2007_08"},{"1":"0","2":"10","_rn_":"2007_09"},{"1":"0","2":"10","_rn_":"2007_10"},{"1":"0","2":"10","_rn_":"2007_11"},{"1":"0","2":"10","_rn_":"2007_12"},{"1":"0","2":"10","_rn_":"2008_01"},{"1":"0","2":"10","_rn_":"2008_02"},{"1":"0","2":"10","_rn_":"2008_03"},{"1":"0","2":"10","_rn_":"2008_04"},{"1":"0","2":"10","_rn_":"2008_05"},{"1":"0","2":"10","_rn_":"2008_06"},{"1":"0","2":"10","_rn_":"2008_07"},{"1":"0","2":"10","_rn_":"2008_08"},{"1":"0","2":"10","_rn_":"2008_09"},{"1":"0","2":"10","_rn_":"2008_10"},{"1":"0","2":"10","_rn_":"2008_11"},{"1":"0","2":"10","_rn_":"2008_12"},{"1":"0","2":"10","_rn_":"2009_01"},{"1":"0","2":"10","_rn_":"2009_02"},{"1":"0","2":"10","_rn_":"2009_03"},{"1":"0","2":"10","_rn_":"2009_04"},{"1":"0","2":"10","_rn_":"2009_05"},{"1":"0","2":"10","_rn_":"2009_06"},{"1":"0","2":"10","_rn_":"2009_07"},{"1":"0","2":"10","_rn_":"2009_08"},{"1":"0","2":"10","_rn_":"2009_09"},{"1":"0","2":"10","_rn_":"2009_10"},{"1":"0","2":"10","_rn_":"2009_11"},{"1":"0","2":"10","_rn_":"2009_12"},{"1":"0","2":"10","_rn_":"2010_01"},{"1":"0","2":"10","_rn_":"2010_02"},{"1":"0","2":"10","_rn_":"2010_03"},{"1":"0","2":"10","_rn_":"2010_04"},{"1":"0","2":"10","_rn_":"2010_05"},{"1":"0","2":"10","_rn_":"2010_06"},{"1":"0","2":"10","_rn_":"2010_07"},{"1":"0","2":"10","_rn_":"2010_08"},{"1":"0","2":"10","_rn_":"2010_09"},{"1":"0","2":"10","_rn_":"2010_10"},{"1":"0","2":"10","_rn_":"2010_11"},{"1":"0","2":"10","_rn_":"2010_12"},{"1":"0","2":"10","_rn_":"2011_01"},{"1":"0","2":"10","_rn_":"2011_02"},{"1":"0","2":"10","_rn_":"2011_03"},{"1":"0","2":"10","_rn_":"2011_04"},{"1":"0","2":"10","_rn_":"2011_05"},{"1":"0","2":"10","_rn_":"2011_06"},{"1":"0","2":"10","_rn_":"2011_07"},{"1":"0","2":"10","_rn_":"2011_08"},{"1":"0","2":"10","_rn_":"2011_09"},{"1":"0","2":"10","_rn_":"2011_10"},{"1":"0","2":"10","_rn_":"2011_11"},{"1":"0","2":"10","_rn_":"2011_12"},{"1":"0","2":"10","_rn_":"2012_01"},{"1":"0","2":"10","_rn_":"2012_02"},{"1":"0","2":"10","_rn_":"2012_03"},{"1":"0","2":"10","_rn_":"2012_04"},{"1":"0","2":"10","_rn_":"2012_05"},{"1":"0","2":"10","_rn_":"2012_06"},{"1":"0","2":"10","_rn_":"2012_07"},{"1":"0","2":"10","_rn_":"2012_08"},{"1":"0","2":"10","_rn_":"2012_09"},{"1":"0","2":"10","_rn_":"2012_10"},{"1":"0","2":"10","_rn_":"2012_11"},{"1":"0","2":"10","_rn_":"2012_12"},{"1":"0","2":"10","_rn_":"2013_01"},{"1":"0","2":"10","_rn_":"2013_02"},{"1":"0","2":"10","_rn_":"2013_03"},{"1":"0","2":"10","_rn_":"2013_04"},{"1":"0","2":"10","_rn_":"2013_05"},{"1":"0","2":"10","_rn_":"2013_06"},{"1":"0","2":"10","_rn_":"2013_07"},{"1":"0","2":"10","_rn_":"2013_08"},{"1":"0","2":"10","_rn_":"2013_09"},{"1":"0","2":"10","_rn_":"2013_10"},{"1":"0","2":"10","_rn_":"2013_11"},{"1":"0","2":"10","_rn_":"2013_12"},{"1":"0","2":"10","_rn_":"2014_01"},{"1":"0","2":"10","_rn_":"2014_02"},{"1":"0","2":"10","_rn_":"2014_03"},{"1":"0","2":"10","_rn_":"2014_04"},{"1":"0","2":"10","_rn_":"2014_05"},{"1":"0","2":"10","_rn_":"2014_06"},{"1":"0","2":"10","_rn_":"2014_07"},{"1":"0","2":"10","_rn_":"2014_08"},{"1":"0","2":"10","_rn_":"2014_09"},{"1":"0","2":"10","_rn_":"2014_10"},{"1":"0","2":"10","_rn_":"2014_11"},{"1":"0","2":"10","_rn_":"2014_12"},{"1":"0","2":"10","_rn_":"2015_01"},{"1":"0","2":"10","_rn_":"2015_02"},{"1":"0","2":"10","_rn_":"2015_03"},{"1":"0","2":"10","_rn_":"2015_04"},{"1":"0","2":"10","_rn_":"2015_05"},{"1":"0","2":"10","_rn_":"2015_06"},{"1":"0","2":"10","_rn_":"2015_07"},{"1":"0","2":"10","_rn_":"2015_08"},{"1":"0","2":"10","_rn_":"2015_09"},{"1":"0","2":"10","_rn_":"2015_10"},{"1":"0","2":"10","_rn_":"2015_11"},{"1":"0","2":"10","_rn_":"2015_12"},{"1":"0","2":"10","_rn_":"2016_01"},{"1":"0","2":"10","_rn_":"2016_02"},{"1":"0","2":"10","_rn_":"2016_03"},{"1":"0","2":"10","_rn_":"2016_04"},{"1":"0","2":"10","_rn_":"2016_05"},{"1":"0","2":"10","_rn_":"2016_06"},{"1":"0","2":"10","_rn_":"2016_07"},{"1":"0","2":"10","_rn_":"2016_08"},{"1":"0","2":"10","_rn_":"2016_09"},{"1":"0","2":"10","_rn_":"2016_10"},{"1":"0","2":"10","_rn_":"2016_11"},{"1":"0","2":"10","_rn_":"2016_12"},{"1":"0","2":"10","_rn_":"2017_01"},{"1":"0","2":"10","_rn_":"2017_02"},{"1":"0","2":"10","_rn_":"2017_03"},{"1":"0","2":"10","_rn_":"2017_04"},{"1":"0","2":"10","_rn_":"2017_05"},{"1":"0","2":"10","_rn_":"2017_06"},{"1":"0","2":"10","_rn_":"2017_07"},{"1":"0","2":"10","_rn_":"2017_08"},{"1":"0","2":"10","_rn_":"2017_09"},{"1":"0","2":"10","_rn_":"2017_10"},{"1":"0","2":"10","_rn_":"2017_11"},{"1":"0","2":"10","_rn_":"2017_12"},{"1":"0","2":"10","_rn_":"2018_01"},{"1":"0","2":"10","_rn_":"2018_02"},{"1":"0","2":"10","_rn_":"2018_03"},{"1":"0","2":"10","_rn_":"2018_04"},{"1":"0","2":"10","_rn_":"2018_05"},{"1":"0","2":"10","_rn_":"2018_06"},{"1":"0","2":"10","_rn_":"2018_07"},{"1":"0","2":"10","_rn_":"2018_08"},{"1":"0","2":"10","_rn_":"2018_09"},{"1":"0","2":"10","_rn_":"2018_10"},{"1":"0","2":"10","_rn_":"2018_11"},{"1":"0","2":"10","_rn_":"2018_12"},{"1":"0","2":"10","_rn_":"2019_01"},{"1":"0","2":"10","_rn_":"2019_02"},{"1":"0","2":"10","_rn_":"2019_03"},{"1":"0","2":"10","_rn_":"2019_04"},{"1":"0","2":"10","_rn_":"2019_05"},{"1":"0","2":"10","_rn_":"2019_06"},{"1":"0","2":"10","_rn_":"2019_07"},{"1":"0","2":"10","_rn_":"2019_08"},{"1":"0","2":"10","_rn_":"2019_09"},{"1":"0","2":"10","_rn_":"2019_10"},{"1":"0","2":"10","_rn_":"2019_11"},{"1":"0","2":"10","_rn_":"2019_12"},{"1":"0","2":"10","_rn_":"2020_01"},{"1":"0","2":"10","_rn_":"2020_02"},{"1":"0","2":"10","_rn_":"2020_03"},{"1":"0","2":"10","_rn_":"2020_04"},{"1":"0","2":"10","_rn_":"2020_05"},{"1":"0","2":"10","_rn_":"2020_06"},{"1":"0","2":"10","_rn_":"2020_07"},{"1":"0","2":"10","_rn_":"2020_08"},{"1":"0","2":"10","_rn_":"2020_09"},{"1":"0","2":"10","_rn_":"2020_10"},{"1":"0","2":"10","_rn_":"2020_11"},{"1":"0","2":"10","_rn_":"2020_12"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>





