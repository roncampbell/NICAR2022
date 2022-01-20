# Importing and cleaning data in R

All the data and maps you need for this class are already in this Github repo. But learning how to find, import and, most importantly, clean Census data are valuable skills. We'll take a little time to demonstrate, using the files for this class as examples.

First, as always with R, we'll grab a few packages to make life easy - the tidyverse and two Census-centric libraries, tidycensus and tigris. The only thing you need, aside from the packages themselves, is a free Census API key; you can get that by signing up here: <https://api.census.gov/data/key_signup.html> You can install it as a default in tidycensus.

Remember that you must install packages before you can use them. To do so, make sure you have a working web connection and then type this command at the console prompt: <code>install.packages("xxx")</code> where xxx (in quote marks) is the name of the package.

> library(tidyverse)

> library(tidycensus)

> library(tigris)

First, we'll import maps (shapefiles) using the tigris package. By default, Census Bureau maps extend offshore; we want maps trimmed to the coastline, known as cartographic boundary (CB) maps. We'll begin by importing a shapefile of Georgia counties.

> ga_counties <- counties("GA", cb = TRUE)

Next, we'll filter that map to get just the eight counties in the Atlanta metro area, using their FIPS codes to identify them.

> metro_counties <- ga_counties %>% 
  filter(COUNTYFP %in% c("121","135","067","089","063","057",
                         "117","151"))

We also want 2020 census tracts for the metro counties. We have to specify the year because the default is 2019.

> metro_tracts20 <- tracts("GA", county = c("121","135","067","089",
                                          "063","057","117","151"), 
                         cb = TRUE, year = 2020)

Finally, we'll filter the metro area tract maps to get just the tracts in Fulton County. Again, we'll use the FIPS code.

> fulton_tracts20 <- metro_tracts20 %>% 
  filter(COUNTYFP == '121')

Data journalists do not live by maps alone. We also hunger for data to hang on those maps. And for this, we have tidycensus to lead us through the Census Bureau maze. We're going to grab a slice of the 2020 redistricting file for Georgia - specifically the Hispanic and NonHispanic by Race file (Table P2). That table has 73 columns. We're going to drastically pare it down to just a handful. If you want to know the number of people of six races by census tract, you're on your own.

use tidycensus to grab redistricting file by county
> ga_race20 <- tidycensus::get_decennial(geography = "county", 
                                       year = 2020,
                                       table = 'P2',
                                       cache_table = TRUE,
                                       state = '13',
                                       geometry = FALSE) 

simplify the categories from 73 to 9
> ga_races <- ga_race20 %>% 
  filter(variable %in% c("P2_001N", "P2_002N", "P2_005N", "P2_006N",
                         "P2_007N", "P2_008N", "P2_009N",
                          "P2_010N", "P2_011N")) %>% 
  pivot_wider(names_from = variable, values_from = value)

replace census variables with labels
use `load_variables(2020, 'pl', cache=TRUE)` for reference 
> colnames(ga_races)[2] <- 'County'
> 
> colnames(ga_races)[3] <- 'Total'
> 
> colnames(ga_races)[4] <- 'Hispanic'
> 
> colnames(ga_races)[5] <- 'White'
> 
> colnames(ga_races)[6] <- 'Black'
> 
> colnames(ga_races)[7] <- 'AmerInd'
> 
> colnames(ga_races)[8] <- 'Asian'
> 
> colnames(ga_races)[9] <- 'PacIsl'
> 
> colnames(ga_races)[10] <- 'Other'
> 
> colnames(ga_races)[11] <- 'Multiracial'

calculate percentages for largest racial groups
> ga_races <- ga_races %>% 
  mutate(White_per = 100 * (White / Total),
         Black_per = 100 * (Black / Total),
         Hispanic_per = 100 * (Hispanic / Total),
         Asian_per = 100 * (Asian / Total))
         
Getting a filtered table for the eight Atlanta metro counties is easy now that we've built the statewide table. All we need to do is check the FIPS codes for the metro counties.

> metro_co_races <- ga_races %>% 
  filter(GEOID %in% c('13057', '13063', '13067', '13089', '13117',
                      '13121', '13135', '13151'))
                      
Now let's get race data at the tract level for the metro counties. We'll use tidycensus again. We specify the FIPS codes for the state ('13') and the counties separately this time.

> metro_tract_race <- get_decennial(geography = "tract", 
                                year = 2020,
                                table = 'P2',
                                cache_table = TRUE,
                                state = '13',
                                county = c('057', '063', 
                                           '067', '089', 
                                           '117',
                                           '121', '135', 
                                           '151'))
                                           
The data frame is a bit messy. It looks like this.

![](https://github.com/roncampbell/NICAR2022/blob/images/metro_tract_race.png)

We can clean it up with just a few lines of code:

> metro_tract_race <- metro_tract_race %>% 
  mutate(NAME = str_remove(NAME, ", Georgia"),
         County = str_extract(NAME, "[A-Z,a-z]+ County$"),
         NAME = str_remove(NAME, ", .*$"))
         
> metro_tract_race <- metro_tract_race[,c(1,5,2:4)]
> colnames(metro_tract_race)[3] <- 'Tract'



