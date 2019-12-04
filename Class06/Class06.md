Untitled
================

our first silly function

``` r
add<-function(x,y=1){
#sum of the input x and y
x+y
}
```

``` r
add(5)
```

    ## [1] 6

``` r
#im guessing this means... you are defining x as 5???
add(c(5,6)) #now there are two x'es, 5 and 6?? appearently not. i do not understand hold on let me try something
```

    ## [1] 6 7

``` r
add(c(5,1,4,3)) #nvm it does read all these are different x inputs! it's important to have the c() otherwise it wont work
```

    ## [1] 6 2 5 4

``` r
#you can redefine y at any time
add(x=c(5,6,1,5,10), y=100)
```

    ## [1] 105 106 101 105 110

``` r
###what if there is some missing data?
add(c(5,5,NA,7)) ##it's ok! it will just spit out NA
```

    ## [1]  6  6 NA  8

time to make a function

``` r
min(c(5,2,7,10))
```

    ## [1] 2

``` r
max(c(5,2,7,10))
```

    ## [1] 10

``` r
range(c(5,2,7,10))
```

    ## [1]  2 10

``` r
#ok so....
x<-range(c(5,2,7,10))
x[1]
```

    ## [1] 2

``` r
x[2]
```

    ## [1] 10

``` r
x
```

    ## [1]  2 10

``` r
###now we make a function, we will call it rescale because we are rescaling a function that we need to use over and over!
rescale<-function(x){
  rng<-range(x)
  (x-rng[1])/(rng[2]-rng[1])
}
rescale(1:10)
```

    ##  [1] 0.0000000 0.1111111 0.2222222 0.3333333 0.4444444 0.5555556 0.6666667
    ##  [8] 0.7777778 0.8888889 1.0000000

``` r
rescale2<-function(x){
  rng<-range(x, na.rm=TRUE)
  #na.rm
  (x-rng[1])/(rng[2]-rng[1])
}
rescale2(c(5,2,NA,7,10))
```

    ## [1] 0.375 0.000    NA 0.625 1.000

``` r
rescale3 <- function(x, na.rm=TRUE, plot=FALSE) {
 if(na.rm) {
 rng <-range(x, na.rm=na.rm)
 } else {
 rng <-range(x)
 }
 print("Hello")
 answer <- (x - rng[1]) / (rng[2] - rng[1])
 print("is it me you are looking for?")
 if(plot) {
 plot(answer, typ="b", lwd=4)
 }
 print("I can see it in ...")
 return(answer)
}
#now test it
rescale3(1:10)
```

    ## [1] "Hello"
    ## [1] "is it me you are looking for?"
    ## [1] "I can see it in ..."

    ##  [1] 0.0000000 0.1111111 0.2222222 0.3333333 0.4444444 0.5555556 0.6666667
    ##  [8] 0.7777778 0.8888889 1.0000000

get that library

``` r
library(bio3d)
```

what the fuck is this package?

``` r
s1 <- read.pdb("4AKE") # kinase with drug
```

    ##   Note: Accessing on-line PDB file

``` r
s2 <- read.pdb("1AKE") # kinase no drug
```

    ##   Note: Accessing on-line PDB file
    ##    PDB has ALT records, taking A only, rm.alt=TRUE

``` r
s3 <- read.pdb("1E4Y") # kinase with drug
```

    ##   Note: Accessing on-line PDB file

``` r
s1.chainA <- trim.pdb(s1, chain="A", elety="CA") #we are trimming to only look at chain A and ONLY the carbon atoms ("CA") we prevoiusly defined s1 as the pdb file 4AKE
s2.chainA <- trim.pdb(s2, chain="A", elety="CA")
s3.chainA <- trim.pdb(s1, chain="A", elety="CA") #this is an error! it should bc s3 not s1
s1.b <- s1.chainA$atom$b
s2.b <- s2.chainA$atom$b
s3.b <- s3.chainA$atom$b
plotb3(s1.b, sse=s1.chainA, typ="l", ylab="Bfactor")
```

![](Class06_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

``` r
plotb3(s2.b, sse=s2.chainA, typ="l", ylab="Bfactor")
```

![](Class06_files/figure-gfm/unnamed-chunk-7-2.png)<!-- -->

``` r
plotb3(s3.b, sse=s3.chainA, typ="l", ylab="Bfactor")
```

![](Class06_files/figure-gfm/unnamed-chunk-7-3.png)<!-- -->

``` r
###let's take a closer look
read.pdb("4AKE")
```

    ##   Note: Accessing on-line PDB file

    ## Warning in get.pdb(file, path = tempdir(), verbose = FALSE): /var/folders/
    ## y6/3n4k53vd3ps8mgpsntfyb18r0000gn/T//RtmpW39f11/4AKE.pdb exists. Skipping
    ## download

    ## 
    ##  Call:  read.pdb(file = "4AKE")
    ## 
    ##    Total Models#: 1
    ##      Total Atoms#: 3459,  XYZs#: 10377  Chains#: 2  (values: A B)
    ## 
    ##      Protein Atoms#: 3312  (residues/Calpha atoms#: 428)
    ##      Nucleic acid Atoms#: 0  (residues/phosphate atoms#: 0)
    ## 
    ##      Non-protein/nucleic Atoms#: 147  (residues: 147)
    ##      Non-protein/nucleic resid values: [ HOH (147) ]
    ## 
    ##    Protein sequence:
    ##       MRIILLGAPGAGKGTQAQFIMEKYGIPQISTGDMLRAAVKSGSELGKQAKDIMDAGKLVT
    ##       DELVIALVKERIAQEDCRNGFLLDGFPRTIPQADAMKEAGINVDYVLEFDVPDELIVDRI
    ##       VGRRVHAPSGRVYHVKFNPPKVEGKDDVTGEELTTRKDDQEETVRKRLVEYHQMTAPLIG
    ##       YYSKEAEAGNTKYAKVDGTKPVAEVRADLEKILGMRIILLGAPGA...<cut>...KILG
    ## 
    ## + attr: atom, xyz, seqres, helix, sheet,
    ##         calpha, remark, call

``` r
###
?trim.pdb
?read.pdb
?plotb3
```

question 6 the condensed
function

``` r
###name of function will be protein.recall because you are calling up proteins from pdb! 
protein.recall<-function(p){ #p is standing in for protein (pdb) file 
  protein<-read.pdb(p) #to load up that pdb file and assign it as protein
p.trimmed<-trim.pdb(protein,chain="A", elety="CA") #define the trimmed version as p.trimmed. We are doing this because there are specific pices of the protein that we want to look at
p.b<-p.trimmed$atom$b #i dont know what im assigning here, i guess a specific atom on the trimmed version
plotb3(p.b, sse=p.trimmed, typ="l", ylab="Bfactor") #this plots it

}
protein.recall("1E4Y") #you can put any pdb code! remember to put it in quotes! 
```

    ##   Note: Accessing on-line PDB file

    ## Warning in get.pdb(file, path = tempdir(), verbose = FALSE): /var/folders/
    ## y6/3n4k53vd3ps8mgpsntfyb18r0000gn/T//RtmpW39f11/1E4Y.pdb exists. Skipping
    ## download

![](Class06_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->
