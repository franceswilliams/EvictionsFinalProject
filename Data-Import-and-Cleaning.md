Data Import/Cleaning
================

#### Data import of eviction data *(Naama)*

#### Data import of gentrification and population density *(Will)*

In this section we’ll import:

1.  NYC gentrification data from *Governing Magazine*’s [**New York City
    Gentrification Maps and
    Data**](https://www.governing.com/gov-data/new-york-gentrification-maps-demographic-data.html)
2.  Population density data from the American Community Survey (ACS)
    [**5-year census
    estimates**](https://factfinder.census.gov/faces/nav/jsf/pages/guided_search.xhtml)

<!-- end list -->

``` r
## For the ACS data, we'll have to use a crude measure of population density we calculate ourselves - census tract population (which changes every year) divided by census tract area (which does not usually change). 

## NUMERATOR
## Importing population per census tract per year


## DENOMINATOR
## Importing area in sq. mi. for each area:
area = 
  read_csv('./data/ACS_09_5YR_G001_with_ann.csv', skip = 1) %>% 
  janitor::clean_names() %>% 
  select(id, id2, geography, area_sqmi = land_area_in_square_miles) %>% 
  mutate(name = as.character(readr::parse_number(geography)),
         geography = str_remove(geography, "Census Tract [0-9]{1,}, "),
         geography = str_remove(geography, "Census Tract [0-9]{1,}\\.[0-9]{1,}, ")) %>% 
  select(id, id2, name, geography, area_sqmi)
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_logical(),
    ##   Id = col_character(),
    ##   Id2 = col_double(),
    ##   Geography = col_character(),
    ##   `Summary Level` = col_double(),
    ##   `Geographic Component` = col_character(),
    ##   Region = col_double(),
    ##   Division = col_double(),
    ##   `State (FIPS)` = col_double(),
    ##   `County (FIPS)` = col_character(),
    ##   `Census Tract` = col_character(),
    ##   `Area Name` = col_double(),
    ##   `Legal/Statistical Area Description Code` = col_character(),
    ##   `Land Area in square miles` = col_double()
    ## )

    ## See spec(...) for full column specifications.

``` r
area %>% 
  head(10) %>% 
  knitr::kable()
```

| id                   |        id2 | name | geography               | area\_sqmi |
| :------------------- | ---------: | :--- | :---------------------- | ---------: |
| 1400000US36001000100 | 3.6001e+10 | 1    | Albany County, New York |      0.940 |
| 1400000US36001000200 | 3.6001e+10 | 2    | Albany County, New York |      0.797 |
| 1400000US36001000300 | 3.6001e+10 | 3    | Albany County, New York |      2.247 |
| 1400000US36001000401 | 3.6001e+10 | 4.01 | Albany County, New York |      3.483 |
| 1400000US36001000403 | 3.6001e+10 | 4.03 | Albany County, New York |      1.211 |
| 1400000US36001000404 | 3.6001e+10 | 4.04 | Albany County, New York |      0.707 |
| 1400000US36001000501 | 3.6001e+10 | 5.01 | Albany County, New York |      0.211 |
| 1400000US36001000502 | 3.6001e+10 | 5.02 | Albany County, New York |      0.300 |
| 1400000US36001000600 | 3.6001e+10 | 6    | Albany County, New York |      0.196 |
| 1400000US36001000700 | 3.6001e+10 | 7    | Albany County, New York |      0.611 |

#### Data import of racial composition and English language data *(Gloria)*
