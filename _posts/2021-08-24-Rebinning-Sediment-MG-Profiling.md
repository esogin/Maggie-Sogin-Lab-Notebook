---
layout: post
title: Rebinning Sediment Metagenomes - Profiling the database
date: '2021-08-24'
categories: Bioinformatics
tags: [DNA]
---

##*Objective* 
Today's goal is set up a script to profile the contigs database using the BAM files. This is an important step in the binning process as it provides coverage statistics needed to accurately place contigs into bins. 

##1. Set up and check workspace 

Log into cologe: 

```
cologn
ssh 8100:localhost:8100 gc-node-2
conda activate anvio-7
cd /opt/extern/bremen/symbiosis/sogin/prj001-comp-gene-chr/
``` 

Re-display contigs database stats: 
```
anvi-display-contigs-stats contigs.db
```

We have something like 12 M contigs!

##2. Set up profiling qsub, bash script

if the bam files are not indexed, you need to add this part to the script: 


```anvi-init-bam SAMPLE-01-RAW.bam -o SAMPLE-01.bam```

This will sort and index your BAM file for sample 1, you can write a loop to repeat this for each BAM file. 

otherwise, move on to anvi-profile comand: 

```
anvi-profile -i SAMPLE-01.bam -c contigs.db --output-dir profiles --sample-name NEW-NAME 
```

Important: New Namae can not have - in name file, only _, letters etc. 

| Names                 | New Name |   
|-----------------------|--------|
| 3847_A_sorted.bam.bai | Out_A  | 
| 3847_B_sorted.bam.bai | Out_B  | 
| 3847_C_sorted.bam.bai | Out_C  | 
| 3847_D_sorted.bam.bai | Edge_D | 
| 3847_E_sorted.bam.bai | Edge_E |
| 3847_F_sorted.bam.bai | Edge_F |
| 3847_G_sorted.bam.bai | In_G   | 
| 3847_H_sorted.bam.bai | In_H   |
| 3847_I_sorted.bam.bai | In_I   |
 

```
#!/bin/bash
#
#$ -cwd
#$ -j y
#$ -S /bin/bash
#$ -pe smp 48
#$ -V
#$ -q main.q@@himem

#make scratch directory; change to directory
mkdir /scratch/sogin/tmp.$JOB_ID/ -p; 
cd /scratch/sogin/tmp.$JOB_ID/

#copy over data 
rsync -r /opt/extern/bremen/symbiosis/sogin/prj001-comp-gene-chr/ /scratch/sogin/tmp.$JOB_ID/

conda activate anvio-7

#cd /scratch/sogin/tmp.$JOB_ID/01_Data/bam
#anvi-init-bam 3847_A_sorted.bam -o 3847_A_sorted_indexed.bam

cd /scratch/sogin/tmp.$JOB_ID/02_Analysis
anvi-profile -i ../01_Data/bam/3847_A_sorted.bam -c contigs.db -o profiles --sample-name Out-A -T 24 -W

rsync -r /scratch/sogin/tmp.$JOB_ID/ /opt/extern/bremen/symbiosis/sogin/prj001-comp-gene-chr/02_Analysis/

## CLEAN UP ##
rm /scratch/sogin/tmp.$JOB_ID/ -R;

echo "job finished: "
date
```

