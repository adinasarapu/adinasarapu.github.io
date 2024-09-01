---
title: 'Quantitative Proteomics: Aptamer-Based Protein Quantification'
date: 2023-02-20
permalink: /posts/2023/02/blog-post-somalogic-proteomics/
tags:
  - SOMALogic
  - Quantitative Proteomics
  - aptamer-based proteomics
  - Emory University
  - Olink Proteomics
  - MS-based Proteomics

---  
Quantitative proteomics is a cutting-edge approach for measuring protein levels in complex biological samples. One innovative method in this field is aptamer-based protein quantification. Aptamers, which are short, single-stranded DNA or RNA molecules, are engineered to specifically bind to target proteins with high precision. 

The [SomaScan®]((https://somalogic.com)) assay is a widely used aptamer-based method for measuring protein abundances. However, there is limited information on how SomaScan correlates with mass spectrometry (MS)-based proteomics and Olink assays, another high-throughput antibody-based platform. Some studies have noted measurement variations between these platforms. Aptamers or SOMAmers in the SomaScan assay are typically selected to bind target proteins in their native conformation or with known post-translational modifications (PTMs). However, novel PTMs induced by pathogens or diseases can alter protein structure, electrophilicity, and interactions. A key drawback of the SomaScan assay is that its quantification relies on DNA microarray chips, which can introduce background noise. On the other hand, the advantages include lower cost and efficient data analysis.  

**Table 1**. Overview of common proteomic platforms (Jiang W et al, Cancers, 2022).  
 
|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|  
| Analytical Technique			| Category 		   | Protein Sample Values 	  | Accepted Biospecimen Types 						   		       |  
|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|  
| Proximity Extension Assay (Olink)	| Antibody 		   | 1 µL			  | Plasma, tissue/cell, synovial fluid, CSF, plaque extract, and saliva	   	       |  
| Reverse Phase Protein Arrays		| Antibody 		   | 5 µg (1.0- 1.5 mg/mL protein)| Tissue/cell, plasma, serum, biopsies, body fluids			   		       |  
| Bio-Plex				| Antibody (bead)	   | 12.5 µL (serum/plasma) 	  | 50 µL (cell culture)	Plasma, serum, tissue/cell			   	       | 			
| Simoa					| Antibody (bead)	   | 25 µL			  | Plasma, serum, urine, tissue/cell, CSF, saliva 			    	     	       |  
| Aptamer Group (Optmer)		| Aptamer		   | 38 µL			  | Plasma (diagnostics and therapeutics), urine, tissue/cell, liquid matrices 		       |  
| Base Pair Technologies		| Aptamer		   | 5–100 µL			  | Plasma, serum, tissue/cell								       |  
| SOMAscan				| Aptamer		   | 55–100 µL			  | Plasma, serum, CSF, urine, cell/tissue, synovial fluid, exosomes 	   		       | 
| Electrochemiluminescence Immunoassay	| ECLIA			   | 50 µL			  | Plasma, serum, tissue/cell, CSF, urine, blood spots, tears, synovial fluid, tissue extracts|	  
| Multiplex ELISA			| ELISA			   | 25–50 µL			  | Plasma, serum, tissue/cell, urine, saliva, CSF				   	       |  
| Singleplex ELISA			| ELISA			   | 100 µL			  | Plasma, serum, tissue/cell, urine, saliva, CSF					       |  
| 2D-PAGE				| Gel electrophoresis	   | ~100 µg (15–50 µL)		  | Plasma, serum, tissue/cell, urine 					   		       |  
| DDA-MS				| MS			   | 10 µL			  | Plasma, serum, tissue/cell 						   		       |  
| SWATH-MS				| MS (DIA)		   | 5–10 µg			  | Plasma, serum tissue/cell, platelets, monocytes/neutrophils 		   	       |  
| iTRAQ					| MS (labeling in LC–MS–MS)| 12 µg			  | Plasma, serum, tissue/cells, saliva 					   	       | 
| SRM/MRM				| MS (LC–MS–MS) 	   | 15 µL			  | Plasma, tissue/cell, dried blood spots 				   	               |  
|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|    

The SomaScan Assay v4.1 enables the simultaneous measurement of approximately 6,600 unique human proteins in a single sample (see **Table 2**). This is achieved using around 7,300 aptamers known as SOMAmers (Slow Off-rate Modified Aptamers). SOMAmers are short, chemically modified single-stranded DNA molecules designed to specifically bind to protein targets. The assay quantifies native proteins in complex samples by converting each protein's concentration into the corresponding SOMAmer reagent concentration, which is then measured using DNA microarrays. SOMAmers are selected to bind proteins in their native, folded states, typically requiring an intact tertiary protein structure for effective binding.  

**Table 2**. The 7k SomaScan Assay v4.1 panel (7,596 aptamers mapping to 6,414 unique human protein targets).  
 
|-----------------------------------------------------------------------------------------------------------------------------------|  
| 	Organism		| SOMAmers | UniProt IDs (all) | UniProt IDs (Unique)|  Protein Targets | Gene IDs   | Gene Symbols |
|-----------------------------------------------------------------------------------------------------------------------------------|  
| Human 			| 7335	   | 7301  	       | 6414   	     |  6610	        | 6408       | 6398	    |  
| Mouse				|  236	   |  236  	       |    4   	     |     4	        |    3	     |    3         |
| African clawed frog		|    3	   |    3  	       |    1   	     |     2	        |    1	     |    1	    |
| Gila monster			|    3	   |    3  	       |    1   	     |     1            |    0	     |    0	    |
| Hornet			|    3	   |    3  	       |    1   	     |     1            |    0	     |    1         |
| Jellyfish			|    3	   |    3  	       |    1   	     |     1            |    0	     |    1	    |
| Thermus thermophilus		|    3 	   |    3  	       |    1   	     |     1            |    0	     |    1         |
| Common eastern firefly	|    2     |    2  	       |    1   	     |     1            |    0	     |    0         |
| Bacillus stearothermophilus	|    1     |    1  	       |    1   	     |     1            |    0	     |    1	    |
| Ensifer meliloti		|    1     |    1  	       |    1   	     |     1            |    0	     |    1	    |
| European elder		|    2     |    2  	       |    1   	     |     1            |    0	     |    0	    |
| HIV-1				|    1     |    1  	       |    1   	     |     1            |    0	     |    1	    |
| HIV-2				|    1     |    1  	       |    1   	     |     1            |    1	     |    1	    |
| Red alga			|    1     |    1  	       |    1   	     |     1            |    1	     |    1	    |
| strain K12			|    1     |    1  	       |    1    	     |     1            |    1	     |    1	    |
|-----------------------------------------------------------------------------------------------------------------------------------|
| Total				| 7596     | 7562              | 6431                |  6628	        | 6415       | 6411         |	
|-----------------------------------------------------------------------------------------------------------------------------------|  

**SomaDataIO**  

[SomaDataIO v5.3.1](https://somalogic.github.io/SomaDataIO/index.html) is an R package for working with the SomaLogic ADAT file format. ADAT is a tab-delimited text file format that contains various data elements, including SOMAmer reagent intensities, sample data, sequence information, and experimental metadata. For each SOMAmer reagent sequence, the ADAT file usually includes the corresponding protein name, UniProt ID, Entrez Gene ID, and Entrez Gene symbol.    

```  
library(SomaDataIO)
library(purrr)
library(tidyr)
library(dplyr)
library(ggplot2)  
```

The `read_adat()` function imports data from ADAT files.  

```  
base.dir = "/Users/adinasa/Documents/"
adat_file <- "example.adat"
my_adat <- read_adat(paste0(base.dir,adat_file))  
```  

Update the ADAT file with sample group information (adding sample group details from external file). Save the updated ADAT file (optional).  

```  
meta_file = paste(base.dir, "MAPPING.csv", sep="/")  
meta <- read.csv(meta_file, header = T, stringsAsFactors = FALSE)  
meta$SampleId <- as.character(meta$SampleId)  
my_adat <- dplyr::left_join(my_adat,meta, by="SampleId", keep=FALSE)
write_adat(my_adat, file = paste(base.dir, "example_updated.adat", sep="/"))  
```  

Utility functions.  

regex for analytes

```  
is_seq <- function(.x) grepl("^seq\\.[0-9]{4}", .x)  
```  

center/scale vector (z-scores).  

```  
cs <- function(.x) {      out <- .x - mean(.x)  
  out / sd(out)       
}
```  

Data were log2-transformed within each sample. Control and Disease are two groups (this may vary in your data).  

```  
cleanData <- my_adat %>% 
  filter(SampleType == "Sample") %>% drop_na(Group) %>% 
  log2() %>% 
  mutate(SampleGroup = as.numeric(factor(Group, levels=c("Control", " Disease"))) - 1) %>% 
  modify_if(is_seq(names(.)), cs)
```  

Human proteins with Uniprot ID and QC=PASS selected.  

```  
t_tests <- getAnalyteInfo(cleanData2) %>% 
  filter(ColCheck == "PASS") %>% 
  filter(Organism == "Human") %>%
  filter(UniProt != "") %>%
  select(AptName, SeqId, Target = TargetFullName,Organism, EntrezGeneID, EntrezGeneSymbol, UniProt, ColCheck)
```  

Performed a Student’s t-test.  

```  
t_tests <- t_tests %>% 
  mutate(
    formula = map(AptName, ~ as.formula(paste(.x, "~ SampleGroup"))), 
    t_test  = map(formula, ~ stats::t.test(.x, data = cleanData,var.equal = TRUE)),  
    t_stat  = map_dbl(t_test, "statistic"),            
    p.value = map_dbl(t_test, "p.value"),              
    fdr     = p.adjust(p.value, method = "BH")         
  ) %>% arrange(p.value)
```  
  
The results were used to identify proteins significantly associated with disease using a Benjamini-Hochberg false discovery rate (FDR) threshold of 1%.  

```
t_tests [t_tests$fdr <= 0.1,]
```  

Further reading ...   
[Tandem Mass Tag (TMT)-based quantitation of proteins](https://adinasarapu.github.io/posts/2020/01/blog-post-tmt/)  
[Label-Free Quantitation (LFQ) of proteins](https://adinasarapu.github.io/posts/2018/04/blog-post-lfq/)  
