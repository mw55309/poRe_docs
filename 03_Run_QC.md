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

## Longest high quality read

We can find this from the metadata:

```R
newbc  <- system.file("/extdata/f5/new_bc", package="poRe")

# extract fast5 info
meta <- read.fast5.info(newbc)

# find the maximum length
max(meta$len2d)

# get the metadata for that read
meta[meta$len2d==max(meta$len2d),]

# We now know the longest read
longest <- rownames(meta[meta$len2d==max(meta$len2d),])[1]
lfq <- get_fastq(longest, which="2D")
lfa <- get_fasta(longest, which="2D")

# write fastq and fasta out to files
cat(lfq[["2D"]], file = "longest.fastq", sep = "\n", fill = FALSE)
cat(lfa[["2D"]], file = "longest.fasta", sep = "\n", fill = FALSE)
```

## Yield

Yield over time can be plotted with plot.cumulative.yield.  Bear in mind that these plots won't make any sesne using just the sample data here!

```R
yield <- plot.cumulative.yield(meta)
```

The calculated cumulative yields are returned as data.frames

```R
head(yield)
```

## Read length histogram

We can plot read lengths histograms.  Bear in mind that these plots won't make any sesne using just the sample data here!

```R
plot.length.histogram(meta)
```

## Channel and pore occupancy

The MinION flowcell is arranged into 512 channels in 4 blocks, and we can see the layout using poRe:

```R
show.layout()
```

Bear in mind that these plots won't make any sesne using just the sample data here!

We first calculate some statistics summarised by channel:

```R
meta <- read.meta.info(newbc)

meta.s <- summarise.by.channel(meta)
head(meta.s)
```

The rows of the result are the channel numbers, and the columns tell us how many channels appear in our summary data, and either the number (n) or cumulative length (l) of template, complement and 2d reads from each channel.

We can plot these:

```R
# the number of times the channel appears in any context
plot.channel.summary(meta.s)

# cumulative 2D length
plot.channel.summary(meta.s, report.col="l2d")
```

