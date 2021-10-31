# H1
## H2
### H3
normal text

**bold text**

*italicized text*

> 1 blockquote
> 2 Block quote line 2

Empty line after blockquote line 2

1. First item
2. Second item
3. Third item

- First item
- Second item
- Third item

`code`

---

https://github.com/TorkamaniLab/preprocessing_reference_panels_for_phasing_imputation

[Torkamani Github](https://github.com/TorkamaniLab/preprocessing_reference_panels_for_phasing_imputation)

![ALTERNATE TEXT](https://www.wikihow.com/images/thumb/d/db/Get-the-URL-for-Pictures-Step-2-Version-6.jpg/v4-460px-Get-the-URL-for-Pictures-Step-2-Version-6.jpg)

# Genotype_Imputation_Pipeline
A tool for imputation of genotype array datasets from dbGaP. The Genotype Imputation Pipeline consists of the following steps:

0. Identify input genome build version outomatically
1. Lift the input to build GRCh37 (hg19)
2. Quality control 1: LD-based fix of strand flips, fix strand swaps, filter variants by missingness 
3. Split samples by ancestry
4. Quality control 2: filter samples by missingness, filter variants by HWE
5. Phase
6. Impute
  

## Dependencies
The pipeline was tested in garibaldi using the following required software and packages:

- R v3.5.1
- vcftools v0.1.14
- PLINK v1.9 
- PLINK v2.00a3LM 64-bit Intel
- samtools v1.9
- GenotypeHarmonizer v1.4.20
- ADMIXTURE
- Eagle v2.4
- Minimac4
- liftOver

If you are in garibaldi, you don't need to install these packages, and tools, they are either loaded by the job script, or executed from the required_tools directory included in this repository (see bellow).

## How to run

Before starting the pipeline, copy the contends of the required_tools folder into your home folder in garibaldi:

```
cp -r required_tools $HOME/
```

After copying the required tools, please read the following instructions on how to run the steps 0-6 manually (automated metod comming soon!).

### Step 0: Check genome build and select chain file

__Prerequisite__  
- N/A

__Usage example__  
```
qsub 0_check_vcf_build.job -v  myinput=/path/to/vcf/genotype_array.vcf,myoutput=/path/to/output/0_check_vcf_build/genotype_array.BuildChecked,gz=yes
```

Where:
- myinput is the full path to the input genotype array dataset in either vcf or vcf.gz format
- myoutput is the full path to save the output of this step
- gz (gz=yes or gz=no) is whether the input file is either vcf or vcf.gz format

The output file will have the sufix *.BuildChecked

### Step 1: Lifeover input genotype array to GRCh37 build

__Prerequisite__  
- N/A

__Usage example__  
```
qsub 1_lift_vcfs_to_GRCh37.job -v myinput=/path/to/vcf/genotype_array.vcf,buildcheck=/path/to/output/0_check_vcf_build/genotype_array.BuildChecked,myoutdir=/path/to/output/1_lift,custom_temp=/my/temp/path/tmp
```
This step will lift the input to GRCh37 build using the *.BuildChecked file generated in the previous steo to select the correct chain file.

Where:
- myinput is the same input file as step 0 (use full path)
- buildcheck is the full path to *.BuildChecked file generated in previous step
- myoutdir is path to the output folder (no file name should be used, just output folder name)
- custom_temp (optional) set a different path to be used as temporary directory, recommended in case your input file is too large (more than 100GB).

The output file will have the suffix *.lifted_[old_build]_to_GRCh37.bed


### Step 2: LD-based fix of strand flips, fix strand swaps and mismatching alleles, and initial quality control (90% missingnes per variant)

__Prerequisite__  
- `/mnt/stsi/stsi0/raqueld/1000G`

__Usage example__  
```
qsub 2_Genotype_Harmonizer.job -v myinput=/path/to/output/1_lift/genotype_array.lifted_NCBI36_to_GRCh37.bed,myoutdir=/path/to/output/2_GH,ref_path=/my/ref/path
```

Where:
- myinput is the path to the *.lifted_[old_build]_to_GRCh37.bed file generated by step 1, in the example above the /path/to/vcf/genotype_array.vcf file was lifted to /path/to/output/1_lift/genotype_array.lifted_NCBI36_to_GRCh37.bed becase the input file was generated from NCBI36 reference build version. 
- myoutdir is path to the output folder (no file name should be used, just output folder name)
- ref_path (optional) is your custom reference file path (just the folder path containing 1000 Genomes reference, no file name needed). WARNING: before using your custom reference, prepare the reference following this ste-by-step documentation: https://github.com/TorkamaniLab/preprocessing_reference_panels_for_phasing_imputation. If no custom reference path is provided, this will be the default path: /mnt/stsi/stsi0/raqueld/1000G.

The output files will have the suffix *.lifted_[old_build]_to_GRCh37.GH.bim, *.lifted_[old_build]_to_GRCh37.GH.fam, and *.lifted_[old_build]_to_GRCh37.GH.bed.

### Step 3: Estimate ancestry and split samples by ancestry

__Prerequisite__  
- `/mnt/stsi/stsi0/raqueld/1000G/ALL.merged.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.clean.vcf.gz`

__Usage example__  
```
qsub 3_ancestry_analysis.job -v myinput=/stsi/raqueld/2_GH/6800_JHS_all_chr_sampleID_c1.lifted_hg19_to_GRCh37.GH.bed,myoutdir=/stsi/raqueld/3_ancestry -N 3_6800_JHS_all_chr_sampleID_c1
```
* myinput=`/path/2_GH/inprefix.bed`
    * with `fam` and `bim` files
* myoutdir=`/path/3_ancestry`  
    * `inprefix.pruned.intersect1KG.vcf.gz`
    * `inprefix.pruned.intersect1KG.pop`
    * `inprefix.pruned.intersect1KG.5.Q.IDs`
    * `inprefix.ancestry-[ancestry_code].bed/fam/bim`


### Step 4: 2nd quality control

__Prerequisite__  
N/A  

__Usage example__ 
```
qsub 4_split_QC2.job -v myinput=/gpfs/home/raqueld/mapping_MESA/mesa_genotypes-black.lifted_NCBI36_to_GRCh37.GH.bed,myoutdir=/stsi/raqueld/N_tests,hwe='',geno=0.1,mind=0.1 -N 4_N_mesa_genotypes-black
```
```
qsub 4_split_QC2.job -v myinput=/stsi/raqueld/3_ancestry/6800_JHS_all_chr_sampleID_c1/6800_JHS_all_chr_sampleID_c1.lifted_hg19_to_GRCh37.GH.ancestry-5.bed,myoutdir=/stsi/raqueld/4_split_QC2,hwe='',geno=0.1,mind=0.1 -N 4_6800_JHS_all_chr_sampleID_c1
```
* myinput=`/path/3_ancestry/inprefix.bed`
* myoutputdir=`/path/4_split_QC2`  
* hwe=`0.1`, geno=`0.1`, mind=`0.1`
    * set as `''` to disable the flag.
> The input file will be split by chromosome and test for missingness per variant, per sample or by Hardy-Weinberg equilibrium filtering.



### Step 5: Phasing

__Prerequisite__  
- `/mnt/stsi/stsi0/raqueld/1000G/map/genetic_map_GRCh37_merged.txt.gz`
- `/mnt/stsi/stsi0/raqueld/HRC/HRC.r1-1.EGA.GRCh37.chr$mychr.haplotypes.bcf`
- `/mnt/stsi/stsi0/raqueld/1000G/ALL.chr$mychr.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.bcf`

```
qsub 5_phase.job -v myinput=/stsi/raqueld/N_tests/aric_genotypes-black/aric_genotypes-black.lifted_NCBI36_to_GRCh37.GH.chr1.bed,myoutdir=/stsi/raqueld/5_N_tests,reftype=HRC -N 5_N_mesa_genotypes-black
```

>the imput must have the suffix \*.lifted\*.chr1.bed, \*lifted\*.chr2.bed, \*.lifted\*.chr3.bed, etc. The previous steps in the pipeline generate those suffixes automatically, but keep these suffixes in mind if you are running this step as a stand alone tools, without running the previous steps


### Step 6: Imputation and post-imputation quality control

__Prerequisite__  
- `/mnt/stsi/stsi0/raqueld/HRC/HRC.r1-1.EGA.GRCh37.chr$mychr.haplotypes.m3vcf.gz`
- `/mnt/stsi/stsi0/raqueld/1000G/ALL.chr$mychr.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.m3vcf.gz`

```
qsub 6_impute.job -v myinput=/stsi/raqueld/5_N_tests/mesa_genotypes-white/mesa_genotypes-white.lifted_NCBI36_to_GRCh37.GH.chr18.phased.vcf.gz,myoutdir=/stsi/raqueld/6_N_tests,reftype=HRC -N 6_mesa_genotypes-white.lifted_NCBI36_to_GRCh37.GH.chr18
```
> if running this step as stand alone tool, input file must have the suffix .lifted_\*.chr\*.phased.vcf.gz, otherwise the pipeline wont work, if you use the previous step to generate this input file, then it will work fine.
date

---

## Running all steps automatically 

This script will setup job dependencies and submit/monitore all the jobs for all the steps automatically.

```
bash genotype_imputation_distributor.sh

 ####################################
 ##                                ##
 ##    Imputation / QC Pipeline    ##
 ##          Torkamani Lab         ##
 ##                                ##
 ##         Author: Raquel Dias    ##
 ##                 Shaun Chen     ##
 ##  Last modified: 12/27/19       ##
 ##                                ##
 ####################################

Usage:    bash script.sh --vcf --out --ref --start --end (--confirm) > LOG

          script.sh      This script
          --vcf -v [STR]      Full path of the input vcf file to be QCed/imputed
          --out -o [STR]      Path of the directory where all the output folders will be created
          --ref -r [STR]      Imputation reference panel (HRC or 1000G)
          --start -s [INT]    First step
          --end -e [INT]      Last step
          --temp -t [STR]     (Optional) Enable larger temp storage in step2 (or use PBSTMPDIR scratch folder)
          --wgs -w            (Optional) Enable variant down-sampling in step3 for WGS/imputed data
          --confirm -c        (Optional) Initiate working mode
          LOG                 (Optional) Log report file name

Prepare input:  for chrom in {1..22}; do printf "${chrom}\t$(ls $(pwd)/[##inprefix##]*chr${chrom}.vcf.gz)\n"; done > [##inprefix##].txt

Debug Example:  bash script.sh --vcf /mnt/stsi/stsi0/raqueld/vcf/SHARE_MESA_c2_flipfix.vcf --out /mnt/stsi/stsi0/raqueld --ref HRC --start 0 --end 1 > MESA_jobs_c1_0-1.txt
Working Example:  bash script.sh --vcf /mnt/stsi/stsi0/raquel
```

Once you ran the script setting the run variable inside the script as `--confirm`, the script will save all the submited job commands and job IDs into a log file that you can use for debugging and locating your results.

## Running job in parallel mode

Use the following command:
```
for chrom in {1..22}; do printf "${chrom}\t$(ls $(pwd)/[##inprefix##]*chr${chrom}.vcf.gz)\n"; done > [##inprefix##].txt
```

To prepare an txt input taken by `--vcf` as:
```
1	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr1.vcf.gz
2	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr2.vcf.gz
3	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr3.vcf.gz
4	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr4.vcf.gz
5	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr5.vcf.gz
6	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr6.vcf.gz
7	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr7.vcf.gz
8	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr8.vcf.gz
9	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr9.vcf.gz
10	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr10.vcf.gz
11	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr11.vcf.gz
12	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr12.vcf.gz
13	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr13.vcf.gz
14	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr14.vcf.gz
15	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr15.vcf.gz
16	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr16.vcf.gz
17	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr17.vcf.gz
18	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr18.vcf.gz
19	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr19.vcf.gz
20	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr20.vcf.gz
21	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr21.vcf.gz
22	/mnt/stsi/stsi0/sfchen/UKBB/split_QC/ukbb_hap_v2_000/ukb_hap_v2_000_chr22.vcf.gz
```

## Contact information

  - Raquel Dias – [@RaquelDiasSRTI](https://twitter.com/RaquelDiasSRTI) – raqueld@scripps.edu  
  - Shaun Chen - [@ShaunFChen](http://twitter.com/ShaunFChen) - sfchen@scripps.edu  



## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b feature/fooBar`)
3. Commit your changes (`git commit -am 'Add some fooBar'`)
4. Push to the branch (`git push origin feature/fooBar`)
5. Create a new Pull Request
