# 1. Data Analysis and Missing Data

The dataset includes 10 variables that capture various factors related to fires, land use, environmental conditions, and climate. It provides a spatial resolution of 500 meters, allowing a detailed analysis of the Amazon rainforest. These variables are measured on a monthly basis, covering the entire period from 2001 to 2020, and each variable contains a substantial monthly dataset, with around 26.8 million observations per month.

Before looking for each variable, we import the Amazon shape file:
```r
# Import shape file
amaz.basin.shp <- st_read(paste0(path.data,"/0. Amazon_shapefile/projected/amazon_shp_projected.shp"))
```
```
    Simple feature collection with 1 feature and 6 fields
    Geometry type: MULTIPOLYGON
    Dimension:     XY
    Bounding box:  xmin: -2156811 ymin: 1625314 xmax: 1745999 ymax: 4555427
    Projected CRS: South_America_Albers_Equal_Area_Conic
    ID     AREA PERIMETER FORMA POLYAREA     AREAPROJ                       geometry
    1  0 548.0098   241.442  <NA>  6725344 6.725344e+12 MULTIPOLYGON (((-2028457 35...
```

Now, let's take a closer look at each variable:

## 1.1. Burnt Area

### Data Analysis

_Burnt Area_ Represents the extent of burned areas in the Amazon rainforest, categorized as burnt (1), unburnt (0), missing (-1), or water (-2).

_**Import data**_

```r
# list of files
amaz.burntArea.list <- list.files(paste0(path.data,"/1. Burnt Area/03. Working Data"),
                                  full.names=TRUE,
                                  pattern = ".tif$")
# Import data with `Terra`
burntArea.rast <- rast(amaz.burntArea.list)
burntArea.rast
```
```
    class       : SpatRaster 
    dimensions  : 5860, 7806, 238  (nrow, ncol, nlyr)
    resolution  : 500, 500  (x, y)
    extent      : -2156811, 1746189, 1625314, 4555314  (xmin, xmax, ymin, ymax)
    coord. ref. : South_America_Albers_Equal_Area_Conic 
    sources     : burntarea_working_2001_1.tif  
                burntarea_working_2001_10.tif  
                burntarea_working_2001_11.tif  
                ... and 235 more source(s)
    names       : fire_~_proj, fire_~_proj, fire_~_proj, fire_~_proj, fire_~_proj, fire_~_proj, ... 
    min values  :          -2,          -2,          -2,          -2,          -2,          -2, ... 
    max values  :           1,           1,           1,           1,           1,           1, ... 
```

_**Rename layers**_

```r
# Rename layers
burntArea.rast <- renameLayers(burntArea.rast, 'burntarea_working_', '')
burntArea.rast
```
```
    class       : SpatRaster 
    dimensions  : 5860, 7806, 240  (nrow, ncol, nlyr)
    resolution  : 500, 500  (x, y)
    extent      : -2156811, 1746189, 1625314, 4555314  (xmin, xmax, ymin, ymax)
    coord. ref. : South_America_Albers_Equal_Area_Conic 
    sources     : burntarea_working_2001_1.tif  
                burntarea_working_2001_10.tif  
                burntarea_working_2001_11.tif  
                ... and 237 more source(s)
    names       : 2001_01, 2001_10, 2001_11, 2001_12, 2001_02, 2001_03, ... 
    min values  :      -2,      -2,      -2,      -2,      -2,      -2, ... 
    max values  :       1,       1,       1,       1,       1,       1, ... 
```

_**Order layers**_

```r
# Order layers
ordered.names <- seq(as.Date("2001-1-1"), as.Date("2020-12-1"), by = "month") %>% format(., '%Y_%m')
burntArea.rast <- burntArea.rast[[ordered.names]]
burntArea.rast
```
```
    class       : SpatRaster 
    dimensions  : 5860, 7806, 240  (nrow, ncol, nlyr)
    resolution  : 500, 500  (x, y)
    extent      : -2156811, 1746189, 1625314, 4555314  (xmin, xmax, ymin, ymax)
    coord. ref. : South_America_Albers_Equal_Area_Conic 
    sources     : burntarea_working_2001_1.tif  
                burntarea_working_2001_2.tif  
                burntarea_working_2001_3.tif  
                ... and 237 more source(s)
    names       : 2001_01, 2001_02, 2001_03, 2001_04, 2001_05, 2001_06, ... 
    min values  :      -2,      -2,      -2,      -2,      -2,      -2, ... 
    max values  :       1,       1,       1,       1,       1,       1, ... 
```

_**Verification of the values**_

```r
# Verification of the values
burntArea.minmax <- minmax(burntArea.rast) %>% t() %>% as.data.frame()
burntArea.minmax[which((burntArea.minmax[,1] != -2) & (burntArea.minmax[,2] != 1)),]
```

<p align="center">
  <img src="img/1.1.BurntArea-Verification of the values.png"  width="60%" />
</p>


_**Create Raster Time Series (`rts`) object**_

```r
# Create a sequence date
seq.dates <- seq(as.Date("2001-1-1"), as.Date("2020-12-1"), by = "month")
# Create the `rts` object
burntArea.rts <- rts(burntArea.rast, seq.dates)
```

_**Plot of the month of October 2020**_

```r
# Upplaying the mask to plot only the amazon area.
ba <- burntArea.rts[['2020-10-01']] %>% mask(mask = amaz.basin.shp)
# Change values as categorical 
levels(ba) <- data.frame(id=c(-2, 0, 1), val=c('-2', '0', '1'))
# Plot
my.colors <- c("mediumblue", "mediumseagreen", "firebrick")
p.ba <- myPlot(ba, title = "Burnt Area") +
  scale_fill_manual(
    name = NULL, 
    values = my.colors, 
    na.translate=FALSE
  ) 
p.ba
```

<p align="center">
  <img src="img/1.2.ba_2.png"  width="60%" />
</p>

```r
# Convert the raster object to a datatable
ba.dt <- as.data.table(ba, cell=T, xy=T)
# Plot
ggplot(data = ba.dt, aes(x = val)) + 
  geom_bar(stat = "count", aes(fill = val), position = "dodge") + 
  labs(title="Burnt Area in October 2020", x="Burnt Area") +
  coord_flip() + 
  scale_fill_manual(name = "Burnt Area", values = my.colors) +
  stat_count(geom = "text", 
             aes(label = ..count..),
             position=position_stack(vjust=0.5),
             colour = "black", size = 3.5) + 
  theme_bw(base_size=16)
```

<p align="center">
  <img src="img/1.3.ba.png"  width="60%" />
</p>

_**Percentage of fires**_

```r
freq.dt <- matrix(nrow = 0, ncol = 3) %>% as.data.table()
colnames(freq.dt) <- c("layer", "0", "1")
for (ras_id in amaz.burntArea.list){
  cat("\n", ras_id)
  ras <- rast(ras_id) %>%
    renameLayers(., 'burntarea_working_', '') %>%
    mask(mask = amaz.basin.shp)
 
  # Replace -2 and -1 value by `NA`
  ras[ras %in% c(-2, -1)] <- NA
  ras.freq <- freq(ras, digits=0, usenames=T) %>% as.data.table()
  tmp <- dcast(ras.freq,layer ~ value,value.var = c("count"))
  freq.dt <- rbind(freq.dt, tmp)
}
percentage.fires <- sum(freq.dt[, '1']) / sum(freq.dt[, c('0', '1')])
percentage.fires
```
```
    [1] 0.00089637
```

### Missing Data

```r
cl <- makeCluster(detectCores() - 1)
registerDoParallel(cl, cores=detectCores() - 1)

# Raster to datatable in parallel: one raster per thread
rasList <- foreach (ras_id=amaz.burntArea.list, .packages=c('terra', 'sf'), .combine='c') %dopar% {
    # Read all rasters into one big stack
    ras <- rast(ras_id)
    # Rename layers
    ras <- renameLayers(ras, 'burntarea_working_', '')
    # Replace negative value by `NA`
    if (names(ras) %in% c("2012_07", "2012_09")) {ras[ras == -1] <- NA}
    # Count the missing data
    ras.nonNA <- not.na(ras)
    ras.nonNA.mask <- mask(ras.nonNA, amaz.basin.shp)
    ras.freq.na <- terra::freq(ras.nonNA.mask, digits=0, value=0, usenames=T)
  
    list(ras.freq.na)
  }
stopCluster(cl)

# Bind all per-raster into one dataframe
burntArea.freq.na <- rbindlist(rasList, fill=T, use.names=T)
#
colnames(burntArea.freq.na)[3] <- "burntArea_na"
burntArea.freq.na <- burntArea.freq.na[order(burntArea.freq.na$layer)]
burntArea.freq.na
```
<p align="center">
  <img src="img/1.4.ba.na2.png"  width="49.5%" />
  <img src="img/1.4.ba.na3.png"  width="49.5%" />
</p>

## Land Cover

_Land Cover_ is a categorical variable with 11 classes, providing information on different land cover types such as water, urban, forest, grassland, and more.


<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["min"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["max"],"name":[2],"type":["dbl"],"align":["right"]}],"data":[{"1":"0","2":"10","_rn_":"2001_01"},{"1":"0","2":"10","_rn_":"2001_02"},{"1":"0","2":"10","_rn_":"2001_03"},{"1":"0","2":"10","_rn_":"2001_04"},{"1":"0","2":"10","_rn_":"2001_05"},{"1":"0","2":"10","_rn_":"2001_06"},{"1":"0","2":"10","_rn_":"2001_07"},{"1":"0","2":"10","_rn_":"2001_08"},{"1":"0","2":"10","_rn_":"2001_09"},{"1":"0","2":"10","_rn_":"2001_10"},{"1":"0","2":"10","_rn_":"2001_11"},{"1":"0","2":"10","_rn_":"2001_12"},{"1":"0","2":"10","_rn_":"2002_01"},{"1":"0","2":"10","_rn_":"2002_02"},{"1":"0","2":"10","_rn_":"2002_03"},{"1":"0","2":"10","_rn_":"2002_04"},{"1":"0","2":"10","_rn_":"2002_05"},{"1":"0","2":"10","_rn_":"2002_06"},{"1":"0","2":"10","_rn_":"2002_07"},{"1":"0","2":"10","_rn_":"2002_08"},{"1":"0","2":"10","_rn_":"2002_09"},{"1":"0","2":"10","_rn_":"2002_10"},{"1":"0","2":"10","_rn_":"2002_11"},{"1":"0","2":"10","_rn_":"2002_12"},{"1":"0","2":"10","_rn_":"2003_01"},{"1":"0","2":"10","_rn_":"2003_02"},{"1":"0","2":"10","_rn_":"2003_03"},{"1":"0","2":"10","_rn_":"2003_04"},{"1":"0","2":"10","_rn_":"2003_05"},{"1":"0","2":"10","_rn_":"2003_06"},{"1":"0","2":"10","_rn_":"2003_07"},{"1":"0","2":"10","_rn_":"2003_08"},{"1":"0","2":"10","_rn_":"2003_09"},{"1":"0","2":"10","_rn_":"2003_10"},{"1":"0","2":"10","_rn_":"2003_11"},{"1":"0","2":"10","_rn_":"2003_12"},{"1":"0","2":"10","_rn_":"2004_01"},{"1":"0","2":"10","_rn_":"2004_02"},{"1":"0","2":"10","_rn_":"2004_03"},{"1":"0","2":"10","_rn_":"2004_04"},{"1":"0","2":"10","_rn_":"2004_05"},{"1":"0","2":"10","_rn_":"2004_06"},{"1":"0","2":"10","_rn_":"2004_07"},{"1":"0","2":"10","_rn_":"2004_08"},{"1":"0","2":"10","_rn_":"2004_09"},{"1":"0","2":"10","_rn_":"2004_10"},{"1":"0","2":"10","_rn_":"2004_11"},{"1":"0","2":"10","_rn_":"2004_12"},{"1":"0","2":"10","_rn_":"2005_01"},{"1":"0","2":"10","_rn_":"2005_02"},{"1":"0","2":"10","_rn_":"2005_03"},{"1":"0","2":"10","_rn_":"2005_04"},{"1":"0","2":"10","_rn_":"2005_05"},{"1":"0","2":"10","_rn_":"2005_06"},{"1":"0","2":"10","_rn_":"2005_07"},{"1":"0","2":"10","_rn_":"2005_08"},{"1":"0","2":"10","_rn_":"2005_09"},{"1":"0","2":"10","_rn_":"2005_10"},{"1":"0","2":"10","_rn_":"2005_11"},{"1":"0","2":"10","_rn_":"2005_12"},{"1":"0","2":"10","_rn_":"2006_01"},{"1":"0","2":"10","_rn_":"2006_02"},{"1":"0","2":"10","_rn_":"2006_03"},{"1":"0","2":"10","_rn_":"2006_04"},{"1":"0","2":"10","_rn_":"2006_05"},{"1":"0","2":"10","_rn_":"2006_06"},{"1":"0","2":"10","_rn_":"2006_07"},{"1":"0","2":"10","_rn_":"2006_08"},{"1":"0","2":"10","_rn_":"2006_09"},{"1":"0","2":"10","_rn_":"2006_10"},{"1":"0","2":"10","_rn_":"2006_11"},{"1":"0","2":"10","_rn_":"2006_12"},{"1":"0","2":"10","_rn_":"2007_01"},{"1":"0","2":"10","_rn_":"2007_02"},{"1":"0","2":"10","_rn_":"2007_03"},{"1":"0","2":"10","_rn_":"2007_04"},{"1":"0","2":"10","_rn_":"2007_05"},{"1":"0","2":"10","_rn_":"2007_06"},{"1":"0","2":"10","_rn_":"2007_07"},{"1":"0","2":"10","_rn_":"2007_08"},{"1":"0","2":"10","_rn_":"2007_09"},{"1":"0","2":"10","_rn_":"2007_10"},{"1":"0","2":"10","_rn_":"2007_11"},{"1":"0","2":"10","_rn_":"2007_12"},{"1":"0","2":"10","_rn_":"2008_01"},{"1":"0","2":"10","_rn_":"2008_02"},{"1":"0","2":"10","_rn_":"2008_03"},{"1":"0","2":"10","_rn_":"2008_04"},{"1":"0","2":"10","_rn_":"2008_05"},{"1":"0","2":"10","_rn_":"2008_06"},{"1":"0","2":"10","_rn_":"2008_07"},{"1":"0","2":"10","_rn_":"2008_08"},{"1":"0","2":"10","_rn_":"2008_09"},{"1":"0","2":"10","_rn_":"2008_10"},{"1":"0","2":"10","_rn_":"2008_11"},{"1":"0","2":"10","_rn_":"2008_12"},{"1":"0","2":"10","_rn_":"2009_01"},{"1":"0","2":"10","_rn_":"2009_02"},{"1":"0","2":"10","_rn_":"2009_03"},{"1":"0","2":"10","_rn_":"2009_04"},{"1":"0","2":"10","_rn_":"2009_05"},{"1":"0","2":"10","_rn_":"2009_06"},{"1":"0","2":"10","_rn_":"2009_07"},{"1":"0","2":"10","_rn_":"2009_08"},{"1":"0","2":"10","_rn_":"2009_09"},{"1":"0","2":"10","_rn_":"2009_10"},{"1":"0","2":"10","_rn_":"2009_11"},{"1":"0","2":"10","_rn_":"2009_12"},{"1":"0","2":"10","_rn_":"2010_01"},{"1":"0","2":"10","_rn_":"2010_02"},{"1":"0","2":"10","_rn_":"2010_03"},{"1":"0","2":"10","_rn_":"2010_04"},{"1":"0","2":"10","_rn_":"2010_05"},{"1":"0","2":"10","_rn_":"2010_06"},{"1":"0","2":"10","_rn_":"2010_07"},{"1":"0","2":"10","_rn_":"2010_08"},{"1":"0","2":"10","_rn_":"2010_09"},{"1":"0","2":"10","_rn_":"2010_10"},{"1":"0","2":"10","_rn_":"2010_11"},{"1":"0","2":"10","_rn_":"2010_12"},{"1":"0","2":"10","_rn_":"2011_01"},{"1":"0","2":"10","_rn_":"2011_02"},{"1":"0","2":"10","_rn_":"2011_03"},{"1":"0","2":"10","_rn_":"2011_04"},{"1":"0","2":"10","_rn_":"2011_05"},{"1":"0","2":"10","_rn_":"2011_06"},{"1":"0","2":"10","_rn_":"2011_07"},{"1":"0","2":"10","_rn_":"2011_08"},{"1":"0","2":"10","_rn_":"2011_09"},{"1":"0","2":"10","_rn_":"2011_10"},{"1":"0","2":"10","_rn_":"2011_11"},{"1":"0","2":"10","_rn_":"2011_12"},{"1":"0","2":"10","_rn_":"2012_01"},{"1":"0","2":"10","_rn_":"2012_02"},{"1":"0","2":"10","_rn_":"2012_03"},{"1":"0","2":"10","_rn_":"2012_04"},{"1":"0","2":"10","_rn_":"2012_05"},{"1":"0","2":"10","_rn_":"2012_06"},{"1":"0","2":"10","_rn_":"2012_07"},{"1":"0","2":"10","_rn_":"2012_08"},{"1":"0","2":"10","_rn_":"2012_09"},{"1":"0","2":"10","_rn_":"2012_10"},{"1":"0","2":"10","_rn_":"2012_11"},{"1":"0","2":"10","_rn_":"2012_12"},{"1":"0","2":"10","_rn_":"2013_01"},{"1":"0","2":"10","_rn_":"2013_02"},{"1":"0","2":"10","_rn_":"2013_03"},{"1":"0","2":"10","_rn_":"2013_04"},{"1":"0","2":"10","_rn_":"2013_05"},{"1":"0","2":"10","_rn_":"2013_06"},{"1":"0","2":"10","_rn_":"2013_07"},{"1":"0","2":"10","_rn_":"2013_08"},{"1":"0","2":"10","_rn_":"2013_09"},{"1":"0","2":"10","_rn_":"2013_10"},{"1":"0","2":"10","_rn_":"2013_11"},{"1":"0","2":"10","_rn_":"2013_12"},{"1":"0","2":"10","_rn_":"2014_01"},{"1":"0","2":"10","_rn_":"2014_02"},{"1":"0","2":"10","_rn_":"2014_03"},{"1":"0","2":"10","_rn_":"2014_04"},{"1":"0","2":"10","_rn_":"2014_05"},{"1":"0","2":"10","_rn_":"2014_06"},{"1":"0","2":"10","_rn_":"2014_07"},{"1":"0","2":"10","_rn_":"2014_08"},{"1":"0","2":"10","_rn_":"2014_09"},{"1":"0","2":"10","_rn_":"2014_10"},{"1":"0","2":"10","_rn_":"2014_11"},{"1":"0","2":"10","_rn_":"2014_12"},{"1":"0","2":"10","_rn_":"2015_01"},{"1":"0","2":"10","_rn_":"2015_02"},{"1":"0","2":"10","_rn_":"2015_03"},{"1":"0","2":"10","_rn_":"2015_04"},{"1":"0","2":"10","_rn_":"2015_05"},{"1":"0","2":"10","_rn_":"2015_06"},{"1":"0","2":"10","_rn_":"2015_07"},{"1":"0","2":"10","_rn_":"2015_08"},{"1":"0","2":"10","_rn_":"2015_09"},{"1":"0","2":"10","_rn_":"2015_10"},{"1":"0","2":"10","_rn_":"2015_11"},{"1":"0","2":"10","_rn_":"2015_12"},{"1":"0","2":"10","_rn_":"2016_01"},{"1":"0","2":"10","_rn_":"2016_02"},{"1":"0","2":"10","_rn_":"2016_03"},{"1":"0","2":"10","_rn_":"2016_04"},{"1":"0","2":"10","_rn_":"2016_05"},{"1":"0","2":"10","_rn_":"2016_06"},{"1":"0","2":"10","_rn_":"2016_07"},{"1":"0","2":"10","_rn_":"2016_08"},{"1":"0","2":"10","_rn_":"2016_09"},{"1":"0","2":"10","_rn_":"2016_10"},{"1":"0","2":"10","_rn_":"2016_11"},{"1":"0","2":"10","_rn_":"2016_12"},{"1":"0","2":"10","_rn_":"2017_01"},{"1":"0","2":"10","_rn_":"2017_02"},{"1":"0","2":"10","_rn_":"2017_03"},{"1":"0","2":"10","_rn_":"2017_04"},{"1":"0","2":"10","_rn_":"2017_05"},{"1":"0","2":"10","_rn_":"2017_06"},{"1":"0","2":"10","_rn_":"2017_07"},{"1":"0","2":"10","_rn_":"2017_08"},{"1":"0","2":"10","_rn_":"2017_09"},{"1":"0","2":"10","_rn_":"2017_10"},{"1":"0","2":"10","_rn_":"2017_11"},{"1":"0","2":"10","_rn_":"2017_12"},{"1":"0","2":"10","_rn_":"2018_01"},{"1":"0","2":"10","_rn_":"2018_02"},{"1":"0","2":"10","_rn_":"2018_03"},{"1":"0","2":"10","_rn_":"2018_04"},{"1":"0","2":"10","_rn_":"2018_05"},{"1":"0","2":"10","_rn_":"2018_06"},{"1":"0","2":"10","_rn_":"2018_07"},{"1":"0","2":"10","_rn_":"2018_08"},{"1":"0","2":"10","_rn_":"2018_09"},{"1":"0","2":"10","_rn_":"2018_10"},{"1":"0","2":"10","_rn_":"2018_11"},{"1":"0","2":"10","_rn_":"2018_12"},{"1":"0","2":"10","_rn_":"2019_01"},{"1":"0","2":"10","_rn_":"2019_02"},{"1":"0","2":"10","_rn_":"2019_03"},{"1":"0","2":"10","_rn_":"2019_04"},{"1":"0","2":"10","_rn_":"2019_05"},{"1":"0","2":"10","_rn_":"2019_06"},{"1":"0","2":"10","_rn_":"2019_07"},{"1":"0","2":"10","_rn_":"2019_08"},{"1":"0","2":"10","_rn_":"2019_09"},{"1":"0","2":"10","_rn_":"2019_10"},{"1":"0","2":"10","_rn_":"2019_11"},{"1":"0","2":"10","_rn_":"2019_12"},{"1":"0","2":"10","_rn_":"2020_01"},{"1":"0","2":"10","_rn_":"2020_02"},{"1":"0","2":"10","_rn_":"2020_03"},{"1":"0","2":"10","_rn_":"2020_04"},{"1":"0","2":"10","_rn_":"2020_05"},{"1":"0","2":"10","_rn_":"2020_06"},{"1":"0","2":"10","_rn_":"2020_07"},{"1":"0","2":"10","_rn_":"2020_08"},{"1":"0","2":"10","_rn_":"2020_09"},{"1":"0","2":"10","_rn_":"2020_10"},{"1":"0","2":"10","_rn_":"2020_11"},{"1":"0","2":"10","_rn_":"2020_12"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>

## Precipitation

_Precipitation_ is measured in millimeters per hour, with a range between 0 and 3300.

## Soil Moisture

_Soil Moisture_ is measured in millimeters, with missing values marked as -9.99e+08, and a range between 0 and 4291.

## Elevation

_Elevation_ is measured in meters, with a range between -85 and 6471.

## Land Surface Temperature

_Land Surface Temperature_ is represented in Kelvin, with values adjusted by a scale factor of 0.02. Different months have varying missing data.

## Specific Humidity

_Specific Humidity_ is represented as kg/kg, indicating the ratio of kilograms of water (moisture) per kilogram of air. It ranges from 9.59e-04 to 2.15e-02.

## Evapotranspiration

_Evapotranspiration_ is measured in kg/m2s, with values ranging between -2.02e-07 and 9.69e-05.

## Wind Speed

_Wind Speed_ is measured in m/s, with values between 0.86 and 9.85.

## Air Temperature

_Air Temperature_ is represented in Kelvin, with values ranging from 268 to 307.






# Handling Missing Data and Data Preparation

## Missing Data
Dealing with missing data is a crucial part of our project. In the context of the "Burnt Area" variable, cells with a value of -2 represent water. However, the status of these cells can change over time to indicate "No fire" (0) or "Fire" (1) in different months. In our analysis, we treat cells with a value of -2 as missing data.

For the covariate "Land Surface Temperature," missing data varies across months. We observed that 26 months had more than 1 million missing data points, with three months exceeding 2 million missing data points. February 2016 had the highest with 3.3 million missing data points. To visually understand these missing data patterns, you can refer to the plots for the months with the most significant gaps.

In contrast, other covariates like "Precipitation," "Soil Moisture," "Specific Humidity," "Evapotranspiration," "Wind Speed," and "Air Temperature" have consistent missing data patterns across all months. These covariates have missing data values mostly at the boundary of the map, and each month's missing data count is below 133,000. We decided to exclude cells with missing data in these covariates as they consistently lacked data throughout the entire study period.