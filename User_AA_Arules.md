# Association Rule Mining
Brandon Stange  
July 9, 2015  

## My Background


- Currently a Data Scientist at Trinity Health
- Previously a Data Analyst for a large physician practice
- MA Economics (Econometrics, Environmental Econ)

Brandon.Stange@gmail.com    

## Accuracy vs. Interpretability
**Maximally Accurate**

- Non-Linear
- Feature Engineering/Interaction
- Ensembling

**Maximally Interpretable**

- Rule-Based
- Single Decision Trees
- Linear/Logistic Regression

## Association Rule Uses

- Originally used for *market basket analysis*
    - Find sets of products that are often bought together
    - Improve product arrangement
    - Targeted Advertising
    - {Beer} -> {Pizza}
- Can be used for any dataset that can be represented as a binary matrix
- Traditionally unsupervised, but can target specific outcomes
- Can target more than one outcome at once

## Algorithms and Implimentations

- Apriori
    - R, Python, SQL Server, Oracle, everywhere
- Eclat
    - R, Python, C, others
- FP Growth
    - C, Python, mahout
- Graph Databases
    - neo4j, Titan, Giraph

## Arules Package Ecosystem {.flexbox .vcenter .hcenter .bigger}
![Arules Ecosystem](images/ArulesEco.jpg)

## Input Data Structure {.flexbox .vcenter}
![Input Data Format](images/DataFormat.jpg)

## Rule Interpretation (Apriori Output)


|   |  lhs  | rhs | support | confidence | lift |
|:--|:-----:|:---:|:-------:|:----------:|:----:|
|16 | {b,c} | {d} |   0.4   |     1      | 2.5  |

<br>
RHS (Outcome) = **{d}**

LHS (Inputs) = **{b,c}**
LHS occurs in **40%** of total population.

RHS occurs in **100%** of these transactions, which is **2.5** times the population at large.

## Apriori Function Call {.bigger}


```r
apriori(data,
         parameter = list(minlen=1, 
                          support=0.05, 
                          confidence=0.4, 
                          maxlen=3),
         appearance = list(rhs=outcomelist, 
                           default="lhs"),
         control = list(verbose=T))
```

## Itemset Graph Representation {.flexbox .vcenter}
![Frequent Item Graph](images/FrequentItems.png)

## Session Info {.smaller}


```r
library(dplyr)
library(readr)
library(tidyr)
library(arules)
library(arulesViz)
print(sessionInfo(),locale=F)
```

```
## R version 3.2.2 (2015-08-14)
## Platform: x86_64-w64-mingw32/x64 (64-bit)
## Running under: Windows Server 2012 x64 (build 9200)
## 
## attached base packages:
## [1] grid      stats     graphics  grDevices utils     datasets  methods  
## [8] base     
## 
## other attached packages:
## [1] arulesViz_1.0-0     arules_1.1-6        Matrix_1.1-5       
## [4] tidyr_0.4.0         readr_0.2.2         dplyr_0.4.3        
## [7] RevoUtilsMath_3.2.2
## 
## loaded via a namespace (and not attached):
##  [1] Rcpp_0.12.3          formatR_1.2.1        plyr_1.8.3          
##  [4] bitops_1.0-6         iterators_1.0.8      tools_3.2.2         
##  [7] digest_0.6.9         evaluate_0.8         lattice_0.20-33     
## [10] foreach_1.4.3        igraph_1.0.1         DBI_0.3.1           
## [13] registry_0.3         yaml_2.1.13          parallel_3.2.2      
## [16] seriation_1.1-3      TSP_1.1-3            stringr_1.0.0       
## [19] knitr_1.12           cluster_2.0.3        caTools_1.17.1      
## [22] gtools_3.5.0         lmtest_0.9-34        vcd_1.4-1           
## [25] scatterplot3d_0.3-36 R6_2.1.1             rmarkdown_0.9.2     
## [28] gdata_2.17.0         magrittr_1.5         scales_0.3.0        
## [31] gplots_2.17.0        codetools_0.2-14     gclus_1.3.1         
## [34] htmltools_0.3        MASS_7.3-43          assertthat_0.1      
## [37] colorspace_1.2-6     KernSmooth_2.23-15   stringi_1.0-1       
## [40] munsell_0.4.2        zoo_1.7-12
```

## Real Data Example

**490k records from New York City resturant inspections**

requires readr, dplyr, tidyr, arules, arulesviz
(https://data.cityofnewyork.us)

**Fields of interest:**

- ID, name, address, phone, etc.
- Inspection Date
- Borough
- Cuisine Description
- Violation Description
- Action

## Data Pre-Processing

1. Clean up dates and special characters
2. Apply custom violation code grouping
3. Use tidyr::gather to put data in 'long' format
4. Replace keys and measures with integer IDs
5. Convert to sparse matrix


## Pre-Processing steps 3-5 Code


```r
## 3. Use tidyr::gather to put data in 'long' format
nycs <- nycw %>%
  select(INSID, BORO, CUISINE_DESCRIPTION, VIOLATION_TYPE) %>%
  gather(MEASURE, VALUE, -INSID)
nycs$MEASURE <- nycs$VALUE
nycs$VALUE <- 1

## 4. Replace keys and measures with integer IDs
ID <- unique(nycs$INSID)
ME <- unique(nycs$MEASURE)
nycs$MEASURE<-match(nycs$MEASURE,ME)
nycs$INSID<-match(nycs$INSID,ID)

## 5. Convert to sparse matrix
sm<-sparseMatrix(i=nycs$INSID, j=nycs$MEASURE,x=nycs$VALUE,
                 dimnames=list(ID,ME),giveCsparse=T)
```


## Example Function Call {.smaller}


```r
rules <- apriori(sm2,
                   parameter = list(minlen=1, supp=0.001, conf=0.4, maxlen=4),
                   appearance = list(rhs=outcomelist, none="Pass or Not Critical",
                                     default="lhs"),
                   control = list(verbose=T))
```

```r
outcomelist
```

```
## [1] "Temperature"        "Rodents/Pests"      "Food Handling"     
## [4] "Employee Practices" "Poor Equipment"     "Adminstrative"     
## [7] "Food Source"
```

```r
class(sm2)
```

```
## [1] "transactions"
## attr(,"package")
## [1] "arules"
```

## Results

21 rules were generated.  The top 5 are:

|   |         lhs         |       rhs       | support | confidence | lift  |
|:--|:-------------------:|:---------------:|:-------:|:----------:|:-----:|
|1  |   {Delicatessen}    |  {Temperature}  |  0.009  |   0.536    | 1.430 |
|24 | {MANHATTAN,Chinese} |  {Temperature}  |  0.013  |   0.516    | 1.376 |
|6  |     {Caribbean}     | {Rodents/Pests} |  0.014  |   0.496    | 1.344 |
|2  |   {Pizza/Italian}   |  {Temperature}  |  0.010  |   0.487    | 1.299 |
|25 | {MANHATTAN,Chinese} | {Rodents/Pests} |  0.011  |   0.473    | 1.280 |

## arulesViz plots

```r
plot(rules.sorted,method="grouped")
```

![](User_AA_Arules_files/figure-html/unnamed-chunk-9-1.png)\

## arulesViz plots contd

```r
plot(rules.sorted,method="paracoord")
```

![](User_AA_Arules_files/figure-html/unnamed-chunk-10-1.png)\

## arulesViz plots contd

```r
plot(rules.sorted,method="graph")
```

![](User_AA_Arules_files/figure-html/unnamed-chunk-11-1.png)\

## Arules Sequences

- Allows for pattern discovery in ordered sets of discrete data
- Ordered, not temporal (no measure of time between events)
- Right-censored, events *after* the event of interest are ignored
- arulesSequences::cspade()
- Parameters are similar: support, size, gap, window

## Resources

**Christian Borgelt's website**

Various publications and implementations of Association Rules

[www.borgelt.net](www.borgelt.net)

[Efficient Analysis of Pattern and Association Rule Mining Approaches](http://arxiv.org/ftp/arxiv/papers/1402/1402.2892.pdf)
