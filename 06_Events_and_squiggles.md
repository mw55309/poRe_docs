# Events and Squiggles

## Events data

We extract events data with the get.events function.  As events data are co-located with FASTQ in the FAST5 file, then for Nick's SQK-MAP-006 data we need to give it the paths again.

get.events again returns a list, wth the template and complement events as data frames

```R
f5 <- "Data/read_data/MAP006-1_2100000-2600000_fast5/LomanLabz_PC_Ecoli_K12_MG1655_20150924_MAP006_1_5005_1_ch56_file159_strand.fast5"
ev <- get.events(f5, path.t = "/Analyses/Basecall_2D_000/", path.c = "/Analyses/Basecall_2D_000/")
names(ev)
head(ev$template)
head(ev$complement)
```

* start: time in seconds
* mean: the mean signal of the event
* stdev: the standard deviation when sampling
* length: length of the event
* model_state: the predicted kmer
* model_level: the value from the model
* move: how many moves the model has made in that step

### Extracting the model

Extracting the model itself in poRe is also easy, but relies on a little legacy code that needs to be updated... next version ;-)

```R
mods <- get.models(f5, tmodel = "/Analyses/Basecall_2D_000/BaseCalled_template/Model", 
                        cmodel = "/Analyses/Basecall_2D_000/BaseCalled_complement/Model")
head(mods$template)
head(mods$complement)
```

Here we can see the models that ONT use to turn events into kmers, but this isn't simply a case of looking up the closest match:

```R
head(ev$template)
# first event kmer is GTGTTT, event mean is 50.23671, model mean is 50.36703
     mean    start      stdv      length model_state model_level move
1 50.23671 65351.52 0.6342780 0.049136786      GTGTTT    50.36703    0

mods$template[mods$template$kmer=="GTGTTT",]

# however the model level for this kmer is quite different:
       kmer level_mean level_stdv  sd_mean  sd_stdv   weight
3008 GTGTTT   43.29339    0.55339 0.78968 0.267732 623.0312
```

What gives?  Well, ONT apply three parameters to the model to "scale" the model to fit each read.  These parameters are drift, scale and shift and they are read dependent!

### Extracting the model parameters

For v1.0:

```R
get.model.params(f5, tsum="/Analyses/Basecall_2D_000/Summary/basecall_1d_template", 
                     csum = "/Analyses/Basecall_2D_000/Summary/basecall_1d_complement")
```

For v1.1:
```R
get.model.params(f5)
```

For this template read, we have:
* drift	0.0003234845
* scale	0.9643541
* shift	8.616875

### Applying the model parameters

The drift parameter is applied per event and is scaled by the number of seconds since the start of the read.  So "drift * (time - min(time))" can be subtracted from the event mean, or added to the model mean.  As we are dealing with the first event then the drift parameter isn't applied.

Scale and shift are then applied in a classic linear model: (model.mean * scale) + shift.  In our case:

* (43.29339 * 0.9643541) + 8.616875 = 50.36703

Which is the model value that shows up in the events table above. 

#### Plotting Squiggles

Plotting squiggles works directly from the events data.  The parameters minseconds and maxseconds control which events to plot:

```R
plot.squiggle(ev$template)
plot.squiggle(ev$template, minseconds=5)
plot.squiggle(ev$complement)
```
