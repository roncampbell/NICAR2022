# Gathering and cleaning data in R

This is the third of a three-part introduction to <code>R</code>. We'll import California demographic data from the web that requires some cleaning. We'll also analyze airport flight delays that will make sense only when it is merged with other data. With a little practice, these skills will become second nature. But first let's load the R packages we'll need. The ace IRE/NICAR staff has already installed them for the conference.

(Hint: If you're not at the conference, remember that you must install packages before you can use them. To do so, make sure you have a working web connection and then type at the console prompt this command: <code>install.packages("xxx")</code> where xxx is the name of the package.)

> library(tidyverse)

> library(xml2)

The xml2 package is a "non-core" part of the <code>tidyverse</code>. It's one of the dozen or so elements of the tidyverse that must be individually activated. Core elements like <code>ggplot</code>, <code>dplyr</code>, <code>tidyr</code> and <code>stringr</code> are activated with the single command <code>library(tidyverse)</code>.

The California Department of Finance tracks demographic trends using census, birth and death records to project population trends. We'll import a file that covers all 58 counties from 1970 through 2050.

> <code>download_html("http://tinyurl.com/y82z52uw", "ca_popest.csv")</code>

This XML2 command does three things. It tells R to download some HTML; it directs the browser to a particular website; and it downloads the content to a local file, to which we give a name. 

> <code>CA_popest <- read_csv("ca_popest.csv")
Parsed with column specification:
cols(
  fips = col_double(),
  county = col_character(),
  year = col_double(),
  age = col_double(),
  pop_female = col_double(),
  pop_male = col_double(),
  pop_total = col_double()
)</code>
  
> View(CA_popest) 

The dataframe contains 474,125 rows and has 7 columns. If you're familiar with FIPS codes, you'll immediately notice a problem with the (lower-case) fips column in CA_popest. The FIPS code for California is "06", but it appears here without the leading zero. This is a potential problem if we want to merge the data or map it. 

Fortunately it's easy to solve with the core tidyverse package stringr, which manipulates strings. The current fips field is 4 characters wide and lacks a leading zero. The field should be 5 characters wide (2 characters to identify the state, 3 to identify the county). Since the state is California in all cases, the first two characters should be "06"; we know from sorting the field in ascending and descending order that every row begins with a "6". All we need to do is widen the field from four characters to five and place an "0" in the first position, before the "6". 

> CA_popest$FIPS <- str_pad(CA_popest$fips, 5, "left", pad = "0")

This command creates a new field, FIPS (all caps), based on the existing field fips (lower case); it's 5 characters wide, padded on the left with the character "0". If you use the command View(CA_popest), you'll see the new field at the far right of the dataframe. Now all we have to do is move it to the left, where it will be more convenient. We can do that with column indexing.

> <code>CA_popest <- CA_popest[,c(8,1,2:7)]</code>

A reminder: Whenever you see numbers in R inside square brackets, separated by a comma, they refer to rows and columns; rows are to the left of the comma, columns to the right. In the code above, we move column 8 in front of column 1 while doing nothing to the rows. We use the "c()" to indicate we're combining or concatenating several objects.

We have a wealth of information here. If you're say, a school district administrator in Calaveras County, you can use this data to see how many 5-year-old children to expect in your kindergarten classes in 2025. But for most people this is too much of a good thing. We need to summmarize. We also want to be able to document our work. We can do both with a script.

To create a script in R Studio, click on the green "+" button at the upper left corner of the screen and then click on the words "R Script". I usually begin a script by listing the packages needed for that script; this way I can run the script independently of any other work I am doing. To stay organized, I usually keep a project in a folder or a set of folders on my computer; all the R scripts go in a subdirectory called, naturally, Scripts. 

Here's the code for summarizing population change by county by decade. 

<code>library(tidyverse)              # make sure tidyverse is on
  
CA_popsum <- CA_popest %>%      # create new dataframe

  group_by(county, year) %>%    # grouping variables
  
  filter(year == 1970 | year == 1980 | year == 1990 | year == 2000 | 
           year == 2010 | year == 2020 | year == 2030 | year == 2040 | 
           year == 2050) %>%    # once-a-decade data
           
  summarise(
  Population = sum(pop_total, na.rm=T)) # get population for each year</code>

This script produces a 522-line dataframe, with a separate line for each year and county.

![](https://github.com/roncampbell/NICAR2019/blob/images/Popest1.png?raw=true)

I commented each line of this script using a hash mark (#). You can comment scripts as much (or as little) as you want. Just remember -- you may be going back to your script months later and wondering "What was I thinking when I wrote this?" Your comments, preceded by a hash (#) mark will be valuable clues. I use old scripts for two reasons. The first is to recycle old data for new stories. The second and by far the more important is to tweak the code for some new and unexpected use. Think of the data that the code produces as bricks that you can tear down and reuse a limited number of times and code as tools that you can employ over and over and over again in an endless combination of ways. The more you use the code, the more ways you will think of using it.

Now think about the format of this dataframe. We've summarized it, but it's still very hard to use. The data would make more sense if each year were in its own column. We can do this with the tidyverse package tidyr. This package has two main functions, gather() and spread(). The former takes wide tables and makes them long; the latter takes long tables and makes them wide. They both work by using key:value pairs. 

Here's an example: On line 1 of CA_popsum the key is 1970 and the value is 1072985; these two values are associated with each other (and with ALAMEDA). We'll make a new dataframe built of key:value pairs like this.

<code>library(tidyverse)
  
CA_popsum1 <- CA_popsum %>% 

  spread(key=year, value=Population)</code>
  
The result is a dataframe with 58 rows -- one for each county -- and 10 columns, the lefthand column for county names and the remaining nine for the decadal years from 1970 through 2050. 
  
![](https://github.com/roncampbell/NICAR2019/blob/images/Popest2.png?raw=true)

In this format it's easy to measure year-by-year and county-by-county population changes for all 58 counties. But there are better ways to grasp the changes transforming California. Click on the 2020 column: Click once and you'll see more than 20 counties with fewer than 100,000 residents each; click a second time so the column is in descending order and you'll see 10 counties with more than 1 million residents each. The top five counties account for more than half of California's population.

We can discover a lot about California (and learn some R in the process) just by focusing on the growth of those five big counties over the past five decades. To get there, we'll filter the CA_popsum dataframe for the top five counties -- Los Angeles, San Diego, Orange, Riverside and San Bernardino -- and use the counterpart to tidyr's spread() function, gather(), to create new Year and Population columns. Since the year columns are numeric, a troublesome issue for R that can trigger errors, we'll refer to those columns by their index numbers.

<code>library(tidyverse)
CA_top5 <- CA_popsum1 %>% 
  filter(county %in% c("LOS ANGELES", "SAN DIEGO", "ORANGE", "RIVERSIDE", 
                       "SAN BERNARDINO")) %>% 
  gather(key=Year, value=Population, c(2:10))</code>

We get a 45-line, 3-column dataframe that's ideal for charting.

![](https://github.com/roncampbell/NICAR2019/blob/images/Poptest3.png?raw=true)

Let's make a chart and see if there's a pattern. And since we've got estimates going out to 2050, let's limit the chart to 2020, since we're more confident of projections for that year than for 2030, 2040 and 2050.

> <code>ggplot(filter(CA_top5, Year <=2020), aes(x=Year, y=Population, fill=county)) +
geom_bar(stat="identity") +
labs(title="California population growth, 1970-2020, 5 largest counties",
                                            caption="Source: State Department of Finance")</code>
  
![]()


There are in descending order, you'll see that the top five counties each are larger than many states; together they comprise more than half of California's population. California's growth has slowed over time and the growth center has shifted from the coast to inland counties. We can see that by calculating the percentage change for each decade from 1970 through 2020 (where we have a very good estimate of the projected population). But another problem creeps: Most of the column headers are numbers -- not something reasonable like "Y1970" but simply 1970. R doesn't like numbers as variables.

The janitor package, which we encountered in the previous class, can fix that. But there's an alternative. We can enclose them with backticks, which you'll find near the upper lefthand corner of the keyboard, next to the 1 / ! key; it's the lower-case to the upper-case tilde (~). With that, here's a script to calculate population changes by decade for every California county, while taking care of those messy column headers.

<code>library(tidyverse)
CA_popchange <- CA_popsum1 %>% 
  mutate(
    Ch70_80 = ((`1980` - `1970`) / `1970`) * 100,
    Ch80_90 = ((`1990` - `1980`) / `1980`) * 100,
    Ch90_00 = ((`2000` - `1990`) / `1990`) * 100,
    Ch00_10 = ((`2010` - `2000`) / `2000`) * 100,
  Ch10_20 = ((`2020` - `2010`) / `2010`) * 100)  # note backticks for years</code>
  
With these new columns we can look for long-term changes in the state's population growth. Since most of that growth has occurred in a handful of counties, we'll use the dplyr filter() command to choose particular counties and the select() command to choose columns. We'll also arrange our selection to see the counties that grew the fastest since 2010.

<code>library(tidyverse)
Pattern <- CA_popchange %>% 
  select(county, `1970`, `2010`, Ch70_80, Ch80_90, Ch90_00, Ch00_10, Ch10_20) %>% 
  filter(county %in% c("LOS ANGELES", "SAN DIEGO", "ORANGE", "SANTA CLARA",
                       "RIVERSIDE", "SAN BERNARDINO", "FRESNO", "KERN", 
                       "SACRAMENTO", "SAN JOAQUIN")) %>% 
  arrange(desc(Ch10_20))</code>
  
  

