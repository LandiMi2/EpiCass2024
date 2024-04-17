
## Quality control and trimming tutorial 

Within this tutorial, we will:
 - Perform [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) analysis on selected EMseq files.  
 - Utilize [TrimGalore](https://github.com/FelixKrueger/TrimGalore/blob/master/Docs/Trim_Galore_User_Guide.md) for trimming and adapter removal (if present), resulting in clean data ready for the subsequent Bismark tutorial.

### Our samples 
We have EM-Seq data that were generated for this course (explain how). The data comprises sequencing results from two treatments: high-yield and low-yield cassava plants. Our analysis will focus on three replicates for each treatment.
Sample IDs:  
1. High-yield samples (A43,A56,A57)
2. Low-yield samples (A04,A13,A21)

### Running QC 

Create a folder for quality control `mkdir QC trim`
Run the quality control for all samples in the data folder and save the outputs in the QC folder with the below command. FastQC tool used for this tuturial is `FastQC v0.12.1`

`fastqc -o QC/ data/*.fastq.gz`  

Output foles for each file:  
 - A04_R1_course_fastqc.html  
 - A04_R1_course_fastqc.zip

 Download/copy the html file to your local computer to view on your browser.  
`scp -r  mlandi@172.30.2.200:"/data01/mlandi/EM-SeqDataAnalysis/EpiCassWorshop2024/trial/QC/*.html" ./`

Questions:
1. After inversitgating the QC plots, do we really have to do a trimming step?

Check out sample A04 sequence quality and sequence content 
![Per Base Sequence Quality](https://github.com/LandiMi2/EpiCass2024/blob/main/seqQuality.png?raw=true)
![Per Base Sequence Contet](https://github.com/LandiMi2/EpiCass2024/blob/main/seqContent.png?raw=true)

What do you think?


### Running Trim Galore
Trim Galore is a wrapper around [Cutadapt](https://github.com/marcelm/cutadapt) and FastQC. 
Important flag while using Trim Galore 

```trim_galore --paired --clip_R1 2 --clip_R2 2 --three_prime_clip_R1 2 --three_prime_clip_R2 2 --fastqc --quality 20 -j 6 --illumina --length 100 <R1.fastq.gz> <R2.fastq.gz>```
 


