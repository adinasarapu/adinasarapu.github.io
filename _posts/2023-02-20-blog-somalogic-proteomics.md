---
title: 'Quantitative proteomics: aptamer-based quantitation of proteins'
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
The aptamer-based [SomaScan®](https://somalogic.com) assay is one of the popular methods of measuring abundances of protein targets. There is very little information on correlation between mass spectrometry (MS)-based proteomics, SOMAscan and Olink assays; Olink is another popular high throughput antibody-based platform. Some studies also reported a measurement variation between those platforms. In general aptamers/SOMAmers are selected against target proteins in their native conformation and in some cases against a functional protein with “known” post translational modifications (PTMs). It’s well known that novel PTMs (pathogen or disease-induced) can impact the protein structure, electrophilicity and interactions with proteins. The other main disadvantage is quantification which is based on DNA microarray chips (background noise). The main advantages are lower cost and data analysis.  

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

The SomaScan Assay v4.1 measures simultaneously ~6,600 unique human proteins in a single sample (see **Table 2**). Those protein targets were evaluated by ~7,300 aptamers called SOMAmers (Slow Off-rate Modified Aptamers). SOMAmers are short single-stranded DNA molecules, which are chemically modified to specifically bind to protein targets. The SOMAscan assay measures native proteins in complex matrices by transforming each individual protein concentration into a corresponding SOMAmer reagent concentration, which is then quantified using DNA microarrays. SOMAmer reagents are selected against proteins in their native folded conformations and are therefore generally found to require an intact, tertiary protein structure for binding.  

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

**ADAT file**
  
ADAT is a tab-delimited text file format. The contents include SOMAmer reagent intensities, sample data, sequence data and experimental metadata. For each SOMAmer reagent sequence, the ADAT file typically contains corresponding protein name, UniProt ID, Entrez Gene ID and Entrez Gene symbol.  

**SomaDataIO**  

[SomaDataIO v5.3.1](https://somalogic.github.io/SomaDataIO/index.html) is an R package for working with the SomaLogic ADAT file format.  

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
