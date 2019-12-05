---
title: 'ATAC-seq data analysis'
date: 2019-12-05
permalink: /posts/2019/12/blog-post-atacseq/
tags:
  - ATAC-seq (Assay for Transposase Accessible Chromatin with high-throughput Sequencing)

---
**Overview**  

**ATAC-seq (Assay for Transposase Accessible Chromatin with high-throughput Sequencing)** is a next-generation sequencing approach for the analysis of open chromatin regions to assess the genome-wise chromatin accessibility. ATAC-seq achieves this by simultaneously fragmenting and tagging genomic DNA with sequencing adapters using the hyperactive Tn5 transposase enzyme [^1]. Other global chromatin accessibility methods include FAIRE-seq and DNase-seq. This document aims to provide accessibility. 

**Pre-processing of raw sequencing reads** – before mapping the raw reads to the genome, trim the adapter sequences. Poor read quality or sequencing
errors often lead to low mapping rate.  

**Mapping/alignment of sequencing reads to a reference genome** – use Burrows-Wheeler Aligner (BWA) for mapping of sequencing reads. The output alignment file will be saved as a sequence alignment/map (SAM) format or binary version of SAM called BAM. Mark the duplicate reads using Picard [^2] and exclude reads mapping to mitochondrial DNA and other chromosomes from analysis together with low quality reads (MAPQ<10 and reads in Encode black list regions) using SAMtools [^3].

**Filtering and shifting of the mapped reads** - shift the read position +4 and -5 bp in the BAM file before peak calling [adjust the reads alignment](https://yiweiniu.github.io/blog/2019/03/ATAC-seq-data-analysis-from-FASTQ-to-peaks/). When the Tn5 transposase cuts open chromatin regions, it introduces two cuts that are separated by 9 bp. Therefore, ATAC-seq reads aligning to the positive and negative strands need to be adjusted by +4 bp and -5 bp respectively to represent the center of the transposase binding site. Picard CollectInsertSizeMetrics will be used to compute the fragment sizes on alignment shifted BAM files.  


**Identification and visualization of the ATAC-seq peaks** – use MACS2 for peak calling with the parameters nomodel or BAMPE [^4] and identify the differentially enriched peaks using the MACS2 `bdgdiff` module. Individual peaks separated by <100 bp will be join together. For peak annotation and functional analysis use the R package ChIPpeakAnno or HOMER [^5],[^6]. First, ATAC-seq peaks will be categorized into different groups based on the nearest RefSeq gene i.e. promoter, untranslated regions (UTRs), intron and exon. Second, peaks that are within 5 kb upstream and 3 kb downstream of the Transcription Start Site (TSS) are associated to the nearest genes. Finally, these genes are then analyzed for over-represented gene ontology (GO) terms and KEGG pathways using ChIPpeakAnno. Visualize all sequencing tracks using the Integrated Genomic Viewer (IGV) [^7].  

Scripts are available for HPC [Cluster](https://bitbucket.org/adinasarapu/atac_seq-data-analysis/src/master/).  

---  
[^1]: [ATAC-seq](https://www.ncbi.nlm.nih.gov/pubmed/24097267) - Nature Methods. 2013; 10:1213–1218.  
[^2]: [PICARD](https://broadinstitute.github.io/picard/)  
[^3]: [HTSLIB](http://www.htslib.org)  
[^4]: [MACS](https://github.com/taoliu/MACS)  
[^5]: [ChIPpeakAnno](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-11-237)  
[^6]: [HOMER](http://homer.ucsd.edu/homer/ngs/)  
[^7]: [IGV](http://software.broadinstitute.org/software/igv/home)
