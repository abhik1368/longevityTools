---
title: longevityTools_eQTL - eQTL, eSNP and GWAS Analysis 
keywords: 
last_updated: Sun Mar  6 19:48:46 2016
---
Authors: Danjuma Quarless, Kuan-Fu Ding, Jamison McCorrison, Nik Schork, Dan Evans

Last update: 06 March, 2016 

Alternative formats of this vignette:
[`Single-page .Rmd HTML`](https://htmlpreview.github.io/?https://github.com/tgirke/longevityTools/blob/master/vignettes/longevityTools_eQTL.html),
[`.Rmd`](https://raw.githubusercontent.com/tgirke/longevityTools/master/vignettes/longevityTools_eQTL.Rmd),
[`.R`](https://raw.githubusercontent.com/tgirke/longevityTools/master/vignettes/longevityTools_eQTL.R)

## Introduction 
This vignette is part of the NIA funded Longevity Genomics project. For more information on this project please visit its 
website [here](http://www.longevitygenomics.org/projects/). The GitHub repository of the corresponding R package 
is available <a href="https://github.com/tgirke/longevityTools">here</a> and the most recent version of this 
vignette can be found <a href="https://htmlpreview.github.io/?https://github.com/tgirke/longevityTools/blob/master/vignettes/longevityTools_eQTL.html">here</a>.

Based on muscle gene expression from 15 young (25 year old) and 15 old (65 year old) participants, Sood et al. identified a 150 probe set that accurately classified young and old individuals in external studies with gene expression data collected from tissues other than muscle (Sood et al., 2015). A gene score based on the classifier was associated with better renal function, increased survival time over follow-up, and decreased Alzheimer's Disease prevalence.

Our goal is to perform Mendelian Randomization using the 150 gene set to determine whether SNPs associated with expression (eSNPs) of the 150 genes are associated with the aging phenotypes identified by Sood et al.

Hypothesis: eSNPs associated with younger expression profile are markers for younger biological age relative to chronological age, which would be associated with longevity.

