---
layout: post
title: Solve common R problems efficiently with data.table
tags: R data.table
---



I was recently browsing stackoverflow.com (often called *SO*) for the most voted questions under [R tag](https://stackoverflow.com/questions/tagged/r?sort=votes).  
To my surprise, many questions on the first page were quite well addressed with the data.table package. I found a few other questions that could benefit from a data.table answer, therefore went ahead and answered them.  
In this post, I’d like to summarise them along with benchmarks (where possible) and my comments if any.  
Many answers under highly voted questions seem to have been posted a while back. data.table is quite actively developed and has had tons of improvements (in terms of speed and memory usage) over the  recent years. It might therefore be entirely possible that some of those answers will have even better performance by now.  

## 50 highest voted questions under R tag

Here’s the list of top 50 questions. I’ve marked those for which a data.table answer is available (which is usually quite performant).  


|  I| Number of votes|Question title                                               |Use data.table solution |
|--:|---------------:|:------------------------------------------------------------|:-----------------------|
|  1|            1153|How to make a great R reproducible example?                  |                        |
|  2|             621|How to sort a dataframe by column(s)?                        |TRUE                    |
|  3|             496|R Grouping functions: sapply vs. lapply vs. apply. vs. tappl |TRUE                    |
|  4|             429|How can we make xkcd style graphs?                           |                        |
|  5|             396|How to join (merge) data frames (inner, outer, left, right)? |TRUE                    |
|  6|             330|What statistics should a programmer (or computer scientist)  |                        |
|  7|             314|Drop columns in R data frame                                 |TRUE                    |
|  8|             290|Tricks to manage the available memory in an R session        |                        |
|  9|             280|Remove rows with NAs in data.frame                           |TRUE                    |
| 10|             279|Quickly reading very large tables as dataframes in R         |TRUE                    |
| 11|             263|How to properly document S4 class slots using Roxygen2?      |                        |
| 12|             250|Assignment operators in R: &#39;=&#39; and &#39;&lt;-&#39;   |                        |
| 13|             236|Drop factor levels in a subsetted data frame                 |TRUE                    |
| 14|             234|Plot two graphs in same plot in R                            |                        |
| 15|             225|What is the difference between require() and library()?      |                        |
| 16|             221|data.table vs dplyr: can one do something well the other can |                        |
| 17|             216|In R, why is `[` better than `subset`?                       |                        |
| 18|             212|R function for testing if a vector contains a given element  |                        |
| 19|             201|Expert R users, what&#39;s in your .Rprofile?                |                        |
| 20|             197|R list to data frame                                         |TRUE                    |
| 21|             197|Rotating and spacing axis labels in ggplot2                  |                        |
| 22|             197|How to Correctly Use Lists in R?                             |                        |
| 23|             192|How to convert a factor to an integer\numeric without a loss |                        |
| 24|             184|How can I read command line parameters from an R script?     |                        |
| 25|             184|How to unload a package without restarting R?                |                        |
| 26|             182|Tools for making latex tables in R                           |                        |
| 27|             181|In R, what is the difference between the [] and [[]] notatio |                        |
| 28|             180|How can I view the source code for a function?               |                        |
| 29|             171|Cluster analysis in R: determine the optimal number of clust |                        |
| 30|             170|How do I install an R package from source?                   |                        |
| 31|             162|How do I replace NA values with zeros in R?                  |                        |
| 32|             152|Counting the number of elements with the values of x in a ve |                        |
| 33|             152|Write lines of text to a file in R                           |                        |
| 34|             151|Standard library function in R for finding the mode?         |                        |
| 35|             150|How to trim leading and trailing whitespace in R?            |                        |
| 36|             143|How to save a plot as image on the disk?                     |                        |
| 37|             139|Most underused data visualization                            |                        |
| 38|             137|Convert data.frame columns from factors to characters        |TRUE                    |
| 39|             136|How to find the length of a string in R?                     |                        |
| 40|             134|Workflow for statistical analysis and report writing         |                        |
| 41|             132|Create an empty data.frame                                   |                        |
| 42|             130|adding leading zeros using R                                 |                        |
| 43|             129|Check existence of directory and create if doesn&#39;t exist |                        |
| 44|             127|Run R script from command line                               |                        |
| 45|             125|Changing column names of a data frame in R                   |TRUE                    |
| 46|             120|How to set limits for axes in ggplot2 R plots?               |                        |
| 47|             114|How to find out which package version is loaded in R?        |                        |
| 48|             112|How to plot two histograms together in R?                    |                        |
| 49|             112|How can 2 strings be concatenated in R                       |                        |
| 50|             112|How to organize large R programs?                            |                        |

Below are the chosen answers where data.table can be applied. Each one supplied with the usage and timing copied from the linked answer. Click on the question title to view SO question or follow the answer link for a reproducible example and benchmark details.  

### [How to sort a dataframe by column(s)?](http://stackoverflow.com/q/1296646/2490497)

Sort dataset `dat` by variables `z` and `b`. Use descending order for `z` and ascending for `b`.  

```r
setorder(dat, -z, b)  
```

Timing and memory consumption:  

```r
# R-session memory usage (BEFORE) = ~2GB (size of 'dat')
# ------------------------------------------------------------
# Package      function    Time (s)  Peak memory   Memory used
# ------------------------------------------------------------
# doBy          orderBy      409.7        6.7 GB        4.7 GB
# taRifx           sort      400.8        6.7 GB        4.7 GB
# plyr          arrange      318.8        5.6 GB        3.6 GB 
# base R          order      299.0        5.6 GB        3.6 GB
# dplyr         arrange       62.7        4.2 GB        2.2 GB
# ------------------------------------------------------------
# data.table      order        6.2        4.2 GB        2.2 GB
# data.table   setorder        4.5        2.4 GB        0.4 GB
# ------------------------------------------------------------
```
[Arun's answer](http://stackoverflow.com/a/29331287/2490497)

### [R Grouping functions: sapply vs. lapply vs. apply. vs. tapply vs. by vs. aggregate](http://stackoverflow.com/q/3505701/2490497)

Having dataset `dt` with variables `x` and `grp` calculate sum of `x` and length of `x` by groups specified by `grp` variable.  

```r
dt[, .(sum(x), .N), grp]
```

Timing in seconds:  

```r
#          fun elapsed
#1:  aggregate 109.139
#2:         by  25.738
#3:      dplyr  18.978
#4:     tapply  17.006
#5:     lapply  11.524
#6:     sapply  11.326
#7: data.table   2.686
```
[jangorecki's answer](http://stackoverflow.com/a/34167477/2490497)

### [How to join (merge) data frames (inner, outer, left, right)?](http://stackoverflow.com/q/1299871/2490497)

Join data.table `dt1` and `dt2` having common join column named *CustomerId*.  

```r
# right outer join keyed data.tables
dt1[dt2]
# right outer join unkeyed data.tables - use `on` argument
dt1[dt2, on = "CustomerId"]
# left outer join - swap dt1 with dt2
dt2[dt1, on = "CustomerId"]
# inner join - use `nomatch` argument
dt1[dt2, nomatch=0L, on = "CustomerId"]
# anti join - use `!` operator
dt1[!dt2, on = "CustomerId"]
# inner join
merge(dt1, dt2, by = "CustomerId")
# full outer join
merge(dt1, dt2, by = "CustomerId", all = TRUE)
# see ?merge.data.table arguments for other cases
```

Timing:  

```r
# inner join
#Unit: milliseconds
#       expr        min         lq      mean     median        uq       max neval
#       base 15546.0097 16083.4915 16687.117 16539.0148 17388.290 18513.216    10
#      sqldf 44392.6685 44709.7128 45096.401 45067.7461 45504.376 45563.472    10
#      dplyr  4124.0068  4248.7758  4281.122  4272.3619  4342.829  4411.388    10
# data.table   937.2461   946.0227  1053.411   973.0805  1214.300  1281.958    10

# left outer join
#Unit: milliseconds
#       expr       min         lq       mean     median         uq       max neval
#       base 16140.791 17107.7366 17441.9538 17414.6263 17821.9035 19453.034    10
#      sqldf 43656.633 44141.9186 44777.1872 44498.7191 45288.7406 47108.900    10
#      dplyr  4062.153  4352.8021  4780.3221  4409.1186  4450.9301  8385.050    10
# data.table   823.218   823.5557   901.0383   837.9206   883.3292  1277.239    10

# right outer join
#Unit: milliseconds
#       expr        min         lq       mean     median        uq       max neval
#       base 15821.3351 15954.9927 16347.3093 16044.3500 16621.887 17604.794    10
#      sqldf 43635.5308 43761.3532 43984.3682 43969.0081 44044.461 44499.891    10
#      dplyr  3936.0329  4028.1239  4102.4167  4045.0854  4219.958  4307.350    10
# data.table   820.8535   835.9101   918.5243   887.0207  1005.721  1068.919    10

# full outer join
#Unit: seconds
#       expr       min        lq      mean    median        uq       max neval
#       base 16.176423 16.908908 17.485457 17.364857 18.271790 18.626762    10
#      dplyr  7.610498  7.666426  7.745850  7.710638  7.832125  7.951426    10
# data.table  2.052590  2.130317  2.352626  2.208913  2.470721  2.951948    10
```
[jangorecki's answer](http://stackoverflow.com/a/34219998/2490497)

### [Drop columns in R data frame](http://stackoverflow.com/q/4605206/2490497)

Drop columns `a` and `b` from dataset `DT`, or drop columns by names stored in a variable. `set` function can be also used, it will work on data.frames too.  

```r
DT[, c('a','b') := NULL]
# or
del <- c('a','b')
DT[, (del) := NULL]
# or
set(DT, j = 'b', value = NULL)
```
[mnel's answer](http://stackoverflow.com/a/13371575/2490497)

No timing here as that process is almost instant, you can expect less memory consumption dropping columns with `:=` operator or `set` function as it is made by reference. In R version earlier than 3.1 regular data.frame methods would copy your data in memory.  

### [Quickly reading very large tables as dataframes in R](http://stackoverflow.com/q/1727772/2490497)

Reading 1 million rows dataset from csv file.  

```r
fread("test.csv")
```

Timing in seconds:  

```r
##    user  system elapsed  Method
##   24.71    0.15   25.42  read.csv (first time)
##   17.85    0.07   17.98  read.csv (second time)
##   10.20    0.03   10.32  Optimized read.table
##    3.12    0.01    3.22  fread
##   12.49    0.09   12.69  sqldf
##   10.21    0.47   10.73  sqldf on SO
##   10.85    0.10   10.99  ffdf
```
[mnel's answer](http://stackoverflow.com/a/15058684/2490497)

### [Remove rows with NAs in data.frame](http://stackoverflow.com/q/4862178/2490497)

There is `na.omit` data.table method which can be handly for that using `cols` argument.  
You should not expect performance impovement over data.frame methods other than faster detection of rows to delete (`NA`s in this example).  
Additional memory efficiency is expected in future thanks to [Delete rows by reference - data.table#635](https://github.com/Rdatatable/data.table/issues/635).  

### [Drop factor levels in a subsetted data frame](http://stackoverflow.com/q/1195826/2490497)

Drop levels for all factor columns in a dataset after making subset on it.  

```r
upd.cols = sapply(subdt, is.factor)
subdt[, names(subdt)[upd.cols] := lapply(.SD, factor), .SDcols = upd.cols]
```
[jangorecki's answer](http://stackoverflow.com/a/34181931/2490497)

No timing, don't expect speed up as the bottleneck is the `factor` function used to recreate the factor columns, this is also true for `droplevels` methods.  

### [R list to data frame](http://stackoverflow.com/q/4227223/2490497)

Having a list of data.frames called `ll`, perform row binding on all elements of the list into single dataset.  

```r
rbindlist(ll)
```

Timing:  

```r
system.time(ans1 <- rbindlist(ll))
#   user  system elapsed
#  3.419   0.278   3.718

system.time(ans2 <- rbindlist(ll, use.names=TRUE))
#   user  system elapsed
#  5.311   0.471   5.914

system.time(ans3 <- do.call("rbind", ll))
#     user   system  elapsed
# 1097.895 1209.823 2438.452 
```
[Arun's answer](http://stackoverflow.com/a/23983648/2490497)

### [Convert data.frame columns from factors to characters](http://stackoverflow.com/q/2851015/2490497)

This is a problem related to the fact that `data.frame()` by default converts character columns to factor. This is not a problem for `data.table()` which keeps character column class. So a question simply is already solved for data.table.  

```r
dt = data.table(col1 = c("a","b","c"), col2 = 1:3)
sapply(dt, class)
```

If you have a factor columns in your dataset already and you want to convert them to character you can do the following.  

```r
upd.cols = sapply(dt, is.factor)
dt[, names(dt)[upd.cols] := lapply(.SD, as.character), .SDcols = upd.cols]
```
[jangorecki's answer](http://stackoverflow.com/a/34188802/2490497)

No timings, speed up is unlikely as these are quite primitive operation. If you are creating factor columns in data.frame (default) then you can get speed up with data.table as you avoid `factor` function which depending on factor levels can be costly. Still those operations shouldn't be a bottleneck in your workflow.  

### [Changing column names of a data frame in R](http://stackoverflow.com/q/6081439/2490497)

Rename columns in `dt` dataset to *good* and *better*.  

```r
setnames(dt, c("good", "better"))
```
[jangorecki's answer](http://stackoverflow.com/a/34202613/2490497)

No timing here, you should not expect speed up but you can expect less memory consumption due to fact that `setnames` update names by reference.  

## Didn't find a problem you are looking for?

It is very likely as I just listed the questions from the first page of the most voted - so among 50 questions only.  
Before you ask new question on SO it may be wise to search for existing one because there is quite a lot already answered.  

> As of 11 Oct 2015, data.table was the 2nd largest tag about an R package

You can effectively search SO using `[r] [data.table] my problem here` in the SO search bar.  
If you decide to ask a question remember about *Minimal Reproducible Example* (MRE). If you need help on that see the highest voted question on R tag: [How to make a great R reproducible example?](http://stackoverflow.com/q/5963269/2490497).  
Also if you are new user to data.table you should take a look at [Getting started](https://github.com/Rdatatable/data.table/wiki/Getting-started) wiki.  

## Final note on performance

Seeing the above timings you can get a valid impression that data.table can dramatically reduce the time (and resources) required in your data processing workflow.  
Yet the above questions doesn't include few other use cases where data.table makes in fact the most stunning impression, those are:  

- reshaping data - [melt](https://stat.ethz.ch/pipermail/r-help/2015-April/427349.html), [dcast](http://stackoverflow.com/a/15669045/2490497)
- overlapping joins - [overlapping range joins](https://github.com/Rdatatable/data.table/wiki/talks/EARL2014_OverlapRangeJoin_Arun.pdf)
- rolling joins - [R - Data.Table Rolling Joins](http://gormanalysis.com/r-data-table-rolling-joins/)
- binary search - [Access data quickly and easily: data.table package](http://www.milanor.net/blog/?p=286)
- data.table index - [Scaling data.table using index](https://jangorecki.github.io/blog/2015-11-23/data.table-index.html)

It is good to be aware of those too.  

Any comments are welcome in the blog github repo as issues.  
  
