For more information here is the [bismark documentation](https://github.com/FelixKrueger/Bismark/tree/master/docs/bismark). Always read the manual. 

Below is a summary of commands for this tutorial. We will map our EMSeq data (high and low samples) to TMEB117 cassava genome chromosome 2.  

Bismark is split into three main steps:

- Genome preparation
- Read Alignment
- Methylation extractor 

## Genome preparation 

Create a directory ```mkdir genome ``` in your working directory for the genome. 


Run ```bismark_genome_preparation```

```bismark_genome_preparation <path_to_genome_folder> ```


## Read Alignment 

We will use a for loop. First, creat folders we will need to use. 

```mkdir -p data/high/{bam,dedups,meth} data/low/{bam,dedups,meth}```

using ```pwd``` save the paths of folders for the ```for loop```

```genome=<path_to_genome_folder>```  
```high=<path_to_high_folder>```  
```bam=<path_to_bam_folder_in_high>```  
```dedups=path_to_dedups_folder_in_high```  
```meth=path_to_meth_folder_in_high```  









