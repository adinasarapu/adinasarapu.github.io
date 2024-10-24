---
title: 'Single cell gene expression data analysis on Cluster (10X Genomics, Cell Ranger)'
date: 2019-11-18
permalink: /posts/2019/01/blog-post-sc-ranseq/
tags:
  - 10XGenomics
  - Chromium Single Cell Gene Expression
  - Single cell RNA-sequencing (scRNA-seq)
  - Single Cell
  - Cell Ranger
  - Seurat
  - Cluster Computing
  - Feature Barcoding

---  
*Updated on November 28, 2023*  

[Cell Ranger](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/what-is-cell-ranger) can be run in cluster mode, using job schedulers like Sun Grid Engine (or simply SGE) or Load Sharing Facility (or simply LSF) as queuing system allows highly parallelizable jobs.

There are 4 steps to analyze Chromium Single Cell data[^1].

**Step 1**: `cellranger mkfastq` demultiplexes raw base call (BCL) files generated by Illumina sequencers into FASTQ files.  
 
**Step 2**: `cellranger count` takes FASTQ files from `cellranger mkfastq` and performs alignment, filtering, barcode counting, and unique molecular identifier (UMI) counting.  

_When doing large studies involving multiple GEM wells, first run `cellranger count` on FASTQ data from each of the GEM wells individually, and then pool the results using `cellranger aggr`, as described [here](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/using/aggregate)_.  

**Step 3**: `cellranger aggr` aggregates outputs from multiple runs of `cellranger count`.  

**Step 4**: Use R package [Seurat](https://satijalab.org/seurat/)[^2] for downstream analysis.


Running pipelines on cluster requires the following:  

**1**. Download and uncompress `cellranger-7.2.0`[^1] at your `$HOME` directory and add PATH in `~/.bashrc`.  

**2**. Update job config file (`external/martian/jobmanagers/config.json`) for threads and memory.  

For example  

`"threads_per_job": 4,`  
`"memGB_per_job": 32,`
`"name":"SGE_ROOT",`
`"description":"/opt/sge"`

**3**. Update template file `sge.template` (`external/martian/jobmanagers/sge.template`).  

``` 
#!/bin/bash
#$ -N __MRO_JOB_NAME__
#$ -V
#$ -pe smp __MRO_THREADS__
#$ -q b.q
#$ -cwd
#$ -l mem_free=__MRO_MEM_GB__G
#$ -o __MRO_STDOUT__
#$ -e __MRO_STDERR__
#$ -m abe
#$ -M <email>
#$ -S /bin/bash
 
cd __MRO_JOB_WORKDIR__
source $HOME/10xgenomics/cellranger-7.2.0/sourceme.bash 
```

For clusters whose job managers do not support memory requests, it is possible to request memory 
in the form of cores via the `--mempercore` command-line option. This option scales up the number 
of threads requested via the `__MRO_THREADS__` variable according to how much memory a stage requires. 
see more at [Cluster Mode](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/advanced/cluster-mode)  

**4**. Download single cell gene expression and reference genome datasets from [10XGenomics](https://www.10xgenomics.com/resources/datasets/).  

**5**. Create `sge.sh` file  

**for Single Cell 3′ gene expression**   

Output files will appear in the out/ subdirectory within this pipeline output directory. For pipeline output directory, the `--id` argument is used i.e 10XGTX_v3.    

```
#!/bin/bash

cd $HOME/10xgenomics/out  

FASTQS="$HOME/pbmc_10k_v3_fastqs"  
TR="$HOME/refdata-gex-GRCh38-2020-A"

cellranger count --disable-ui \  
  --id=10XGTX_v3 \  
  --transcriptome=${TR} \  
  --fastqs=${FASTQS} \  
  --sample=pbmc_10k_v3 \  
  --expect-cells=10000 \  
  --jobmode=sge \  
  --mempercore=8 \  
  --jobinterval=5000 \  
  --maxjobs=3  
```

Execute a command in screen and, detach and reconnect:  

Start a screen as `screen -S some_name`.  
Run the above script as `sh sge.sh`  
To detach the screen session from the terminal use `control` + `a` followed by `d`  
To reconnect to the screen: `screen -R some_name`  

If the job is Eqw: Job waiting in error state  
`qstat -j jobid | grep error`  

If you understand the reason and can get it fixed, you can clear the error state with  
`qmod -cj jobid`  


**for Single Cell 5′ gene expression**

Use either `--force-cells` or `--expect-cells`  
  
```  
#!/bin/bash  

cd $HOME/10xgenomics/out  

TR="$HOME/refdata-cellranger-GRCh38-3.0.0
FASTQS="$HOME/vdj_v1_hs_nsclc_5gex_fastqs"

cellranger count \  
  --id=10XGTX_v5 \  
  --fastqs=${FASTQS} \  
  --transcriptome=${TR} \  
  --sample=vdj_v1_hs_nsclc_5gex \  
  --force-cells=7802 \  
  --jobmode=sge \  
  --mempercore=8 \  
  --maxjobs=3 \  
  --jobinterval=2000  
```  

Execute a command in screen and, detach and reconnect:  

Start a screen as `screen -S some_name`.  
Run the above script as `sh sge.sh`  
To detach the screen session from the terminal use `control` + `a` followed by `d`  
To reconnect to the screen: `screen -R some_name`  

**for Feature Barcode Analysis**  

Tested on Single Cell 5′ gene expression and cell surface protein (Feature Barcoding/Antibody Capture Assay) data.  

For more information, please visit the [Single Cell Gene Expression with Feature Barcoding](https://support.10xgenomics.com/single-cell-gene-expression/overview/doc/getting-started-single-cell-gene-expression-with-feature-barcoding-technology) page (Single Cell 3') or the [Single Cell Immune Profiling with Feature Barcoding](https://support.10xgenomics.com/single-cell-vdj/overview/doc/getting-started-immune-profiling-feature-barcoding) page (Single Cell 5').  

Currently available Feature Barcode kits for Single Cell Gene Expression Feature Barcode Technology  

|-----------------------------------------------------------------------------------------------------------|  
| 10x Solution			  		| Gene Expression | Cell Surface Protein | CRISPR Screening |  
|-----------------------------------------------------------------------------------------------------------|  
| Single Cell Gene Expression v2  		| ✓		  | -			 | -		    |  
| Single Cell Gene Expression v3		| ✓		  | ✓			 | ✓		    |  
| Single Cell Gene Expression v3.1		| ✓		  | ✓			 | ✓                |  
| Single Cell Gene Expression v3.1 (Dual Index)	| ✓		  | ✓			 | ✓ 		    |  
|-----------------------------------------------------------------------------------------------------------|  

Currently available Feature Barcoding kits for Single Cell Immune Profiling Feature Barcoding Technology  

|------------------------------------------------------------------------------------------------------------------------------------------|  
| 10x Solution							| TCR/Ig | Gene Expression | Cell Surface Marker | TCR-Antigen Specificity |  
|------------------------------------------------------------------------------------------------------------------------------------------|  
| Single Cell Immune Profiling 					| ✓	 | ✓		   | -			 | -			   |  
| Single Cell Immune Profiling with Feature Barcoding technology| ✓	 | ✓		   | ✓			 | ✓			   |  
|------------------------------------------------------------------------------------------------------------------------------------------|  

To enable Feature Barcode analysis, `cellranger count` needs two inputs:  

First you need a csv file declaring input library data sources; one for the normal single-cell gene expression reads, and one for the Feature Barcode reads (the FASTQ file directory and library type for each input dataset).  

`LIBRARY=$HOME/vdj_v1_hs_pbmc2_5gex_protein_fastqs/vdj_v1_hs_pbmc2_5gex_protein_library.csv`  
  
|------------------------------------------------------------------------------------|  
| fastqs			 |   sample			| library_type       |  
|------------------------------- |------------------------------|--------------------|  
| /path/to/antibody_fastqs	 |   vdj_v1_hs_pbmc2_antibody	| Antibody Capture   |  
| /path/to/gene_expression_fastqs|   vdj_v1_hs_pbmc2_5gex	| Gene Expression    |  
|------------------------------------------------------------------------------------|  

Second, you need Feature reference csv file, declaring feature-barcode constructs and associated barcodes. The pattern will be used to extract the Feature Barcode sequence from the read sequence.  

`FEATURE_REF=$HOME/vdj_v1_hs_pbmc2_5gex_protein_fastqs/vdj_v1_hs_pbmc2_5gex_protein_feature_ref.csv`  

| ---------------------------------------------------------------------------------------------------|  
| id	 | name		    | read  | pattern 			| sequence 	  | feature_type     |  
|--------|------------------|-------|---------------------------|-----------------|------------------|  
| CD3    | CD3_TotalSeqC    | R2    | 5PNNNNNNNNNN(BC)NNNNNNNNN	| CTCATTGTAACTCCT | Antibody Capture |  
| CD19   | CD19_TotalSeqC   | R2    | 5PNNNNNNNNNN(BC)NNNNNNNNN | CTGGGCAATTACTCG | Antibody Capture |  
| CD45RA | CD45RA_TotalSeqC | R2    | 5PNNNNNNNNNN(BC)NNNNNNNNN | TCAATCCTTCCGCTT | Antibody Capture |  
| ...    | ...              | ...   | ... 			| ...		  | ...		     |  
|----------------------------------------------------------------------------------------------------|  

*Feature and Library Types* - When inputting Feature Barcode data to Cell Ranger via the Libraries CSV File, you must declare the library_type of each library. Examples include *Antibody Capture*, *CRISPR Guide Capture* or *Custom*. If your assay scheme creates a library containing multiple library_types, for example if you're using CRISPR Guide Capture and Antibody Capture features, you will need to run Cell Ranger multiple times, passing different library_type values in the Libraries CSV File. If Targeted Gene Expression data is analyzed in conjunction with CRISPR-based Feature Barcode data, there are additional requirements imposed for the Feature Reference CSV file.  

[TotalSeq™ Reagents for Single-Cell Proteogenomics](https://www.biolegend.com/en-us/totalseq)  

[TotalSeq™-B](https://cf.10xgenomics.com/samples/cell-exp/3.0.0/pbmc_1k_protein_v3/pbmc_1k_protein_v3_feature_ref.csv) is a line of antibody-oligonucleotide conjugates supplied by BioLegend that are compatible with the Single Cell 3' v3 assay.  
[TotalSeq™-C](https://support.10xgenomics.com/csv/vdj_v1_hs_pbmc2_5gex_protein_feature_ref.csv) is a line of antibody-oligonucleotide conjugates supplied by BioLegend that are compatible with the Single Cell 5' assay.  
[TotalSeq™-A](https://support.10xgenomics.com/csv/TotalSeqA_example_feature_ref.csv) is a line of antibody-oligonucleotide conjugates supplied by BioLegend that are compatible with the Single Cell 3' v2 and Single Cell 3' v3 kits. Although TotalSeq™-A can be used with the CITE-Seq assay, CITE-Seq is not a 10x supported assay.  

CITE-seq (Cellular Indexing of Transcriptomes and Epitopes by Sequencing) allows simultaneous analysis of transcriptome and cell surface protein information at the level of a single cell.[^3]  

*"The pipeline first extracts and corrects the cell barcode and UMI from the feature library using the same methods as gene expression read processing. It then matches the Feature Barcode read against the list of features declared in the above [Feature Barcode Reference](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/using/feature-bc-analysis#feature-ref). The counts for each feature are available in the feature-barcode matrix output files."*  
 
```
#!/bin/bash  

cd $HOME/10xgenomics/out

TR="$HOME/refdata-cellranger-GRCh38-3.0.0

LIBRARY=$HOME/vdj_v1_hs_pbmc2_5gex_protein_fastqs/vdj_v1_hs_pbmc2_5gex_protein_library.csv  
FEATURE_REF=$HOME/vdj_v1_hs_pbmc2_5gex_protein_fastqs/vdj_v1_hs_pbmc2_5gex_protein_feature_ref.csv  

cellranger count \  
 --libraries=${LIBRARY} \  
 --feature-ref=${FEATURE_REF} \  
 --id=PBMC_5GEX \  
 --transcriptome=${TR} \  
 --expect-cells=9000 \  
 --jobmode=sge \  
 --mempercore=8 \  
 --maxjobs=3 \  
 --jobinterval=5000  
```

Execute a command in screen and, detach and reconnect:  

Start a screen as `screen -S some_name`.  
Run the above script as `sh sge.sh`.  
To detach the screen session from the terminal use `control` + `a` followed by `d`  
To reconnect to the screen: `screen -R some_name`  

**6**. Monitor work progress through a web browser  

Open `_log` file present in output folder `PBMC_5GEX`  

If you see serving UI as `http://cluster.university.edu:3600?auth=rlSdT_QLzQ9O7fxEo-INTj1nQManinD21RzTAzkDVJ8`, then type the following from your laptop  

`ssh -NT -L 9000:cluster.university.edu:3600 user@cluster.university.edu`  

`user@cluster.university.edu's password:`  

Then access the UI using the following URL in your web browser
`http://localhost:9000/`  

**7**. Single Cell Integration in Seurat  

Seurat is an R package designed for QC, analysis, and exploration of single cell RNA-seq data. Seurat aims to enable users to identify and interpret sources of heterogeneity from single cell transcriptomic measurements, and to integrate diverse types of single cell data. Seurat starts by reading cellranger data (barcodes.tsv.gz, features.tsv.gz and matrix.mtx.gz)  

`pbmc.data <- Read10X(data.dir = "~/PBMC_5GEX/outs/filtered_feature_bc_matrix/")`  
 
---

[^1]: [10XGenomics](https://support.10xgenomics.com/single-cell-gene-expression/software/overview/welcome)
[^2]: [Seurat](https://satijalab.org/seurat/)  
[^3]: Stoeckius, M. et al. Simultaneous epitope and transcriptome measurement in single cells. Nat. Methods 14, 865–868 (2017).  

