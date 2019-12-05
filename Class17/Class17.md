Untitled
================

\#\#oh hey\! instead of writing bio3d::function you can library(bio3d)
and all the functions can be used as normal \#3. Querying the GDC from R
\#We will typically start our interaction with the GDC by searching the
resource to find \#data that we are interested in investigating further.
In GDC speak this is called “Querying GDC metadata”. \#Metadata here
refers to the extra descriptive information associated with the actual
patient data (i.e. ‘cases’) in the GDC.

\#For example: Our query might be \#‘find how many patients were studied
for each major project’ \#or \#‘find and download all gene expression
quantification data files for all pancreatic cancer patients’. \#We will
answer both of these questions below. \#The are four main sets of
metadata that we can query, \#namely projects(), cases(), files(), and
annotations(). \#We will start with projects()

``` r
#projects <- getGDCprojects()

#head(projects)
```

\#If you use the View(projects) function call you can see all the
project names (such as Neuroblastoma, Pancreatic Adenocarcinoma, etc.)
along with their project IDs (such as TARGET-NBL, TCGA-PAAD, etc.) and
associated information.

``` r
#View(projects)
```

\#Moving onto cases() we can use an example from the package associated
publication to answer our first from question above (i.e. find the
number of cases/patients across different projects within the GDC):

``` r
#cases_by_project <- cases() %>%
 # facet("project.project_id") %>%
  #aggregations()
#head(cases_by_project)
```

\#Note that the facet() and aggregations() functions here are from the
GenomicDataCommons package and act to group all cases by the project id
and then count them up.

\#If you use the View() function on our new cases\_by\_project object
you will find that the data we are after is accessible via
cases\_by\_project$project.project\_id.

#### Design a Cancer Vaccine

\#Q1: Identify sequence regions that contain all 9-mer peptides that are
only found in the tumor. Hint: You will need to first identify the sites
of mutation in the above sequences and then extract the surrounding
subsequence region. This subsequence should encompass all possible
9-mers in the tumor derived sequence. In other words extract the
subsequence from 8 residues before and 8 residues after all point
mutations in the tumor sequence.

## read in the alignment

\#We start by 1. reading the provided sequences
(lecture18\_sequences.fa) into R, then 2. aligning, 3. looking for sites
of cancer specific mutation (i.e. differences between the two
sequences), and finally 4. outputing all 9-mer contaning subsequences
encompasing these mutant sites.

``` r
library(bio3d)
read.fasta("lecture18_sequences.fa")
```

    ##              1        .         .         .         .         .         60 
    ## P53_wt       MEEPQSDPSVEPPLSQETFSDLWKLLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGP
    ## P53_mutant   MEEPQSDPSVEPPLSQETFSDLWKLLPENNVLSPLPSQAMLDLMLSPDDIEQWFTEDPGP
    ##              **************************************** ******************* 
    ##              1        .         .         .         .         .         60 
    ## 
    ##             61        .         .         .         .         .         120 
    ## P53_wt       DEAPRMPEAAPPVAPAPAAPTPAAPAPAPSWPLSSSVPSQKTYQGSYGFRLGFLHSGTAK
    ## P53_mutant   DEAPWMPEAAPPVAPAPAAPTPAAPAPAPSWPLSSSVPSQKTYQGSYGFRLGFLHSGTAK
    ##              **** ******************************************************* 
    ##             61        .         .         .         .         .         120 
    ## 
    ##            121        .         .         .         .         .         180 
    ## P53_wt       SVTCTYSPALNKMFCQLAKTCPVQLWVDSTPPPGTRVRAMAIYKQSQHMTEVVRRCPHHE
    ## P53_mutant   SVTCTYSPALNKMFCQLAKTCPVQLWVDSTPPPGTRVRAMAIYKQSQHMTEVVRRCPHHE
    ##              ************************************************************ 
    ##            121        .         .         .         .         .         180 
    ## 
    ##            181        .         .         .         .         .         240 
    ## P53_wt       RCSDSDGLAPPQHLIRVEGNLRVEYLDDRNTFRHSVVVPYEPPEVGSDCTTIHYNYMCNS
    ## P53_mutant   RCSDSDGLAPPQHLIRVEGNLRVEYLDDRNTFVHSVVVPYEPPEVGSDCTTIHYNYMCNS
    ##              ******************************** *************************** 
    ##            181        .         .         .         .         .         240 
    ## 
    ##            241        .         .         .         .         .         300 
    ## P53_wt       SCMGGMNRRPILTIITLEDSSGNLLGRNSFEVRVCACPGRDRRTEEENLRKKGEPHHELP
    ## P53_mutant   SCMGGMNRRPILTIITLEV-----------------------------------------
    ##              ******************                                           
    ##            241        .         .         .         .         .         300 
    ## 
    ##            301        .         .         .         .         .         360 
    ## P53_wt       PGSTKRALPNNTSSSPQPKKKPLDGEYFTLQIRGRERFEMFRELNEALELKDAQAGKEPG
    ## P53_mutant   ------------------------------------------------------------
    ##                                                                           
    ##            301        .         .         .         .         .         360 
    ## 
    ##            361        .         .         .  393 
    ## P53_wt       GSRAHSSHLKSKKGQSTSRHKKLMFKTEGPDSD
    ## P53_mutant   ---------------------------------
    ##                                                
    ##            361        .         .         .  393 
    ## 
    ## Call:
    ##   read.fasta(file = "lecture18_sequences.fa")
    ## 
    ## Class:
    ##   fasta
    ## 
    ## Alignment dimensions:
    ##   2 sequence rows; 393 position columns (259 non-gap, 134 gap) 
    ## 
    ## + attr: id, ali, call

``` r
#assign it
p53_alignment<-read.fasta("lecture18_sequences.fa")
```

\#We can optionally align these sequences to make sure we have residue
position correspondences correctly mapped between wt and mutant (incase
of indels) with the following code. However, this appears to be
unnecessary in this case as the provided sequences are already aligned.
\#\#THIS IS FOR ALIGNING SEQUENCES, BUT WHAT IF YOU HAD TWO SEPERATE
FILES??)

\#seqs \<- bio3d::seqaln(seqs, exe=“filepath for
muscle”)

``` r
p53_alignment <- bio3d::seqaln(p53_alignment, exefile="/Users/DinaZ/Desktop/UCSD_Documents/BioInformatics_resource_1_files/muscle3.8.31_i86darwin32")
p53_alignment
```

    ##              1        .         .         .         .         .         60 
    ## P53_wt       MEEPQSDPSVEPPLSQETFSDLWKLLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGP
    ## P53_mutant   MEEPQSDPSVEPPLSQETFSDLWKLLPENNVLSPLPSQAMLDLMLSPDDIEQWFTEDPGP
    ##              **************************************** ******************* 
    ##              1        .         .         .         .         .         60 
    ## 
    ##             61        .         .         .         .         .         120 
    ## P53_wt       DEAPRMPEAAPPVAPAPAAPTPAAPAPAPSWPLSSSVPSQKTYQGSYGFRLGFLHSGTAK
    ## P53_mutant   DEAPWMPEAAPPVAPAPAAPTPAAPAPAPSWPLSSSVPSQKTYQGSYGFRLGFLHSGTAK
    ##              **** ******************************************************* 
    ##             61        .         .         .         .         .         120 
    ## 
    ##            121        .         .         .         .         .         180 
    ## P53_wt       SVTCTYSPALNKMFCQLAKTCPVQLWVDSTPPPGTRVRAMAIYKQSQHMTEVVRRCPHHE
    ## P53_mutant   SVTCTYSPALNKMFCQLAKTCPVQLWVDSTPPPGTRVRAMAIYKQSQHMTEVVRRCPHHE
    ##              ************************************************************ 
    ##            121        .         .         .         .         .         180 
    ## 
    ##            181        .         .         .         .         .         240 
    ## P53_wt       RCSDSDGLAPPQHLIRVEGNLRVEYLDDRNTFRHSVVVPYEPPEVGSDCTTIHYNYMCNS
    ## P53_mutant   RCSDSDGLAPPQHLIRVEGNLRVEYLDDRNTFVHSVVVPYEPPEVGSDCTTIHYNYMCNS
    ##              ******************************** *************************** 
    ##            181        .         .         .         .         .         240 
    ## 
    ##            241        .         .         .         .         .         300 
    ## P53_wt       SCMGGMNRRPILTIITLEDSSGNLLGRNSFEVRVCACPGRDRRTEEENLRKKGEPHHELP
    ## P53_mutant   SCMGGMNRRPILTIITLEV-----------------------------------------
    ##              ******************                                           
    ##            241        .         .         .         .         .         300 
    ## 
    ##            301        .         .         .         .         .         360 
    ## P53_wt       PGSTKRALPNNTSSSPQPKKKPLDGEYFTLQIRGRERFEMFRELNEALELKDAQAGKEPG
    ## P53_mutant   ------------------------------------------------------------
    ##                                                                           
    ##            301        .         .         .         .         .         360 
    ## 
    ##            361        .         .         .  393 
    ## P53_wt       GSRAHSSHLKSKKGQSTSRHKKLMFKTEGPDSD
    ## P53_mutant   ---------------------------------
    ##                                                
    ##            361        .         .         .  393 
    ## 
    ## Call:
    ##   bio3d::seqaln(aln = p53_alignment, exefile = "/Users/DinaZ/Desktop/UCSD_Documents/BioInformatics_resource_1_files/muscle3.8.31_i86darwin32")
    ## 
    ## Class:
    ##   fasta
    ## 
    ## Alignment dimensions:
    ##   2 sequence rows; 393 position columns (259 non-gap, 134 gap) 
    ## 
    ## + attr: id, ali, call

\#Next we calculate identity per equivalent (i.e. aligned) position and
then use this information to find non identical sites that do not
contain gaps (i.e. indels).

## Calculate positional identity scores

\#ide \<- conserv(seqs$ali, method=“identity”) \#mutant.sites \<-
which(ide \<
1)

``` r
p53_ide<-conserv(p53_alignment$ali, method="identity") #our method is identity but there are others
p53_ide#look, most are 1.0, but we want to look for the less than 1.0 which are the sites
```

    ##   [1] 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
    ##  [18] 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
    ##  [35] 1.0 1.0 1.0 1.0 1.0 1.0 0.5 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
    ##  [52] 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 0.5 1.0 1.0 1.0
    ##  [69] 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
    ##  [86] 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
    ## [103] 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
    ## [120] 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
    ## [137] 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
    ## [154] 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
    ## [171] 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
    ## [188] 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
    ## [205] 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 0.5 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
    ## [222] 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
    ## [239] 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
    ## [256] 1.0 1.0 1.0 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5
    ## [273] 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5
    ## [290] 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5
    ## [307] 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5
    ## [324] 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5
    ## [341] 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5
    ## [358] 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5
    ## [375] 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5 0.5
    ## [392] 0.5 0.5

``` r
p53_ide<1
```

    ##   [1] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ##  [12] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ##  [23] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ##  [34] FALSE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE
    ##  [45] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ##  [56] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE FALSE
    ##  [67] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ##  [78] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ##  [89] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [100] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [111] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [122] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [133] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [144] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [155] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [166] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [177] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [188] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [199] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [210] FALSE FALSE FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [221] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [232] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [243] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
    ## [254] FALSE FALSE FALSE FALSE FALSE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
    ## [265]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
    ## [276]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
    ## [287]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
    ## [298]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
    ## [309]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
    ## [320]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
    ## [331]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
    ## [342]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
    ## [353]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
    ## [364]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
    ## [375]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
    ## [386]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE

``` r
which(p53_ide<1)
```

    ##   [1]  41  65 213 259 260 261 262 263 264 265 266 267 268 269 270 271 272
    ##  [18] 273 274 275 276 277 278 279 280 281 282 283 284 285 286 287 288 289
    ##  [35] 290 291 292 293 294 295 296 297 298 299 300 301 302 303 304 305 306
    ##  [52] 307 308 309 310 311 312 313 314 315 316 317 318 319 320 321 322 323
    ##  [69] 324 325 326 327 328 329 330 331 332 333 334 335 336 337 338 339 340
    ##  [86] 341 342 343 344 345 346 347 348 349 350 351 352 353 354 355 356 357
    ## [103] 358 359 360 361 362 363 364 365 366 367 368 369 370 371 372 373 374
    ## [120] 375 376 377 378 379 380 381 382 383 384 385 386 387 388 389 390 391
    ## [137] 392 393

``` r
mutant_sites<-which(p53_ide<1) 
##let's look at that first mutation site
mutant_sites[1] #oh! it's #41
```

    ## [1] 41

``` r
mutant_sites #ugh, all the "gaps" are annoying, bc the sequence just ends so there are so many gaps!!! we want them gone!!
```

    ##   [1]  41  65 213 259 260 261 262 263 264 265 266 267 268 269 270 271 272
    ##  [18] 273 274 275 276 277 278 279 280 281 282 283 284 285 286 287 288 289
    ##  [35] 290 291 292 293 294 295 296 297 298 299 300 301 302 303 304 305 306
    ##  [52] 307 308 309 310 311 312 313 314 315 316 317 318 319 320 321 322 323
    ##  [69] 324 325 326 327 328 329 330 331 332 333 334 335 336 337 338 339 340
    ##  [86] 341 342 343 344 345 346 347 348 349 350 351 352 353 354 355 356 357
    ## [103] 358 359 360 361 362 363 364 365 366 367 368 369 370 371 372 373 374
    ## [120] 375 376 377 378 379 380 381 382 383 384 385 386 387 388 389 390 391
    ## [137] 392 393

``` r
p53_alignment$ali[,mutant_sites]
```

    ##            [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10] [,11] [,12]
    ## P53_wt     "D"  "R"  "R"  "D"  "S"  "S"  "G"  "N"  "L"  "L"   "G"   "R"  
    ## P53_mutant "L"  "W"  "V"  "V"  "-"  "-"  "-"  "-"  "-"  "-"   "-"   "-"  
    ##            [,13] [,14] [,15] [,16] [,17] [,18] [,19] [,20] [,21] [,22]
    ## P53_wt     "N"   "S"   "F"   "E"   "V"   "R"   "V"   "C"   "A"   "C"  
    ## P53_mutant "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"  
    ##            [,23] [,24] [,25] [,26] [,27] [,28] [,29] [,30] [,31] [,32]
    ## P53_wt     "P"   "G"   "R"   "D"   "R"   "R"   "T"   "E"   "E"   "E"  
    ## P53_mutant "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"  
    ##            [,33] [,34] [,35] [,36] [,37] [,38] [,39] [,40] [,41] [,42]
    ## P53_wt     "N"   "L"   "R"   "K"   "K"   "G"   "E"   "P"   "H"   "H"  
    ## P53_mutant "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"  
    ##            [,43] [,44] [,45] [,46] [,47] [,48] [,49] [,50] [,51] [,52]
    ## P53_wt     "E"   "L"   "P"   "P"   "G"   "S"   "T"   "K"   "R"   "A"  
    ## P53_mutant "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"  
    ##            [,53] [,54] [,55] [,56] [,57] [,58] [,59] [,60] [,61] [,62]
    ## P53_wt     "L"   "P"   "N"   "N"   "T"   "S"   "S"   "S"   "P"   "Q"  
    ## P53_mutant "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"  
    ##            [,63] [,64] [,65] [,66] [,67] [,68] [,69] [,70] [,71] [,72]
    ## P53_wt     "P"   "K"   "K"   "K"   "P"   "L"   "D"   "G"   "E"   "Y"  
    ## P53_mutant "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"  
    ##            [,73] [,74] [,75] [,76] [,77] [,78] [,79] [,80] [,81] [,82]
    ## P53_wt     "F"   "T"   "L"   "Q"   "I"   "R"   "G"   "R"   "E"   "R"  
    ## P53_mutant "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"  
    ##            [,83] [,84] [,85] [,86] [,87] [,88] [,89] [,90] [,91] [,92]
    ## P53_wt     "F"   "E"   "M"   "F"   "R"   "E"   "L"   "N"   "E"   "A"  
    ## P53_mutant "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"  
    ##            [,93] [,94] [,95] [,96] [,97] [,98] [,99] [,100] [,101] [,102]
    ## P53_wt     "L"   "E"   "L"   "K"   "D"   "A"   "Q"   "A"    "G"    "K"   
    ## P53_mutant "-"   "-"   "-"   "-"   "-"   "-"   "-"   "-"    "-"    "-"   
    ##            [,103] [,104] [,105] [,106] [,107] [,108] [,109] [,110] [,111]
    ## P53_wt     "E"    "P"    "G"    "G"    "S"    "R"    "A"    "H"    "S"   
    ## P53_mutant "-"    "-"    "-"    "-"    "-"    "-"    "-"    "-"    "-"   
    ##            [,112] [,113] [,114] [,115] [,116] [,117] [,118] [,119] [,120]
    ## P53_wt     "S"    "H"    "L"    "K"    "S"    "K"    "K"    "G"    "Q"   
    ## P53_mutant "-"    "-"    "-"    "-"    "-"    "-"    "-"    "-"    "-"   
    ##            [,121] [,122] [,123] [,124] [,125] [,126] [,127] [,128] [,129]
    ## P53_wt     "S"    "T"    "S"    "R"    "H"    "K"    "K"    "L"    "M"   
    ## P53_mutant "-"    "-"    "-"    "-"    "-"    "-"    "-"    "-"    "-"   
    ##            [,130] [,131] [,132] [,133] [,134] [,135] [,136] [,137] [,138]
    ## P53_wt     "F"    "K"    "T"    "E"    "G"    "P"    "D"    "S"    "D"   
    ## P53_mutant "-"    "-"    "-"    "-"    "-"    "-"    "-"    "-"    "-"

``` r
p53_alignment$ali[2,c((mutant_sites-8):(mutant_sites+8))]
```

    ## Warning in (mutant_sites - 8):(mutant_sites + 8): numerical expression has
    ## 138 elements: only the first used
    
    ## Warning in (mutant_sites - 8):(mutant_sites + 8): numerical expression has
    ## 138 elements: only the first used

    ##  [1] "S" "P" "L" "P" "S" "Q" "A" "M" "L" "D" "L" "M" "L" "S" "P" "D" "D"

## Exclude gap possitions from analysis

\#gaps \<- gap.inspect(seqs) \#mutant.sites \<-
mutant.sites\[mutant.sites %in% gaps$f.inds\]

``` r
p53_gaps<-gap.inspect(p53_alignment)
p53_mutant_sites<-mutant_sites[mutant_sites %in% p53_gaps$f.inds]
p53_mutant_sites
```

    ## [1]  41  65 213 259

\#mutant.sites \#We can use these indices in mutant.sites to extract
subsequences as required for the hands-on session. First however we come
up with suitable names for these subsequences based on the mutation.
This will help us later to make sense and keep track of our results.
\#\# Make a “names” label for our output sequences (one per mutant)
\#mutant.names \<-
paste0(seqs\(ali["P53_wt",mutant.sites],  #mutant.sites,  #seqs\)ali\[“P53\_mutant”,mutant.sites\])

\#mutant.names

``` r
p53_mutant_names<- paste0(p53_alignment$ali["P53_wt",p53_mutant_sites],
                          p53_mutant_sites,
                          p53_alignment$ali["P53_mutant", p53_mutant_sites])
p53_mutant_names
```

    ## [1] "D41L"  "R65W"  "R213V" "D259V"

\#for once you have your mutant subsequences you want to predict MCH
binding Afinity\! there is an online tool for this
<http://tools.iedb.org/mhci/> \#\# once you have your best binders BLAST
for the human proteome bc you dont want it to target something else\!
(obviously if it’s getting hits for tumor antigens, thats a GOOD THING)

### how to align sequences that are not\!

``` r
s1<-read.fasta("seq1.fa")
s2<-read.fasta("seq2.fa")
s3<-read.fasta("seq3.fa")
### you can do this for as many as you want!
#now bind them!
s_compiled<-seqbind(s1,s2,s3,...) #for all of the sequences
###OR in your TERMINAL
##make sure you are in the folder of ONLY YOUR SEQUENCES TO BE ALGIGNED (ls)
##cat *.fa or cat *.fasta will compile all the fasta files
##cat *.fasta > my_compiled_sequences
#the > is a redirect, telling it to make a file
###uhhh idk what to do next
```
