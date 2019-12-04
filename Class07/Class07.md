Untitled
================

## 

``` r
source("http://tinyurl.com/rescale-R")
```

``` r
rescale(1:10)
```

    ##  [1] 0.0000000 0.1111111 0.2222222 0.3333333 0.4444444 0.5555556 0.6666667
    ##  [8] 0.7777778 0.8888889 1.0000000

``` r
rescale(c(3,10,NA,7,))
```

\#\#\#important stuff to put into our functions\! stop() and warning().
These are functions themselves but you put them into functions\! \#look
at rescale2

``` r
function(x, na.rm=TRUE, plot=FALSE, ...) {
  # Our rescale function from lecture 10

  if( !is.numeric(x) ) {
    stop("Input x should be numeric", call.=FALSE)
  }
  
  rng <-range(x, na.rm=TRUE)

  answer <- (x - rng[1]) / (rng[2] - rng[1])
  if(plot) { 
    plot(answer, ...) 
  }

  return(answer)
}

rescale2(c(3,10,NA,7,"barry"))
##you should get an error that says "x should be numeric!". it's printing the stop message
```

## take a look at is.numeric(). it returns a boolean\!

is.numeric(5) is.numeric(c(5,10)) \#the bang \! is like the NOT symbol
in symbolic logic\! so \!is.numeric() asks if something is NOT numeric.

\#\#what makes a good function? \#\#\#understandable (remember that
humans and computers are using them) \#\#\#correct + understandable
\#\#\#sensible and thoughtout \#\#\#good names to make it obvious what
it does\! \#\#\#\#\#\# EXAMPLES \#\#\#we want a function called
“both\_na()” that counts how many positisions in two input values, x
and y, that have missing values in the same spot \#\#\#\#\# do not start
with this: \# Should we start like this?

``` r
#both_na <- function(x, y) {
# something goes here?
```

## always start with the simple definition of the problem\!

# Lets define an example x and y

``` r
x <- c( 1, 2, NA, 3, NA)
y <- c(NA, 3, NA, 3, 4)
```

## because both sets have NA as the third data point

## he google searched for a starting point but i dont know how he got the is.na function. i guess it checks for things that are NA

``` r
is.na(x)
```

    ## [1] FALSE FALSE  TRUE FALSE  TRUE

``` r
is.na(y)
```

    ## [1]  TRUE FALSE  TRUE FALSE FALSE

# should return “\[1\] FALSE FALSE TRUE FALSE TRUE” / “\[1\] TRUE FALSE TRUE FALSE FALSE”

\#\#now the which() function. it tells you which element is the thing
you are looking for

``` r
which(is.na(x))
```

    ## [1] 3 5

``` r
which(is.na(y))
```

    ## [1] 1 3

# you get \[1\] 3 5 / \[1\] 1 3

## apply logic\! using &

``` r
is.na(x) & is.na(y)
```

    ## [1] FALSE FALSE  TRUE FALSE FALSE

### woo that works\! let’s make the function

``` r
both_na<- function(x,y){
  which(is.na(x)&is.na(y))
}
both_na(x,y)
```

    ## [1] 3

\#\#\#OMG IT WORKED\!\! \#\# BUT WAIT, we wanted to know HOW MANY\!

``` r
both_na_sum<- function(x,y){
  sum(is.na(x)&is.na(y))
}
both_na_sum(x,y)
```

    ## [1] 1

### and boom, it works

### ANOTHER EXAMPLE

``` r
x <- c(NA, NA, NA)
y1 <- c( 1, NA, NA)
y2 <- c( 1, NA, NA, NA)
both_na_sum(x, y1)
```

    ## [1] 2

``` r
both_na_sum(x, y2)
```

    ## Warning in is.na(x) & is.na(y): longer object length is not a multiple of
    ## shorter object length

    ## [1] 3

\#\#\#it worked for x and y1, but not exactly x and y2. it tells us 3,
when it only matches in 2 places, because it sees 3 NAs, and y2 is
longer than x, it seems to align it. \#\# IT IS NOT\! IT’S
RECYCLING\!\!\!

``` r
x <- c(NA, NA, NA)
y1 <- c( 1, NA, NA)
y2 <- c( 1, NA, NA, NA)
y3 <- c( 1, NA, NA, NA, NA, NA)
y4 <- c(1, NA, NA, NA, NA, NA, NA)
both_na_sum(x, y3)
```

    ## [1] 5

``` r
both_na_sum(x, y4)
```

    ## Warning in is.na(x) & is.na(y): longer object length is not a multiple of
    ## shorter object length

    ## [1] 6

### see how for 5 NAs it gives us just 5, but for 6 it tells us the length is off\!

### we can do better\!

\#we want to use “==”, but we want to catch when == is false \! so “\!=”

``` r
#both_na_sum2<- function(x,y){
 # if(length(x) != length(y)) {
 #   stop("your vectors are not equal")
 # }
 # sum(is.na(x)&is.na(y))
#}

#x <- c(NA, NA, NA)
#y1 <- c( 1, NA, NA)
#y2 <- c( 1, NA, NA, NA)
#y3 <- c( 1, NA, NA, NA, NA, NA)
#y4 <- c(1, NA, NA, NA, NA, NA, NA)
#both_na_sum2(x, y1)
#both_na_sum2(x, y2)
#both_na_sum2(x, y3)
#both_na_sum2(x, y4)
```

### MORE EXAMPLES\!\!\!

``` r
function(x, y) {
  ## Print some info on where NA's are as well as the number of them 
  if(length(x) != length(y)) {
    stop("Input x and y should be vectors of the same length", call.=FALSE)
  }
  na.in.both <- ( is.na(x) & is.na(y) )
  na.number  <- sum(na.in.both)
  na.which   <- which(na.in.both)

  message("Found ", na.number, " NA's at position(s):", 
          paste(na.which, collapse=", ") ) 
  
  return( list(number=na.number, which=na.which) )
}
```

    ## function(x, y) {
    ##   ## Print some info on where NA's are as well as the number of them 
    ##   if(length(x) != length(y)) {
    ##     stop("Input x and y should be vectors of the same length", call.=FALSE)
    ##   }
    ##   na.in.both <- ( is.na(x) & is.na(y) )
    ##   na.number  <- sum(na.in.both)
    ##   na.which   <- which(na.in.both)
    ## 
    ##   message("Found ", na.number, " NA's at position(s):", 
    ##           paste(na.which, collapse=", ") ) 
    ##   
    ##   return( list(number=na.number, which=na.which) )
    ## }

#### ok we are writing anothe function\!\!\!

### what do we want to do?

### we want to assess student overall grades\! because fuck that takes a long time, especially if you want to drop the lowest grade to give the student a boost\!

\#how do we exclude things

``` r
# student 1
#student1<- c(100, 100, 100, 100, 100, 100, 100, 90)
# student 2
#student2 <- c(100, NA, 90, 90, 90, 90, 97, 80)
## search google i guess
## i didn't find shit but i guess this does something??? it calls the data desired and adding "-" does the opposite, all the data that is not that!
#student1[which.min(student1)]
#student1[-which.min(student1)]
### ok but for student 2
#student2[which.min(student2)]
### we get the 80, not the NA, the 0 assignment. maybe tell them that NA == 0
#student2[which.min(student2)]
#### i tried this 
#student2[which.min(student2, na.rm=TRUE)]
### you have to put it into the mean function
#mean(student2[which.min(student2)], na.rm=TRUE)
#mean(student2[-which.min(student2)], na.rm=TRUE)
### that works but now if a student has more than one NA, it drops all of that! we need to tell it NA = 0 so it drops just of those! 
## make a student 3
#student3<- c(100, 100, 100,NA, 100, NA, 100, 90)
### let's look at something
#student3[-which.min(student3)]
#sum(student3[-which.min(student3)], na.rm=TRUE)/(length(student3-1))
##this gives us 62.5 that seems a little low
## now the na.rm is called TRUE and reads as 0 but it exluded the 90, when it needs to drop and NA. check the ""student3[which.min(student3)]" and "student3[-which.min(student3)]"
## x[is.na(x)]<-0 will do this 
###sum(x[-which.min(x)]/(length(x-1))
#gradesum<- function(x) {
 #  x[is.na(x)]<-0
   # sum(x[-which.min(x)])/(length(x-1))
#}
#gradesum(student3)
## now that gives us 73.75 a much better grade! 
```

### ok i guess we are moving on?? whatever…

### CRAN and Bioconductor

\#CRAN is an archive of R packages, 15,000 packages and counting \#
cran.r-project.org \# installing a package is
install.packages(“package\_name”) \#library(“package\_name”) \#Bio
Conductor is a softwhare repository of R packages with some rules and
guiding principles. 1741 packages and counting \#\#\# Emphasis on
reproducible research \#\#\# install bioconductor from CRAN first (see
lecture 7 slides)
