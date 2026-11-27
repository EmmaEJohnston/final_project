Data Cleaning
================
Everyone
2026-11-17

## Load packages:

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.2.1     ✔ readr     2.2.0
    ## ✔ forcats   1.0.1     ✔ stringr   1.6.0
    ## ✔ ggplot2   4.0.3     ✔ tibble    3.3.1
    ## ✔ lubridate 1.9.5     ✔ tidyr     1.3.2
    ## ✔ purrr     1.2.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(tidycensus)
library(lubridate)
library(sf)
```

    ## Linking to GEOS 3.13.0, GDAL 3.8.5, PROJ 9.5.1; sf_use_s2() is TRUE

``` r
library(tigris)
```

    ## To enable caching of data, set `options(tigris_use_cache = TRUE)`
    ## in your R script or .Rprofile.

``` r
library(patchwork)
```

## Load and process 311 data:

``` r
pigeon_311_df = read_csv("data/contains_'Pigeon'_20261117.csv") |> 
  janitor::clean_names() |> 
  select(unique_key, created_date, closed_date, problem_formerly_complaint_type, 
problem_detail_formerly_descriptor, location_type, incident_zip, latitude, longitude, borough, status,  
resolution_description, resolution_action_updated_date, location) |> 
  mutate(
    created_date = ymd_hms(created_date),
    closed_date = ymd_hms(closed_date),
    complaint_type = factor(problem_formerly_complaint_type),
    complaint_descriptor = factor(problem_detail_formerly_descriptor),
    location_type = factor(location_type),
    incident_zip = factor(incident_zip),
    borough = factor(borough),
    length_open = difftime(closed_date, created_date, units = "days")
  ) |> 
  select(-contains("problem")) |> 
  relocate(unique_key, created_date, closed_date, length_open, status, location, latitude, longitude, incident_zip, borough, location_type, complaint_type, complaint_descriptor, resolution_description, resolution_action_updated_date) 
```

    ## Rows: 8644 Columns: 30
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (23): Created Date, Closed Date, Agency, Agency Name, Problem (formerly ...
    ## dbl  (4): Unique Key, Incident Zip, Latitude, Longitude
    ## num  (2): X Coordinate (State Plane), Y Coordinate (State Plane)
    ## lgl  (1): Due Date
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

## Import income data:

``` r
## Intialize API key if necessary. Otherwise, keep line commented out:
## census_api_key("f8e602dfbaa32070e224824940f2b356d38db646", overwrite = TRUE, install = TRUE)

## Pull data from 2024:
income_df =
  get_acs(
    geography = "tract",
    variable     = "B19013_001",
    state     = "NY",
    county    = c("Bronx", "Kings", "New York", "Queens", "Richmond"),
    year      = 2024,        
    survey    = "acs5",
    cache_table = TRUE,
    geometry = TRUE) |> 
  st_transform(4326) |> 
  select(geometry, estimate, moe)
```

    ## Getting data from the 2020-2024 5-year ACS

    ## Downloading feature geometry from the Census website.  To cache shapefiles for use in future sessions, set `options(tigris_use_cache = TRUE)`.

## Merge income data with the 311 data:

``` r
## Make a list of sf data:
pigeon_311_sf = pigeon_311_df |> 
  filter(!is.na(longitude)) |> 
  st_as_sf(coords = c("longitude", "latitude"), crs = 4326) |> 
  select(unique_key, geometry)

## Create a merged data frame of unique id's and census data:
pigeon_311_merge =  pigeon_311_sf |> 
  st_join(income_df, left = T)

## Merge with the pigeon df:
pigeon_311_df = pigeon_311_df |> 
  left_join(pigeon_311_merge, by = "unique_key") |> 
  rename(median_inc_est = estimate, median_inc_moe = moe)
```

## Add E-Bird Data:

``` r
## Load and process ebird data:
pigeon_ebird_df = read_csv("data/ebird.csv.zip") |> 
  select(gbifID, occurrenceID, recordedBy, eventDate, individualCount, 
         locality, occurrenceStatus, decimalLatitude, decimalLongitude) |> 
  janitor::clean_names() |> 
  rename(latitude = decimal_latitude, longitude = decimal_longitude) |> 
  mutate(event_date = mdy(event_date)) |> 
  st_as_sf(coords = c("longitude", "latitude"), crs = 4326)
```

    ## Multiple files in zip: reading 'ebird.csv'
    ## Rows: 387975 Columns: 50
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (27): datasetKey, occurrenceID, kingdom, phylum, class, order, family, ...
    ## dbl   (7): gbifID, individualCount, decimalLatitude, decimalLongitude, day, ...
    ## lgl  (15): infraspecificEpithet, verbatimScientificNameAuthorship, coordinat...
    ## dttm  (1): lastInterpreted
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
## Load nyc boundary:
nyc <- counties(state = "NY", cb = TRUE) |>
  filter(COUNTYFP %in% c("005", "047", "061", "081", "085")) |>
  st_transform(st_crs(pigeon_ebird_df)) |>
  select(county = NAME)
```

    ## Retrieving data for the year 2025

``` r
## Subset the ebird data by nyc boundaries:
pigeon_ebird_df <- st_join(pigeon_ebird_df, nyc, left = FALSE) 
```

## Data is loaded and cleaned. Use pigeon_ebird_df for e-bird data and pigeon_311_df for 311 data.

## Next steps:

1.  Create new rmd with final report.
2.  Use API to load pigeon data.
3.  Have rmd with dashboard code.
4.  Visualizations:

- Mapping - GM
- Temporal Trends
- Complaint resolution length.
- Complaint closure randomly dispersed???
  - Local Moran’s I

##### Planned analyses:

1)  Temporal trend of complaints

- 6 years of data - have the amount of complaints changed?

Yearly trends in the number of complaints

``` r
complaints_by_year = 
  pigeon_311_df |> 
  mutate(
    year = year(created_date)
  ) |> 
  count(year, name = "n_reports")

complaints_by_year |> 
  ggplot(aes(x = year, y = n_reports)) +
  geom_line() +
  geom_point() +
  scale_x_continuous() +
  scale_y_continuous() +
  labs(
    title = "311 Pigeon Related Complaints per Year",
    x = "Year",
    y = "Number of Pigeon Related Complaints"
  ) +
  theme_minimal()
```

![](data_cleaning_files/figure-gfm/unnamed-chunk-6-1.png)<!-- --> The
number of pigeon-related 311 complaints steadily increased between 2020
and 2025. 2026 not over yet –\> looks like it plateued but will most
likely still be an increase (albeit a smaller one)

Seasonal trends in the level of complaints, across all years

``` r
seasonal_complaints = 
  pigeon_311_df |> 
  mutate(
    month = month(created_date, label = TRUE, abbr = TRUE)
  ) |> 
  count(month, name = "n_reports")

seasonal_complaints |> 
  ggplot(aes(x = month, y = n_reports, group = 1)) +
  geom_line() +
  geom_point() +
  scale_y_continuous() +
  labs(
    title = "311 Pigeon Related Complaints per Month Across Years",
    x = "Month",
    y = "Number of Pigeon Related Complaints"
  ) +
  theme_minimal()
```

![](data_cleaning_files/figure-gfm/unnamed-chunk-7-1.png)<!-- --> Each
year, the number of 311 pigeon related complaints peaks in late summer
before reaching a low point in December.

2)  Geographical distribution of complaints

- By borough and neighborhood (incorporate income)
- Temporal trends of pigeon complaints by borough

3)  Type and details of complaints

- Problem: Unsanitary pigeon condition versus trapping pigeon
- Problem detail: Pigeon waste versus pigeon odor

``` r
complaint_type_plot = 
  pigeon_311_df |> 
  count(complaint_type, name = "n_reports") |> 
  ggplot(aes(x = complaint_type, y = n_reports)) +
  geom_col() +
  labs(
    title = "311 Pigeon Related Complaint Types",
    x = "Complaint Type",
    y = "Number of Pigeon Related Complaints"
  ) +
  theme_minimal()

complaint_descriptor_plot = 
  pigeon_311_df |> 
  filter(!is.na(complaint_descriptor), complaint_descriptor != "N/A") |>
  count(complaint_descriptor, name = "n_reports") |> 
  ggplot(aes(x = complaint_descriptor, y = n_reports)) +
  geom_col() +
  labs(
    title = "311 Pigeon Related Complaint Descriptors",
    x = "Complaint Descriptor",
    y = "Number of Pigeon Related Complaints"
  ) +
  theme_minimal()

complaint_type_plot + complaint_descriptor_plot
```

![](data_cleaning_files/figure-gfm/unnamed-chunk-8-1.png)<!-- --> I can
make all of these plots more attractive once we commit to including
them!!

4)  Pigeon complaint by building type

- Residential building versus commercial building versus school versus
  street area, etc.

``` r
pigeon_311_df |> 
  count(location_type, name = "n_reports") |> 
  mutate(location_type = fct_reorder(location_type, n_reports)) |>
  ggplot(aes(x = location_type, y = n_reports)) +
  geom_col() +
  coord_flip() +
  scale_y_continuous() +
  labs(
    title = "311 Pigeon Related Complaints by Building Type",
    x = "Building Type",
    y = "Number of Pigeon Related Complaints"
  ) +
  theme_minimal()
```

![](data_cleaning_files/figure-gfm/unnamed-chunk-9-1.png)<!-- --> Note:
we should probably combine some of these categories (ie. 1-2
FamilyDwelling with 1-2 Family Dwelling)

5)  Brief scientific overview of pigeon species in NYC (EJ)

6)  Are there correlations between where pigeon sightings are reported
    on eBird and where people file pigeon complaints?

7)  Use the API and have a dashboard of complaint monitoring

8)  How long it takes for a 311 Pigeon call to get resolved

- Overall and by geography (incorporate income)
