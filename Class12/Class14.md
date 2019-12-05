Class12
================

## set up BioConductor DESeq2

``` r
#install.packages("BiocManager")
#BiocManager::install()
#BiocManager::install("DESeq2")
```

## Data for today’s class saved into the project

``` r
counts<- read.csv("airway_scaledcounts.csv", stringsAsFactors = FALSE)
metadata<- read.csv("airway_metadata.csv", stringsAsFactors = FALSE)
```

## let’s have a peak

``` r
head(counts)
```

    ##           ensgene SRR1039508 SRR1039509 SRR1039512 SRR1039513 SRR1039516
    ## 1 ENSG00000000003        723        486        904        445       1170
    ## 2 ENSG00000000005          0          0          0          0          0
    ## 3 ENSG00000000419        467        523        616        371        582
    ## 4 ENSG00000000457        347        258        364        237        318
    ## 5 ENSG00000000460         96         81         73         66        118
    ## 6 ENSG00000000938          0          0          1          0          2
    ##   SRR1039517 SRR1039520 SRR1039521
    ## 1       1097        806        604
    ## 2          0          0          0
    ## 3        781        417        509
    ## 4        447        330        324
    ## 5         94        102         74
    ## 6          0          0          0

### how many genes do we have

``` r
nrow(counts)
```

    ## [1] 38694

### how many experiments

``` r
ncol(counts)-1
```

    ## [1] 8

### what are our controls ??? we check the metadata

### let’s make sure the metadata ID col matches the colnames of counts

``` r
colnames(counts)[-1]
```

    ## [1] "SRR1039508" "SRR1039509" "SRR1039512" "SRR1039513" "SRR1039516"
    ## [6] "SRR1039517" "SRR1039520" "SRR1039521"

``` r
metadata$id
```

    ## [1] "SRR1039508" "SRR1039509" "SRR1039512" "SRR1039513" "SRR1039516"
    ## [6] "SRR1039517" "SRR1039520" "SRR1039521"

``` r
(colnames(counts)[-1])== (metadata$id)
```

    ## [1] TRUE TRUE TRUE TRUE TRUE TRUE TRUE TRUE

### if you have a lage metadata you can condense it

``` r
all((colnames(counts)[-1])== (metadata$id))
```

    ## [1] TRUE

## analysis compare the control to drug tested

## first we need ot access the columns of our countDaa that are control and find their mean

``` r
metadata$dex
```

    ## [1] "control" "treated" "control" "treated" "control" "treated" "control"
    ## [8] "treated"

### but i want the specific columns to access\!\!\!

``` r
control_id<-metadata[ metadata$dex=="control", ]$id
control_id
```

    ## [1] "SRR1039508" "SRR1039512" "SRR1039516" "SRR1039520"

``` r
head(counts[ ,control_id])
```

    ##   SRR1039508 SRR1039512 SRR1039516 SRR1039520
    ## 1        723        904       1170        806
    ## 2          0          0          0          0
    ## 3        467        616        582        417
    ## 4        347        364        318        330
    ## 5         96         73        118        102
    ## 6          0          1          2          0

#### ok we want the mean across the control rows\!\!\!

``` r
control_mean<-rowSums(counts[,control_id])/(length(control_id))
names(control_mean)<- counts$ensgene
```

## now for the treated

``` r
treated_id<-metadata[ metadata$dex=="treated", ]$id
treated_id
```

    ## [1] "SRR1039509" "SRR1039513" "SRR1039517" "SRR1039521"

``` r
head(counts[ ,treated_id])
```

    ##   SRR1039509 SRR1039513 SRR1039517 SRR1039521
    ## 1        486        445       1097        604
    ## 2          0          0          0          0
    ## 3        523        371        781        509
    ## 4        258        237        447        324
    ## 5         81         66         94         74
    ## 6          0          0          0          0

\#\#now for the treated

``` r
treated_mean<-rowSums(counts[,treated_id])/(length(treated_id))
names(treated_mean)<- counts$ensgene
```

### store them together\!\!

``` r
meancounts<- data.frame(control_mean, treated_mean)
```

### plot control vs tested\!\!

``` r
plot(meancounts$control_mean, meancounts$treated_mean)
```

![](Class14_files/figure-gfm/unnamed-chunk-18-1.png)<!-- --> \#\#\# well
that’s no good\!\!\! so let’s plot it on a log scale (bc like, only a
few have high expression\! so it willbe easier to
    visualize)

``` r
plot(meancounts$control_mean, meancounts$treated_mean, log="xy")
```

    ## Warning in xy.coords(x, y, xlabel, ylabel, log): 15032 x values <= 0
    ## omitted from logarithmic plot

    ## Warning in xy.coords(x, y, xlabel, ylabel, log): 15281 y values <= 0
    ## omitted from logarithmic plot

![](Class14_files/figure-gfm/unnamed-chunk-19-1.png)<!-- -->

## points that are abberant (have differential expression) are noticably above or below the dark center line

### now lets get the fold change\! if fold change is 0 then there was no difference

``` r
meancounts$log2fc <- log2(meancounts[,"treated_mean"]/meancounts[,"control_mean"])
head(meancounts)
```

    ##                 control_mean treated_mean      log2fc
    ## ENSG00000000003       900.75       658.00 -0.45303916
    ## ENSG00000000005         0.00         0.00         NaN
    ## ENSG00000000419       520.50       546.00  0.06900279
    ## ENSG00000000457       339.75       316.50 -0.10226805
    ## ENSG00000000460        97.25        78.75 -0.30441833
    ## ENSG00000000938         0.75         0.00        -Inf

#### the next step is to EXCLUDE THE 0 VALUES\!\!\! we DON’T WANT THEM\!\!\!

### 

``` r
x<- data.frame(happy=c(5,0,4,3), sad=c(0,5,5,0))
x==0
```

    ##      happy   sad
    ## [1,] FALSE  TRUE
    ## [2,]  TRUE FALSE
    ## [3,] FALSE FALSE
    ## [4,] FALSE  TRUE

``` r
which(x==0, arr.ind =TRUE)
```

    ##      row col
    ## [1,]   2   1
    ## [2,]   1   2
    ## [3,]   4   2

``` r
unique(which(x==0, arr.ind =TRUE)[,1])
```

    ## [1] 2 1 4

## now let’s apply this to our counts\!

``` r
#which(meancounts==0, arr.ind = TRUE)
to_rm<-unique(which(meancounts[,1:2]==0, arr.ind =TRUE)[,1])
zero_vals <- which(meancounts[,1:2]==0, arr.ind=TRUE)

to.rm <- unique(zero_vals[,1])
mycounts <- meancounts[-to.rm,]
head(mycounts)
```

    ##                 control_mean treated_mean      log2fc
    ## ENSG00000000003       900.75       658.00 -0.45303916
    ## ENSG00000000419       520.50       546.00  0.06900279
    ## ENSG00000000457       339.75       316.50 -0.10226805
    ## ENSG00000000460        97.25        78.75 -0.30441833
    ## ENSG00000000971      5219.00      6687.50  0.35769358
    ## ENSG00000001036      2327.00      1785.75 -0.38194109

### fold change time \!\!\! we have set a threshold

``` r
up.ind <- mycounts$log2fc > 2
down.ind <- mycounts$log2fc < (-2)
```

``` r
sum(up.ind)
```

    ## [1] 250

``` r
head( mycounts[up.ind,] )
```

    ##                 control_mean treated_mean   log2fc
    ## ENSG00000004799       270.50      1429.25 2.401558
    ## ENSG00000006788         2.75        19.75 2.844349
    ## ENSG00000008438         0.50         2.75 2.459432
    ## ENSG00000011677         0.50         2.25 2.169925
    ## ENSG00000015413         0.50         3.00 2.584963
    ## ENSG00000015592         0.50         2.25 2.169925

``` r
#columns(org.Hs.eg.db)
#mycounts$symbol <- mapIds(org.Hs.eg.db,
                     #keys=row.names(mycounts), # Our genenames
                     #keytype="ENSEMBL",        # The format of our genenames
                     #column="SYMBOL",          # The new format we want to add
                     #multiVals="first")
#head(mycounts)
```

### we are gonna make our summary plot\!\!

### also hey remember that fold change is great but you need it to be significant\! let’s work with that

### DESeq Analysis

``` r
library(DESeq2)
```

    ## Loading required package: S4Vectors

    ## Loading required package: stats4

    ## Loading required package: BiocGenerics

    ## Loading required package: parallel

    ## 
    ## Attaching package: 'BiocGenerics'

    ## The following objects are masked from 'package:parallel':
    ## 
    ##     clusterApply, clusterApplyLB, clusterCall, clusterEvalQ,
    ##     clusterExport, clusterMap, parApply, parCapply, parLapply,
    ##     parLapplyLB, parRapply, parSapply, parSapplyLB

    ## The following objects are masked from 'package:stats':
    ## 
    ##     IQR, mad, sd, var, xtabs

    ## The following objects are masked from 'package:base':
    ## 
    ##     anyDuplicated, append, as.data.frame, basename, cbind,
    ##     colnames, dirname, do.call, duplicated, eval, evalq, Filter,
    ##     Find, get, grep, grepl, intersect, is.unsorted, lapply, Map,
    ##     mapply, match, mget, order, paste, pmax, pmax.int, pmin,
    ##     pmin.int, Position, rank, rbind, Reduce, rownames, sapply,
    ##     setdiff, sort, table, tapply, union, unique, unsplit, which,
    ##     which.max, which.min

    ## 
    ## Attaching package: 'S4Vectors'

    ## The following object is masked from 'package:base':
    ## 
    ##     expand.grid

    ## Loading required package: IRanges

    ## Loading required package: GenomicRanges

    ## Loading required package: GenomeInfoDb

    ## Loading required package: SummarizedExperiment

    ## Loading required package: Biobase

    ## Welcome to Bioconductor
    ## 
    ##     Vignettes contain introductory material; view with
    ##     'browseVignettes()'. To cite Bioconductor, see
    ##     'citation("Biobase")', and for packages 'citation("pkgname")'.

    ## Loading required package: DelayedArray

    ## Loading required package: matrixStats

    ## 
    ## Attaching package: 'matrixStats'

    ## The following objects are masked from 'package:Biobase':
    ## 
    ##     anyMissing, rowMedians

    ## Loading required package: BiocParallel

    ## 
    ## Attaching package: 'DelayedArray'

    ## The following objects are masked from 'package:matrixStats':
    ## 
    ##     colMaxs, colMins, colRanges, rowMaxs, rowMins, rowRanges

    ## The following objects are masked from 'package:base':
    ## 
    ##     aperm, apply, rowsum

``` r
## ok time to do some DESeq
#set up our obejct for DESeq analysis 

dds <- DESeqDataSetFromMatrix(countData=counts, 
                              colData=metadata, 
                              design=~dex, 
                              tidy=TRUE)
```

    ## converting counts to integer mode

    ## Warning in DESeqDataSet(se, design = design, ignoreRank): some variables in
    ## design formula are characters, converting to factors

``` r
dds
```

    ## class: DESeqDataSet 
    ## dim: 38694 8 
    ## metadata(1): version
    ## assays(1): counts
    ## rownames(38694): ENSG00000000003 ENSG00000000005 ...
    ##   ENSG00000283120 ENSG00000283123
    ## rowData names(0):
    ## colnames(8): SRR1039508 SRR1039509 ... SRR1039520 SRR1039521
    ## colData names(4): id dex celltype geo_id

``` r
dds<- DESeq(dds)
```

    ## estimating size factors

    ## estimating dispersions

    ## gene-wise dispersion estimates

    ## mean-dispersion relationship

    ## final dispersion estimates

    ## fitting model and testing

``` r
res<-results(dds)
res
```

    ## log2 fold change (MLE): dex treated vs control 
    ## Wald test p-value: dex treated vs control 
    ## DataFrame with 38694 rows and 6 columns
    ##                          baseMean     log2FoldChange             lfcSE
    ##                         <numeric>          <numeric>         <numeric>
    ## ENSG00000000003  747.194195359907 -0.350703020686579 0.168245681332529
    ## ENSG00000000005                 0                 NA                NA
    ## ENSG00000000419  520.134160051965  0.206107766417861 0.101059218008052
    ## ENSG00000000457  322.664843927049 0.0245269479387471 0.145145067649248
    ## ENSG00000000460   87.682625164828  -0.14714204922212 0.257007253994673
    ## ...                           ...                ...               ...
    ## ENSG00000283115                 0                 NA                NA
    ## ENSG00000283116                 0                 NA                NA
    ## ENSG00000283119                 0                 NA                NA
    ## ENSG00000283120 0.974916032393564  -0.66825846051647  1.69456285241871
    ## ENSG00000283123                 0                 NA                NA
    ##                               stat             pvalue              padj
    ##                          <numeric>          <numeric>         <numeric>
    ## ENSG00000000003   -2.0844696749953 0.0371174658432827 0.163034808641681
    ## ENSG00000000005                 NA                 NA                NA
    ## ENSG00000000419    2.0394751758463 0.0414026263001167 0.176031664879168
    ## ENSG00000000457  0.168982303952746  0.865810560623561 0.961694238404388
    ## ENSG00000000460  -0.57252099672319  0.566969065257939 0.815848587637724
    ## ...                            ...                ...               ...
    ## ENSG00000283115                 NA                 NA                NA
    ## ENSG00000283116                 NA                 NA                NA
    ## ENSG00000283119                 NA                 NA                NA
    ## ENSG00000283120 -0.394354484734893  0.693319342566817                NA
    ## ENSG00000283123                 NA                 NA                NA

``` r
summary(res)
```

    ## 
    ## out of 25258 with nonzero total read count
    ## adjusted p-value < 0.1
    ## LFC > 0 (up)       : 1563, 6.2%
    ## LFC < 0 (down)     : 1188, 4.7%
    ## outliers [1]       : 142, 0.56%
    ## low counts [2]     : 9971, 39%
    ## (mean count < 10)
    ## [1] see 'cooksCutoff' argument of ?results
    ## [2] see 'independentFiltering' argument of ?results

### cool let’s plot it\!\!\!

\#VOLCANO PLOT

``` r
plot(res$log2FoldChange, res$padj)
```

![](Class14_files/figure-gfm/unnamed-chunk-30-1.png)<!-- --> \#\#\# ok
but that’s no good\! we want to see the small p values easily\!\! \#take
the log\! but the -log of res$padj \# in this way, the larger numbers
are the smallest p values

``` r
plot(res$log2FoldChange, -log(res$padj))
abline(v=c(-2,2), col="gray", lty=2)
abline(h=-log(0.05), col="gray", lty=2)
```

![](Class14_files/figure-gfm/unnamed-chunk-31-1.png)<!-- --> \#\#\#\#
let’s color it

``` r
mycols<- rep("grey", length(res$padj))
mycols[abs(res$log2FoldChange)>2]= "blue"
mycols[ res$padj > 0.05 ]="red"
plot(res$log2FoldChange, -log(res$padj), col=mycols)
```

![](Class14_files/figure-gfm/unnamed-chunk-32-1.png)<!-- -->
