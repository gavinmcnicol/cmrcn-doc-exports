---
title: "RCN DOC Exports - Predictive Modeling"
author: "Gavin McNicol"
date: "2022-11-05"
output: 
  html_document:
    theme: journal
    highlight: monochrome
    keep_md: true
---



## Purpose

Code for predictive modeling of stream dissolved organic carbon (DOC) using random forest and spatial (forward) feature selection in support of the manuscript:

*Carbon Export along Coastal Temperate Rainforest Margins Dominated by Small Watersheds* by G. McNicol, E. Hood, D. E. Butman, S. Tank, I.J.W. Giesbrecht, W. Floyd, D. D’Amore, J.B. Fellman (2022-2023)

## Analysis



Packages loaded: Yes



## Part 1: Create Training Data

### Load and View DOC Data

Much of Part 1 code can be suppressed `eval = F` for Part 1 as `training-data.csv` exists in repo.




### Load and View Predictive Features

Exclude geolocation features: `Poly_x`, `Poly_y`


```
## Rows: 2,705
## Columns: 19
## $ IP_ID     <dbl> 1, 10, 101, 102, 10278, 104, 105, 107, 108, 109, 110, 114, 1…
## $ Cluster   <dbl> 1, 1, 2, 2, 7, 2, 2, 2, 2, 2, 2, 2, 2, 2, 4, 2, 2, 2, 2, 7, …
## $ Area_km2  <dbl> 12.87594, 29.93291, 42.80481, 12.56653, 17.30269, 39.30523, …
## $ PAratio   <dbl> 2.1778479, 1.3328560, 1.1284217, 1.5158169, 1.6098820, 0.871…
## $ Wtb_prct  <dbl> 0.0000000, 0.0000000, 1.1195686, 0.0000000, 0.0000000, 1.012…
## $ Glc_prct  <dbl> 0.0000000, 0.0000000, 0.0000000, 0.0000000, 0.0000000, 0.000…
## $ Elev_avg  <dbl> 70.87788, 259.52785, 589.80864, 607.63224, 33.95881, 504.535…
## $ Elev_max  <dbl> 129, 1204, 1423, 1309, 66, 1512, 1895, 1948, 1503, 1513, 161…
## $ Slpe_avg  <dbl> 3.784544, 8.974770, 26.764982, 28.421681, 2.401219, 22.03952…
## $ Slpe5     <dbl> 70.3251137, 48.8825402, 2.5293305, 0.4124427, 86.2074731, 2.…
## $ Slpe10    <dbl> 82.729751, 68.572787, 7.828095, 2.032192, 97.674639, 13.8316…
## $ MAP_avg   <dbl> 1261.750, 1972.083, 3154.780, 3177.636, 2980.667, 2925.139, …
## $ PAS_avg   <dbl> 40.58333, 147.70833, 569.78049, 621.36364, 117.26667, 470.47…
## $ PrctSnow  <dbl> 3.216432, 7.489964, 18.060860, 19.554271, 3.934243, 16.08375…
## $ MAT_avg   <dbl> 10.000000, 8.958333, 6.512195, 6.545455, 8.000000, 6.805556,…
## $ MSP_avg   <dbl> 274.4167, 418.4583, 618.2927, 600.6364, 784.2667, 581.9167, …
## $ TD_avg    <dbl> 13.83333, 14.08333, 14.68293, 14.45455, 10.13333, 14.25000, …
## $ Eref_avg  <dbl> 663.0000, 618.9167, 550.1951, 535.5455, 491.5333, 540.6111, …
## $ VegHt_avg <dbl> 5.181818, 9.220000, 28.105263, 25.173913, 15.892857, 23.3289…
```

### Feature Objects



`feat_l` = 17  

`feat_names`: Area_km2, PAratio, Wtb_prct, Glc_prct, Elev_avg, Elev_max, Slpe_avg, Slpe5, Slpe10, MAP_avg, PAS_avg, PrctSnow, MAT_avg, MSP_avg, TD_avg, Eref_avg, VegHt_avg

### Geneate Combinations

Minimum number of features = 2
Maximum number of features = 5

Takes a couple of minute to run for max. = 5. Longer for more.





**Output:** Predictive modeling feature combinations list, `data/feat_comb.rds`


```r
saveRDS(feat_comb, "data/feat_comb.rds")
```

There are ~9,000 feature combinations for 2-5 predictors.

### Join DOC and Features



**Output:** Predictive modeling training dataset, `data/training-data.csv`


```r
write_csv(data, "data/training-data.csv")
```

## Part 2: Train Random Forest


```r
train <- read_csv("data/training-data.csv")
feat_comb <- readRDS("data/feat_comb.rds")
```


```r
set.seed(23)
IP_IDs <- train %>% 
  select(IP_ID) %>% pull
```

### Cluster Watersheds


```r
# site_loc <- read_csv("data/WtsData_For_RCN_DOCflux.csv") %>% 
#   group_by(IP_ID) %>% 
#   summarize(Lat = Poly_y[1],
#             Lon = Poly_x[1]) 
# 
# x <- site_loc$Lon
# y <- site_loc$Lat
# xy <- SpatialPointsDataFrame(matrix(c(x,y), ncol=2), data.frame(IP_ID=site_loc$IP_ID),
#                              proj4string = CRS("+proj=aea +ellps=GRS80 +datum=NAD83"))
# plot(xy)
# crs(xy)
# 
# xy <- spTransform(xy, CRS("+proj=longlat +ellps=WGS84 +datum=WGS84")) 

# # calculate Euclidian distance 
# site_dist <- xy %>% distm()
# 
# # cluster
# site_clusters <- hclust(as.dist(site_dist), method ='complete')
# 
# # define 300 km threshold
# d <- 100000
# xy$fold <- cutree(site_clusters, h=d) 
# xy <- xy %>% as_tibble() %>% 
#   dplyr::select(IP_ID, fold)
# 
# # rejoin folds
# train <- train %>% 
#   left_join(xy, by = c("IP_ID")) %>%
#   dplyr::select(fold = fold, everything())
```


