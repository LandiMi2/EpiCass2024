For more information here is the [bismark documentation](https://github.com/FelixKrueger/Bismark/tree/master/docs/bismark). Always read the manual. 

Below is a summary of commands for this tutorial. We will map our EMSeq data (high and low samples) to TMEB117 cassava genome chromosome 2.  

Bismark is split into three main steps:

- Genome preparation
- Read Alignment
- Methylation extractor

For this analysis we will use for loops to run all the sample. For one samples we can run all the steps to see the expected outputs before running the for loops. 

## Genome preparation 

Create a directory `mkdir genome` in your working directory for the genome. 


Run `bismark_genome_preparation`

`bismark_genome_preparation <path_to_genome_folder> `


## Read Alignment 

`bismark [options:-p, --un ,--ambiguous] --genome <genome_folder> -1 <mates1> -2 <mates2> -o <path_to_alignment_folder>`

**-p** - _threads_  
**--un** - _write all reads that could not be aligned to a file in the output directory_   
**--ambiguous** - _write all reads which produce more than one valid alignment with the same number of lowest mismatches or other reads that fail to align uniquely to a file in the output directory_  


Create folders we will need to use. 

`mkdir -p data/high/{bam,dedups,meth} data/low/{bam,dedups,meth}`

using `pwd` save the paths of folders for the ```for loop```

`genome=<path_to_genome_folder>`  
`high=<path_to_high_folder>`  
`bam=<path_to_bam_folder_in_high>`  
`dedups=path_to_dedups_folder_in_high`  
`meth=path_to_meth_folder_in_high`  

Running mapping 

```
for i in "$high"/*_R1_001_val_1.fq.gz;

do
   prefix=$(basename $i _R1_001_val_1.fq.gz)
   bismark -p 12 --un --ambiguous --genome "$genome" -1 "$high"/"${prefix}"_R1_001_val_1.fq.gz \
   -2 "$high"/"${prefix}"_R2_001_val_2.fq.gz -o "$bam";

done
```

Removing duplicates

`deduplicate_bismark -p <path_to_bam_file>`

```
for i in "$bam"/*_R1_001_val_1_bismark_bt2_pe.bam;

do
   prefix=$(basename $i _R1_001_val_1_bismark_bt2_pe.bam)
   deduplicate_bismark -p "$bam"/"${prefix}"_R1_001_val_1_bismark_bt2_pe.bam \
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
for i in "$dedups"/*_R1_001_val_1_bismark_bt2_pe.deduplicated.bam;

do
   prefix=$(basename $i _R1_001_val_1_bismark_bt2_pe.deduplicated.bam)
   bismark_methylation_extractor -p --no_overlap --cytosine_report --CX --comprehensive --genome_folder "$genome" \
   --ignore 6 --ignore_r2 7 "$dedups"/"${prefix}"_R1_001_val_1_bismark_bt2_pe.deduplicated.bam -o "$meth" ;

done
```

## Reports 

`bismark2report --alignment_report <path_to_alignment_report> --dedup_report <path_to_deduplication_report> --mbias_report <path_to_MBias_report>`


Outputs to use for the downstream analysis - R:
`{fileName}_bismark_bt2_pe.deduplicated.CX_report.txt`










