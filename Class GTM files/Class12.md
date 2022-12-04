Class12
================
Gregory Jordan

# 1. Bioconductor and DESeq2 Setup

``` r
#install.packages(BiocManager)
#library(BiocManager)
#BiocManager::Install(DESeq2)
library(DESeq2)
```

# 2. Import CountData and ColData

## Input Data:

we need at least 2 things: -count data (genes in rows and exp in cols)
-metadata (a.k.a. `colData`)

``` r
counts <- read.csv("https://bioboot.github.io/bimm143_W18/class-material/airway_scaledcounts.csv",row.names = 1)
metadata <- read.csv("https://bioboot.github.io/bimm143_W18/class-material/airway_metadata.csv")
```

``` r
head(metadata)
```

              id     dex celltype     geo_id
    1 SRR1039508 control   N61311 GSM1275862
    2 SRR1039509 treated   N61311 GSM1275863
    3 SRR1039512 control  N052611 GSM1275866
    4 SRR1039513 treated  N052611 GSM1275867
    5 SRR1039516 control  N080611 GSM1275870
    6 SRR1039517 treated  N080611 GSM1275871

``` r
head(counts)
```

                    SRR1039508 SRR1039509 SRR1039512 SRR1039513 SRR1039516
    ENSG00000000003        723        486        904        445       1170
    ENSG00000000005          0          0          0          0          0
    ENSG00000000419        467        523        616        371        582
    ENSG00000000457        347        258        364        237        318
    ENSG00000000460         96         81         73         66        118
    ENSG00000000938          0          0          1          0          2
                    SRR1039517 SRR1039520 SRR1039521
    ENSG00000000003       1097        806        604
    ENSG00000000005          0          0          0
    ENSG00000000419        781        417        509
    ENSG00000000457        447        330        324
    ENSG00000000460         94        102         74
    ENSG00000000938          0          0          0

We need to make sure that the metadata and our counts match!

``` r
metadata$id
```

    [1] "SRR1039508" "SRR1039509" "SRR1039512" "SRR1039513" "SRR1039516"
    [6] "SRR1039517" "SRR1039520" "SRR1039521"

``` r
colnames(counts)
```

    [1] "SRR1039508" "SRR1039509" "SRR1039512" "SRR1039513" "SRR1039516"
    [6] "SRR1039517" "SRR1039520" "SRR1039521"

``` r
#all function to test if every occurance is True
all(metadata$id==colnames(counts))
```

    [1] TRUE

Q1. How many genes are in this dataset?

``` r
#goood ol' nrow
nrow(counts)
```

    [1] 38694

Q2. How many ‘control’ cell lines do we have?

``` r
#filtering by dex==control
nrow(metadata[metadata$dex=="control",])
```

    [1] 4

# 3. Toy Differential Expression

First silo control and treatment into separate datasets with their
corresponding count data

``` r
control <- metadata[metadata$dex=="control",]
head(control)
```

              id     dex celltype     geo_id
    1 SRR1039508 control   N61311 GSM1275862
    3 SRR1039512 control  N052611 GSM1275866
    5 SRR1039516 control  N080611 GSM1275870
    7 SRR1039520 control  N061011 GSM1275874

``` r
#link control metadata to the control ids
control.counts <- counts[,control$id]
head(control.counts)
```

                    SRR1039508 SRR1039512 SRR1039516 SRR1039520
    ENSG00000000003        723        904       1170        806
    ENSG00000000005          0          0          0          0
    ENSG00000000419        467        616        582        417
    ENSG00000000457        347        364        318        330
    ENSG00000000460         96         73        118        102
    ENSG00000000938          0          1          2          0

``` r
#to calcute means for each gene use rowmeans
control.means<-rowMeans(control.counts)
head(control.means)
```

    ENSG00000000003 ENSG00000000005 ENSG00000000419 ENSG00000000457 ENSG00000000460 
             900.75            0.00          520.50          339.75           97.25 
    ENSG00000000938 
               0.75 

Q3. How would you make the above code in either approach more robust?

You could combine all of the chunks onto one line, because R does not
care how it is input. However, people care! Definitely do it in multiple
lines so it is easy to follow

Q4. Follow the same procedure for the treated samples (i.e. calculate
the mean per gene across drug treated samples and assign to a labeled
vector called treated.mean)

``` r
#filter by dex==treated
treated <- metadata[metadata$dex=="treated",]
head(treated)
```

              id     dex celltype     geo_id
    2 SRR1039509 treated   N61311 GSM1275863
    4 SRR1039513 treated  N052611 GSM1275867
    6 SRR1039517 treated  N080611 GSM1275871
    8 SRR1039521 treated  N061011 GSM1275875

``` r
#link treatment metadata to treatment id
treated.counts <- counts[,treated$id]
head(treated.counts)
```

                    SRR1039509 SRR1039513 SRR1039517 SRR1039521
    ENSG00000000003        486        445       1097        604
    ENSG00000000005          0          0          0          0
    ENSG00000000419        523        371        781        509
    ENSG00000000457        258        237        447        324
    ENSG00000000460         81         66         94         74
    ENSG00000000938          0          0          0          0

``` r
#get row means 
head(treated.means<-rowMeans(treated.counts))
```

    ENSG00000000003 ENSG00000000005 ENSG00000000419 ENSG00000000457 ENSG00000000460 
             658.00            0.00          546.00          316.50           78.75 
    ENSG00000000938 
               0.00 

Q5 (a). Create a scatter plot showing the mean of the treated samples
against the mean of the control samples. Your plot should look something
like the following.

create a dataframe of control and treated means

``` r
meancounts<-data.frame(control.means,treated.means)
```

``` r
plot(meancounts)
```

![](Class12_files/figure-commonmark/unnamed-chunk-34-1.png)

Q5 (b).You could also use the ggplot2 package to make this figure
producing the plot below. What geom\_?() function would you use for this
plot?

geom_point()

Q6. Try plotting both axes on a log scale. What is the argument to
plot() that allows you to do this?

``` r
#use log argument to logscale your axes
plot(meancounts,log="xy")
```

    Warning in xy.coords(x, y, xlabel, ylabel, log): 15032 x values <= 0 omitted
    from logarithmic plot

    Warning in xy.coords(x, y, xlabel, ylabel, log): 15281 y values <= 0 omitted
    from logarithmic plot

![](Class12_files/figure-commonmark/unnamed-chunk-36-1.png)

log2 transforms are useful to determine fold change for genes in control
vs treated. This is b/c log2 of 1 is 0, log2 of 2 is 1, log2 of 0.5 is
-1, etc.

Log2 Fold Change

``` r
#make dataframe of meancounts
meancounts$log2fc <- log2(meancounts$treated.means / meancounts$control.means)
```

``` r
head(meancounts)
```

                    control.means treated.means      log2fc
    ENSG00000000003        900.75        658.00 -0.45303916
    ENSG00000000005          0.00          0.00         NaN
    ENSG00000000419        520.50        546.00  0.06900279
    ENSG00000000457        339.75        316.50 -0.10226805
    ENSG00000000460         97.25         78.75 -0.30441833
    ENSG00000000938          0.75          0.00        -Inf

need to remove the -infinity (had zero expression) and NA values (data
missing)

``` r
mycounts <- meancounts[rowSums(meancounts[,1:2] == 0) == 0,]
head(mycounts)
```

                    control.means treated.means      log2fc
    ENSG00000000003        900.75        658.00 -0.45303916
    ENSG00000000419        520.50        546.00  0.06900279
    ENSG00000000457        339.75        316.50 -0.10226805
    ENSG00000000460         97.25         78.75 -0.30441833
    ENSG00000000971       5219.00       6687.50  0.35769358
    ENSG00000001036       2327.00       1785.75 -0.38194109

``` r
nrow(mycounts)
```

    [1] 21817

about 20,000-25,000 genes in the human genome. this looks good

Q7. What is the purpose of the arr.ind argument in the which() function
call above? Why would we then take the first column of the output and
need to call the unique() function?

The arr.ind=TRUE argument will clause which() to return both the row and
column indices (i.e. positions) where there are TRUE values. In this
case this will tell us which genes (rows) and samples (columns) have
zero counts. We are going to ignore any genes that have zero counts in
any sample so we just focus on the row answer. Calling unique() will
ensure we dont count any row twice if it has zer entries in both
samples.

Commonly called that Log2 fold change of +2 is considered upregulated
(and vice versa for downregulated)

Q8. Using the up.ind vector above can you determine how many up
regulated genes we have at the greater than 2 fc level?

``` r
#we did not use the up.ind vector in class
sum(mycounts$log2fc>=2)
```

    [1] 314

Q9. Using the down.ind vector above can you determine how many down
regulated genes we have at the greater than 2 fc level?

``` r
#we did not use the down.ind vector in class
sum(mycounts$log2fc<=-2)
```

    [1] 485

Q10. Do you trust these results? Why or why not?

Not really. We don’t have any statistical significance values

# 4. DESeq2 Analysis

Time to do things the way the rest of the world does them. with DeSeq2

DeSeq2 wants counts and coldata and the “design” i.e. what to compare to
what

``` r
#use the tilda ~ to get the design column
dds <- DESeqDataSetFromMatrix(countData = counts,colData = metadata,design = ~dex)
```

    converting counts to integer mode

    Warning in DESeqDataSet(se, design = design, ignoreRank): some variables in
    design formula are characters, converting to factors

``` r
#use DESeq to do what we just did except better and with p values
dds <- DESeq(dds)
```

    estimating size factors

    estimating dispersions

    gene-wise dispersion estimates

    mean-dispersion relationship

    final dispersion estimates

    fitting model and testing

``` r
res <- results(dds)
```

``` r
res
```

    log2 fold change (MLE): dex treated vs control 
    Wald test p-value: dex treated vs control 
    DataFrame with 38694 rows and 6 columns
                     baseMean log2FoldChange     lfcSE      stat    pvalue
                    <numeric>      <numeric> <numeric> <numeric> <numeric>
    ENSG00000000003  747.1942     -0.3507030  0.168246 -2.084470 0.0371175
    ENSG00000000005    0.0000             NA        NA        NA        NA
    ENSG00000000419  520.1342      0.2061078  0.101059  2.039475 0.0414026
    ENSG00000000457  322.6648      0.0245269  0.145145  0.168982 0.8658106
    ENSG00000000460   87.6826     -0.1471420  0.257007 -0.572521 0.5669691
    ...                   ...            ...       ...       ...       ...
    ENSG00000283115  0.000000             NA        NA        NA        NA
    ENSG00000283116  0.000000             NA        NA        NA        NA
    ENSG00000283119  0.000000             NA        NA        NA        NA
    ENSG00000283120  0.974916      -0.668258   1.69456 -0.394354  0.693319
    ENSG00000283123  0.000000             NA        NA        NA        NA
                         padj
                    <numeric>
    ENSG00000000003  0.163035
    ENSG00000000005        NA
    ENSG00000000419  0.176032
    ENSG00000000457  0.961694
    ENSG00000000460  0.815849
    ...                   ...
    ENSG00000283115        NA
    ENSG00000283116        NA
    ENSG00000283119        NA
    ENSG00000283120        NA
    ENSG00000283123        NA

``` r
summary(res)
```


    out of 25258 with nonzero total read count
    adjusted p-value < 0.1
    LFC > 0 (up)       : 1563, 6.2%
    LFC < 0 (down)     : 1188, 4.7%
    outliers [1]       : 142, 0.56%
    low counts [2]     : 9971, 39%
    (mean count < 10)
    [1] see 'cooksCutoff' argument of ?results
    [2] see 'independentFiltering' argument of ?results

``` r
#if we wanted we can add the alpha argument to change significance we care about
res05<-results(dds,alpha=0.5)
summary(res05)
```


    out of 25258 with nonzero total read count
    adjusted p-value < 0.5
    LFC > 0 (up)       : 3375, 13%
    LFC < 0 (down)     : 3108, 12%
    outliers [1]       : 142, 0.56%
    low counts [2]     : 8564, 34%
    (mean count < 4)
    [1] see 'cooksCutoff' argument of ?results
    [2] see 'independentFiltering' argument of ?results

# 5. Adding Annotation Data

we skipped this in class

…revisiting on 11/9/22

``` r
#need new bioc annotation packages
library("AnnotationDbi")
library("org.Hs.eg.db")
```

``` r
columns(org.Hs.eg.db)
```

     [1] "ACCNUM"       "ALIAS"        "ENSEMBL"      "ENSEMBLPROT"  "ENSEMBLTRANS"
     [6] "ENTREZID"     "ENZYME"       "EVIDENCE"     "EVIDENCEALL"  "GENENAME"    
    [11] "GO"           "GOALL"        "IPI"          "MAP"          "OMIM"        
    [16] "ONTOLOGY"     "ONTOLOGYALL"  "PATH"         "PFAM"         "PMID"        
    [21] "PROSITE"      "REFSEQ"       "SYMBOL"       "UCSCKG"       "UNIGENE"     
    [26] "UNIPROT"     

``` r
res$symbol <- mapIds(org.Hs.eg.db,
                     keys=row.names(res), # Our genenames
                     keytype="ENSEMBL",        # The format of our genenames
                     column="SYMBOL",          # The new format we want to add
                     multiVals="first")
```

    'select()' returned 1:many mapping between keys and columns

``` r
head(res)
```

    log2 fold change (MLE): dex treated vs control 
    Wald test p-value: dex treated vs control 
    DataFrame with 6 rows and 7 columns
                      baseMean log2FoldChange     lfcSE      stat    pvalue
                     <numeric>      <numeric> <numeric> <numeric> <numeric>
    ENSG00000000003 747.194195     -0.3507030  0.168246 -2.084470 0.0371175
    ENSG00000000005   0.000000             NA        NA        NA        NA
    ENSG00000000419 520.134160      0.2061078  0.101059  2.039475 0.0414026
    ENSG00000000457 322.664844      0.0245269  0.145145  0.168982 0.8658106
    ENSG00000000460  87.682625     -0.1471420  0.257007 -0.572521 0.5669691
    ENSG00000000938   0.319167     -1.7322890  3.493601 -0.495846 0.6200029
                         padj      symbol
                    <numeric> <character>
    ENSG00000000003  0.163035      TSPAN6
    ENSG00000000005        NA        TNMD
    ENSG00000000419  0.176032        DPM1
    ENSG00000000457  0.961694       SCYL3
    ENSG00000000460  0.815849    C1orf112
    ENSG00000000938        NA         FGR

Q11. Run the mapIds() function two more times to add the Entrez ID and
UniProt accession and GENENAME as new columns called
res$entrez, res$uniprot and res\$genename.

``` r
res$entrez <- mapIds(org.Hs.eg.db,
                     keys=row.names(res),
                     column="ENTREZID",
                     keytype="ENSEMBL",
                     multiVals="first")
```

    'select()' returned 1:many mapping between keys and columns

``` r
res$uniprot <- mapIds(org.Hs.eg.db,
                     keys=row.names(res),
                     column="UNIPROT",
                     keytype="ENSEMBL",
                     multiVals="first")
```

    'select()' returned 1:many mapping between keys and columns

``` r
res$genename <- mapIds(org.Hs.eg.db,
                     keys=row.names(res),
                     column="GENENAME",
                     keytype="ENSEMBL",
                     multiVals="first")
```

    'select()' returned 1:many mapping between keys and columns

``` r
head(res$entrez)
```

    ENSG00000000003 ENSG00000000005 ENSG00000000419 ENSG00000000457 ENSG00000000460 
             "7105"         "64102"          "8813"         "57147"         "55732" 
    ENSG00000000938 
             "2268" 

``` r
head(res$uniprot)
```

    ENSG00000000003 ENSG00000000005 ENSG00000000419 ENSG00000000457 ENSG00000000460 
       "A0A024RCI0"        "Q9H2S6"        "O60762"        "Q8IZE3"    "A0A024R922" 
    ENSG00000000938 
           "P09769" 

``` r
head(res$genename)
```

                                                  ENSG00000000003 
                                                  "tetraspanin 6" 
                                                  ENSG00000000005 
                                                    "tenomodulin" 
                                                  ENSG00000000419 
    "dolichyl-phosphate mannosyltransferase subunit 1, catalytic" 
                                                  ENSG00000000457 
                                       "SCY1 like pseudokinase 3" 
                                                  ENSG00000000460 
                            "chromosome 1 open reading frame 112" 
                                                  ENSG00000000938 
                 "FGR proto-oncogene, Src family tyrosine kinase" 

# 6. Data Visualization

## Volcano Plots

Let’s see log2 up/down regulation and the significance graphed using a
volcanooooo plot

``` r
#remember to make it log2 fold change
plot(res$log2FoldChange,res$padj)
```

![](Class12_files/figure-commonmark/unnamed-chunk-72-1.png)

This graph is coolish, but ugly and not informative enough. let’s take
the log of the p value to adjust

``` r
#log scale now
plot(res$log2FoldChange,log(res$padj))
```

![](Class12_files/figure-commonmark/unnamed-chunk-74-1.png)

ok this is cool now but we usually want to associate lower p values with
higher signfiicance so let’s flip it to make it easier to interpret

``` r
#use abline to add lines to cut off our data based off what we want. i.e. +/-2 fold change and p value > .05
plot(res$log2FoldChange,-log(res$padj),xlab="Log2 Fold-Change",ylab="-Log(P-value)")
abline(v=2,col="red")
abline(v=-2,col="red")
abline(h=-log(.05),col="red")
```

![](Class12_files/figure-commonmark/unnamed-chunk-76-1.png)

# 7. Pathway Analysis

the 2 main datasets for pathway analysis are GO and KEGG

we will use the **gage** package to start with.

First load in our packages to set up the KEGG data sets

``` r
library(pathview)
```

    ##############################################################################
    Pathview is an open source software package distributed under GNU General
    Public License version 3 (GPLv3). Details of GPLv3 is available at
    http://www.gnu.org/licenses/gpl-3.0.html. Particullary, users are required to
    formally cite the original Pathview paper (not just mention it) in publications
    or products. For details, do citation("pathview") within R.

    The pathview downloads and uses KEGG data. Non-academic uses may require a KEGG
    license agreement (details at http://www.kegg.jp/kegg/legal.html).
    ##############################################################################

``` r
library(gage)
```

``` r
library(gageData)

#loading in data from kegg
data(kegg.sets.hs)

#looking at the first 2 pathways in the kegg dataset for humans
head(kegg.sets.hs,2)
```

    $`hsa00232 Caffeine metabolism`
    [1] "10"   "1544" "1548" "1549" "1553" "7498" "9"   

    $`hsa00983 Drug metabolism - other enzymes`
     [1] "10"     "1066"   "10720"  "10941"  "151531" "1548"   "1549"   "1551"  
     [9] "1553"   "1576"   "1577"   "1806"   "1807"   "1890"   "221223" "2990"  
    [17] "3251"   "3614"   "3615"   "3704"   "51733"  "54490"  "54575"  "54576" 
    [25] "54577"  "54578"  "54579"  "54600"  "54657"  "54658"  "54659"  "54963" 
    [33] "574537" "64816"  "7083"   "7084"   "7172"   "7363"   "7364"   "7365"  
    [41] "7366"   "7367"   "7371"   "7372"   "7378"   "7498"   "79799"  "83549" 
    [49] "8824"   "8833"   "9"      "978"   

The main gage() function requires a named vector of fold changes, where
the names of the values are the Entrez gene IDs.

Note that we used the mapIDs() function above to obtain Entrez gene IDs
(stored in
res$entrez) and we have the fold change results from DESeq2 analysis (stored in res$log2FoldChange).

``` r
#setting the names of my fold changes to be the entrez IDs 
foldchanges <- res$log2FoldChange
names(foldchanges) <- res$entrez
head(foldchanges)
```

           7105       64102        8813       57147       55732        2268 
    -0.35070302          NA  0.20610777  0.02452695 -0.14714205 -1.73228897 

now I can do pathway analysis

``` r
#if I wanted to do GO as my pathway I would change the gsets to Go
keggres <- gage(foldchanges,gsets=kegg.sets.hs)
```

``` r
#check out the attributes
attributes(keggres)
```

    $names
    [1] "greater" "less"    "stats"  

It appears we are looking at genes that are \>0 in the greater object
and \<0 in the less object

``` r
#Each keggres$less and keggres$greater object is data matrix with gene sets as rows sorted by p-value.
head(keggres$less,3)
```

                                          p.geomean stat.mean        p.val
    hsa05332 Graft-versus-host disease 0.0004250461 -3.473346 0.0004250461
    hsa04940 Type I diabetes mellitus  0.0017820293 -3.002352 0.0017820293
    hsa05310 Asthma                    0.0020045888 -3.009050 0.0020045888
                                            q.val set.size         exp1
    hsa05332 Graft-versus-host disease 0.09053483       40 0.0004250461
    hsa04940 Type I diabetes mellitus  0.14232581       42 0.0017820293
    hsa05310 Asthma                    0.14232581       29 0.0020045888

Now let’s see it visually in the pathway!

``` r
#our gene data is our named foldchanges (i.e. fold change and gene names) and our pathway id is whatever we put in as a pathway we want to analyze. So cool!
pathview(gene.data=foldchanges,pathway.id = "hsa05310")
```

    Info: Downloading xml files for hsa05310, 1/1 pathways..

    Info: Downloading png files for hsa05310, 1/1 pathways..

    'select()' returned 1:1 mapping between keys and columns

    Info: Working in directory /Users/gregoryjordan/Desktop/BGGN213/BGGN_213_R_Project/Classes_qmd_compiled

    Info: Writing image file hsa05310.pathview.png

![The Asthma pathway with our
genes](/Users/gregoryjordan/Desktop/BGGN213/BGGN%20213_R%20Project/Class12/hsa05310.pathview.png)
