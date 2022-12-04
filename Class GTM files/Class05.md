Class 5
================
Gregory Jordan

- <a href="#our-first-plot" id="toc-our-first-plot">Our first plot</a>
- <a href="#section-5.-a-more-interesting-plot"
  id="toc-section-5.-a-more-interesting-plot">Section 5. A more
  interesting plot</a>

# Our first plot

R has base graphics

``` r
plot(cars)
```

![](Class05_files/figure-commonmark/unnamed-chunk-2-1.png)

How would I plot this with ‘ggplot2’? We need to install and load the
ggplot2 package first because it does not come with Base R. Use
‘install.packages()’ function to install any packages in R. Then use
‘library()’ function to load the package in.

``` r
#install.packages("ggplot2")
library(ggplot2)
ggplot(data=cars)
```

![](Class05_files/figure-commonmark/unnamed-chunk-4-1.png)

Note that you can’t just type ggplot and put in your data. You need to
add the 3 required layers in order to map your data to variables and
create visuals.

Every ggplot needs at least 3 layers:

- **Data** (i.e. the data.frame we have)
- **Aes** (the aesthetic mapping of our data to what we want to plot)
- **Geoms** (how we want to plot this stuff!)

``` r
ggplot(data=cars) + 
  aes(x=speed, y=dist) + 
  geom_point() + 
  geom_line()
```

![](Class05_files/figure-commonmark/unnamed-chunk-6-1.png)

Change ‘geom_line()’ to ‘geom_smooth()’

``` r
ggplot(data=cars) + 
  aes(x=speed, y=dist) + 
  geom_point() + 
  geom_smooth(se=FALSE,method=lm)
```

    `geom_smooth()` using formula 'y ~ x'

![](Class05_files/figure-commonmark/unnamed-chunk-8-1.png)

------------------------------------------------------------------------

# Section 5. A more interesting plot

``` r
url <- "https://bioboot.github.io/bimm143_S20/class-material/up_down_expression.txt"
genes <- read.delim(url)
```

Q1. How many genes are in this dataset?

There are 5196 genes in this dataset.

Q2. What are the column names and number of columns?

The column names are Gene, Condition1, Condition2, State and there are 4
columns

Q3. Use the ‘table()’ function to figure out number of up-regulated
genes

``` r
table(genes$State)
```


          down unchanging         up 
            72       4997        127 

Q4. What fraction of total genes are up-regulated?

``` r
"Percent Change:"
```

    [1] "Percent Change:"

``` r
round(table(genes$State)/nrow(genes)*100,2)
```


          down unchanging         up 
          1.39      96.17       2.44 

------------------------------------------------------------------------

Read in the genes data

Now plot the data

``` r
ggplot(data = genes) +
  aes(x=Condition1, y= Condition2, color=State) +
  scale_color_manual(values=c("blue","gray","red")) +
  geom_point(alpha=0.5) +
  labs(title="Gene Expression Changes Upon Drug Treatment",x="Control (no drug)",y="Drug Treatment")+
  theme_gray() 
```

![](Class05_files/figure-commonmark/unnamed-chunk-16-1.png)
