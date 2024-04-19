For more information here is the [bismark documentation](https://github.com/FelixKrueger/Bismark/tree/master/docs/bismark). Always read the manual. 

Below is a summary of the commands for this tutorial. We will map our EMSeq data (high and low samples) to the TMEB117 cassava genome chromosome 16.

Bismark is divided into three main steps:

- Genome preparation
- Read alignment
- Methylation extraction

For this analysis, we will use for loops to run all the samples. Before running the for loops, we can run all the steps for one sample to see the expected outputs.

## Genome preparation 

Create a directory `mkdir genome` in your working directory for the genome. Copy chromosome 16 (chr16.fasta) to genome folder `cp data/chr16.fasta genome` 

Run `bismark_genome_preparation`

`bismark_genome_preparation genome/ `

## Read Alignment 

`bismark [options:-p, --un ,--ambiguous] --genome <genome_folder> -1 <mates1> -2 <mates2> -o <path_to_alignment_folder>`

**-p** - _threads_  
**--un** - _write all reads that could not be aligned to a file in the output directory_   
**--ambiguous** - _write all reads which produce more than one valid alignment with the same number of lowest mismatches or other reads that fail to align uniquely to a file in the output directory_  

Create folders we will need to use. 

First separate the sample in data folder into high and low
`mkdir high low` - High samples (A43,A56 and A57) Low samples (A04,A13 and A21)  
`mv A43_R* A56_R* A57_R* high/`  
`mv A04_R* A13_R* A21_R* low/`  
Create folder to run bismark:  
`mkdir -p result/high/{bam,dedups,meth} result/low/{bam,dedups,meth}`  

We will create a bash script to run bismark for high and low samples separetely. 

`nano mapHigh.sh`

```#!/bin/bash

#define paths
genome="/data01/mlandi/EM-SeqDataAnalysis/EpiCassWorshop2024/trial/genome"  
high="/data01/mlandi/EM-SeqDataAnalysis/EpiCassWorshop2024/trial/data/high"  
bam="/data01/mlandi/EM-SeqDataAnalysis/EpiCassWorshop2024/trial/result/high/bam" 
dedups="/data01/mlandi/EM-SeqDataAnalysis/EpiCassWorshop2024/trial/result/high/dedups"
meth="/data01/mlandi/EM-SeqDataAnalysis/EpiCassWorshop2024/trial/result/high/meth"

```
Running mapping 

```
for i in "$high"/*_R1_course.fastq.gz;

do
   prefix=$(basename $i _R1_course.fastq.gz)
   bismark -p 2 --un --ambiguous --genome "$genome" -1 "$high"/"${prefix}"_R1_course.fastq.gz \
   -2 "$high"/"${prefix}"_R2_course.fastq.gz -o "$bam";

done
```

Removing duplicates

`deduplicate_bismark -p <path_to_bam_file>`

```
for i in "$bam"/*_R1_course_bismark_bt2_pe.bam;

do
   prefix=$(basename $i _R1_course_bismark_bt2_pe.bam)
   deduplicate_bismark -p "$bam"/"${prefix}"_R1_course_bismark_bt2_pe.bam \
   --output_dir "$dedups";

done
```

## Methylation extractor 

`bismark_methylation_extractor [options: -p, --no_overlap,--cytosine_report,--CX,--comprehensive,--ignore,--ignore_r2] -o <path_to_methylation_calls_outputs>`  

**-p**-_paired-end read data_  
**--no_overlap**-_For paired-end reads it is theoretically possible that read_1 and read_2 overlap. This option avoids scoring overlapping methylation calls twice (only methylation calls of read 1 are used for in the process since read 1 has historically higher quality basecalls than read 2)_  
**--cytosine_report**-_produces a genome-wide methylation report for all cytosines in the genome. The output considers all Cs on both forward and reverse strands and reports their position, strand, trinucleotide content and methylation state_  
**--CX**-_outputs all the cytosine contexts. Default = CpG context only. This applies to both forward and reverse strands_  
**--comprehensive**-_Specifying this option will merge all four possible strand-specific methylation info into context-dependent output files. The default context - CpG context CHG context and CHH context_  
**--ignore**-_Ignore the first <int> bp from the 5' end of Read 1_  
**--ignore_r2**-_Ignore the first <int> bp from the 5' end of Read 2 of paired-end data only_  


```
for i in "$dedups"/*_R1_course_bismark_bt2_pe.deduplicated.bam;

do
   prefix=$(basename $i _R1_course_bismark_bt2_pe.deduplicated.bam)
   bismark_methylation_extractor -p --no_overlap --cytosine_report --CX --comprehensive --genome_folder "$genome" \
   --ignore 6 --ignore_r2 7 "$dedups"/"${prefix}"_R1_course_bismark_bt2_pe.deduplicated.bam -o "$meth" ;

done
```

## Reports 

`bismark2report --alignment_report <path_to_alignment_report> --dedup_report <path_to_deduplication_report> --mbias_report <path_to_MBias_report>`

```bismark2report --alignment_report ../bam/A43_R1_course_bismark_bt2_PE_report.txt --dedup_report ../dedups/A43_R1_course_bismark_bt2_pe.deduplication_report.txt --mbias_report ../meth/A43_R1_course_bismark_bt2_pe.deduplicated.M-bias.txt -o A43_report.html```

```bismark2report --alignment_report ../bam/A56_R1_course_bismark_bt2_PE_report.txt --dedup_report ../dedups/A56_R1_course_bismark_bt2_pe.deduplication_report.txt --mbias_report ../meth/A56_R1_course_bismark_bt2_pe.deduplicated.M-bias.txt -o A56_report.html```

```bismark2report --alignment_report ../bam/A57_R1_course_bismark_bt2_PE_report.txt --dedup_report ../dedups/A57_R1_course_bismark_bt2_pe.deduplication_report.txt --mbias_report ../meth/A57_R1_course_bismark_bt2_pe.deduplicated.M-bias.txt -o A57_report.html```

Download the reports 
`scp -r  mlandi@172.30.2.200:"/data01/mlandi/EM-SeqDataAnalysis/EpiCassWorshop2024/trial/result/high/report/*.html" ./`

Check the report before downloading the CX report file generated in the `meth` folder. Most importantly is the M-Bias plot. 
![MBias](https://github.com/LandiMi2/EpiCass2024/blob/main/images/MBias-A04.png?raw=true)

Looks good! 

We will either work on the server Rstudion or our own local machine. The methylation calls are not huge files (about 400M).

Download to your local `mkdir -p methCalls/high methCalls/low`
`cd methCalls/high`  
`scp -r  mlandi@172.30.2.200:"/data01/mlandi/EM-SeqDataAnalysis/EpiCassWorshop2024/trial/result/high/meth/*.CX_report.txt" ./`  
`scp -r  mlandi@172.30.2.200:"/data01/mlandi/EM-SeqDataAnalysis/EpiCassWorshop2024/trial/result/low/meth/*.CX_report.txt" ./`

Let's dive into some R coding fun! Catch you in the next tutorial!









