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
filenames_DP05 = 
  list.files('./data/') %>%
  paste0('./data/', .) %>% 
  as_tibble() %>% 
  filter(str_detect(value, 'DP05')) %>% 
  pull(., value) ## coerces tibble back to vector for reading by map_df()

participant_data = 
  cbind(map_df(filenames_DP05, read_csv))
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
## to do 
  ## make first row as column names
  ## extract only population variable per census tract per year
  ## merge with denominator, below
  ## calculate crude density per year (should have 17 measures per census tract, one for each year)

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
| 2010 | 1400000US36061000100 | 36061000100 |       NA |
| 2010 | 1400000US36061000201 | 36061000201 |     52.1 |
| 2010 | 1400000US36061000202 | 36061000202 |     28.3 |
| 2010 | 1400000US36061000500 | 36061000500 |       NA |
| 2010 | 1400000US36061000600 | 36061000600 |     55.4 |
| 2010 | 1400000US36061000700 | 36061000700 |      3.5 |
| 2010 | 1400000US36061000800 | 36061000800 |     66.0 |
| 2010 | 1400000US36061000900 | 36061000900 |      0.0 |
| 2010 | 1400000US36061001001 | 36061001001 |      5.9 |
| 2010 | 1400000US36061001002 | 36061001002 |     32.6 |

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
| 2010 | 1400000US36061000100 | 36061000100 |    NA |    NA |   NA |    NA |   NA |    NA |
| 2010 | 1400000US36061000201 | 36061000201 |  23.9 |  16.6 |  3.8 |  47.0 |  0.0 |  17.2 |
| 2010 | 1400000US36061000202 | 36061000202 |  47.4 |  17.2 |  0.7 |  17.6 |  0.0 |  20.8 |
| 2010 | 1400000US36061000500 | 36061000500 |    NA |    NA |   NA |    NA |   NA |    NA |
| 2010 | 1400000US36061000600 | 36061000600 |  13.6 |   6.9 |  2.8 |  72.3 |  0.1 |   7.0 |
| 2010 | 1400000US36061000700 | 36061000700 |  68.6 |   6.0 |  0.4 |  29.9 |  0.0 |   0.6 |
| 2010 | 1400000US36061000800 | 36061000800 |  10.7 |   1.4 |  0.4 |  87.8 |  0.0 |   1.1 |
| 2010 | 1400000US36061000900 | 36061000900 |  55.9 |   9.8 |  0.0 |  36.7 |  0.0 |   0.0 |
| 2010 | 1400000US36061001001 | 36061001001 |  89.4 |   1.7 |  0.0 |   1.8 |  0.0 |   7.1 |
| 2010 | 1400000US36061001002 | 36061001002 |  27.1 |  21.0 |  1.1 |  13.4 |  0.0 |  41.0 |
