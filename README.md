# KAML [![](https://img.shields.io/badge/Issues-1%2B-brightgreen.svg)](https://github.com/YinLiLin/R-KAML/issues) [![](https://img.shields.io/badge/Release-v1.0.1-ff69b4.svg)](https://github.com/YinLiLin/R-KAML/commits/master)

## Kinship Adjusted Multiple Locus Best Linear Unbiased Prediction

## Contents
* [OVERVIEW](#overview)
* [CITATION](#citation)
* [GETTING STARTED](#getting-started)
  - [Installation](#installation)
  - [Test Datasets](#test-datasets)
* [INPUT](#input)
  - [Phenotype file](#phenotype-file)/[Covariate file](#covariate-file)/[Kinship file](#kinship-file) 
  - [Genotype file](#genotype-file)
    * [Hapmap](#hapmap)/[PLINK Binary](#plink-binary)/[Numeric](#numeric)
* [USAGE](#usage)
  - [Basic](#basic)
  - [Advanced](#advanced)
* [OUTPUT](#output)
* [FAQ AND HINTS](#faq-and-hints)

---
## OVERVIEW
***`KAML`*** is originally designed to predict phenotypic value using genome- or chromosome-wide SNPs for sample traits which are controled by limited major markers or complex traits that are influenced by many minor-polygene. In brief, ***`KAML`*** incorporates pseudo QTNs as fixed effects and a trait-specific K matrix as random effect in a mixed linear model. Both pseudo QTNs and trait-specific K matrix are optimized using a parallel-accelerated machine learning strategy.

***`KAML`*** is developed by [***Lilin Yin***](https://github.com/YinLiLin), [***Xiaolei Liu***](https://github.com/XiaoleiLiuBio)**\***, [***Xinyun Li***](http://pdc.hzau.edu.cn/jgfw/desktop/grzy/show.htm?zgh=105042007017)**\***, [***Shuhong Zhao***](http://pdc.hzau.edu.cn/jgfw/desktop/grzy/show.htm?zgh=105041992036) at the [***Huazhong(Centra of China) agriculture University***](http://www.hzau.edu.cn/2014/ch/)

---
## CITATION
(waiting for updating)

---
## GETTING STARTED
***`KAML`*** is compatible with both [R](https://www.r-project.org/) and [Microsoft R Open](https://mran.microsoft.com/open/), We strongly recommend **MRO** instead of **R** for running ***`KAML`***. **MRO** is the enhanced distribution of **R**, it includes multi-threaded math libraries. These libraries make it possible for so many common R operations, ***such as matrix multiply/inverse, matrix decomposition, and some higher-level matrix operations***, to compute in parallel and use all of the processing power available to [reduce computation times](https://mran.microsoft.com/documents/rro/multithread/#mt-bench).

### Installation
***`KAML`*** is not available on CRAN, but can be installed using the R package **"devtools"**. There are two packages should be installed beforehand, **"snpStats"** and **"rfunctions"**. ***`KAML`*** can be installed with the following R code:
```r
#if "devtools" isn't installed, please "install.packages(devtools)" first.
devtools::install_version('RcppEigen', version = "0.3.2.9.0")
devtools::install_github("Bioconductor-mirror/snpStats")
devtools::install_github("jaredhuling/rfunctions")
devtools::install_github("YinLiLin/R-KAML/KAML")
```
After installed successfully, ***`KAML`*** can be loaded by typing ```library(KAML)```. Typing `?KAML` could get the details of all parameters.

### Test Datasets

```bash
wget https://raw.githubusercontent.com/YinLiLin/R-KAML/master/example/example.zip
unzip example.zip
cd example
```

**Or** click [here](https://raw.githubusercontent.com/YinLiLin/R-KAML/master/example/example.zip) in your browser to download.

---

## INPUT
### Phenotype file
The file must contain a header row. Missing values should be denoted by NA, which will be treated as candidates. Notice that only numeric values are allowed and characters will not be recognized. However, if a phenotype takes only values 0, 1(or only two levels), then ***`KAML`*** would consider it to be a case-control trait, and the predicted value could be directly interpreted as the probability of being a case. <br>
When a phenotype file contains more than one phenotype, users should specify which to analyse using the option "pheno=N", for example: KAML(..., pheno=1) means the trait in first column would be predicted. 

> `mouse.Pheno.txt`

| Trait1 | Trait2 | Trait3 | Trait4 | Trait5 | Trait6 |
| :---: | :---: |  :---: |  :---: |  :---: | :---: |
| 0.224992 | 0.224991 | NA | 1 | NA | -0.285427 |
| -0.974543 | -0.974542 | NA | 0 | NA | -2.333531 |
| 0.195909 | 0.195909 | NA | 1 | NA | 0.046818 |
| NA | NA | NA | NA | NA | NA |
| NA | NA | NA | NA | NA | NA |
| ... | ... | ... | ... | ... | ... |
| NA | NA | NA | NA | NA | 0.720009 |

### Covariate file
Generally, there are no covariates when predicting candidates in most cases, especially genomic selection of animal breeding, because the predicted values are not original phenotypes but the (genomic) estimated breeding value(GEBV/EBV), which have been corrected by covariates. So **Covariate file** is **optional**, in order to fit the model for original phenotype prediction, users can provide the covariates in file. If provided, NAs are not allowed in the file, the order of all individuals must be corresponding to phenotype file. As is the example below, the elements can be either character or numeric, ***`KAML`*** will regard the column whose levels is less than 50% of total individuals as fixed effect, and transform the (0, 1) identity matrix for it automatically, other columns will be treated as covariates directely.

> `CV.txt` ***(optional)***

| female | group1 | 1 | ... | 55 |
| :---: | :---: |  :---: |  :---: |  :---: |
| female | group2 | 1| ... | 57 |
| male | group2 | 2 | ... | 62 |
| male | group3 | 2| ... | 75 |
| male | group2 | 2| ... | 45 |
| ... | ... | ... | ... | ... |
| female | group3 | 1 | ... | 80 |

### Kinship file
***`KAML`*** requires a n×n relatedness matrix. By default, it is automatically calculated using choosed one type of three algorithms(
***"scale","center","vanraden"***
), but can be supplied in file by users. If in this case, the order of individuals for each row and each column in the file must correspond to phenotype file, no column and row names.

> `mouse.Kin.txt` ***(optional)***

| 0.3032 | -0.0193 | 0.0094 | 0.0024 | 0.0381 | ... | -0.0072 |
| :---: | :---: |  :---: |  :---: |  :---: |  :---: |  :---: |
| -0.0193 | 0.274 | -0.0243 | 0.0032 | -0.0081 | ... | 0.0056 |
| 0.0094 | -0.0243 | 0.3207 | -0.0071 | -0.0045 | ... | -0.0407 |
| 0.0024 | 0.0032 | -0.0071 | 0.321 | -0.008 | ... | -0.0093 |
| 0.0381 | -0.0081 | -0.0045 | -0.008 | 0.3498 | ... | -0.0238 |
| ... | ... | ... | ... | ... | ... | ... | 
| -0.0072 | 0.0056 | -0.0407 | -0.0093 | -0.0238 | ... | 0.3436 |

### Genotype file

With the increasing number of SNPs through whole genome, the storage of genotype is a big problem. Obviously, it's not a good choice to read it into memory with a memory-limited PC directly. Here, ***`KAML`*** is integrated with a memory-efficient tool named ***bigmemory*** and could obtain the genotype information from disk instead, which can save much of memory to do more analysis.<br>
By default, total three files should be provided: `*.map`, `*.geno.bin`, `*.geno.desc`. The `*.map` file contains SNP information, the first column is SNP id, the second column is its chromosome number, and the third column is its base-pair position; `*.geno.bin` is the numeric m(number of markers) by n(number of individuals) genotype file in *big.matrix* format, and `*.geno.desc` is the description file of `*.geno.bin`. Actually, users could manually make those files, but time-consuming and error prone, so ***`KAML`*** provides a function ***`KAML.Data()`*** for genotype format transformation. Currently, genotype file could be in three formats, either in the ***Hapmap*** format, ***PLINK binary ped*** format, or the m by n ***Numeric*** format. When transformin, missing genotypes are replaced by the mean genotype value of a given marker.<br>
** Note: No matter what type of format of genotype, the order of individuals in columns of the file must correspond to phenotype file in rows.**

#### Hapmap
Hapmap is the most popular used format for storing genotype data. As the example below, the SNP information is stored in the rows and taxa information is stored in the columns. The first 11 columns display attributes of the SNPs and the remaining columns show the nucleotides genotyped at each SNP for all individuals.

> `mouse.hmp.txt`
| rs# | alleles | chrom | pos | strand | assembly# | center | protLSID | assayLSID | panelLSID | QCcode | A048005080 | A048006063 | A048006555 | A048007096 | A048010273 | ... | A084292044 |
| :---: | :---: |  :---: |  :---: |  :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| rs3683945 | G/A | 1 | 3197400 | + | NA | NA | NA | NA | NA | NA | AG | AG | GG | AG | GG | ... | AA |
| rs3707673 | A/G | 1 | 3407393 | + | NA | NA | NA | NA | NA | NA | GA | GA | AA | GA | AA | ... | GG |
| rs6269442 | G/A | 1 | 3492195 | + | NA | NA | NA | NA | NA | NA | AG | GG | GG | AG | GG | ... | AA |
| rs6336442 | G/A | 1 | 3580634 | + | NA | NA | NA | NA | NA | NA | AG | AG | GG | AG | GG | ... | AA |
| rs13475699 | G | 1 | 3860406 | + | NA | NA | NA | NA | NA | NA | GG | GG | GG | GG | GG | ... | GG |

This type of file can be transformed by the following codes:

```r
KAML.Data(hfile="mouse.hmp.txt", out="mouse")
```

Normally, all chromosomes are stored in one file, but can be stored in separated files for different chromosomes. If in this case, it can be transformed by following codes:

```r
KAML.Data(hfile=c("mouse.chr1.hmp.txt", "mouse.chr2.hmp.txt",...), out="mouse")
```

#### PLINK Binary

`mouse.fam`, `mouse.bim`, `mouse.bed`

```r
KAML.Data(bfile="mouse", out="mouse")
```

#### Numeric

> `mouse.Numeric.txt`

| 1 | 1 | 2 | 1 | 2 | … | 0 |
| :---: | :---: |  :---: |  :---: |  :---: | :---: | :---: |
| 1 | 1 | 0 | 1 | 0 | … | 2 |
| 1 | 2 | 2 | 1 | 2 | … | 0 |
| 1 | 1 | 2 | 1 | 2 | … | 0 |
| 0 | 0 | 0 | 0 | 0 | … | 0 |

```r
KAML.Data(numfile="mouse.Numeric.txt", mapfile="mouse.map", out="mouse")
```

> `mouse.map, mouse.geno.desc, mouse.geno.bin`

---
## USAGE
### Basic
```r
KAML(pfile="mouse.Pheno.txt", pheno=1, gfile="mouse")
```
```r
KAML(pfile="mouse.Pheno.txt", pheno=1, gfile="mouse", cfile="CV.txt", kfile="mouse.Kin.txt")
```
### Advanced
<p align="center">
<a href="https://raw.githubusercontent.com/YinLiLin/R-KAML/master/figures/Trait1.jpg">
<img src="/figures/Trait1.jpg" height="300px" width="840px">
</a>
</p>

---
## OUTPUT

## FAQ and Hints

:sos: **Question1:** Failing to install "devtools":

***ERROR: configuration failed for package ‘git2r’***

***removing ‘/Users/acer/R/3.4/library/git2r’***

***ERROR: dependency ‘git2r’ is not available for package ‘devtools’***

***removing ‘/Users/acer/R/3.4/library/devtools’***

:yum: **Answer:** Please type the following codes in terminal.
```bash
apt-get install libssl-dev/unstable
```
---
:sos: **Question2:** When installing packages from Github with "devtools", there is a error:
 
 ***Error in curl::curl_fetch_disk(url, x$path, handle = handle): Problem with the SSL CA cert (path? access rights?)***
 
:yum: **Answer:** Please type the following codes and then try agian.
```r
library(httr)
set_config(config(ssl_verifypeer = 0L))
```

**Questions, suggestions, and bug reports are welcome and appreciated.** [:arrow_right:](https://github.com/YinLiLin/R-KAML/issues)

