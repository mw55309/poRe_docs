# Installation

poRe is written for R version 3 or above, which is available from [CRAN](https://cran.r-project.org/)

The first step is to install dependencies, which can be done from within R:

```R

# packages from BioConductor
source("http://www.bioconductor.org/biocLite.R")
biocLite("rhdf5")

# packages from CRAN
install.packages(c("shiny","svDialogs","data.table","bit64"))

```

### Windows
Download the latest zip file from [SourceForge](https://sourceforge.net/projects/rpore/)

Open R and choose Packages -> Install package(s) from local zip file. Navigate to the downloaded zip file.

### Linux/Mac
Download the latest tar.gz file from [SourceForge](https://sourceforge.net/projects/rpore/)

From the command line, run: R CMD INSTALL poRe_0.17.tar.gz


