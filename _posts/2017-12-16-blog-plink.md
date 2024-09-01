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
A crucial step in analyzing genome-wide association studies (GWAS) is identifying problematic subjects and markers. Quality control (QC) in GWAS involves removing unreliable markers and individuals, significantly enhancing the accuracy of results.  

Key QC steps include checking for gender discrepancies (where genetic sex does not match phenotypic gender), genotyping rate (call rate), minor allele frequency (MAF), Hardy-Weinberg equilibrium deviation (HWE), heterozygosity rate, and identical by descent (IBD) allele sharing. Standard tools such as PLINK and KING are often utilized for these QC processes through automated scripts.  

The first step is to load VCF or BCF file into PLINK.
```bash
PLINK --bcf file.bcf.gz \
	--allow-no-sex \
	--keep-allele-order \
	--vcf-idspace-to _ \
	--make-bed \
	--out plink.load
```
Then remove SNPs with more than 10 percent missing genotype calls.
```bash
PLINK --bfile plink.load \
         --allow-no-sex \
         --keep-allele-order \
         --geno \
         --make-bed \
         --out plink.geno10pc
```
The command check-sex compares the sex reported in the .fam file and the sex imputed from the X chromosome inbreeding coefficients.
```bash
PLINK --bfile plink.geno10pc \
	--allow-no-sex \
	--keep-allele-order \
	--split-x hg38 \
	--make-bed \
	--out plink.geno10pc.split

PLINK --bfile plink.geno10pc.split \
	--allow-no-sex \
	--check-sex \
	--out plink.geno10pc.sexcheck

```
Complete pipeline can be found [here](https://bitbucket.org/adinasarapu/clustercomputing/src/80fe2e327b605d134454fe99c9cf272d7271b0aa/job_post_variant_qc.sh).
Now, you need to go into your SGE computer (our's is called HGCC), and run:
```
qsub job_post_variant_qc.sh
```
