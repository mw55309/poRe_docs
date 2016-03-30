# FASTQ and FASTA extraction

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

## Extract FASTQ in the client

We'll see how to extract FASTQ from entire directories below, but here are some exemples of single file analysis

### New format

Starting off with the new format of FAST5 (what we have called 1.1) is pretty easy as all of the defaults are set correctly

```R
# we will use example data packaged with poRe
newbc  <- system.file("/extdata/f5/new_bc", package="poRe")

# get a list of all fast5 files in the directory
f5files <- dir(path = newbc, pattern = "\\.fast5", full.names = TRUE)
 
# just look at the first one
f5 <- f5files[1]

# run get_fastq on just that file
get_fastq(f5)
```

This returns a list with three fastq datasets, "template", "complement" and "2D".  

### Old format

poRe can still handle the old format, we just need to change the defaults

```R
# we will use example data packaged with poRe
oldbc  <- system.file("/extdata/f5/old_bc", package="poRe")

# get a list of all fast5 files in the directory
f5files <- dir(path = oldbc, pattern = "\\.fast5", full.names = TRUE)

# just look at the first one
f5 <- f5files[1]

# run get_fastq on just that file
get_fastq(f5)

# run get_fastq with the default paths changed
get_fastq(f5, path.t="/Analyses/Basecall_2D_000/", path.c="/Analyses/Basecall_2D_000/")
```

If we don't want to extract all 3, we can choose which to extract using the "which" argument

```R
get_fastq(f5, path.t="/Analyses/Basecall_2D_000/", which="template")
```

Working with lists is easy in R, and if you want the FASTQ as a string:

```R
fq <- get_fastq(f5, path.t="/Analyses/Basecall_2D_000/", path.c="/Analyses/Basecall_2D_000/", which="all")
names(fq)
fq$template
```

From here you can see that it's incredibly simple to write a FASTQ extraction script:

```R
# path to fast5 files
f5dir <- system.file("/extdata/f5/new_bc", package="poRe")
# get vector of all fast5 files
f5files <- dir(f5dir, pattern="\\.fast5$", full.names = TRUE)
# iterate over files
for (f5 in f5files) {
    # extract 2D fastq
    fq <- get_fastq(f5, which="2D")
    # check fq is a list and contains 2D
    if (typeof(fq) == "list" && exists("2D", where=fq)) {
        # cat to "" (STDOUT but could be the name of a file
        # change the cat = "" to a filename to see what happens
        cat(fq[["2D"]], file = "", sep = "\n", fill = FALSE)
    }
}
```

The above will output the FASTQ data to the console, but if we want to output to a file, we simply change one line:

```R
# path to fast5 files
f5dir <- system.file("/extdata/f5/new_bc", package="poRe")
# get vector of all fast5 files
f5files <- dir(f5dir, pattern="\\.fast5$", full.names = TRUE)
# iterate over files
for (f5 in f5files) {
    # extract 2D fastq
    fq <- get_fastq(f5, which="2D")
    # check fq is a list and contains 2D
    if (typeof(fq) == "list" && exists("2D", where=fq)) {
        # cat to "" (STDOUT but could be the name of a file
        # change the cat = "" to a filename to see what happens
        cat(fq[["2D"]], file = "output.2D.fastq", sep = "\n", fill = FALSE)
    }
}
```

(note line **cat(fq[["2D"]], file = "output.2D.fastq", sep = "\n", fill = FALSE)** has changed)

However, we have scripts (see below) that do this already, so there is no need to write your own!

## Extract FASTA in the client

Simply use get_fasta instead of get_fastq :-)

## Extracting FASTQ from the command-line


Command-line scripts for extracting FASTQ can be pulled from [github](https://github.com/mw55309/poRe_scripts).  There are scripts there that can parse both the old and new format of fast5 file, but we have installed the old format scripts here:

```sh
# 2D
extract2D Data/read_data/MAP006-1_2100000-2600000_fast5/ > MAP006-1.2D.fastq

# template
extractTemplate Data/read_data/MAP006-1_2100000-2600000_fast5/ > MAP006-1.template.fastq

# complement
extractComplement Data/read_data/MAP006-1_2100000-2600000_fast5/ > MAP006-1.complement.fastq
```

FASTQ can be converted to FASTA using a small script

```sh
porefq2fa MAP006-1.2D.fastq > MAP006-1.2D.fasta
```
