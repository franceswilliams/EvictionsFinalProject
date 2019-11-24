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
  filter(parent_location %in% c("New York County, New York", "Queens County, New York", "Kings County, New York", "Bronx County, New York", "Richmond County, New York"),
         year %in% c(2010:2016),
         population != 0) %>% 
  
  ## selecting and renaming variables
  mutate(pct_nonwhite = (100 - pct_white)) %>% 
  select(geoid, year, poverty_rate, rent_burden, pct_nonwhite_evictiondata = pct_nonwhite, evictions, eviction_filings, eviction_rate, eviction_filing_rate, total_pop_evictiondata = population, renter_occupied_households) %>% 
  filter(renter_occupied_households > 10) # census tracts with few renter households were creating eviction rates of 150, 200, etc.

## note: eviction filing rate calculated (# evictions / # renter occupied households)
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

##### Gentrification (not using for now)

``` r
gentrification =
  read_excel('./data/udp_ny_final_typology_jan_2019.xlsx') %>% 
  filter(startsWith(as.character(geoid), '36')) %>% ## just New York state (FIPS = 36, first two digits)
  rename(id2 = geoid,
         gent_status = Type_1.19) %>% 
  mutate(gent_indicator = if_else(gent_status %in% c('LI - Ongoing Gentrification', 'MHI - Advanced Gentrification'),
                                  1,
                                  0))

# gentrification %>% 
#   arrange(id2) %>% 
#   head(10) %>% 
#   knitr::kable(digits = 11)
```

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
  select(id, id2, name, geography, total_pop, year) %>% 
  filter(total_pop != 0)
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
  select(id2, area_sqmi)
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
  left_join(population_data, area, by = "id2") %>% 
  mutate(density = total_pop / area_sqmi) %>% 

  ## selecting and renaming meaningful variables

  filter(geography %in% c("New York County, New York", "Queens County, New York", "Kings County, New York", "Bronx County, New York", "Richmond County, New York"),
       year %in% c(2010:2016)) %>% 
  select(geoid = id2, year, total_pop_densitydata = total_pop, area_sqmi, density)
```

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
  bind_rows(englang_2010, englang_2011, englang_2012, englang_2013, englang_2014, englang_2015, englang_2016) %>% 
  mutate(geo_id2 = as.numeric(geo_id2),
         year = as.numeric(year)) %>% 
  
  ## restricting and renaming etc.
  select(geoid = geo_id2, year, pct_eng) %>% 
  
  ## bronx, kings, new york, queens, richmond counties - using codes (need to triple-check these match up with County Names)
  filter(substr(geoid, 1,5) %in% c(36005, 36047, 36061, 36081, 36085),
         year %in% c(2010:2016))
```

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
  bind_rows(race_2010, race_2011, race_2012, race_2013, race_2014, race_2015, race_2016) %>% 
  
  ## creating % nonwhite
  mutate(pct_nonwhite_racedata = (100 - white),
         geo_id2 = as.numeric(geo_id2),
         year = as.numeric(year)) %>% 
  
  ## restricting and renaming etc.
  select(geoid = geo_id2, year, pct_nonwhite_racedata) %>% 
  
  ## bronx, kings, new york, queens, richmond counties - using codes (need to triple-check these match up with County Names)
  filter(substr(geoid, 1,5) %in% c(36005, 36047, 36061, 36081, 36085),
         year %in% c(2010:2016))
```

#### Joining datasets

``` r
## creating key for geoid <--> boro
geoid_boro_key = 
  read.csv('./data/EvictionData_NY.csv') %>% 
  janitor::clean_names() %>% 
  filter(parent_location %in% c("New York County, New York", "Queens County, New York", "Kings County, New York", "Bronx County, New York", "Richmond County, New York"),
         year %in% c(2010:2016)) %>% 
  select(geoid, county = parent_location) %>% 
  mutate(boro = recode(county, "New York County, New York" = "Manhattan", 
                               "Queens County, New York" = "Queens", 
                               "Kings County, New York" = "Brooklyn", 
                               "Bronx County, New York" = "Bronx", 
                               "Richmond County, New York" = "Staten Island")) %>% 
  distinct()
```

``` r
## all rows regardless of NA, using outer (full) join

joined_data = 
  eviction %>% 
  full_join(., density_data, by = c("geoid", "year")) %>% 
  full_join(., englang_data,  by = c("geoid", "year")) %>% 
  full_join(., race_data,  by = c("geoid", "year")) %>% 
  inner_join(., geoid_boro_key, by = "geoid") %>% 
  
  ## reordering variables
  
  select(geoid, year, boro, county, ## geo/time
         evictions, eviction_rate, eviction_filings, eviction_filing_rate, ## outcomes
         pct_nonwhite_evictiondata, pct_nonwhite_racedata, rent_burden, pct_eng, density, ## predictors
         renter_occupied_households, total_pop_evictiondata, total_pop_densitydata, area_sqmi, ## calculation variable
         poverty_rate ## contextual variable
         )

## just brooklyn
joined_data_bklyn =
  joined_data %>% 
  filter(boro == "Brooklyn")

## all rows with all needed information, no missing
joined_data_bklyn_nomissing =
  joined_data_bklyn %>% 
  drop_na()
```

``` r
## just to clean up your environment - uncomment the below chuck with Ctrl+Shift+C, then run, then re-comment with Ctrl+Shift+C

rm(area,
   englang_2010, englang_2011, englang_2012, englang_2013, englang_2014, englang_2015, englang_2016,
   race_2010, race_2011, race_2012, race_2013, race_2014, race_2015, race_2016,
   filenames_DP05)
```

## Regressions (Question 3)

The generalized estimating equation for eviction rates (continuous) on
each of our predictors of interest are as follows:

E(evictions\_ij) = β0 + β1(year\_ij) + β2(predictor\_ij) +
β3(year\_ij\*predictor\_ij)

Regression function w/ output:

``` r
univariate = function(x){
  
  gee_line = geeglm(eviction_rate ~ x + year + year*x, data = joined_data_bklyn_nomissing, id = geoid, family = gaussian, corstr = "unstructured") %>%
  broom::tidy(conf.int = FALSE, conf.level = 0.95,
  exponentiate = FALSE, quick = FALSE)
  
  tibble(
    b0_est = gee_line[[1,2]],
    b0_p = gee_line[[1,5]],
    b1_est = gee_line[[2,2]],
    b1_p = gee_line[[2,5]],
    year_est = gee_line[[3,2]],
    year_p = gee_line[[3,5]],
    interaction_est = gee_line[[4,2]],
    interaction_p = gee_line[[4,5]]
  )
  
}

pred_list = list("pct_eng" = pull(joined_data_bklyn_nomissing, pct_eng), 
              "rent_burden" = pull(joined_data_bklyn_nomissing, rent_burden), 
              "pct_nonwhite" = pull(joined_data_bklyn_nomissing, pct_nonwhite_racedata), 
              "pop_density" = pull(joined_data_bklyn_nomissing, total_pop_densitydata)
              )

gee_output = map(pred_list, univariate) %>%
  do.call(rbind, .) %>%
  knitr::kable(digits = 2)

gee_output
```

|               |  b0\_est | b0\_p | b1\_est | b1\_p | year\_est | year\_p | interaction\_est | interaction\_p |
| ------------- | -------: | ----: | ------: | ----: | --------: | ------: | ---------------: | -------------: |
| pct\_eng      |   142.61 |  0.00 |    0.40 |  0.68 |    \-0.07 |    0.00 |             0.00 |           0.67 |
| rent\_burden  | \-110.04 |  0.47 |    7.47 |  0.07 |      0.05 |    0.47 |             0.00 |           0.07 |
| pct\_nonwhite |    20.98 |  0.44 | \-12.94 |  0.00 |    \-0.01 |    0.45 |             0.01 |           0.00 |
| pop\_density  |   202.14 |  0.00 |  \-0.02 |  0.39 |    \-0.10 |    0.00 |             0.00 |           0.39 |

**Example interpretation:**

  - For each 1% increase in people who speak English less than “very
    well” in a census tract, the eviction rate increases by 0.40, on
    average. However, this relationship is not significant at the 5%
    level, and does not appear to change over time.

  - For each 1% increase in the non-White population, the eviction rate
    appears to decrease by 12.94, and this relationship is significant
    at the 5% level. However, this relationship appears to be
    attenuating over time.
