Class11
================
Gregory Jordan

# Question 13

read in the document:

``` r
expr <- read.table("https://bioboot.github.io/bggn213_W19/class-material/rs8067378_ENSG00000172057.6.txt")
head(expr)
```

       sample geno      exp
    1 HG00367  A/G 28.96038
    2 NA20768  A/G 20.24449
    3 HG00361  A/A 31.32628
    4 HG00135  A/A 34.11169
    5 NA18870  G/G 18.25141
    6 NA11993  A/A 32.89721

``` r
nrow(expr)
```

    [1] 462

``` r
table(expr$geno)
```


    A/A A/G G/G 
    108 233 121 

``` r
library(ggplot2)
```

let’s make a boxplot to display our data

``` r
ggplot(expr) + aes(geno, exp, fill=geno) + geom_boxplot(notch=TRUE)
```

![](Class11_files/figure-commonmark/unnamed-chunk-10-1.png)

# Q14

The median expression of A/A is elevated compared to G/G. The A/A SNP
increases the expression of ORMDL3 more than the G/G variant!
