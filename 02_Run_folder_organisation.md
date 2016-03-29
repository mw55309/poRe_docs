# Run folder organisation

Organising MinION run folders can still be a problem, especially if you have Metrichor set to download data to a single directory, which then ends up keeping data from multiple runs.

For this problem we have the copy.runs function, which inspects each FAST5, extracts metadata and copies to an appropriate folder.

Example usage:

```R
library(poRe)

# we will use example data packaged with poRe
f5dir <- system.file("/extdata/f5/new_bc", package="poRe")

# we will create a "test" directory in the current working directory
dir.create("test")

# now copy from f5dir to test
copy.runs(dir=f5dir, dest="test")
```

The ouutput of the above is the path of the resulting folder that the reads have been copied to

```
"test/e40030db5d1193cc903f8d505011be8e9a628f6c/ONT Sequencing Workflow"
```


