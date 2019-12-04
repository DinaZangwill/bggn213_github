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
