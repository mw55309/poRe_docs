# Installation

poRe is written for R version 3 or above, which is available from [CRAN](https://cran.r-project.org/)

The first step is to install dependencies, which can be done from within R:

```R
# RUN THIS CODE FROM WITHIN R

# packages from BioConductor
source("http://www.bioconductor.org/biocLite.R")
biocLite("rhdf5")

# packages from CRAN
install.packages(c("shiny","svDialogs","data.table","bit64","optparse"))

```

### Windows
Download the latest zip file from [SourceForge](https://sourceforge.net/projects/rpore/)

Open R and choose Packages -> Install package(s) from local zip file. Navigate to the downloaded zip file.

### Linux/Mac
Download the latest tar.gz file from [SourceForge](https://sourceforge.net/projects/rpore/)

From the command line, run: 

```sh
# RUN THIS COMMAND FROM THE TERMINAL

R CMD INSTALL poRe_0.17.tar.gz
```

### Mac specific issues

Some people have reported problems installing poRe on Mac due to dependency issues.

In particular, if installation of data.table fails with:

```
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
```

Then this can be fixed with

```
xcode-select --install
```
