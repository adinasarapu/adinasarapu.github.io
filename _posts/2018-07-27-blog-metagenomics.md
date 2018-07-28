---
title: 'Taxonomic and functional profiling of shotgun metagenomics sequencing data '
date: 2018-07-27
permalink: /posts/2018/07/blog-post-metagenomics/
tags:
  - metagenomics
  - MetaPhlAn2 
  - HUMAnN2
  - taxonomic profiling
  - functional profiling
  - shotgun metagenomics sequencing
  - 

---
This workflow consists of taxonomic and functional profiling of shotgun metagenomics sequencing (MGS) reads using MetaPhlAn2 and HUMAnN2, respectively. To perform taxonomic (phyla, genera or species level) profiling of the MGS data, the MetaPhlAn2 pipeline was run on a high performance multicore cluster computing environment. 

MetaPhlAn2 provides microbial (bacterial, archaeal, viral, and eukaryotic) taxonomic profiling allowing the quantification of individual species across metagenomic samples. MetaPhlAn2 relies on ~1M unique clade-specific marker genes identified from ~17,000 reference genomes. Microbial reads, aligned by MetaPhlAn2, belonging to clades with no sequenced genomes available are reported as an “unclassified” subclade of the closest ancestor with available sequence data.
HUMAnN2 utilizes the MetaCyc database as well as the UniRef gene family catalog to characterize the microbial pathways present in samples. HUMAnN2 relies on programs such as BowTie (for accelerated nucleotide-level searches) and Diamond (for accelerated translated searches) to compute the abundance of gene families and metabolic pathways present. HUMAnN2 generates three outputs: 1) gene families based on UniRef proteins and their abundances reported in reads per kilobase, 2) MetaCyc pathways and their coverage, and 3) MetaCyc pathways and their abundances reported in reads per kilobase.

Scripts are available at [shotgun_metagenomics](https://bitbucket.org/adinasarapu/shotgun_metagenomics/src)
