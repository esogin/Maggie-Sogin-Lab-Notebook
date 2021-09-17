---
layout: post
title: Rebinning-Sediment-MG-merge-profiles
date: '2021-09-17'
categories: Analysis
tags: bioinformatics
---

_**Objective:**_ 
Continue with Anvio protocol for binning metagenomes

##SetUp

Base Directory (in cologn):

```bash
cd /opt/extern/bremen/symbiosis/sogin/prj001-comp-gene-chr
conda activate anvio-7
```

_**Notes from last time**_: 

The anvi-profile function seems to have worked and we now have profile data for each library in the 02_Analysis folder. 

1. Merge anvi profiles 
One issue: I forgot to assign a sample name to the profile database

base script to change: 

```bash
anvi-db-info in_I/PROFILE.db --self-key sample_id \
                                     --self-value "in_I" \
                                     --just-do-it

#check if it worked
anvi-db-info edge_E/PROFILE.db
```

```bash
anvi-merge */PROFILE.db -o SAMPLES-MERGED -c contigs.db
```
