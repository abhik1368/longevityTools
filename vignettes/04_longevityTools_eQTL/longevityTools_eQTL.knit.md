---
title: "longevityToolseQTL - eQTL, eSNP and GWAS Analysis" 
author: "Authors: Danjuma Quarless, Kuan-Fu Ding, Jamison McCorrison, Nik Schork, Dan Evans"
date: "Last update: 06 March, 2016" 
package: "longevityTools 1.0.4"
output:
  BiocStyle::html_document:
    toc: true
    toc_depth: 3
    fig_caption: yes

fontsize: 14pt
bibliography: bibtex.bib
---
<!--
%% \VignetteEngine{knitr::rmarkdown}
%\VignetteIndexEntry{Overview Vignette}
%% \VignetteDepends{methods}
%% \VignetteKeywords{compute cluster, pipeline, reports}
%% \VignettePackage{longevityTools}
-->

<!---
- Compile from command-line
echo "rmarkdown::render('longevityTools_eQTL.Rmd')" | R -slave; R CMD Stangle longevityTools_eQTL.Rmd

- Commit to github
git commit -am "some edits"; git push -u origin master

- To customize font size and other style features, add this line to output section in preamble:  
    css: style.css
-->

<script type="text/javascript">
document.addEventListener("DOMContentLoaded", function() {
  document.querySelector("h1").className = "title";
});
</script>
<script type="text/javascript">
document.addEventListener("DOMContentLoaded", function() {
  var links = document.links;  
  for (var i = 0, linksLength = links.length; i < linksLength; i++)
    if (links[i].hostname != window.location.hostname)
      links[i].target = '_blank';
});
</script>



# Introduction 
This vignette is part of the NIA funded Longevity Genomics project. For more information on this project please visit its 
website [here](http://www.longevitygenomics.org/projects/). The GitHub repository of the corresponding R package 
is available <a href="https://github.com/tgirke/longevityTools">here</a> and the most recent version of this 
vignette can be found <a href="https://htmlpreview.github.io/?https://github.com/tgirke/longevityTools/blob/master/vignettes/longevityTools_eQTL.html">here</a>.

Based on muscle gene expression from 15 young (25 year old) and 15 old (65 year old) participants, Sood et al. identified a 150 probe set that accurately classified young and old individuals in external studies with gene expression data collected from tissues other than muscle [@Sood2015-pb]. A gene score based on the classifier was associated with better renal function, increased survival time over follow-up, and decreased Alzheimer's Disease prevalence.

Our goal is to perform Mendelian Randomization using the 150 gene set to determine whether SNPs associated with expression (eSNPs) of the 150 genes are associated with the aging phenotypes identified by Sood et al.

Hypothesis: eSNPs associated with younger expression profile are markers for younger biological age relative to chronological age, which would be associated with longevity.

<div align="right">[Back to Table of Contents]()</div>

# General Analytical Approach

## Construction of MR eSNP risk score

File sood-2015-fileS1.xls reports the expression direction relative to age for each probe from the 150 probe set. See Top 150 worksheet. A gene marked as 'down' in column 'Ratio of Y:O muscle' means that the expression of the gene is lower in young vs old muscle. 

The goal is to generate an eSNP risk score constructed from eSNP alleles associated with increasing gene expression for genes that decrease with age and from eSNP alleles associated with decreasing gene expression for genes that increase with age. This eSNP risk score would represent an expression profile that more closely mimics the younger age profile rather than the older age profile.  

For the 150 probes that map to genes, identify cis-eSNPs using GTEx preanalyzed results from whole blood. Preanalyzed GTEx results correct for multiple testing of SNPs for each gene. To identify independent cis-eSNPs for each gene, prune cis-eSNPs in LD using 1000G EUR population group genotypes. Once independent cis-eSNPs are identified, create a file listing the SNPs to be included in a risk score and which allele is associated with decreasing expression for genes that increase with age (upregulated gene) and which allele is associated with increasing expression for genes that decrease with age (downregulated gene). Upregulated and downregulated genes based on column 'Ratio of Y:O muscle' from Sood et al supplemental file.

Individual level eSNP risk scores will be constructed as below. The eSNP risk score represents the MR instrumental variable.

$$ eSNP Risk Score_i = \sum_{j=1}^{N} eSNP_j^{AlleleDown} + \sum_{k=1}^{N} eSNP_k^{AlleleUp}  $$

# Getting Started

## Installation

The R software for running [_`longevityTools`_](https://github.com/tgirke/longevityTools) can be downloaded from [_CRAN_](http://cran.at.r-project.org/). The _`longevityTools`_ package can be installed from the R console using the following _`biocLite`_ install command. 


```r
source("http://bioconductor.org/biocLite.R") # Sources the biocLite.R installation script 
biocLite("tgirke/longevityTools", build_vignettes=FALSE) # Installs package from GitHub
```
<div align="right">[Back to Table of Contents]()</div>

## Loading package and documentation


```r
library("longevityTools") # Loads the package
library(help="longevityTools") # Lists package info
vignette("longevityTools") # Opens vignette
```
<div align="right">[Back to Table of Contents]()</div>

# Setup of working environment

## Import custom functions 

Preliminary functions that are still under developement, or not fully tested and documented 
can be imported with the `source` command from the `inst/extdata` directory of the package.


```r
fctpath <- system.file("extdata", "longevityTools_eQTL_Fct.R", package="longevityTools")
source(fctpath)
```
<div align="right">[Back to Table of Contents]()</div>

## Create expected directory structure 

The following creates the directory structure expected by this workflow. Input data
will be stored in the `data` directory and results will be written to the `results` directory.
All paths are given relative to the present working directory of the user's R session.


```r
dir.create("data"); dir.create("results") 
```

<div align="right">[Back to Table of Contents]()</div>

# Sample data from GTEx
The eQTL data used in this vignette can be downloaded from the [GTEx site](http://www.gtexportal.org/home/datasets).
Note, the following downloads a 1.3GB file, which will take some time.


```r
download.file("http://www.gtexportal.org/static/datasets/gtex_analysis_v6/single_tissue_eqtl_data/GTEx_Analysis_V6_eQTLs.tar.gz", "./GTEx_Analysis_V6_eQTLs.tar.gz")
untar("GTEx_Analysis_V6_eQTLs.tar.gz")
```

# Utilities
The _`geneGrep`_ function takes GTEx results and gene list as arguments,
returns GTEx eSNP results for genes in gene list. Genes without significant
eSNPs are removed from results. The following sample file _Thyroid\_Analysis.snpgenes.TXNRD2_ has been subsetted to include eSNPs for TXNRD2.

```r
library(longevityTools)
samplepath <- system.file("extdata", "Thyroid_Analysis.snpgenes.TXNRD2", package="longevityTools") 
dat <- read.delim(samplepath)
myGenes <- c("TXNRD2")
result <- geneGrep(dat, myGenes)
head(result[order(result$p_value),])
```

```
##                    snp       beta      p_value ref alt gene_name   nom_thresh snp_chrom  snp_pos
## 83 22_19867771_C_T_b37 -0.4596489 6.598661e-21   C   T    TXNRD2 1.215557e-05        22 19867771
## 86 22_19868678_T_C_b37 -0.4522136 1.109763e-20   T   C    TXNRD2 1.215557e-05        22 19868678
## 82 22_19867276_T_C_b37 -0.4527145 5.145122e-20   T   C    TXNRD2 1.215557e-05        22 19867276
## 80 22_19866194_G_C_b37 -0.4547658 6.891885e-20   G   C    TXNRD2 1.215557e-05        22 19866194
## 95 22_19871691_C_T_b37 -0.4709824 8.145810e-20   C   T    TXNRD2 1.215557e-05        22 19871691
## 91 22_19870036_C_G_b37 -0.4454200 1.026483e-19   C   G    TXNRD2 1.215557e-05        22 19870036
##    snp_id_1kg_project_phaseI_v3 rs_id_dbSNP142_GRCh37p13
## 83                    rs1139795                rs1139795
## 86                   rs45465601               rs45465601
## 82                   rs34175429               rs34175429
## 80                    rs5993849                rs5993849
## 95                   rs12158214               rs12158214
## 91                    rs8141451                rs8141451
```

```r
#output results in PLINK format
result2<-result[,c("snp_id_1kg_project_phaseI_v3","p_value")]
names(result2)<-c("SNP","P")
#write.table(result2,file="data/eSNP.assoc",quote=F,row.names=F)
```

There are 139 eSNPs.

# SNP clumping using PLINK
GTEx V6 analysis results are based on genotypes imputed to 1000 Genomes (1KG) Phase I version 3. Thus, significant results could be LD-filtered using Phase I data. However, to make use of the larger sample size in later projects, 1KG Phase 3 genotypes will be used.


```r
#genotypes
download.file("ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/ALL.chr22.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.vcf.gz", "./ALL.chr22.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.vcf.gz")
untar("ALL.chr22.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.vcf.gz")

#sample ped file
download.file("ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/integrated_call_samples_v2.20130502.ALL.ped", "./integrated_call_samples_v2.20130502.ALL.ped")

#sample super population file
download.file("ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/release/20130502/integrated_call_samples_v3.20130502.ALL.panel", "./integrated_call_samples_v3.20130502.ALL.panel")

#identify EUR unrelated samples from 1KG phase 3
ped2<-read.table("data/integrated_call_samples_v2.20130502.ALL.ped", stringsAsFactors = F, header = T, sep="\t")
ped3<-read.table("data/integrated_call_samples_v3.20130502.ALL.panel", stringsAsFactors = F, header = T, sep="\t")

samples1KG <- filter_1KGsamples("EUR",ped2,ped3)
samples1KG_ID <- samples1KG[,"Individual.ID",drop=F]
write.table(samples1KG_ID,file="data/samples1KG.txt",quote=F,row.names=F,col.names=F)
```

Create region file to use with bcftools for LD.

```r
regions<-data.frame(chr="chr1",pos=0,pos_to=0,stringsAsFactors = F)
regions$chr[1]<-gsub("chr","",result$snp_chrom[1]) #1KG genotype files do not have chr
regions$pos[1]<-min(result$snp_pos)
regions$pos_to[1]<-max(result$snp_pos)
write.table(regions,file="data/regions.txt",quote=F,row.names=F,col.names=F,sep="\t")
```


Filter 1KG genotypes to only include EUR unrelated individuals and eQTL region.
```
>bcftools --version
bcftools 1.3
Using htslib 1.3
Copyright (C) 2015 Genome Research Ltd.
License Expat: The MIT/Expat license
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

```

```
bcftools view -Ov -o results/1KGgeno.vcf -S data/samples1KG.txt -R data/regions.txt -v snps data/ALL.chr22.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.vcf.gz
#without -v snps, multiple indels with same rsID were outputted, and plink would not read that in.
#Quick solution, only include SNPs, not indels.
```

Run PLINK clump command using default settings, but might want to change with different nominal significance thresholds.

```
plink -vcf results/1KGgeno.vcf --clump data/eSNP.assoc 

```

PLINK clump command identifies 8 independent eSNPs in the region. 

Next step, extract independent eSNPs from individual level genotype data, build MR risk score, evaluate for association with survival time.

<div align="right">[Back to Table of Contents]()</div>

# Funding
This project is funded by NIH grant U24AG051129 awarded by the National Intitute on Aging (NIA).

<div align="right">[Back to Table of Contents]()</div>


# Version information


```r
sessionInfo()
```

```
## R version 3.2.2 (2015-08-14)
## Platform: x86_64-pc-linux-gnu (64-bit)
## Running under: CentOS Linux 7 (Core)
## 
## locale:
## [1] C
## 
## attached base packages:
## [1] stats     graphics  utils     datasets  grDevices methods   base     
## 
## other attached packages:
## [1] ggplot2_2.0.0        longevityTools_1.0.1 BiocStyle_1.8.0     
## 
## loaded via a namespace (and not attached):
##  [1] Rcpp_0.12.3      codetools_0.2-14 digest_0.6.9     plyr_1.8.3       grid_3.2.2      
##  [6] gtable_0.1.2     formatR_1.2.1    magrittr_1.5     evaluate_0.8     scales_0.3.0    
## [11] stringi_1.0-1    rmarkdown_0.9.2  tools_3.2.2      stringr_1.0.0    munsell_0.4.2   
## [16] yaml_2.1.13      colorspace_1.2-6 htmltools_0.3    knitr_1.12.3
```
<div align="right">[Back to Table of Contents]()</div>

# References