# Overview
This directory provides example code for you to calculate PRS from a discovery GWAS in a target sample. The pipeline automatically provides files for target samples of interest, specifically COGA, and reference files so that you do not need to download them or move files to scratch. This pipeline uses [PRScs](https://github.com/getian107/PRScs/blob/master/README.md "PRScs") and [PRScsx](https://github.com/getian107/PRScsx) and assumes that your GWAS summary statistics are already in the correct file format. 
# Prep for pipeline
"The summary statistics file must have the following format (including the header line):

```
    SNP          A1   A2   BETA      P
    rs4970383    C    A    -0.0064   4.7780e-01
    rs4475691    C    T    -0.0145   1.2450e-01
    rs13302982   A    G    -0.0232   2.4290e-01
    ...
```
Or:
```
    SNP          A1   A2   OR        P
    rs4970383    A    C    0.9825    0.5737                 
    rs4475691    T    C    0.9436    0.0691
    rs13302982   A    G    1.1337    0.0209
    ...
```
where SNP is the rs ID, A1 is the effect allele, A2 is the alternative allele, BETA/OR is the effect/odds ratio of the A1 allele, P is the p-value of the effect. In fact, BETA/OR is only used to determine the direction of an association. Therefore if z-scores or even +1/-1 indicating effect directions are presented in the BETA column, the algorithm should still work properly."

# Computing PRS in the EUR subsets of ABCD or COGA or UK Biobank using PRS-cs
### Setup: Copy over the master script for the cohort you are using into your working directory

COGA master script = /lts/aalab/arpana_data/community_scripts/coga_eur_prscs/0-master_prscs.sh  

### Step 1: Run PRS-cs

To run the first step of the master script you use the command format:  
bash 0-master_prscs.sh step1 [NAME OF TRAIT] [path to sum stats] [GWAS sample size]

For example:  
bash 0-master_prscs.sh step1 pau /scratch/aalab/suri/data/cleaned_data/pau_forPRS.txt 435563

This will start 22 array jobs (one for each chromosome) that should finish in <1 day. Once all of these jobs are complete (check the queue), you can move on to step 2. 

### Step 2: Concatenate files and score individuals

To run the second step of the master script you use the command format:  
bash 0-master_prscs.sh step2 [NAME OF TRAIT]

For example:  
bash 0-master_prscs.sh step2 pau

Your scores will be located in a file with the following naming scheme:

```
${WORKING_DIR}/${trait}_prs/output/${trait}_${coga_or_abcd}_eur_prs_complete.txt
```

# Computing PRS in the AFR subsets of COGA using PRS-csx

To compute PRS in our AFR ancestry subsets, we use PRS-csx which incorporates GWAS from multiple ancestries. In our case, we use GWAS of EUR and AFR samples to increase power.

### Setup: Copy over the master script for the cohort you are using into your working directory

COGA master script = /lts/aalab/arpana_data/community_scripts/coga_afr_prscsx/0-master_prscsx.sh  

### Step 1: Run PRS-cs

To run the first step of the master script you use the command format:  
bash 0-master_prscsx.sh step1 [NAME OF TRAIT] [path to sum stats] [order of GWAS ancestries] [GWAS sample sizes]

For example:  
bash 0-master_prscsx.sh step1 pau ../data/pau_no_coga_forPRS.txt ../data/aud_afr_forPRS.txt EUR AFR 351113 56648

This will start 22 array jobs (one for each chromosome) that should finish in <1 day. Once all of these jobs are complete (check the queue), you can move on to step 2. 

### Step 2: Concatenate files and score individuals

To run the second step of the master script you use the command format:  
bash 0-master_prscsx.sh step2 [NAME OF TRAIT]

For example:  
bash 0-master_prscsx.sh step2 pau

Your scores will be located in a file with the following naming scheme:
```
${WORKING_DIR}/${trait}_afr_prs/output/${trait}_${coga_or_abcd}_afr_prs_complete.txt
```

# Info on PRS-cs settings

- phi = default
- n_iter = 10000
- n_burnin = 5000
- seed = 1234
- reference files = 1000 genomes files located in /ref/aalab/data/prscs
- for AA analysis using PRS-csx, meta = TRUE 

# Info on COGA target dataset:

- EUR files imputed using hrc
- AFR files imputed using caapa
- filtered to MAF > 0.01, HWE>10^-6, INFO>=0.8 and marker genotype rate >95%
