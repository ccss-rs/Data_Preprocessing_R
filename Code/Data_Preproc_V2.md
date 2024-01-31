---
title: "Data_Preprocessing"
author: "Aishat Sadiq"
date: "February 7, 2024"
output:
  ioslides_presentation:
    incremental: true
  slidy_presentation:
    incremental: true
    self_contained: false
    smart: false
    keep_md: true
editor_options:
  markdown:
    wrap: 72
  chunk_output_type: inline
---

## Houston, we have a problem! \| Big Data

How much data is created each day?

[It's estimated that 1.7 MB of data are created every second for every
person on earth---the same amount of data needed to store an 850 page
book, per second. Researchers believe that data consumption will increase by 23% annually until 2025](https://time.com/6108001/data-protection-richard-stengel/)

## Folder structure

subdirectories: raw data, processed data, code, documentation

-   getwd, working directory should be code folder

```{r, eval=TRUE, echo=TRUE}
###### run getwd()
getwd()

###### run list.files() to see all files in working directory
list.files() 

###### Run list.files() with pattern = "Preproc" to find specific files
list.files(pattern = "Preproc")

```

-   relative paths and reproducibility

Data Structure

-   A `data.frame` is a collection of vectors of identical lengths. Each
    vector represents a column, and each vector can be of a different
    data type (e.g., characters, integers, factors).

-   You should separate the original data (raw data) from intermediate
    datasets that you may create for the need of a particular analysis.
    For instance, you may want to create a `data/` directory within your
    working directory that stores the raw data, and have a
    `data_output/` directory for intermediate datasets and a
    `figure_output/` directory for the plots you will generate.

## Why Does It Matter?

**Tidy data...**

1.  Makes it easier to focus on manipulating and analyzing data

2.  Makes data visualization simpler and more intuitive

3.  Help ML algorithms perform and learn more effectively

> "Modeling is the driving inspiration of this work because most
> modeling tools work best with tidy datasets." Hadley Wickham ----

## Load Packages/Libraries

-   Base R vs. packages

-   Tidyverse package was created by Hadley Wickham. Learn more about
    the 'tidyverse' online at <https://www.tidyverse.org> or
    <https://github.com/tidyverse/tidyverse>

-   Key information: Version, Depends, Imports

```{r Tidyverse package, eval = TRUE, results='markup'}

## install.packages("tidyverse")
library(tidyverse)
library(help = "tidyverse") 

```

## Import messy data

-   Easiest Option: Download from github
-   Intermediate Option: Download directly from site: [Harvard
    Dataverse\>Mass Mobilization Data Project
    Dataverse](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/HTTWYL)

Raw Data Files:

-   mmALL_073120_csv.tab - .csv file

-   mmALL_073120_v16.tab - Stata file

-   Both have the same number of variables and observations!

-   Click the triangle to the right of the download/access file icon \>
    Select RData\*

-   If the original file was created in state it is usually better to
    download the Stata file & convert in RStudio to maintain attributes
    like column labels.

## Import messy data cont.

-   Depending on your file type, "read in" the data.

```{r Import raw data, eval = TRUE}
###### Method 1 ###### 
# Mass_Mobilizaton_Data <- haven::read_dta("Mass_Mobilizaton_Data.dta")

###### Method 2 ###### 
load("../Data/mmALL_073120_csv.RData")
View(mmALL_073120_csv)

###### Method 3 w/dupes ###### 
mmALL_073120_csv <- read_csv("../Data/mmALL_073120_csv.csv")
View(mmALL_073120_csv)

```

## Background on Mass Mobilization (MM)

The Mass Mobilization (MM) data are an effort to understand citizen
movements against governments, what citizens want when they demonstrate
against governments, and how governments respond to citizens. The
project codes protests against governments - the data cover 162
countries between 1990 and March 2020. For each protest event, the
project records protester demands, government responses, protest
location, and protester identities. The unit of observation is the 
protest-country-year, where protests are recorded individually within 
each country and year.

The Principle Investigators for the Mass Mobilization project are David
H. Clark (Binghamton University) and Patrick M. Regan (University of
Notre Dame). The Mass Mobilization project is sponsored by the Political
Instability Task Force (PITF). The PITF is funded by the Central
Intelligence Agency. The views expressed herein are the Principal
Investigators' alone and do not represent the views of the US
Government.

------------------------------------------------------------------------

## Define Research Question...to select key variables

Using the Mass Mobilization data, what are the geopolitical differences
in citizen participation in protest movements against governments over
time? In other words, can we determine if the [number of
participants]{.ul} and the [number of protests]{.ul} change by [country,
region, or year.]{.ul}

## Subset Columns of Interest

*   Use filter() to get the rows you want

*   Use select() to get the columns you want

*   Use pipe commands (\|\> vs %\>%) to input/output the data you want

*   Pipes in R look like `%>%` and are made available via the `magrittr` package installed as part of `dplyr`

*   Data will not save if it created into an object, unless using the last baseR script
        

```{r subset necessary columns, eval=TRUE}

###### View all column names in df ###### 
colnames(mmALL_073120_csv)

###### Braindump all potentially relevant variables/columns ###### 
PreprocData <- mmALL_073120_csv %>%
  dplyr::select(protest, participants, participants_category, protestnumber, protesterviolence, region, year, country, ccode, id, startday, startmonth, startyear, endday, endmonth, endyear)

###### Filter rows by Region ###### 
preprec_North_America <- PreprocData %>%
  filter(region == "North America")
View(preprec_North_America)

###### Remove the year column ###### 
PreprocData %>% select(-year)

###### Rename the columns w/ dplry and base R ###### 
PreprocData %>%
  rename(countrycode = ccode)
  
names(PreprocData)[names(PreprocData) == "ccode"] <- "countrycode"

```

```{r compiled edits w/ pipe, eval=TRUE}

###### All together, magic of the pipe tool ###### 

PreprocData <- mmALL_073120_csv %>%
  dplyr::select(protest, participants, participants_category, protestnumber, protesterviolence, region, year, country, ccode, id, startday, startmonth, startyear, endday, endmonth, endyear) %>%
  select(-year) %>%
  rename(countrycode = ccode)

###### View all column names in new df ###### 
colnames(PreprocData)

```

## Transforming Variable Type

-   Use str() function to detail your dataframe's structure.

    -   `str()` - structure of the object and information about the
        class, length and content of each column


```{r Transforming Variable Type}

str(PreprocData)

## transforms selected vars
PreprocData_transformed <- PreprocData %>%
mutate(
  participants = as.numeric(participants),
  participants_category = as.factor(participants_category),
  protestnumber = as.integer(protestnumber),
  protesterviolence = as.factor(protesterviolence),
  startmonth = as.integer(startmonth),
  startyear = as.integer(startyear),
  endday = as.integer(endday),
  endmonth = as.integer(endmonth),
  endyear = as.integer(endyear),
  region = as.factor(region),
  country = as.factor(country)
) 
## Known warning message for participants NA numeric data

###### reassign levels of factor variable: participants_category ######
levels(PreprocData_transformed$participants_category) <- list(
       ">10000" = ">10000",
       "100-999" = "100-999",
       "1000-1999" = "1000-1999",
       "2000-4999" = "2000-4999",
       "50-99" = "50-99",
       "5000-10000" = "5000-10000",   
       "NA"="NA")


```

## Identifying and removing duplicate observations/rows

* With big data, it is inefficient to manually remove duplicates
* Also, using code makes for a more reproducible result
```{r removing duplicates, eval=TRUE}

rownames(PreprocData_transformed)

###### Method 1: Remove duplicate rows ###### 
PreprocData2 <- PreprocData_transformed[!duplicated(PreprocData_transformed), ]

###### Method 2: Remove duplicates by single column ###### 
PreprocData2 <- PreprocData_transformed[!duplicated(PreprocData_transformed$id), ]

###### Method 3: Using dplyr - Remove duplicate rows (all columns) 
PreprocData2 <- PreprocData_transformed %>% distinct()

```

## Remove Missing Data

* What should we do when we encounter missing data in our datasets? \*
Listwise-deletion/ Complete Case Analysis - watch what happens to the
number of observations! \* Mean/median Imputation \* Multiple Imputation

* Ask yourself if you were to guess what those missing values were without
any additional information, which would you choose?

-   Listwise-deletion/ Complete Case Analysis

-   When dealing with simple statistics like the mean, the easiest way to ignore `NA` (the missing data) is to use `na.rm=TRUE` (`rm` stands for remove).

```{r, Remove Missing Data w/Listwise-deletion }
###### Find missing cells
is.na(PreprocData2)

###### Find rows w/ NA
complete.cases(PreprocData2)

###### Count NA 
sum(is.na(PreprocData2))       ## count all missing observations 
colSums(is.na(PreprocData2))   ## count the number of missing values in each column 

######  Listwise NA Removal and confirmation - compare number of observations
Preproc_NArm <- na.exclude(PreprocData2)

######  Other methods of Listwise NA removal
PreprocData2[complete.cases(PreprocData2),]
na.omit(PreprocData2)
sum(is.na(PreprocData2))
PreprocData2 %>% drop_na()

######  Remove missing values of specified single column
PreprocData2 %>% drop_na(participants)    

######  Remove missing values of specified multiple columns
PreprocData2 %>% drop_na(participants, protesterviolence) 

######  confirm no missing observations in new df
sum(is.na(Preproc_NArm))   

######  confirm no missing values in each column of new df
colSums(is.na(Preproc_NArm))   
```

## Multiple Imputation of NAs

-   In statistics, imputation means "filling in the data." Mice is an
    algorithm that allows us to fill in the missing data by creating a
    multivariate imputation model and conditional probability
    distribution for each variable in our dataset.
-   Think about it as an iterative series of matrix calculations!
-   mice includes functions for diagnostic plots to inspect the quality of the imputations
-   When to not use mice? Large p, small n.
-   `summary()` - summary statistics for each column

Mean/median Imputation steps:
- Start the mutate() function.
- Specify the column in which you want to replace the missing values.
- Use the ifelse() function to identify NA's, and once found, replace them with the median.
- Finish the mutate() function.

```{r Mean/Median Imputation}

###### Impute mean for participants column ######
preproc_participants_mean_impute <- PreprocData2 %>% 
  mutate(participants = ifelse(is.na(participants),
                            mean(participants, na.rm = T),
                            participants))
summary(preproc_participants_mean_impute$participants)

###### Impute median for participants column ######
preproc_participants_median_impute <- PreprocData2 %>% 
  mutate(participants = ifelse(is.na(participants),
                            median(participants, na.rm = T),
                            participants))
summary(preproc_participants_median_impute$participants)

###### Impute mean for all numeric columns ######                            
preproc_mean_impute <- PreprocData2 %>% 
  mutate_if(is.numeric, function(x) ifelse(is.na(x), mean(x, na.rm = T), x))
summary(preproc_mean_impute)  

###### Impute median for all numeric columns ######  
preproc_median_impute <- PreprocData2 %>% 
  mutate_if(is.numeric, function(x) ifelse(is.na(x), median(x, na.rm = T), x))
summary(preproc_median_impute)   

###### Group by region, Impute NAs participants ######
###### RUN
preproc_median_tidyimpute <- PreprocData2 %>% 
  group_by(region) %>% 
  mutate(participants = ifelse(is.na(participants), 
                            median(participants, na.rm = TRUE), 
                            participants)) 
colSums(is.na(preproc_median_tidyimpute))    

######  confirm no missing observations in new df
sum(is.na(preproc_median_tidyimpute))   

######  confirm no missing values in each column of new df
colSums(is.na(preproc_median_tidyimpute)) 
  
```

-   Multiple Imputation using MICE
* Note: objects masked by packages!
* For more: https://www.gerkovink.com/miceVignettes/Sensitivity_analysis/Sensitivity_analysis.html

```{r Imputing Missing Data}

######  install.packages("mice")
######  install.packages("ggmice")

library(mice)
library(ggmice)

library(help = "mice") 
?mice() ## Look at Built-in univariate imputation methods for "methods = "" & Vignettes 


######  run simple multiple imputation over whole df
PreprocData_imp <- mice(PreprocData2, m = 1, seed = 31415, method = "cart")

######  list the actual imputations for startyear
PreprocData_imp$imp$startyear

######  fill data matrix
PreprocData_compl <- complete(PreprocData_imp)

View(PreprocData_compl)
colSums(is.na(PreprocData_compl))   
```

## Computing new variables

```{r }


######  use lubridate to correct date formatting
Preproc_dates <- preproc_median_tidyimpute %>% 
  mutate(start = lubridate::make_datetime(startyear, startday, startmonth)) %>%   ## error w/o including lubridate::
  mutate(end = lubridate::make_datetime(endyear, endday, endmonth))
summary(Preproc_dates)


######  split date/time columns
###### DO NOT RUN
Preproc_dates_sep <- Preproc_dates %>%
tidyr::separate(start, c("startyear", "startday", "startmonth"))  %>%
tidyr::separate(end, c("endyear", "endday", "endmonth"))
summary(Preproc_dates)


######  computing new variables w/ multiple existing w/difftime()?
Preproc_dates$protest_length = difftime(Preproc_dates$end, Preproc_dates$start, units = "days")
View(Preproc_dates)

```



## Merge & Join Separate DFs

-   merge, join, rbind, cbind
-   modified by the "by" preposition

```{r, eval=FALSE, include=FALSE}

## read in and format raw population df
global_population <- read_csv("../Data/population.csv") %>% 
rename("country"="Country Name",
"population"="Value") %>% 
select(-"Country Code")

## check changes
head(global_population)

## match population by country and startyear
Preproc_join <- Preproc_dates %>% 
dplyr::semi_join(Preproc_dates, global_population, by = c("country", "startyear"))
View(Preproc_join)



```

## Flagging and dropping outliers

When we analyze data, it's important to recognize outliers because they
can affect the accuracy of our models. Outliers can cause bias when
performing statistical calculations like mean, standard deviation, and
correlation on continuous variables. Although outliers can have a big
impact, experts suggest not automatically removing them from our
observations during analysis!!

-   Methods of removal:
    -   InterQuartile Range
    -   Mean +/- 3 Standard Deviations
        - Median Absolute Deviation (MAD)
    -   Z-score & modified Z-score
    -   Statistical Tests
        - Dixon's *Q* Test; Grubb's test; Rosner's test/generalized ESD many-outliers test (GESD); Chi-squared test


```{r dropping outliers, }

###### Review summary stats of df to select numeric columns with possible outliers 
summary(Preproc_join)

###### Visual approach: Histogram, scatter plot, and boxplot
#par(mfrow=c(1,1))
hist(Preproc_join$participants, main = "Histogram")
boxplot(Preproc_join$participants, main = "Boxplot")
plot(Preproc_join$region, Preproc_join$participants)
plot(Preproc_join$region, Preproc_join$protest_length)
qqnorm(Preproc_join$participants, main = "Normal Q-Q plot")

###### IQR method on participants column ###### 
Preproc_OutIQR <- Preproc_join %>% 
  select(id, region, country, protest, protestnumber, participants, start, end, participants, protesterviolence) %>%
  group_by(participants) %>% 
  mutate(IQR = IQR(Preproc_join$participants),
            O_upper = quantile(participants, probs=c( .75), na.rm = FALSE)+1.5*IQR,  
            O_lower = quantile(participants, probs=c( .25), na.rm = FALSE)-1.5*IQR  
  ) %>% 
  filter(O_lower <= participants & participants <= O_upper)

## Results: Shows No outliers present
dim(Preproc_join)
dim(Preproc_OutIQR)
                          

###### Mean +/- 3 Standard Deviations

ptps = Preproc_join$participants
mean = mean(ptps)
std = sd(ptps)

Tmin = mean-(3*std)  ##find outliers
Tmax = mean+(3*std)

ptps[which(ptps < Tmin | ptps > Tmax)]   ##remove outliers
ptps[which(ptps > Tmin & ptps < Tmax)]

###### Median Absolute Deviation (MAD)

med = median(ptps)
abs_dev = abs(ptps-med)
mad = 1.4826 * median(abs_dev)

Tmin = med-(3*mad) 
Tmax = med+(3*mad) 

ptps[which(ptps < Tmin | ptps > Tmax)]
ptps[which(ptps > Tmin & ptps < Tmax)]

```



## Reshaping: transform long to wide data using melt()

-   Data comes in two primary shapes: *wide* and *long*.

-   Dataframes often come in less-than-ideal formats, especially when
    you are using secondary data. Helpful to prepare it for tables or
    for plotting with `ggplot2`

-   Wide = a row has more than one observation, and the units of
    observation (e.g., individuals, countries, households) are on one
    row each. (eg. survey or assessment data, or if you have ever
    downloaded data from Qualtrics)

-   Long = when a row has only one observation, but the units of
    observation are repeated down a column. (eg. Longitudinal data)

```{r}

###### Inspect DF to determine if data we want is long or wide
View(Preproc_OutIQR)

###### Making long region data wide w/ pivot_wider()
ncol(Preproc_OutIQR)
Preproc_wide <- 
  Preproc_OutIQR  %>% 
  tidyr::pivot_wider(names_from = "region", 
              values_from = "participants")
ncol(Preproc_wide)


```

## sort observations and reorder columns

```{r, sort observations}

###### Sort observations in descending order by protest startyear
Preproc_wide$start %>% 
  sort(decreasing = TRUE)

###### Reordering columns ######
head(Preproc_wide)

Preproc_wide <- Preproc_wide %>%
  select(id, country, protest, protestnumber, start, end, protesterviolence)


        
```

## Aggregate Summary Stats

* Use plyr::summarise() to apply a function to the variables in a
    dataset, especially important for aggregating across grouping
    variables.
    
* Summarise multiple columns:
-   summarise_all() affects every variable
-   summarise_at() affects variables selected with a character
        vector or vars()
-   summarise_if() affects variables selected with a predicate
        function
-   <https://dplyr.tidyverse.org/reference/summarise_all.html>



```{r,  arrange group by summarize by}

Preproc_long <- Preproc_NArm %>% 
  arrange(region, country) 
View(Preproc_long)

Preproc_long %>%
  group_by(region) %>%
  summarise(median = median(participants), n = n())

###### calculate summary statistics values by month for each variable
Preproc_long %>% 
  group_by(region) %>% 
  summarize(violence = sum(as.numeric(protesterviolence), na.rm = T))


```


## Sources

-   Gillespie, C. & Lovelace, R. (2016). Efficient R programming
    <https://bookdown.org/csgillespie/efficientR/>
-   Wickham, H. (2014). Tidy Data. Journal of Statistical Software,
    59(10), 1--23. <https://doi.org/10.18637/jss.v059.i10>
-   Wickham, H., Çetinkaya-Rundel, M., Grolemund, G. (2023) R for Data
    Science, 2e. <https://r4ds.hadley.nz/>
-   <https://bookdown.org/yihui/bookdown/>
-   **YAML Syntax -**
-   <https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html>
-   <https://rstudio.github.io/cheatsheets/html/lubridate.html?_gl=1*iabz4g*_ga*MTAwNzU0ODg4NC4xNzA1NTExNDA4*_ga_2C0WZ1JHG0*MTcwNTUxMzk2Ni4yLjEuMTcwNTUxNTExNC4wLjAuMA>
-   <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4731595/>
-   <https://www.jstatsoft.org/article/view/v045i03>
-   <https://rpubs.com/brouwern/meanimpute>
-   **Data Wrangling with R, [UNIVERSITY of WISCONSIN--MADISONSocial
    Science Computing Cooperative](http://www.wisc.edu/):**
    <https://sscc.wisc.edu/sscc/pubs/dwr/index.html>
-   
