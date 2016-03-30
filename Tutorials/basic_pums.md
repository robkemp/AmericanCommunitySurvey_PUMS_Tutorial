Recoding and tabulating a single variable.
==========================================

PUMS Overview
-------------

The Census Bureau has been releasing what it calls micro-data in files
called Public Use Micro-data Samples for a very long time. Traditionally
this was done for the decennial census' and data sets such as the
Current Population Survey. They continued this practice when they rolled
out the American Community Survey data in 2005. While there is data for
2005, we generally recommend not using that year due to group quarters
issues. Using 2006 and forward is much better from a time-series
perspective.

PUMS data are released in the same way as the tabulated files. They are
released in 1-year and 5-year increments (i.e. 2006-2010). There are
3-year files through 2013, however these files have be discontinued and
will not be released in the future. A general rule of thumb for choosing
the right file is the intent and goals of the analysis. If you need to
have a time-series or the most current data are required, then the
1-year files are best. If you require more precision and currency isn't
as important, then the 5-year files are a good choice. The 5-year files
for PUMS do not have the normal benefit of more geographic levels, they
are only larger samples and can provide more precise estimates. For most
uses, the 1-year files work well.

PUMS data are a sample of all the responses from the ACS, not the full
sample, but methods are provided to account for that. The data are
designed for researchers and other interested parties to be able to
create their own tabulations and analyses, but that are not provided by
the Census Bureau.

To do this, we can use software and a bit of code to easily create
summations that can be re-run easily with small changes as the analysis
progresses or needs change or we need it for more places.

Data - a deeper dive.
---------------------

The PUMS files are distributed as .CSV or SAS files that are designed to
be read into computer programs for analysis. They are broken up into two
files, one for the person records and one for the housing unit records.
Housing units are the primary unit of analysis for the survey, but they
ask about each member of the household. This yields multiple
person-level records within each household. There is a method to link
them using the household id that can be found in the documentation.

The data used in this example comes from the ACS 2014 1-year file. That
file can be downloaded along with other files
[here](https://www.census.gov/programs-surveys/acs/data/pums.html). They
come with a basic readme on usage, including geography notes, weighting
considerations, and standard errors (used for margins of error).

I've also thrown the detailed documentation into the Data and
Documentation folder that shows the coding of each variables and which
variables are included. I use this document religiously through out the
process because the data are not usually very usable in their
distributed form. The Census leaves as much detail as it can while still
protecting privacy to allow for the most customization options. The also
means that the researcher needs to make far more decisions that you
would when using a prepared table. These include collapsing or changing
categories, dealing with missing data, and decisions around validity of
the variables for the goals of the analysis. The following section will
walk through making these decisions and using the documentation to
create a custom poverty level tabulation.

One final consideration that is a very important feature and limitation
of the PUMS data: geography. A pinnacle concern of all released Census
products is the privacy of survey or census respondents. Toward that
end, the Census will always attempt to prevent the identification of
individual respondents, including by letting the public have very small
geographies to work with. The Census solved this by creating a custom
geography called Public-Use Micro-data Areas. Most urban areas are well
covered by them, but getting rural areas covered. They are updated after
every Census, so there are two versions currently in use. The Census
2000 defined PUMA areas are used for any data before 2012 in the 1-year
files and the Census 2010 definitions from 2012 into the future.
Multi-year files that cross 2012 have both, but are often missing and
are difficult to use because of privacy concerns. For a map of the PUMAs
for both Census' check out the Integrated Public Use Micro-data Survey
site [here](https://usa.ipums.org/usa/volii/pumas10.shtml). At the
state-level, the PUMS are very accurate, but they do struggle with some
sub-state areas, particularly rural areas where the geographies are very
large and cover a lot of territory that might not be appropriate to
group together.

Some R Code Basics
------------------

This tutorial is going to rely on normal, commonly accepted R functions,
but a few of which are not typically taught in online tutorials. The
tutorial relies heavily on the `dplyr` package from Hadley Wickham to
create some logic and readability in the code. The `dplyr` package uses
'verbs' to help with data analysis. They parallel SQL concepts like
grouping and selection. I'll explain each verb as it is used below, but
for more information on that package check out the package
[vignette](https://cran.rstudio.com/web/packages/dplyr/vignettes/introduction.html).
The tutorial also makes use of a concept called piping using the `%>%`
operation. Basically, piping takes the previous command in the chain and
pass that to the next function. This should become clear in the
tutorial.

Finally, this tutorial uses a function from the `car` package called
`recode()` to do all recoding. This makes the code far more intuitive
and understandable, which makes finding errors easier. The function
takes the variable to recode and a string that allows for a direct way
to show which values need to be change to which new values (i.e.
`"22:27=2"` codes any value in the original variable between 22 and 27
to 2 in the new variable).

All of the code below can be copy-pasted directly into the R Console or
saved in an R Script. See the Introduction for setting up R and
installing packages. See
[here](https://cran.r-project.org/doc/contrib/Lemon-kickstart/kr_scrpt.html)
for information on R Scripts.

Custom Poverty Levels
---------------------

Frequently programs and research projects require looking at the number
of people below or between various poverty levels. Often, the Census
doesn't provide the exact poverty level of interest. PUMS data can be
used to identify very specific levels of poverty using the `POVPIP`
variable.

This analysis uses the data placed in the Data and Documentation folder
of the repository. It is the person-level records for Colorado.

The first this we need to do is load the required libraries:

    library(car)
    library(dplyr)

After this, we need to load our data in. I decided NOT to make this an
RStudio project to make sure people that don't want to use RStudio can
use it easily. The most important part is telling R where the folder for
the tutorial is. If you're using Windows, make sure you change the `\`
to `/` so that R can read it. The first command does that, the second
loads the data into an object called `data`. NOTE: Larger states or the
US files are very large, R uses RAM to load them and some files may not
fit into memory depending on the specifications of your computer. Feel
free to email me about this if there are questions.

    # Change the file path to the proper location

    # setwd(PATH_TO_TUTORIAL_FOLDER) #uncomment this line to change directory.

    # reads in the csv file and stores it. stringsAsFactors=FALSE keeps R from tansforming variables you don't want transformed.

    data <- read.csv("Data and Documentation/ss14pco.csv", stringsAsFactors=FALSE) 

Now that the data is loaded into R, we can start working with it.
Poverty status is coded from 000 to 500 to represent the percent of the
poverty-level for that individual (i.e. 142% would indicate and
individual living at 142% of the federal poverty level (FPL)). The code
of 501 is a top-code for any individual over 500% of the poverty line. R
also notes missing data using the `NA` code. The following code will
take the data, create a new variable called `poverty_new` that codes
`POVPIP` into the following categories: 0 to 124%=1, 125% to 183%=2, and
183% or higher=3. The `mutate` verb in `dplyr` is the call we'll use for
this.

    data <- data%>% # passes the data object we created above to the next function
              mutate(poverty_new=recode(POVPIP, "000:124=1; 125:183=2; 183:501=3")) # adds the poverty_new variable to the end of the dataset, it doesn't over write POVPIP.

The final step to generating our basic estimates is to deal with the
weighting variable. If we were to just simply count the number of
observations within each category, we'd be only counting each record as
one person, but they are not. The PUMS files are samples from a survey
of a sample. To make the survey generalizable to the general population
(in this case Colorado) the Census includes survey weights that
represent how many times to count each person. The variable `PWGTP` is
the main weighting variable for the person-level file (there is a `WGTP`
weight for housing unit and household work). To accomplish this, I will
use a `group_by` function to tell R which groups to perform operations
in and a `summarize` function to sum the weighting variable within those
groups. This means that is a person was coded 1, which means they are
below 125% of the poverty line, and their value on the `PWGTP` variable
was 43, instead of being counted as one person, they'd be counted as 43
people. We coded a new variable that would put each person into our
categories. We just want to know how many people in Colorado fall into
each category. The code to do this is below.

    poverty <- data%>%
      group_by(poverty_new)%>%
      summarize(total=sum(PWGTP))

    poverty

    ## Source: local data frame [4 x 2]
    ## 
    ##   poverty_new   total
    ##         (dbl)   (int)
    ## 1           1  850367
    ## 2           2  504153
    ## 3           3 3883084
    ## 4          NA  118262

We created a new R object that has each of our codes and a `NA` row.
Consulting the documentation shows that those are missing values on the
poverty variable. For poverty this occurs when it the Census is unable
to figure out the person's poverty level. This output shows that 850,367
people live below 125% of the poverty line in Colorado in 2014. This is
the most basic use of the PUMS data to generate estimates.
