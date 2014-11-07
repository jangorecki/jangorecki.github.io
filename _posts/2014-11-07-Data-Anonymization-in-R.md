---
layout: post
title: Data anonymization in R
tags: R digest data.table
---

# Use cases

* Public reports.  
* Public data sharing, e.g. R packages download logs from CRAN's RStudio mirror - [cran-logs.rstudio.com](http://cran-logs.rstudio.com/) - mask ip addresses.
* Reports or data sharing for external vendor.
* Development works can operate on anonymized PRODUCTION data.  
Manually or semi-manually populated data can often brings some new issue after migration to PRODUCTION data.  
Such anonymized PRODUCTION data can be quite handy for the devs.  

# Dependencies


```r
suppressPackageStartupMessages({
  library(data.table)
  library(digest)
  library(knitr) # used only for post creation
})
```

# Sample of survey data

Anonymize sensitive information in survey data, data storage in a single table.


```r
# pretty print
kable(head(SURV))
```



|City    |Postal Code |Address           |Name           |Sex | Age| Height| Weight| Score|
|:-------|:-----------|:-----------------|:--------------|:---|---:|------:|------:|-----:|
|London  |SW1H 0QW    |Silk Road 17      |John Lennon    |M   |  48|    176|     94|     3|
|Cardiff |CF23 9AE    |Queen Road 19     |Edward Snowden |M   |  55|    185|     74|     2|
|London  |SW1P 3BU    |Edinburgh Road 19 |John Kennedy   |M   |  46|    156|     84|     1|
|London  |SW1P 3BU    |Cardiff Road 21   |Mahatma Gandhi |M   |  56|    186|     54|     5|
|Cardiff |CF23 9AE    |King Road 10      |Nelson Mandela |M   |  61|    181|     84|     2|
|London  |SW1P 2EE    |Cardiff Road 23   |Vandana Shiva  |F   |  41|    192|     64|     5|

# Anonymize function

Function will calculate hashes only for unique inputs and return vector of masked inputs.  
My version will use `digest(x, algo="crc32")` because it fits better into html tables, algo `crc32` is not really secure.  
Read `?digest::digest` for supported `algo`, also consider to salt your input vector, e.g. `x=paste0("prefix",x,"suffix")`.  
Performance improvement possible using `Rcpp` / `C`: [digest #2](https://github.com/eddelbuettel/digest/issues/2).

```r
anonymize <- function(x, algo="crc32"){
  unq_hashes <- vapply(unique(x), function(object) digest(object, algo=algo), FUN.VALUE="", USE.NAMES=TRUE)
  unname(unq_hashes[x])
}
```

# Anonymize survey data

We will keep *city* and *sex* fields unmasked.

```r
# choose columns to mask
cols_to_mask <- c("name","address","postal_code")
# backup original data
SURV_ORG <- copy(SURV)
# anonymize
SURV[,cols_to_mask := lapply(.SD, anonymize),.SDcols=cols_to_mask,with=FALSE]
# pretty print
kable(head(SURV))
```



|City    |Postal Code |Address  |Name     |Sex | Age| Height| Weight| Score|
|:-------|:-----------|:--------|:--------|:---|---:|------:|------:|-----:|
|London  |913ad86c    |c26dc5a8 |a6ccb226 |M   |  48|    176|     94|     3|
|Cardiff |921485db    |58be1ead |14404453 |M   |  55|    185|     74|     2|
|London  |4c0d9ac8    |7996c8e1 |66dc3ad0 |M   |  46|    156|     84|     1|
|London  |4c0d9ac8    |1a5ecf8b |44f84c46 |M   |  56|    186|     54|     5|
|Cardiff |921485db    |b4dce820 |b3445a6d |M   |  61|    181|     84|     2|
|London  |1f39765c    |f450aea7 |56efd861 |F   |  41|    192|     64|     5|

# Why not just random data or integer sequence

When using the `digest` function to hide sensitive data you:

* keep rows distribution:  
aggregates by masked columns will still match to aggregates on original columns, see simple grouping below:

```r
SURV_ORG[,.(.N,mean_age=mean(age),mean_score=mean(score)),by=.(city,postal_code)
         ][,kable(.SD)]
```



|City    |Postal Code |  N| Mean Age| Mean Score|
|:-------|:-----------|--:|--------:|----------:|
|London  |SW1H 0QW    |  1|    48.00|       3.00|
|Cardiff |CF23 9AE    |  3|    65.33|       2.33|
|London  |SW1P 3BU    |  2|    51.00|       3.00|
|London  |SW1P 2EE    |  2|    36.50|       3.50|
|Glasgow |G40 3AS     |  1|    53.00|       2.00|

```r
SURV[,.(.N,mean_age=mean(age),mean_score=mean(score)),by=.(city,postal_code)
     ][,kable(.SD)]
```



|City    |Postal Code |  N| Mean Age| Mean Score|
|:-------|:-----------|--:|--------:|----------:|
|London  |913ad86c    |  1|    48.00|       3.00|
|Cardiff |921485db    |  3|    65.33|       2.33|
|London  |4c0d9ac8    |  2|    51.00|       3.00|
|London  |1f39765c    |  2|    36.50|       3.50|
|Glasgow |90b79e54    |  1|    53.00|       2.00|
* keep relationships on equi joins:  
if `t1.col1 == t2.col4` TRUE then also `digest(t1.col1) == digest(t2.col4)` TRUE  
Example in next section below.

# Sample of sales data

Anonymize relational data in sales data, data normalized into *SALES* and *CUSTOMER* tables.



```r
kable(head(SALES,4))
```



|Customer Uid |Product Name |Transaction Date | Quantity| Value|
|:------------|:------------|:----------------|--------:|-----:|
|CUST_3       |rgr          |2014-10-28       |       34|   612|
|CUST_4       |jfc          |2014-10-13       |       42|   588|
|CUST_6       |hnm          |2014-11-06       |       40|   200|
|CUST_9       |zgm          |2014-11-04       |       40|   760|

```r
kable(head(CUSTOMER,2))
```



|Customer Uid |City    |Postal Code |Address       |Name           |Sex |
|:------------|:-------|:-----------|:-------------|:--------------|:---|
|CUST_1       |London  |SW1H 0QW    |Silk Road 17  |John Lennon    |M   |
|CUST_2       |Cardiff |CF23 9AE    |Queen Road 19 |Edward Snowden |M   |

```r
# join
kable(head(
  CUSTOMER[SALES]
))
```



|Customer Uid |City    |Postal Code |Address           |Name           |Sex |Product Name |Transaction Date | Quantity| Value|
|:------------|:-------|:-----------|:-----------------|:--------------|:---|:------------|:----------------|--------:|-----:|
|CUST_3       |London  |SW1P 3BU    |Edinburgh Road 19 |John Kennedy   |M   |rgr          |2014-10-28       |       34|   612|
|CUST_4       |London  |SW1P 3BU    |Cardiff Road 21   |Mahatma Gandhi |M   |jfc          |2014-10-13       |       42|   588|
|CUST_6       |London  |SW1P 2EE    |Cardiff Road 23   |Vandana Shiva  |F   |hnm          |2014-11-06       |       40|   200|
|CUST_9       |Glasgow |G40 3AS     |Simple Road 11    |Bob Marley     |M   |zgm          |2014-11-04       |       40|   760|
|CUST_2       |Cardiff |CF23 9AE    |Queen Road 19     |Edward Snowden |M   |qej          |2014-11-06       |       29|   493|
|CUST_9       |Glasgow |G40 3AS     |Simple Road 11    |Bob Marley     |M   |fnz          |2014-10-30       |       59|   649|

```r
# join and aggregate
kable(head(
  CUSTOMER[SALES][,.(quantity = sum(quantity),value = sum(value)),by=.(city,postal_code)]
))
```



|City    |Postal Code | Quantity| Value|
|:-------|:-----------|--------:|-----:|
|London  |SW1P 3BU    |      845| 10783|
|London  |SW1P 2EE    |      729|  9732|
|Glasgow |G40 3AS     |      376|  4887|
|Cardiff |CF23 9AE    |      981| 12983|
|London  |SW1H 0QW    |      329|  4099|

# Anonymize sales data


```r
SALES[, customer_uid := anonymize(customer_uid)]
cols_to_mask <- c("customer_uid","name","address","postal_code")
CUSTOMER[,cols_to_mask := lapply(.SD, anonymize),.SDcols=cols_to_mask,with=FALSE]
setkey(CUSTOMER,customer_uid)
```


```r
# preview result
kable(head(CUSTOMER,2))
```



|Customer Uid |City    |Postal Code |Address  |Name     |Sex |
|:------------|:-------|:-----------|:--------|:--------|:---|
|4a7d777      |Cardiff |921485db    |a759d95  |b51a2e5c |F   |
|73a0e7e1     |Glasgow |90b79e54    |a8708751 |7c739cd6 |M   |

```r
kable(head(SALES,2))
```



|Customer Uid |Product Name |Transaction Date | Quantity| Value|
|:------------|:------------|:----------------|--------:|-----:|
|93750eff     |rgr          |2014-10-28       |       34|   612|
|d119b5c      |jfc          |2014-10-13       |       42|   588|


```r
# datasets will still join correctly even on masked columns
kable(head(
  CUSTOMER[SALES]
))
```



|Customer Uid |City    |Postal Code |Address  |Name     |Sex |Product Name |Transaction Date | Quantity| Value|
|:------------|:-------|:-----------|:--------|:--------|:---|:------------|:----------------|--------:|-----:|
|93750eff     |London  |4c0d9ac8    |7996c8e1 |66dc3ad0 |M   |rgr          |2014-10-28       |       34|   612|
|d119b5c      |London  |4c0d9ac8    |1a5ecf8b |44f84c46 |M   |jfc          |2014-10-13       |       42|   588|
|e31ffa70     |London  |1f39765c    |f450aea7 |56efd861 |F   |hnm          |2014-11-06       |       40|   200|
|73a0e7e1     |Glasgow |90b79e54    |a8708751 |7c739cd6 |M   |zgm          |2014-11-04       |       40|   760|
|e4723e69     |Cardiff |921485db    |58be1ead |14404453 |M   |qej          |2014-11-06       |       29|   493|
|73a0e7e1     |Glasgow |90b79e54    |a8708751 |7c739cd6 |M   |fnz          |2014-10-30       |       59|   649|


```r
# also the aggregates on masked columns will match to the origin
kable(head(
    CUSTOMER[SALES][,.(quantity = sum(quantity),value = sum(value)),by=.(city,postal_code)]
))
```



|City    |Postal Code | Quantity| Value|
|:-------|:-----------|--------:|-----:|
|London  |4c0d9ac8    |      845| 10783|
|London  |1f39765c    |      729|  9732|
|Glasgow |90b79e54    |      376|  4887|
|Cardiff |921485db    |      981| 12983|
|London  |913ad86c    |      329|  4099|

# Reproduce from Rmd

Script used to produce this post is available in the github repo (link in the page footer) as `Rmd` file and can be easily reproduced locally in R (required [knitr](http://cran.r-project.org/web/packages/knitr/index.html) or [rmarkdown](http://cran.r-project.org/web/packages/rmarkdown/index.html)) to any format (`md`, `html`, `pdf`, `docx`).

```r
# html output
rmarkdown::render("2014-11-07-Data-Anonymization-in-R.Rmd", html_document())
# markdown file used as current post
knitr::knit("2014-11-07-Data-Anonymization-in-R.Rmd")
```

# Minimal script

Minimal script example on survey data as `SURV_ORG` data.table:


```r
anonymize <- function(x, algo="crc32"){
  unq_hashes <- vapply(unique(x), function(object) digest(object, algo=algo), FUN.VALUE="", USE.NAMES=TRUE)
  unname(unq_hashes[x])
}
cols_to_mask <- c("name","address","postal_code")
SURV_ORG[, cols_to_mask := lapply(.SD, anonymize), .SDcols=cols_to_mask, with=FALSE][]
```
