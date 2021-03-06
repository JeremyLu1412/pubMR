pubMR
==========
pubMR是R平台下一个高效的PubMed文本挖掘工具，集合了：检索下载、解析抽取、基本统计、多维矩阵、论文相似、热点分析、概念识别和网络分析等多种功能。

pubMR is an R package designed for text mining of the PubMed database. Additionally, it provide some highly customized metics to evaluate and visualize results for downstream analysis.


## Authors

[崔雷] (Cuilei)

[周晓北] (Zhou Xiaobei)

## Installation

```r
install.packages("devtools")
devtools::install_github("xizhou/pubMR")
```

## Vignettes
[pubMR.pdf](https://github.com/xizhou/pubMR/tree/master/vignettes/pubMR.pdf)

## Usage
A quick start:
```r
library(pubMR)
m <- '"neoplasms"[MeSH Terms] AND "serine/metabolism"[Mesh Terms] AND ("2017/01/01"[PDAT] : "2018/12/31"[PDAT])'
obj <- AB(query=m,output='ABprofile')
p <- pubtator(obj@PMID)
```
Import an xml file downloaded from the PubMed database into R program:

<p align="center">
  <img src="https://github.com/xizhou/pubMR/blob/master/screenshot.png?raw=true" alt="xml"/>
</p>

```r
library(pubMR)
obj <- AB(input="pubmed_result.xml")
```

Save as an xml file:
```r
library(XML)
obj1 <- AB(query=m,output='xml')
saveXML(obj1,file="result.xml")
```

A MS(MeSH/subheading)-PMID co-occurrence-matrix can be generated by:
```r
library(data.table)
library(tidyr)
options(datatable.prettyprint.char=10L) 
obj1=data.table(PMID=obj@PMID,MS=obj@MS)
MS <- obj1[,MS]
idx <- sapply(MS,is.null)
obj1 <- obj1[!idx,]
obj1 = obj1 %>% unnest(MS) 
V <- table(obj1[,c("MS","PMID")])
V
                                                 PMID
MS                                                27297361 27748765 27777073 27889207 27890529
  A Kinase Anchor Proteins/metabolism                    0        0        0        0        0
  Acquired Immunodeficiency Syndrome/metabolism          0        0        0        0        0
  Activating Transcription Factor 4/metabolism           0        0        0        0        0
  Adaptor Proteins, Signal Transducing/genetics          0        0        0        0        0
  Adaptor Proteins, Signal Transducing/metabolism        0        0        0        0        0
```
Additionally, a MS-MS co-occurrence-matrix can be generated by:
```r
V1 <- crossprod(t(V))
V1                                                 MS
MS                                                A Kinase Anchor Proteins/metabolism Acquired Immunodeficiency Syndrome/metabolism Activating Transcription Factor 4/metabolism
  A Kinase Anchor Proteins/metabolism                                               1                                             0                                            0
  Acquired Immunodeficiency Syndrome/metabolism                                     0                                             1                                            0
  Activating Transcription Factor 4/metabolism                                      0                                             0                                            1
  Adaptor Proteins, Signal Transducing/genetics                                     0                                             0                                            0
  Adaptor Proteins, Signal Transducing/metabolism                                   0                                             0                                            0
```
The MS-MS co-occurrence-matrix can be transferred as a sparse matrix format file, then be clustered and visualized by gCluto program:
```r
library(slam)
x <- V1
diag(x) <- 0
x <- x[!rowSums(x)==0,] 
x <- x[!colSums(x)==0,] 
slam::write_stm_CLUTO(x,file="dat.mat")
```
<p align="center">
  <img src="https://github.com/xizhou/pubMR/blob/master/fig.png?raw=true" alt="gcluto"/>
</p>