
# miRNA-mRNA_Integration
A detailed guide to analise and integrate small-RNASeq and RNASeq samples using [miARma-Seq](https://miarmaseq.com "miARma's Homepage") focus on differentially expresed genes and miRNAs + miRNA-mRNA interactions and a statistical correlation.


----------------------------
# Table of Contents
----------------------------

   * [Overview](#overview)
   * [miARma Installation](#installation)
   * [Indexes and other needed files](#indexes_and_other_needed_files)
       * [Genome Indexes](#genome_indexes)
       * [Organism Anottation](#organism_anottation)
   * [A complete Analysis](#complete_analysis)
       * [Summary of experiment](#summary)
       * [Data retrieval](#data_retrieval)
       * [Analysis](#analysis)
         * [mRNA Analysis](#mRNA)
         * [miRNA Analysis](#miRNA)
         * [Integration](#integration)
       * [Results](#results)
   * [Conclusions](#conclusions)

----------------------------
# Overview
----------------------------

miARma-Seq, miRNA-Seq And RNA-Seq Multiprocess Analysis tool, is a comprehensive pipeline analysis suite designed for NGS transcriptomic analysis:
- Perform a quality analysis from fastq files
- Remove (and predict) adapter sequences over-represented in the samples
- Aligns reads using different aligners: HISAT2, STAR, BWA, TopHat, Bowtie1 or Bowtie2
- Identify mRNA, miRNA and circRNAs. Also De Novo miRNAs is offered
- Differential expresion analysis among any type of conditions (edgeR and NOISeq)
- Functional enrichment for RNASeq (Gene Ontology and KEGG)
- miRNA-mRNA target prediction for RNASeq and/or small RNASeq
- Data integration for miRNA and mRNA based on statistical correlation (Pearson or Spearman)

You can get more information about miARma at its [webpage](https://miarmaseq.com "miARma's Homepage") or you can read the [miARma-Seq article](https://www.nature.com/articles/srep25749).

<p align="center">
  <img src="https://github.com/eandresleon/miRNA-mRNA_Integration/blob/master/src/images/Figure1.png">
</p>


**Fig 1.** miARma-Seq is presented as a stand-alone tool that provides different well-established softwares at ease of installation process. Our suite can analyse a large number of samples due to its multithread design. Here we show that the analyses of miRNA, mRNA and circRNAs against validated datasets can be easily accessible to research community.

----------------------------
# Installation
----------------------------

miARma has been mainly developed in Perl and R. Our tool has been designed to reduce prerequisites to minimun so, in order to run a complete analysis, you just need:

- If you are using an OSX computer, please install Xcode.
- Java JDK runtime (javac)
- C compiler (gcc) [Included in Xcode]
- [R](https://cran.r-project.org)
- [Bioconductor](https://www.bioconductor.org/install/)
- miARma-Seq. You can use the version included in this [repository](https://github.com/eandresleon/miRNA-mRNA_Integration/raw/master/src/soft/miARma-Seq.1.7.5.tar.gz) or find the latest version at the [installation webpage](http://miarmaseq.idoproteins.com/installation). There you'll find docker containers, virtualbox images and source code. In this guide we will use the source code as it only implies to uncompress a file.

So, to install miARma-Seq, please download the [source code](https://github.com/eandresleon/miRNA-mRNA_Integration/raw/master/src/soft/miARma-Seq.1.7.5.tar.gz) and uncompress it:

```
mkdir ~/bin/
mv miARma-Seq.1.7.5.tar.gz ~/bin/
cd ~/bin/
tar -xzf miARma-Seq.1.7.5.tar.gz
```
In that way miARma will be installed inside the folder bin in your home directory. Finally to perform this example you will need the [sra toolkit](https://www.ncbi.nlm.nih.gov/sra/docs/toolkitsoft/), as fastq samples are provided as sra files.

----------------------------
# Indexes and other needed files
----------------------------

In order to analyse transcriptomic data, we need to download the following information.
## Genome Indexes ##
Sequenced reads must be aligned (placed) into the reference genome. As we will study human samples, miARma-Seq will need to download human genome data. Besides we will use the [HISAT2](https://ccb.jhu.edu/software/hisat2/index.shtml) aligner, so human HISAT2 indexes will be downloaded following these steps:

```
cd ~/bin/miARma-Seq.1.7.5
curl -L -O https://sourceforge.net/projects/miarma/files/Genomes/Index_hisat2_hg19.tar.bz2
tar -xjf Index_hisat2_hg19.tar.bz2
```
Once uncompressed, a Genome folder will be created including the whole human genome in fasta format and its HISAT indexes.



## Organism anottation ##

When sequences are localized in the reference genome, these must be quantified. The process of counting reads is called read summarization, and to perform this task miARma-Seq includes [FeatureCounts](http://bioinf.wehi.edu.au/featureCounts/). As a result of this part, we obtain the total number of sequences (proportional to the expression levels of the RNAs to be studied). This lets us to find out which genes (coding or not) have been transcribed and expressed in the form of RNA from the genomic DNA.
To perfom this step, miARma-Seq needs to know the position of each genomic feature (exon, transcript, gene) in the genome. To do that we will use the GENCODE v26 anottation file. For miRNAs we will use the microRNA annotation file from miRBase version 20.
These files can be obtained from this git-hub repository:

```
cd ~/bin/miARma-Seq.1.7.5
mkdir data
cd data

curl -L -O https://github.com/eandresleon/miRNA-mRNA_Integration/raw/master/src/data/gencode.v26_GRCh37.annotation.gtf.gz
gunzip gencode.v26_GRCh37.annotation.gtf.gz

curl -L -O https://raw.githubusercontent.com/eandresleon/miRNA-mRNA_Integration/master/src/data/miRBase_Annotation_20_for_hsa_mature_miRNA.gtf
```

## Complete Analysis

At this moment, we have already downloaded all needed software to perform a complete anaylysis. Besides we have obtained the human genome, its HISAT indexes and all human gene and miRNA anottation.
So we are ready to perfom the complete study.

## Summary

Briefly, the work by [Lu et al](https://www.nature.com/articles/nm.4424)  is focused on colorectal cancer (CRC), as it remains the leading cause of cancer-related death worldwide. Cetuximab and panitumumab are typical CRC treatments that bind the extracellular domain of the EGF receptor enhancing its internalization and degradation. When these are combined with chemotherapy, up to 72% response rates are reported. However, de novo and acquired drug resistance frequently arises, and little is known about non-genetic resistance mechanisms.

In order to increase our knowledge about these mechanisms, CRC samples from colon adenocarcinoma cancer cell line HCA-7 were treated with cetuximab for approximately four months to induce resistance and deposited at the [NCBI Gene Expression Omnibus (GEO) ](https://www.ncbi.nlm.nih.gov/geo/) repository with accession number [GSE82236](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE82236).

## Data retrieval

SRA part

## Complete Analysis

Once fastq files are obtained ans saved in a folder, we will perfom the mRNA analysis and the miRNA analysis by miARma-Seq in two different steps. The last part of the second analysis will include the integration analysis:

### **mRNA**
-------
Three cetuximab-resistant and three non-resistant colon cancer samples were downloaded from GEO and inspected using FastQC to identify sequencing errors such as bad quality reads or possible accumulation of adapter sequences. Subsequently, sequences were aligned using Hisat2 and those with a quality score above 25 were counted and summarized as expression values for each gene using the GENCODE version 26 annotation file. To obtain those genes with a different expression pattern amongst both types of samples, we use edgeR as it enables the calculation of differential expression values. As it was done by Lu et al, genes having a |log2FC|≥1 and a FDR ≤0.01 were considered as differentially expressed. See Figure 2.

### **miRNA**
-------

Correspondingly, three CRC-CR and three CRC miRNASeq colon cancer samples were obtained and studied. Single-end sequenced reads were inspected using FastQC. Once this process, reads were pre-processed using Minion to predict (as it was not facilitated by the authors) and to remove the accumulated adapter sequences detected in the previous step. Afterwards, trimmed reads with a size comprised between 18 and 35 nucleotides were aligned using Bowtie1 and quantified using FeatureCounts and the microRNA annotation file from miRBase version 20. miRNAs having a |log2FC|≥1 and a FDR ≤0.01 were considered as differentially expressed. See Figure 3 for more detail.

### **Integration**

miARma-seq enables to explore miRNA-mRNA interactions using the miRGate query API [34] among genes and miRNAs differentially expressed to identify alterations in regulation patterns linked to phenotype. On the contrary, to include in the analysis all expressed genes and miRNAs, a threshold FDR value equal to 1 can be specified. All those interactions deposited in miRGate and constituted by miRNAs and genes of interest, are retrieved and stored in a file for later analysis. In addition, a statistical correlation study is carried out for the subsequent integration of data. In this way, the correlation (Pearson or Spearman are available) between the expression values ​​of the deregulated genes and the miRNAs is calculated. 
As a final result, we obtain a curated dataset that contains differentially expressed genes and miRNAs with a predicted miRNA-mRNA regulatory interaction and, in addition, formed by two elements that undergo a change of expression level which exhibits an elevated statistical correlation.


## Results

Resistant and non-resistance to cetuximab colon cancer transcriptome samples of both genes and miRNAs were analysed and a total of 368 genes and 29 miRNAs with statistically significant expression alteration were identified. Our results show 165 up-regulated and 203 downregulated genes (|log2FC|≥1 and FDR ≤0.01). In Figure 2a we show the five genes that exhibit the lowest (OLFM1, SERP2, CRABP1, DKK1 and ZNF608) and the five genes presenting the greater fold changes values (MIR100HG, DUSP4, XAF1, BHLHE41 and HS3ST5) while having minimum FDR scores. Figures 2b and 2c illustrate in detail the change of expression of the two most over-expressed and inhibited genes in our dataset. 

<p align="center">
  <img src="https://github.com/eandresleon/miRNA-mRNA_Integration/blob/master/src/images/Figure2.png">
</p>
**Fig 2.** Most overexpressed and repressed genes from resistant (CRC-CR) and non-resistance (CRC) cetuximab samples from colorectal cancer patients. A) Shows a volcano plot including the five most inhibited (OLFM1, SERP2, CRABP1, DKK1 and ZNF608) and overexpressed genes (MIR100HG, DUSP4, XAF1, BHLHE41 and HS3ST5) including log2FC and its FDR (-log10 FDR). B) Display the difference in expression level among CRC and CRC-RC samples from the two most differentially over-expressed genes: MIR100HG and BHLHE41. C) Exhibit the modification in expression level among CRC and CRC-RC samples from the two most differentially repressed genes: OLFM1 and SERP2.

On the contrary, from the small RNA analysis, we have obtained seven up-regulated and 22 downregulated miRNAs (. According with the Figure 3a, the highest overexpressed miRNAs are miR-100-5p, miR-125b-1-3p, miR-125b-5p and let-7a-2-3p. At the same time, the most repressed miRNAs are miR-99a-5p, miR-34b-5p, miR-125b-2-3p and the tumour suppressors miR-1, miR-145-5p and miR-143-3p. Figures 3b and 3c represent the variation of expression of the two most over-expressed and repressed miRNAs among our results. 

<p align="center">
  <img src="https://github.com/eandresleon/miRNA-mRNA_Integration/blob/master/src/images/Figure3.png">
</p>
**Fig 3.** . Most overexpressed and repressed miRNAs from resistant (CRC-CR) and non-resistance (CRC) cetuximab samples from colorectal cancer patients. A) Shows a volcano plot including the five most over-expressed (miR-100-5p, miR-125b-1-3p, miR-125b-5p, let-7a-2-3p and miR-4286) and repressed miRNAs (miR-1, miR-99a-5p, miR-125b-2-3p, miR-145-5p and miR-143-3p) including log2FC and its FDR (-log10 FDR). B) Display the difference in expression level among CRC and CRC-RC samples from the two most differentially over-expressed miRNAs: miR-100-5p and miR-125b-1-3p. C) Exhibit the modification in expression level among CRC and CRC-RC samples from the two most differentially inhibited miRNAs: miR-1 and miR-125b-2-3p.

Once the 29 miRNAs and the 368 differentially expressed genes were identified, miARma-Seq performs a statistical integration between these elements. Giving to the total possible number of 10672 correlations, 6287 were statically significant (p-value ≤ 0.05) according to the Pearson correlation method. Among the most overexpressed genes, the positive correlation that appears with several of the most overexpressed microRNAs stands out. 
In this way, as shown in Table 1, we can see that MIR100HG (gene with extremely high fold change value) has an average positive R coefficient score higher than 0.97 and a p-value < 0.05 for the four most overexpressed microRNAs: miR-100-5p, let-7a-2-3p, miR-125b-5p and miR-125b-1-3p. The explanation of this event is simple since miR100HG is a long non-coding RNA that acts as a host cluster gene. In this way, this RNA functions as a policistron that encodes exactly for the four miRNAs (miR-100-5p, let-7a-2-3p, miR-125b-5p and miR-125b-1-3p) that are overexpressed.

In turn, in Table 1 we can also observe another interesting result, the changes of expression of MIR100HG correlate negatively with three of the most repressed microRNAs: miR-99a-5p, miR-125b-2-3p and miR-34b- 5p, which could indicate a possible second-level regulation. To explain these negative correlations of expression based on the possible regulatory interaction carried out by a microRNA on a gene, our tool collects all possible miRNA-mRNA interactions stored in the miRGate database for these genes and miRNAs having an altered expression between resistant and not resistant to the drug samples. All relevant information such as the prediction method, target site, method score and energy among others, is saved in an excel compatible file to be studied in detail.

Finally, to provide a final result as relevant as possible, miARma-Seq performs an integration analysis between the three available data, that is:  differential expression values, miRNA-mRNA target pairs and statistical correlation. In this way, out tool combines all information a store the relevant results in an excel file. This report contains the miRNA-mRNA connexions formed by differentially expressed genes and miRNAs having a statistically significant correlation in expression levels. As an example, Table 2 shows the results for the five most overexpressed and repressed genes of our data set. It includes in detail, putative miRNAs acting as regulators and the correlation of expression found. In turn, Figure 4 display these results in a graphical manner.

<p align="center">
  <img src="https://github.com/eandresleon/miRNA-mRNA_Integration/blob/master/src/images/Figure4.png">
</p>
**Fig 4.** Set of miRNA-mRNA predicted interactions for the five most overexpressed and repressed genes with those miRNAs differentially expressed having a negative correlation of expression (R Pearson's coefficient) ≥ -0.9 and p-value ≤ 0.05. 


# Conclusions
The integration of data from different sources helps to improve our knowledge about possible mechanisms that governs underlying relationships arising from the pathophysiology of diseases. These techniques allow the development of new strategies for the early detection and treatment of human diseases. An example of this is the recent article published by the consortium of the cancer genome atlas (TCGA) that has integrated the information of about 11 thousand samples from 33 types of tumours, using up to 4 ‘-omic’ technologies (exome sequencing, miRNA and mRNA transcriptome sequencing and DNA methylation). This combination of data has resulted in a molecular grouping of tumours unknown until now [85].
The necessary methodology for this data integration includes the management of a large number of programs that makes it difficult to perform this task for users less experienced in computer-based environments. Therefore, it is advisable to create tools useful to minimize the technical challenges to conduct these analyses. In this work we have described the methodological details of the miARma-seq pipeline to combine transcriptome information from mRNA and miRNA regulation. We have also described the statistical underlying framework in the context of previous work dealing with the identification of novel miRNA-mRNA interactions conserved through different cancer tumour types [6]. 
The purpose of this tool, is to facilitate the extraction of meaningful information regarding relationships of mRNA-miRNA regulation from biological samples. 
