Evictions Project Proposal
================
11/6/2019

### **Project Title:** Demographic and housing correlates of evictions in NYC, 2000-2016

### **Project Overview:**

**Group members:** Frances Williams (fw2334), Gloria Hu (gh2518), Naama
Kipperman (nk2814), Will Simmons (wes2121)

**Motivation for this project:** Evictions represent a major source of
housing instability. We are interested in census tract- and
neighborhood-level predictors of eviction rates in NYC.

**Intended final products:** A website providing an overview of eviction
rate predictors at the census tract level, as well as interactive maps
illustrating the relationship between eviction rates and census-tract
and neighborhood-level predictors.

### **Anticipated Data Sources:**

  - Primary source: [Eviction Lab](https://evictionlab.org/)

  - For gentrification measures: [Mapping Displacement and
    Gentrification in the New York Metropolitan Area (Urban Displacement
    Project)](https://www.urbandisplacement.org/maps/ny)

  - For English language usage and population density: [American
    FactFinder](https://factfinder.census.gov/faces/nav/jsf/pages/guided_search.xhtml)

  - Supplemental data source: [NYC Open
    Data](https://www1.nyc.gov/site/planning/data-maps/open-data/dwn-nynta.page)

### Planned Analyses and Visualizations

  - Analyses: Regressions of the following predictors on our primary
    outcome, *eviction rates*.
    
      - **Rent burden** (in dataset)
    
      - **Percent Non-white** (in dataset)
    
      - **Percent of English-language speakers** (external dataset)
    
      - **Population density** (external dataset)
    
    *Note: We’ll then compare the above simple linear regression models
    via cross-validation to gauge which, if any, is the best predictor
    of eviction rates in NYC.*

  - Visualizations:
    
      - Descriptive
        
          - Line graph of eviction rates over time by neighborhood
            (assuming data available for enough years); if not enough
            data, then grouped bar chart of eviction rates
        
          - Map of eviction rates by census tract over time (choropleth)
        
          - Stacked bar chart (or line graph) of discrepancy between
            eviction filing rates and eviction rates, by neighborhood
    
      - Analytic
        
          - Dynamic maps of each predictor and eviction rate by
            neighborhood/census tract, over time
        
          - (If time permits) Neighborhood-level evictions by
            gentrification indices (external dataset)

### Anticipated Challenges

  - Integration of existing R and GIS knowledge to create cohesive,
    self-explanatory geographic visualizations

  - Missing data across multiple datasets in use - making sure that:
    
      - Models are comparable given potential for different missing data
        structures
    
      - Models are interpretable both in aggregate across NYC and for
        specific Census Tracts
    
      - Addressing the fact that our data exist across time - how do we
        model predictor-outcome relationship from 2000-2016?

### Planned Timeline

  - **November 7**: Confirm appropriate data sources

  - **November 12**: (1) Table of contents for deliverable website
    (pages, subpages, content) (2) All relevant datasets loaded into
    repo

  - **November 16**: Have cleaned and merged dataset, initial
    descriptive plots, and website layout

  - **November 30**: Visualizations complete

  - **December 1**: Final write-up complete

### Background Reading:

[The Gentrification of Gotham
(CityLab)](https://www.citylab.com/life/2017/04/the-gentrification-of-gotham/524694/)

[It’s Manhattan’s Last Affordable Neighborhood. But for How Long?
(NYT)](https://www.nytimes.com/2019/09/27/nyregion/inwood-manhattan-affordable-housing.html)
