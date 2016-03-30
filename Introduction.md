# American Community Survey Public Use Microdata Series Tutorial in R

Hello.  This tutorial is intended to help you get started using the American Community Survey PUMS files in R.  The Census does provide some tools for using these files, but they generally aren't very flexible or reproducible.  This guide should help almost anyone use R to get useful cross tabulations and estimates from the PUMS Files.  It will provide resources for learning R, getting R and RStudio set-up to do this, and provide code examples and explanations for making estimates. If you are interested in margins of error, check out the `survey` package for R.  It's a whole tutorial unto itself so I won't be covering that here.

## Sections

1. [Basics of PUMS - Recode and Single Variable](Tutorials/basic_pums.md)
2. [Cross-tabulation, Labels, and Exporting](Tutorials/cross_tabulation.md)



## R Basics

If you are 100% new to R, I recommend looking at a few resources to get you started.  I've provided a list below:

PUT THE LIST FROM THE OLD DOCS HERE

## Setting Up R and RStudio (Skip this if you already have them)

I **strongly** recommend that anyone using R use RStudio to interact with R.  When you download R for the first time, it does come with a GUI that you can use, but RStudio has TONS of very useful features that we'll only begin to tap into in this tutorial.  

To download R visit [CRAN](https://cran.r-project.org/) and click on the download link for your operating system. Run through your systems dialog for installing R.  As of this writing, the most recent version is 3.2.4, which is what I'm using. 

To download RStudio visit [RStudio Download Site](https://www.rstudio.com/products/rstudio/download/) and click the download link that corresponds to your system.  Run the installation dialog to install it.  RStudio will automatically find the installation of R you just did.

If you're using Windows, you'll need to install RTools, which is a piece of software that you'll need to build R packages that you download from online. I won't be using it, but it's a good idea to install whenever you install R.  Click on the link for [RTools Installers](https://cran.r-project.org/bin/windows/Rtools/) and you can just download the highest version number.

## Installing Required R 'Packages'

In R there are a set of functions and capabilities that are in the basic installation, called base-R.  To add functionality, R allows for additional function to be added by installing R 'packages' that provide different functionality.  This tutorial uses a few R packages that are not in base-R.  I'll describe them in more detail later, but for now, just open RStudio and copy-paste the following code into the window labeled 'Console'.  

```r
install.packages(c("dplyr", "car", "survey"))

```

## Contact 

If you run into any questions or issues in this tutorial, please contact me.  I've put my basic contact information below.  I prefer either an email or an issue filed in this GitHub repo.  I'm happy to schedule calls to deal with specific issues or spend a small amount of time looking at code that is sent or posted.

Rob Kemp
Work Ph: 303-864-7755
Email: robert.kemp@state.co.us

