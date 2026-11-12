Final Project Proposal
================
Emma, Giuliana, Max, Jonathan;
2026-11-12

#### The group members (names and UNIs)

Giuliana Motta (gem2176), Jonathan Morris (jsm2298), Max Hyman
(mjh2289), Emma Johnston (eej2124)

#### The tentative project title

311, What’s Your Emergency? Pigeons.

#### The motivation for this project

As an avid NYC birder, I look through my binoculars each weekend. And
what do I see? Pigeons. From a distance, these birds are, at most, a
minor nuisance (as they scare away other birds). However, to many
people, they are enough of a bother to submit pigeon-related complaints
via 311. We were curious to investigate the issues these pigeons cause
in the lives of New Yorkers.

#### The intended final products

We want to describe and analyze pigeon-related complaints in NYC using
publicly available data from 311 Complaint Data which feature the word
“pigeon.” Our project will include static visualizations which
characterize complaints by time, location, income level, and building
type. Furthermore, we will identify the species of pigeons which are
present in NYC using eBird. We also plan to create a dashboard which
utilizes the API available via the 311 data source to provide real-time,
pigeon-related complaint monitoring. Finally, we will explore factors
which may be associated with pigeon-related complaints and 311’s
response time, including time, building type, location/borough, and
socioeconomic factors, such as income.

#### The anticipated data sources

311 Pigeon Complaint Data. NYC Open Data.
<https://data.cityofnewyork.us/Social-Services/contains-Pigeon-/iea7-sinw/about_data>

EOD – eBird Observation Dataset. Cornell Lab of Ornithology.
<https://www.gbif.org/dataset/4fa7b334-ce0d-4e88-aaae-2e0c138d049e>

Income Data Tables. United States Census Bureau.
<https://www.census.gov/topics/income-poverty/income/data/tables.html>

#### The planned analyses / visualizations / coding challenges

##### Planned analyses:

1)  Temporal trend of complaints

- 16 years of data - have the amount of complaints changed?
- Yearly and seasonal trends in the level of complaints

2)  Geographical distribution of complaints

- By borough and neighborhood (incorporate income)
- Temporal trends of pigeon complaints by borough

3)  Type and details of complaints

- Problem: Unsanitary pigeon condition versus trapping pigeon
- Problem detail: Pigeon waste versus pigeon odor

4)  Pigeon complaint by building type

- Residential building versus commercial building versus school versus
  street area, etc.

5)  Brief scientific overview of pigeon species in NYC

6)  Are there correlations between where pigeon sightings are reported
    on eBird and where people file pigeon complaints?

7)  Use the API and have a dashboard of complaint monitoring

8)  How long it takes for a 311 Pigeon call to get resolved

- Overall and by geography (incorporate income)

##### Visualizations:

- Tables and bar charts for counts (especially location type and
  neighborhood)
- Line charts showing temporal trends
- Histogram for days to resolve a complaint, potentially faceted or
  stratified by geography or income level
- Maps of pigeon sightings and complaints

##### Potential coding challenges:

- Amount of pigeon data available (we have 16 years of data, but
  realistically how many people are submitting pigeon complaints and how
  representative are those people?)
- Understanding how data are categorized (e.g., pigeon waste versus
  pigeon odor or 3+ family apartment building versus 3+ family mixed use
  building)?
- Co-locating pigeon species data to NYC using longitude and latitude
- Working with dates and timestamps for when complaints were opened and
  closed
- Some complaints have not been closed and have missing data

#### The planned timeline

- 11/17 Meet to plan data cleaning (Room 657)
- 11/22 Tidy the data
- 12/2 Meet and figure out analysis
- 12/6 Do the analysis, write the report, make visualizations
- 12/10 Report, website, and screencast
- 12/12 Submit everything
