---
layout: post
title: Rebinning Sediment Metagenomes
date: '2021-07-26'
categories: Bioinformatics
tags: [DNA]
---

_**Objective:**_ Sediments isolated from P. oceanica Rhizosphere were initially binned using co-assembly approaches and automated binning. I am interested trying to get a better chromatiales bin that we might use for a pangenomics appraoch (current one is only 85% complete). 

The goal today will be to run the metagenomics pipeline from Anvio to get a better chromatiales bin from our sediment libraries.  

### Set up
Project Nr: prj001-comp-gene-chr

Base Directory (in cologne):

```/opt/extern/bremen/symbiosis/sogin/prj001-comp-gene-chr```

Starting data: 

1. A coassembly from all genomic libraries done in megahit

2. BAM files for coverage statistics 
Data and workflow published in Sogin et al. 2021

### File directory structure
1. 01_Data; contains contigs.fa; and a subdirectory of BAM files (sorted and indexed)
2. 02_Analysis; contains directory files for anvio analysis 
3. 03_Scripts; contaains scripts for anvio analysis


### Workflow
Based on Anvio metagneomics workflow published [here](https://merenlab.org/2016/06/22/anvio-tutorial-v2/)

Log into Cologn Server using ssh tunnel

```ssh -L 6060:localhost:6060: maggie@server.university.de```

Activate Anvio: 


1. Generate an anvio database

This takes a while to run; run using a bash script:


```
#!/bin/bash
#
#$ -cwd
#$ -j y
#$ -S /bin/bash
#$ -pe smp 24
#$ -V
#$ -q main.q

#make scratch directory; change to directory
mkdir /scratch/sogin/tmp.$JOB_ID/ -p; 
cd /scratch/sogin/tmp.$JOB_ID/

#copy over data 
rsync -r /opt/extern/bremen/symbiosis/sogin/prj001-comp-gene-chr /scratch/sogin/tmp.$JOB_ID/

conda activate anvio-7

#generate database
anvi-gen-contigs-database -f ../01_Data/contigs.fa -o contigs.db -n 'Sediment coassembly db --num-threads 24'

# get hmms for contigs database
anvi-run-hmms -c contigs.db --num-threads 24

# get COGs
anvi-run-ncbi-cogs -c contigs.db --num-threads 24

rsync -r /scratch/sogin/tmp.$JOB_ID/ /opt/extern/bremen/symbiosis/sogin/prj001-comp-gene-chr/

## CLEAN UP ##
rm /scratch/sogin/tmp.$JOB_ID/ -R;

echo "job finished: "
date
```

After script runs, display stats

```
# distplay stats
anvi-display-contigs-stats contigs.db
```

stats: 

Unforunately, the run hmms didn't work. Run hmms on indivdiual bins following binning. 

