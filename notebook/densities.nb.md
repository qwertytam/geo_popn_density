Population Densities - Comparing Across (Selected) Countries
================

## Notebook Summary

This notebook looks at population density metrics for a set of selected
countries. The basic flow is reading in the data, some data
manipulation, then reviewing various plots

Current countries in the data source: - Australia - United States of
America

------------------------------------------------------------------------

### 1. Initialize and Read in the Data

Data is sourced from the relevant national statistical agencies and
gathered using scripts in the `data-raw` directory with results stored
as `csv` files in the `inst/extdata` directory and as `rda` files in the
`data` directory.

``` r
# Load library for successful knit
library(geo.popn.density)

# Set up variables we will reuse in the notebook
ccode = "Country_Code"
msa_name = "MSA_Name"
ppn_est = "Popn_Est"
area = "Area_km2"
density = "Density"
ppn_pct = "Perc_Popn"
cum_pct = "Cum_Popn_Perc"
method = "method"

# Get the data
all_data <- GetAllData()
```

### 2. Data Review

#### 2.1 Density Calculations by Country

``` r
densities <- CalcDensities(all_data, ccode = ccode, ppn_est = ppn_est,
                               area = area, density = density, group_by = ccode)
PlotDensities(densities, density = density, ccode = ccode)
```

![](densities.nb_files/figure-gfm/by%20country-1.png)<!-- -->

#### 2.2 The Median Statistical Area by Density

``` r
# Sorting the data by descending data and calculating the cumulative percentage population to find the median
all_by_density <- CalcCumPercent(all_data, cum_pct = cum_pct, metric = ppn_est, sort_by = density, group_by = ccode)
mdn_sa <- FindMedian(all_by_density, cum_pct = cum_pct)
dplyr::select(mdn_sa, Country_Code, State_Abbr, MSA_Name, SA_Code, Popn_Est, Area_km2, Density, Cum_Popn_Perc)
```

    ## # A tibble: 2 × 8
    ## # Groups:   Country_Code [2]
    ##   Country_Code State_Abbr MSA_Name             SA_Code Popn_Est Area_km2 Density
    ##   <chr>        <chr>      <chr>                  <dbl>    <dbl>    <dbl>   <dbl>
    ## 1 AU           QLD        Brisbane              3.01e8    15813    11.9    1329.
    ## 2 US           CO         Denver-Aurora-Lakew…  8.06e9     2983     3.41    875.
    ## # … with 1 more variable: Cum_Popn_Perc <dbl>

#### 2.3 Statistical Areas by Descending Density

``` r
PlotAllDataByDensity(all_by_density, density = density, ccode = ccode, cum_pct = cum_pct)
```

![](densities.nb_files/figure-gfm/SA%20desc%20density-1.png)<!-- -->

#### 2.4 The Median Metropolitan Area by Population

``` r
# Group the data into its metropolitan statistical areas
msas <- GroupData(all_data, ppn_est = ppn_est, area = area, ppn_pct = ppn_pct,
                  density = density,
                  group_by = c(ccode, "MSA_Code", msa_name, "Metro_Micro"))

# Sort by population descending
msas_by_popn <- SortMSAsbyPopn(msas, cum_pct = cum_pct, ppn_est = ppn_est, ppn_pct = ppn_pct, msa_name = msa_name, group_by = ccode)
mdn_msa_popn <- FindMedian(msas_by_popn, cum_pct = cum_pct)
dplyr::select(mdn_msa_popn, Country_Code, MSA_Name, Popn_Est, Area_km2, Density, Cum_Popn_Perc)
```

    ## # A tibble: 2 × 6
    ## # Groups:   Country_Code [2]
    ##   Country_Code MSA_Name             Popn_Est Area_km2 Density Cum_Popn_Perc
    ##   <chr>        <fct>                   <dbl>    <dbl>   <dbl>         <dbl>
    ## 1 AU           Perth                 2099533    3532.    594.          56.2
    ## 2 US           Cincinnati, OH-KY-IN  2256884   11994.    188.          50.5

#### 2.5 Metroplitan Areas by Descending Population

``` r
PlotMSAsByPopn(msas_by_popn, ppn_est = ppn_est, ccode = ccode, cum_pct = cum_pct)
```

![](densities.nb_files/figure-gfm/MSA%20desc%20popn-1.png)<!-- -->

#### 2.6 The Median Metropolitan Area by Simple Density

``` r
msas_by_density <- SortMSAsbyDensity(msas, cum_pct = cum_pct, density = density, ppn_pct = ppn_pct, group_by = ccode)
mdn_msa_density <- FindMedian(msas_by_density, cum_pct = cum_pct)
dplyr::select(mdn_msa_density, Country_Code, MSA_Name, Popn_Est, Area_km2, Density, Cum_Popn_Perc)
```

    ## # A tibble: 2 × 6
    ## # Groups:   Country_Code [2]
    ##   Country_Code MSA_Name       Popn_Est Area_km2 Density Cum_Popn_Perc
    ##   <chr>        <chr>             <dbl>    <dbl>   <dbl>         <dbl>
    ## 1 AU           Perth           2099533    3532.    594.          55.7
    ## 2 US           Fort Wayne, IN   419601    2585.    162.          50.1

#### 2.7 Metropolitan Areas by Descending Simple Density

``` r
PlotMSAsByDensity(msas_by_density, density = density, ccode = ccode, cum_pct = cum_pct)
```

![](densities.nb_files/figure-gfm/MSA%20desc%20density-1.png)<!-- -->

## Source Data

### Australia

Data sourced from the Australian Bureau of Statistics. Population data
sourced at the Statistical Area 2 level. See references below for
further detail on Australian statistical geographic standards. Density
is given by the ABS but is recalculated here as the
population-area-density numbers do not line up as given. Data is output
to `au.csv` in the data directory.

#### References:

-   [Description of the Australian Statistical Geography
    Standard](https://www.abs.gov.au/statistics/standards/australian-statistical-geography-standard-asgs-edition-3/jul2021-jun2026#asgs-diagram)
-   [ABS population data by
    SA2](https://www.abs.gov.au/statistics/people/population/regional-population/2020-21#data-download)
-   [Significant Urban Area to
    SA2](https://www.abs.gov.au/AUSSTATS/abs@.nsf/DetailsPage/1270.0.55.004July%202016?OpenDocument)

The repo for this code is
[here](https://github.com/qwertytam/geo.popn.density)