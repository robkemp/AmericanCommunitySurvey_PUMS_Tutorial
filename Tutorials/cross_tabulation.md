Creating a cross-tablulation
============================

Using Basic Recoding
--------------------

The next step is to add additional variables to the tabulations that we
went through in the first tutorial. This tutorial should be much shorter
now that the basics are out of the way. We'll continue using the
poverty\_new variable that we created. The code to load the libraries
and data, then recode the variable is below.

    library(car)
    library(dplyr)

    # Change the file path to the proper location

    # setwd(PATH_TO_TUTORIAL_FOLDER) #uncomment this line to change directory.

    # reads in the csv file and stores it. stringsAsFactors=FALSE keeps R from transforming variables you don't want transformed.

    data <- read.csv("Data and Documentation/ss14pco.csv", stringsAsFactors=FALSE) 

    data <- data%>% # passes the data object we created above to the next function
              mutate(poverty_new=recode(POVPIP, "000:124=1; 125:183=2; 183:501=3")) # adds the poverty_new variable to the end of the dataset, it doesn't over write POVPIP.

Let's say that we want to know about children and poverty using the same
poverty levels we used before. We'd need to create another variable,
based on the `AGEP` variable (see the data dictionary page 32) that
indicated if a person was under 18 or not. To do this, we'll use the
base R command `ifelse` to create a statement that will make those under
18 coded to 1 and those 18 and over coded 2.

    data <- data%>%
      mutate(child=ifelse(AGEP<18, 1, 2))

While I just used this approach, it is always worthwhile to check on
missing variables before moving forward as they would be coded 2 here.
AGEP doesn't have any missing variables (as shown by the code below) so
we don't have to worry. We can chain two `ifelse` statements together to
deal with this if it becomes and issue. You can choose to give the
missing data a real code or to just keep it missing. I've provided and
example below.

    # to determine if there are missing variables add exclude=NULL to table()
    table(data$AGEP, exclude=NULL)

    ## 
    ##    0    1    2    3    4    5    6    7    8    9   10   11   12   13   14 
    ##  558  598  619  602  583  699  635  701  690  626  711  705  653  665  709 
    ##   15   16   17   18   19   20   21   22   23   24   25   26   27   28   29 
    ##  699  657  636  754  709  604  568  603  607  654  671  601  660  704  700 
    ##   30   31   32   33   34   35   36   37   38   39   40   41   42   43   44 
    ##  674  701  714  719  710  706  673  655  679  608  684  589  654  667  682 
    ##   45   46   47   48   49   50   51   52   53   54   55   56   57   58   59 
    ##  659  660  626  645  737  798  765  792  813  797  754  863  814  758  761 
    ##   60   61   62   63   64   65   66   67   68   69   70   71   72   73   74 
    ##  772  750  765  688  662  701  641  628  503  469  472  482  392  365  328 
    ##   75   76   77   78   79   80   81   82   83   84   85   86   87   88   89 
    ##  312  328  286  264  232  215  213  178  182  158  162  130  138  100   89 
    ##   93 <NA> 
    ##  386    0

    # We can eliminate NA being coded by explicitly telling the code to deal with them on their own.
    # This variable is identical to the child variable because AGEP has no missing values.
    # is.na() is used to identify missing values in the data.

    data <- data%>%
              mutate(child_wMissing=ifelse(is.na(AGEP), NA,
                                           ifelse(AGEP<18, 1, 2)))
    # This line tests if the two variables are equivalent, is will print TRUE if they are.

    identical(data$child, data$child_wMissing)

    ## [1] TRUE

We've now added the additional variable we need to the data set we can
create the cross tabulation. Essentially, a cross tabulation is taking
one basic tabulation and finding out how the values within each category
would be broken up within other categories. To do this practically, the
`group_by` function is of great use. We can just add our new variable to
the call we used before. This tells R that rather than just grouping by
the `poverty_new` variable, group by that first then also group by our
`child` variable within that first grouping. In this case ordering isn't
important because the output is going to show us the unique
intersections of each category for each variable. Basically the output
is taking a cross-tab table and turning each cell into a row.

    children_poverty <- data%>%
      group_by(poverty_new, child)%>%
      summarize(total=sum(PWGTP))

    children_poverty

    ## Source: local data frame [8 x 3]
    ## Groups: poverty_new [?]
    ## 
    ##   poverty_new child   total
    ##         (dbl) (dbl)   (int)
    ## 1           1     1  258233
    ## 2           1     2  592134
    ## 3           2     1  148270
    ## 4           2     2  355883
    ## 5           3     1  825280
    ## 6           3     2 3057804
    ## 7          NA     1   15549
    ## 8          NA     2  102713

Label Recoded Variables
-----------------------

In our previous example we needed to remember or refer to which category
received which code. In this demonstration, I'll add labels so we are
able to more quickly derive insights.

R has a specific type of variable that is useful for many operations,
but I use it the most for labeling recoded variables. This variable type
is called a factor. It contains two pieces of information, a level
(numeric but doesn't need to indicate ordering even though it can) and a
label for that level. There are ordered and unordered factors, but I'll
be using an ordered factor for both since each variable has an intrinsic
ordering.

Quick side note, you can ignore this if you'd like. R also has a text
variable called a "string" that I could just use in the recode calls
directly. It would make the actual values for each recode into their
labels rather than the numeric values I used. I don't necessarily think
that this approach is wrong, but it is certainly less flexible.
Particularly if you'd like to graph the data. I can control the ordering
of the labels by assigning the level in the order I'd like them
displayed using factors, I can't do this on strings, which use an
alphabetical sorting method. I can also always pull the text out of the
factor labels if needed.

Technically, this next bit of code will take the variables we already
made and use them to create new variables as factors with labels. I'll
use the function `ordered` to create the factor since I'm implying an
order to the levels, but if your data are not ordered and you don't mind
an alphabetical sort, `factor` will work as well. A couple more
technical notes: `:` in R means through, so `1:3` means use values 1
through 3 and the `c()` command concatenates whatever you put into the
parentheses into a vector of values. R uses that a TON within functions
to help keep things orderly.

    # adding the new labeled variables.
    data <- data%>%
              mutate(poverty_labels=ordered(poverty_new, levels=1:3, labels =
                                              c("Less than 125%", "125% to 183%", "More than 183%")),
                     child_labels= ordered(child, levels=1:2, labels = 
                                             c("Child", "Adult")))
    # Creating a new object to store the labeled table
    children_poverty_labels <- data%>%
      group_by(poverty_labels, child_labels)%>%
      summarize(total=sum(PWGTP))

    # tests that the results are numerically identical
    identical(children_poverty$total, children_poverty_labels$total)

    ## [1] TRUE

    # prints out the new table
    children_poverty_labels

    ## Source: local data frame [8 x 3]
    ## Groups: poverty_labels [?]
    ## 
    ##   poverty_labels child_labels   total
    ##           (fctr)       (fctr)   (int)
    ## 1 Less than 125%        Child  258233
    ## 2 Less than 125%        Adult  592134
    ## 3   125% to 183%        Child  148270
    ## 4   125% to 183%        Adult  355883
    ## 5 More than 183%        Child  825280
    ## 6 More than 183%        Adult 3057804
    ## 7             NA        Child   15549
    ## 8             NA        Adult  102713

Exporting Results
-----------------

It's nice to be able to generate these tables and output them in R, but
regardless of where you work or what you do you'll need to export them
to other programs. R supports exporting data to CSV files natively and
that is the method I'll be using here. There are also a very strong set
of packages to export and format data for Excel specifically, but those
are beyond the scope of this tutorial. Check out the `xlsx` package as a
good place to start, but there are many others, that's just the one I
use most of the time.

The following code uses the base R function `write.csv` to export the
tables into their own CSV files that can be read in any spreadsheet
program.

    # Specify the name of the file to write to first, then the object to write.
    # row.names = FALSE is set to prevent R from writing a column of values out with the row number in the final output.
    write.csv("children_poverty_labels.csv", children_poverty_labels, row.names = FALSE)
