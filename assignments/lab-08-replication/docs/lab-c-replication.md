Lab-C Replication
================
Christopher Prener, Ph.D.
(April 23, 2020)

## Introduction

This notebook replicates the Lab-C assignment.

## Dependencies

This notebook requires the following packages:

``` r
# tidyverse packages
library(dplyr)       # data wrangling tools
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
# spatial packages
library(nngeo)       # eliminate holes
```

    ## Loading required package: sf

    ## Linking to GEOS 3.7.2, GDAL 2.4.2, PROJ 5.2.0

``` r
library(mapview)     # preview spatial data
library(sf)          # spatial data tools

# other packages
library(here)        # file path management
```

    ## here() starts at /Users/prenercg/GitHub/slu-soc5650/lecture-C/assignments/lab-c-replication

## Load Data

This notebook requires two shapefiles, one with precincts in
unincorporated parts of St. Louis County and one with precincts in
incorporated
municipalities:

``` r
corp <- st_read(here("data", "STLC_ELECT_IncorporatedPrecincts"), stringsAsFactors = FALSE)
```

    ## Reading layer `STLC_ELECT_IncorporatedPrecincts' from data source `/Users/prenercg/GitHub/slu-soc5650/lecture-C/assignments/lab-c-replication/data/STLC_ELECT_IncorporatedPrecincts' using driver `ESRI Shapefile'
    ## Simple feature collection with 1064 features and 4 fields
    ## geometry type:  POLYGON
    ## dimension:      XY
    ## bbox:           xmin: -90.73619 ymin: 38.46679 xmax: -90.1968 ymax: 38.83229
    ## CRS:            4326

``` r
unincorp <- st_read(here("data", "STLC_ELECT_UnincorporatedPrecincts"), stringsAsFactors = FALSE)
```

    ## Reading layer `STLC_ELECT_UnincorporatedPrecincts' from data source `/Users/prenercg/GitHub/slu-soc5650/lecture-C/assignments/lab-c-replication/data/STLC_ELECT_UnincorporatedPrecincts' using driver `ESRI Shapefile'
    ## Simple feature collection with 398 features and 5 fields
    ## geometry type:  MULTIPOLYGON
    ## dimension:      XY
    ## bbox:           xmin: -90.7356 ymin: 38.3878 xmax: -90.11739 ymax: 38.89174
    ## CRS:            4326

## Part 1

### Question 1

There are three problems with the unincorporated data - the variable
names do not match the incorporated data, and the precinct ID column is
character (as opposed to numeric in the incorporated data). There is
also an extra column, `WARD`. We’ll rename the variables, remove WARD,
and then convert the precinct ID column to numeric. The goal is to make
the unincorporated data match the incorporated data:

``` r
unincorp %>%
  select(-WARD) %>%
  rename(
    id = PRECINCTID,
    precinct = PRECINCT,
    council = COUNTY_COU,
    muni = MUNI
  ) %>%
  mutate(id = as.numeric(id)) -> unincorp
```

Now we have two matching data frames that can be merged (or bound)
together.

### Question 2

To combine these together, we’ll use `rbind()`:

``` r
precincts <- rbind(corp, unincorp)
```

We now have a single data frame with all preincts.

### Question 3

Next, we need to visually inspect the `precincts` object to see if it is
a geometry collection or if it has polygons. If you look in the global
enviornment, you can see that `geometry:sfc_GEOMETRY` appears in the
`precincts` object. We’ll extract this to `"POLYGON"` before proceeding
with the dissolve.

``` r
precincts <- st_collection_extract(precincts, type = "POLYGON")
```

Now we have data prepared for a dissolve.

### Question 4

Next, we’ll combine all of the preincts together based on the council
district they fall into:

``` r
precincts %>%
  select(council) %>%
  group_by(council) %>%
  summarise() -> council
```

We’ll preview the changes using `mapview`:

``` r
mapview(council)
```

![](lab-c-replication_files/figure-gfm/p1-q4-preview-1.png)<!-- -->

We can see a variety of holes throughout most of the council districts.

### Question 5

In order to address these holes, we can use `st_remove_holes()` to
clean-up the output:

``` r
council <- st_remove_holes(council)
```

We’ll preview the changes using `mapview`:

``` r
mapview(council)
```

![](lab-c-replication_files/figure-gfm/p1-q5-preview-1.png)<!-- -->
