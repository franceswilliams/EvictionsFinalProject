Data Import/Cleaning
================

#### Data import of eviction data *(Naama)*

In this section, we’ll import eviction data, which includes eviction
rates and other relevant metrics by census tract.

``` r
# CSV file contains all census tracts for NY state, so we'll import the file and then filter such that the dataframe 'eviction' only contains NYC census tracts.

# contains years 2000 - 2016
# n = 1880 per year (1880 census tracts)


eviction = 
  read.csv('./data/EvictionData_NY.csv') %>% 
  janitor::clean_names() %>% 
  filter(parent_location %in% c("Manhattan County, New York", "Queens County, New York", "Kings County, New York", "Bronx County, New York", "Richmond County, New York")) 
```

#### Data import of gentrification and population density *(Will)*

In this section we’ll import:

1.  NYC gentrification data from *Governing Magazine*’s [**New York City
    Gentrification Maps and
    Data**](https://www.governing.com/gov-data/new-york-gentrification-maps-demographic-data.html)  
    a. Since the data aren’t available in a format that can be analyzed
    directly, we’ll instead import data from the *Urban Displacement
    Project*’s [**Mapping Displacement and Gentrification in the New
    York Metropolitan Area**](https://www.urbandisplacement.org/maps/ny)
2.  Population density data from the American Community Survey (ACS)
    [**5-year census
    estimates**](https://factfinder.census.gov/faces/nav/jsf/pages/guided_search.xhtml)

##### Gentrification

``` r
gentrification =
  read_excel('./data/udp_ny_final_typology_jan_2019.xlsx') %>% 
  filter(startsWith(as.character(geoid), '36')) %>% ## just New York state (FIPS = 36, first two digits)
  rename(id2 = geoid,
         gent_status = Type_1.19) %>% 
  mutate(gent_indicator = if_else(gent_status %in% c('LI - Ongoing Gentrification', 'MHI - Advanced Gentrification'),
                                  1,
                                  0))

gentrification %>% 
  arrange(id2) %>% 
  head(10) %>% 
  knitr::kable(digits = 11)
```

|        id2 | gent\_status                                       | gent\_indicator |
| ---------: | :------------------------------------------------- | --------------: |
| 3.6005e+10 | MHI - Stable Exclusion                             |               0 |
| 3.6005e+10 | MHI - Stable Exclusion                             |               0 |
| 3.6005e+10 | MHI - Stable Exclusion                             |               0 |
| 3.6005e+10 | LI - At Risk of Gentrification                     |               0 |
| 3.6005e+10 | LI - Ongoing Gentrification                        |               1 |
| 3.6005e+10 | LI - At Risk of Gentrification                     |               0 |
| 3.6005e+10 | LI - At Risk of Gentrification                     |               0 |
| 3.6005e+10 | Missing Data                                       |               0 |
| 3.6005e+10 | LI - Ongoing Displacement of Low-Income Households |               0 |
| 3.6005e+10 | LI - Ongoing Gentrification                        |               1 |

##### Population density

``` r
## For the ACS data, we'll have to use a crude measure of population density we calculate ourselves - census tract population (which changes every year) divided by census tract area (which does not usually change). 

## NUMERATOR
## Importing population per census tract per year
filenames_DP05 = 
  list.files('./data/') %>%
  paste0('./data/', .) %>% 
  as_tibble() %>% 
  filter(str_detect(value, 'DP05')) %>% 
  pull(., value) ## coerces tibble back to vector for reading by map_df()

population_data = 
  map_dfr(filenames_DP05, read_csv, .id = "input") %>% 
  janitor::clean_names() %>% 
  select(id = geo_id, id2 = geo_id2, geography = geo_display_label, total_pop = hc01_vc03, year = input) %>% 
  filter(id != "Id") %>% 
  mutate(id2 = as.numeric(id2),
         year = as.numeric(year) + 2009,
         name = as.character(readr::parse_number(geography)),
         geography = str_remove(geography, "Census Tract [0-9]{1,}, "),
         geography = str_remove(geography, "Census Tract [0-9]{1,}\\.[0-9]{1,}, "),
         total_pop = as.numeric(total_pop)) %>% 
  select(id, id2, name, geography, total_pop, year)
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character()
    ## )

    ## See spec(...) for full column specifications.

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character()
    ## )

    ## See spec(...) for full column specifications.

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character()
    ## )

    ## See spec(...) for full column specifications.

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character()
    ## )

    ## See spec(...) for full column specifications.

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character()
    ## )

    ## See spec(...) for full column specifications.

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character()
    ## )

    ## See spec(...) for full column specifications.

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character()
    ## )

    ## See spec(...) for full column specifications.

``` r
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
## JOINING DATASETS & CALCULATING DENSITY
density_data =
  left_join(population_data, area) %>% 
  mutate(density = total_pop / area_sqmi) 
```

    ## Joining, by = c("id", "id2", "name", "geography")

``` r
## EXAMPLE DATALINES
density_data %>% 
  head(10) %>% 
  knitr::kable()
```

| id                   |        id2 | name | geography               | total\_pop | year | area\_sqmi |    density |
| :------------------- | ---------: | :--- | :---------------------- | ---------: | ---: | ---------: | ---------: |
| 1400000US36001000100 | 3.6001e+10 | 1    | Albany County, New York |       2308 | 2010 |      0.940 |  2455.3191 |
| 1400000US36001000200 | 3.6001e+10 | 2    | Albany County, New York |       5506 | 2010 |      0.797 |  6908.4065 |
| 1400000US36001000300 | 3.6001e+10 | 3    | Albany County, New York |       6471 | 2010 |      2.247 |  2879.8398 |
| 1400000US36001000401 | 3.6001e+10 | 4.01 | Albany County, New York |       2211 | 2010 |      3.483 |   634.7976 |
| 1400000US36001000403 | 3.6001e+10 | 4.03 | Albany County, New York |       4672 | 2010 |      1.211 |  3857.9686 |
| 1400000US36001000404 | 3.6001e+10 | 4.04 | Albany County, New York |       5129 | 2010 |      0.707 |  7254.5969 |
| 1400000US36001000501 | 3.6001e+10 | 5.01 | Albany County, New York |       3247 | 2010 |      0.211 | 15388.6256 |
| 1400000US36001000502 | 3.6001e+10 | 5.02 | Albany County, New York |       3914 | 2010 |      0.300 | 13046.6667 |
| 1400000US36001000600 | 3.6001e+10 | 6    | Albany County, New York |       3694 | 2010 |      0.196 | 18846.9388 |
| 1400000US36001000700 | 3.6001e+10 | 7    | Albany County, New York |       4237 | 2010 |      0.611 |  6934.5336 |

#### Data import of racial composition and English language data *(Gloria)*

In this section we’ll import:

1.  English language usage data from the American Community Survey
    (ACS), with our primary measure of interest being the percentage of
    residents who speak English less than “very well” [**5-year census
    estimates**](https://factfinder.census.gov/faces/nav/jsf/pages/guided_search.xhtml)
2.  Data on racial composition by census tract from the American
    Community Survey (ACS) [**5-year census
    estimates**](https://factfinder.census.gov/faces/nav/jsf/pages/guided_search.xhtml)

<!-- end list -->

``` r
## English language usage data- we import data from 2010-2016
## Relevant variable (the percentage of the population 5 and over that speaks English less than "very well"), changes by year- files are therefore individually imported

englang_2010 = 
  read.csv("./data/ACS_10_5YR_DP02_with_ann.csv") %>%
  janitor::clean_names() %>%
  mutate(year = "2010", geo_id = as.character(geo_id), geo_id2 = as.character(geo_id2)) %>%
  select(year, geo_id, geo_id2, hc03_vc170) %>%
  filter(geo_id != "Id") %>%
  mutate(pct_eng = as.numeric(as.character(hc03_vc170))) %>%
  select(-hc03_vc170)

englang_2011 = 
  read.csv("./data/ACS_11_5YR_DP02_with_ann.csv") %>%
  janitor::clean_names() %>%
  mutate(year = "2011", geo_id = as.character(geo_id), geo_id2 = as.character(geo_id2)) %>%
  select(year, geo_id, geo_id2, hc03_vc170) %>%
  filter(geo_id != "Id") %>%
  mutate(pct_eng = as.numeric(as.character(hc03_vc170))) %>%
  select(-hc03_vc170)

englang_2012 = 
  read.csv("./data/ACS_12_5YR_DP02_with_ann.csv") %>%
  janitor::clean_names() %>%
  mutate(year = "2012", geo_id = as.character(geo_id), geo_id2 = as.character(geo_id2)) %>%
  select(year, geo_id, geo_id2, hc03_vc170) %>%
  filter(geo_id != "Id") %>%
  mutate(pct_eng = as.numeric(as.character(hc03_vc170))) %>%
  select(-hc03_vc170)

englang_2013 = 
  read.csv("./data/ACS_13_5YR_DP02_with_ann.csv") %>%
  janitor::clean_names() %>%
  mutate(year = "2013", geo_id = as.character(geo_id), geo_id2 = as.character(geo_id2)) %>%
  select(year, geo_id, geo_id2, hc03_vc173) %>%
  filter(geo_id != "Id") %>%
  mutate(pct_eng = as.numeric(as.character(hc03_vc173))) %>%
  select(-hc03_vc173)

englang_2014 = 
  read.csv("./data/ACS_14_5YR_DP02_with_ann.csv") %>%
  janitor::clean_names() %>%
  mutate(year = "2014", geo_id = as.character(geo_id), geo_id2 = as.character(geo_id2)) %>%
  select(year, geo_id, geo_id2, hc03_vc173) %>%
  filter(geo_id != "Id") %>%
  mutate(pct_eng = as.numeric(as.character(hc03_vc173))) %>%
  select(-hc03_vc173)

englang_2015 = 
  read.csv("./data/ACS_15_5YR_DP02_with_ann.csv") %>%
  janitor::clean_names() %>%
  mutate(year = "2015", geo_id = as.character(geo_id), geo_id2 = as.character(geo_id2)) %>%
  select(year, geo_id, geo_id2, hc03_vc173) %>%
  filter(geo_id != "Id") %>%
  mutate(pct_eng = as.numeric(as.character(hc03_vc173))) %>%
  select(-hc03_vc173)

englang_2016 = 
  read.csv("./data/ACS_16_5YR_DP02_with_ann.csv") %>%
  janitor::clean_names() %>%
  mutate(year = "2016", geo_id = as.character(geo_id), geo_id2 = as.character(geo_id2)) %>%
  select(year, geo_id, geo_id2, hc03_vc173) %>%
  filter(geo_id != "Id") %>%
  mutate(pct_eng = as.numeric(as.character(hc03_vc173))) %>%
  select(-hc03_vc173)

englang_data = 
  bind_rows(englang_2010, englang_2011, englang_2012, englang_2013, englang_2014, englang_2015, englang_2016)

englang_data %>% 
  head(10) %>% 
  knitr::kable()
```

| year | geo\_id              | geo\_id2    | pct\_eng |
| :--- | :------------------- | :---------- | -------: |
| 2010 | 1400000US36001000100 | 36001000100 |     13.8 |
| 2010 | 1400000US36001000200 | 36001000200 |      1.7 |
| 2010 | 1400000US36001000300 | 36001000300 |      4.6 |
| 2010 | 1400000US36001000401 | 36001000401 |      1.0 |
| 2010 | 1400000US36001000403 | 36001000403 |      8.5 |
| 2010 | 1400000US36001000404 | 36001000404 |      5.1 |
| 2010 | 1400000US36001000501 | 36001000501 |      2.2 |
| 2010 | 1400000US36001000502 | 36001000502 |      3.2 |
| 2010 | 1400000US36001000600 | 36001000600 |     15.4 |
| 2010 | 1400000US36001000700 | 36001000700 |      3.0 |

``` r
## Racial composition data- we import data from 2010-2016
## Racial categories are not mutually exclusive, so total percentages may sum to more than 100%
## Relevant variable (the percentage of the population 5 and over that speaks English less than "very well"), changes by year- files are therefore individually imported

race_2010 = 
  read.csv("./data/ACS_10_5YR_DP05_with_ann.csv") %>%
  janitor::clean_names() %>%
  mutate(year = "2010", geo_id = as.character(geo_id), geo_id2 = as.character(geo_id2)) %>%
  select(year, geo_id, geo_id2, hc03_vc72, hc03_vc73, hc03_vc74, hc03_vc75, hc03_vc76, hc03_vc77) %>%
  filter(geo_id != "Id") %>%
  rename(white = hc03_vc72, black = hc03_vc73, aian = hc03_vc74, asian = hc03_vc75, nhpi = hc03_vc76, other = hc03_vc77) %>%
  mutate(
    white = as.numeric(as.character(white)),
    black = as.numeric(as.character(black)),
    aian = as.numeric(as.character(aian)),
    asian = as.numeric(as.character(asian)),
    nhpi = as.numeric(as.character(nhpi)),
    other = as.numeric(as.character(other))
  )

race_2011 = 
  read.csv("./data/ACS_11_5YR_DP05_with_ann.csv") %>%
  janitor::clean_names() %>%
  mutate(year = "2011", geo_id = as.character(geo_id), geo_id2 = as.character(geo_id2)) %>%
  select(year, geo_id, geo_id2, hc03_vc72, hc03_vc73, hc03_vc74, hc03_vc75, hc03_vc76, hc03_vc77) %>%
  filter(geo_id != "Id") %>%
  rename(white = hc03_vc72, black = hc03_vc73, aian = hc03_vc74, asian = hc03_vc75, nhpi = hc03_vc76, other = hc03_vc77) %>%
  mutate(
    white = as.numeric(as.character(white)),
    black = as.numeric(as.character(black)),
    aian = as.numeric(as.character(aian)),
    asian = as.numeric(as.character(asian)),
    nhpi = as.numeric(as.character(nhpi)),
    other = as.numeric(as.character(other))
  )

race_2012 = 
  read.csv("./data/ACS_12_5YR_DP05_with_ann.csv") %>%
  janitor::clean_names() %>%
  mutate(year = "2012", geo_id = as.character(geo_id), geo_id2 = as.character(geo_id2)) %>%
  select(year, geo_id, geo_id2, hc03_vc72, hc03_vc73, hc03_vc74, hc03_vc75, hc03_vc76, hc03_vc77) %>%
  filter(geo_id != "Id") %>%
  rename(white = hc03_vc72, black = hc03_vc73, aian = hc03_vc74, asian = hc03_vc75, nhpi = hc03_vc76, other = hc03_vc77) %>%
  mutate(
    white = as.numeric(as.character(white)),
    black = as.numeric(as.character(black)),
    aian = as.numeric(as.character(aian)),
    asian = as.numeric(as.character(asian)),
    nhpi = as.numeric(as.character(nhpi)),
    other = as.numeric(as.character(other))
  )

race_2013 = 
  read.csv("./data/ACS_13_5YR_DP05_with_ann.csv") %>%
  janitor::clean_names() %>%
  mutate(year = "2013", geo_id = as.character(geo_id), geo_id2 = as.character(geo_id2)) %>%
  select(year, geo_id, geo_id2, hc03_vc78, hc03_vc79, hc03_vc80, hc03_vc81, hc03_vc82, hc03_vc83) %>%
  filter(geo_id != "Id") %>%
  rename(white = hc03_vc78, black = hc03_vc79, aian = hc03_vc80, asian = hc03_vc81, nhpi = hc03_vc82, other = hc03_vc83) %>%
  mutate(
    white = as.numeric(as.character(white)),
    black = as.numeric(as.character(black)),
    aian = as.numeric(as.character(aian)),
    asian = as.numeric(as.character(asian)),
    nhpi = as.numeric(as.character(nhpi)),
    other = as.numeric(as.character(other))
  )

race_2014 = 
  read.csv("./data/ACS_14_5YR_DP05_with_ann.csv") %>%
  janitor::clean_names() %>%
  mutate(year = "2014", geo_id = as.character(geo_id), geo_id2 = as.character(geo_id2)) %>%
  select(year, geo_id, geo_id2, hc03_vc78, hc03_vc79, hc03_vc80, hc03_vc81, hc03_vc82, hc03_vc83) %>%
  filter(geo_id != "Id") %>%
  rename(white = hc03_vc78, black = hc03_vc79, aian = hc03_vc80, asian = hc03_vc81, nhpi = hc03_vc82, other = hc03_vc83) %>%
  mutate(
    white = as.numeric(as.character(white)),
    black = as.numeric(as.character(black)),
    aian = as.numeric(as.character(aian)),
    asian = as.numeric(as.character(asian)),
    nhpi = as.numeric(as.character(nhpi)),
    other = as.numeric(as.character(other))
  )

race_2015 = 
  read.csv("./data/ACS_15_5YR_DP05_with_ann.csv") %>%
  janitor::clean_names() %>%
  mutate(year = "2015", geo_id = as.character(geo_id), geo_id2 = as.character(geo_id2)) %>%
  select(year, geo_id, geo_id2, hc03_vc78, hc03_vc79, hc03_vc80, hc03_vc81, hc03_vc82, hc03_vc83) %>%
  filter(geo_id != "Id") %>%
  rename(white = hc03_vc78, black = hc03_vc79, aian = hc03_vc80, asian = hc03_vc81, nhpi = hc03_vc82, other = hc03_vc83) %>%
  mutate(
    white = as.numeric(as.character(white)),
    black = as.numeric(as.character(black)),
    aian = as.numeric(as.character(aian)),
    asian = as.numeric(as.character(asian)),
    nhpi = as.numeric(as.character(nhpi)),
    other = as.numeric(as.character(other))
  )

race_2016 = 
  read.csv("./data/ACS_16_5YR_DP05_with_ann.csv") %>%
  janitor::clean_names() %>%
  mutate(year = "2016", geo_id = as.character(geo_id), geo_id2 = as.character(geo_id2)) %>%
  select(year, geo_id, geo_id2, hc03_vc78, hc03_vc79, hc03_vc80, hc03_vc81, hc03_vc82, hc03_vc83) %>%
  filter(geo_id != "Id") %>%
  rename(white = hc03_vc78, black = hc03_vc79, aian = hc03_vc80, asian = hc03_vc81, nhpi = hc03_vc82, other = hc03_vc83) %>%
  mutate(
    white = as.numeric(as.character(white)),
    black = as.numeric(as.character(black)),
    aian = as.numeric(as.character(aian)),
    asian = as.numeric(as.character(asian)),
    nhpi = as.numeric(as.character(nhpi)),
    other = as.numeric(as.character(other))
  )

race_data = 
  bind_rows(race_2010, race_2011, race_2012, race_2013, race_2014, race_2015, race_2016)

race_data %>% 
  head(10) %>% 
  knitr::kable()
```

| year | geo\_id              | geo\_id2    | white | black | aian | asian | nhpi | other |
| :--- | :------------------- | :---------- | ----: | ----: | ---: | ----: | ---: | ----: |
| 2010 | 1400000US36001000100 | 36001000100 |  39.5 |  55.7 |  0.0 |   0.4 |  0.0 |   8.4 |
| 2010 | 1400000US36001000200 | 36001000200 |  19.1 |  80.3 |  0.0 |   6.3 |  0.0 |   1.9 |
| 2010 | 1400000US36001000300 | 36001000300 |  49.6 |  45.9 |  0.5 |   2.5 |  0.0 |   4.8 |
| 2010 | 1400000US36001000401 | 36001000401 |  88.4 |   9.2 |  1.4 |   1.0 |  0.0 |   0.8 |
| 2010 | 1400000US36001000403 | 36001000403 |  72.0 |   6.4 |  0.8 |  24.0 |  1.2 |   0.6 |
| 2010 | 1400000US36001000404 | 36001000404 |  68.4 |  13.0 |  1.3 |  10.9 |  0.0 |   7.4 |
| 2010 | 1400000US36001000501 | 36001000501 |  68.5 |  31.0 |  6.2 |   3.7 |  0.0 |   2.0 |
| 2010 | 1400000US36001000502 | 36001000502 |  76.6 |  20.1 |  2.9 |   5.1 |  0.3 |   4.2 |
| 2010 | 1400000US36001000600 | 36001000600 |  43.3 |  39.3 |  1.7 |   7.5 |  0.0 |   9.5 |
| 2010 | 1400000US36001000700 | 36001000700 |  15.5 |  81.3 |  4.9 |   0.0 |  0.0 |   4.3 |
