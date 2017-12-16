---
title: 'Quality control for GWAS studies'
date: 2017-12-16
permalink: /posts/2017/12/blog-post-plink/
tags:
  - plink
  - discordant sex informaion
  - missingness
  - heterozygosity rate
  - runs of homozygosity
---
An important step in the analysis of genome-wide association studies (GWAS) is to identify problematic subjects and markers. Quality control  (QC) in GWAS removes markers and individuals, and greatly increases the accuracy of findings.

Checking for gender (individuals whose genetic sex is discordant to their phenotypic gender), genotyping rate (call rate), minor allele frequency (MAF), Hardy-Weinberg equilibrium deviation (HWE), heterozygosity rate and identical by descent (IBD) allele sharing are useful QC steps. Standard tools like plink and king are called by the scripts. 

This script can be found [here](https://bitbucket.org/adinasarapu/clustercomputing/src/6bad66b2e4ce1f98d3392cb4af7cbf37bb08f73f/job_post_variant_qc.sh?at=master).
Now, you need to go into your SGE computer (our's is called HGCC), and run:
```bash 
qsub job_post_variant_qc.sh
```
