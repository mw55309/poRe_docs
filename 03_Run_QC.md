# Run QC

## File formats

Oxford Nanopore are very bad at releasing official definitions of file formats, therefore unfortunately much guess work is involved.

Most of the early ONT data was released from the SQK-MAP-005 kits - this includes 
[the MARC data](http://f1000research.com/articles/4-1075/v1), [Mick's B fragilis dataset](http://gigadb.org/dataset/100177) and [Nick Loman's first E coli dataset](http://gigadb.org/dataset/100102).  These data were encoded in what can be best described as FAST5 v.1.0 (ONT don't actually assign version numbers!)

Then SQK-MAP-006 came along, which was a major chemistry change that increased throughput.  A major change is that metrichor has switched from a 5mer model to a 6mer model.   [Nick has also released E coli SQK-MAP-006 data](http://lab.loman.net/2015/09/24/first-sqk-map-006-experiment/), and because he was very quick to do this, the files are still in FAST5 v.1.0

However, in November 2015, ONT released a new file format, which we can call FAST5 v1.1.  The major difference is that the template and complement FASTQ and events data have been moved to a new group within the FAST5 file, separate to the 2D data.  This actually makes logical sense, but can make data analysis difficult.

### Major file format differences

The major difference is where the template and complement data are.  In version 1.0 they are all in a group called Basecall_2D_000; however, in v1.1 they have been moved to Basecall_1D_000

FAST5 v1.0
* /Analyses/Basecall_**2D**_000/BaseCalled_2D/
* /Analyses/Basecall_**2D**_000/BaseCalled_template/
* /Analyses/Basecall_**2D**_000/BaseCalled_complement/

FAST5 v1.1
* /Analyses/Basecall_**2D**_000/BaseCalled_2D/
* /Analyses/Basecall_**1D**_000/BaseCalled_template/
* /Analyses/Basecall_**1D**_000/BaseCalled_complement/


## Extracting meta-data from fast5
If you haven't run pore_rt(), then you can extract meta-data directly from the fast5 files.  This takes a long time as we have to open each file and extract the attributes.  We will use a simple example here.

There are three functions for extracting metadata: **read.fast5.info**, **read.meta.info** and **read.essential.info**.  These extract similar subsets of information, so please check the help, but all three are compatible with downstream plots

### New format

```R
newbc  <- system.file("/extdata/f5/new_bc", package="poRe")

# extract fast5 info
meta <- read.fast5.info(newbc)

```

### Old format

```R
oldbc  <- system.file("/extdata/f5/old_bc", package="poRe")

# extract fast5 info
meta <- read.fast5.info(oldbc, 
                        path.t="/Analyses/Basecall_2D_000/", 
                        path.c="/Analyses/Basecall_2D_000/")

```

## Run QC in poRe

If you ran pore_rt() during your nanopore run then you will have access to a metadata text file that can be used for run QC etc.  Otherwise we will have to create one (see above).  

```R
# load in pass data
pass <- read.table("Data/run_metadata/pass.meta.txt", sep="\t", header=TRUE, stringsAsFactors=FALSE)

# set standard/expected column names
colnames(pass) <- c("filename","channel_num","read_num","read_start_time",
                    "status","tlen","clen","len2d",
                    "run_id","read_id","barcode","exp_start")

# load in the fail data
fail <- read.table("Data/run_metadata/fail.meta.txt", sep="\t", header=TRUE, stringsAsFactors=FALSE)

# set standard/expected column names
colnames(fail) <- c("filename","channel_num","read_num","read_start_time",
                    "status","tlen","clen","len2d",
                    "run_id","read_id","barcode","exp_start")
head(pass)
head(fail)
```

### Longest high quality read

We can find this from the metadata:

```R
# find the maximum length
max(pass$len2d)

# get the metadata for that read
pass[pass$len2d==max(pass$len2d),]

# We now know the longest read
longest <- "Data/read_data/MAP006-1_2100000-2600000_fast5/LomanLabz_PC_Ecoli_K12_MG1655_20150924_MAP006_1_5005_1_ch153_file57_strand.fast5"
lfq <- get_fastq(longest, which="2D")
lfa <- get_fasta(longest, which="2D")

# write fastq and fasta out to files
cat(lfq[["2D"]], file = "longest.fastq", sep = "\n", fill = FALSE)
cat(lfa[["2D"]], file = "longest.fasta", sep = "\n", fill = FALSE)
```


### Yield

Yield over time can be plotted with plot.cumulative.yield

```R
yield.p <- plot.cumulative.yield(pass)
yield.f <- plot.cumulative.yield(fail)
```

The calculated cumulative yields are returned as data.frames

```R
head(yield.p)
head(yield.f)
```

### Read length histogram

We can plot read lengths histograms

```R
plot.length.histogram(pass)
plot.length.histogram(fail)
```

There's quite a long failed template read!

```R
max(fail$tlen)
# [1] 379692
fail [fail$tlen==379692,]
```

Plotting in ggplot2 if you really want to

```R
library(ggplot2)
m <- ggplot(pass, aes(x=len2d))
m + geom_histogram(binwidth=500)
```

### Channel and pore occupancy

The MinION flowcell is arranged into 512 channels in 4 blocks, and we can see the layout using poRe:

```R
show.layout()
```

We first calculate some statistics summarised by channel:

```R
# pass data
pass.s <- summarise.by.channel(pass)
head(pass.s)

fail.s <- summarise.by.channel(fail)
head(fail.s)
```

The rows of the result are the channel numbers, and the columns tell us how many channels appear in our summary data, and either the number (n) or cumulative length (l) of template, complement and 2d reads from each channel.

We can plot these:

```R
# pass

# the number of times the channel appears in any context
plot.channel.summary(pass.s)

# cumulative 2D length
plot.channel.summary(pass.s, report.col="l2d")

# fail

# the number of times the channel appears in any context
plot.channel.summary(fail.s)

# cumulative 2D length
plot.channel.summary(fail.s, report.col="l2d")

```









