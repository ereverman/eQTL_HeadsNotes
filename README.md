# eQTL_HeadsNotes

# ML_CopperWildPops

Description of set up for analysis of sequencing data. 3 samples, whole genome sequenced: Avg, Cu, SR. PE 150.

## Project was initiated with Atom: DH_2018

1. Open and link new project in Atom:
  * Open new window.
  * File
    * Add Project Folder (Select from directories)
  * Packages
    * Remote-FTP
      * Create config file (copy from existing project)
        * Edit to match path to folder on Cluster
  * Packages
    * Remote-FTP
      * Toggle
  * Connect, enter password

2. Edit and save the following files:
  * DH_2018_config.sh: establishes the directory structure
  * generate_JOBARRAY_input.sh: genetates a file with path to data
  * DH_2018_fastpARRAY.sh: array file that references config and path file.

3. In terminal, on hpc,run config file in existing DH_2018 directory.

```
sh DH_2018_config.sh
```

4. Run generate_JOBARRAY_input.sh

```
sh generate_JOBARRAY_input.sh <PROJECTNAME> <PATH_TO_FILES>
```

5. fastp quality filtering for paired end data.
 * used ARRAY method.
 * check that the first job runs, then do the rest.
 
```
sbatch --array [1] PATH_TO/DH_2018_fastpARRAY.sh

sbatch --array [2-3] PATH_TO/DH_2018_fastpARRAY.sh
```

## First pass on data using Popoolation2

1. Open an interactive node on a remote screen:

```
screen -S <UNIQUE_NAME>
source ~/.bash_profile
```

2. Open an interactive node:

```
srun --time=12:00:00 --ntasks=1 --cpus-per-task=16 --mem=125g --partition=sjmac --pty /bin/bash -l
```

3. Create a conda environment and install packages

```
conda create -n Popoolation2 --yes
source activate Popoolation2
conda install java-jdk --yes
conda install bwa samtools bcftools --yes
```

5. Index the reference genome

```
# Download from flybase:

wget ftp://ftp.flybase.net/releases/FB2018_06/dmel_r6.25/fasta/dmel-all-chromosome-r6.25.fasta.gz

# Index

bwa refs/REFERENCE_GENOME # Generates several files. 
```

6. Map reads to the reference genome with paired-end filtered reads:

```
bwa aln -n 0.01 -l 100 -o 1 -d 12 -e 12 -t 8 refs/REFERENCE_GENOME PATH_TO_DATA/DH2018_Avg.S3.R1.filt.fastq.gz > map/Avg.R1.sai
bwa aln -n 0.01 -l 100 -o 1 -d 12 -e 12 -t 8 refs/REFERENCE_GENOME PATH_TO_DATA/DH2018_Avg.S3.R2.filt.fastq.gz > map/Avg.R2.sai

bwa aln -n 0.01 -l 100 -o 1 -d 12 -e 12 -t 8 refs/REFERENCE_GENOME PATH_TO_DATA/DH2018_Cu.S1.R1.filt.fastq.gz > map/Cu.R1.sai
bwa aln -n 0.01 -l 100 -o 1 -d 12 -e 12 -t 8 refs/REFERENCE_GENOME PATH_TO_DATA/DH2018_Cu.S1.R2.filt.fastq.gz > map/Cu.R2.sai

bwa aln -n 0.01 -l 100 -o 1 -d 12 -e 12 -t 8 refs/REFERENCE_GENOME PATH_TO_DATA/DH2018_SR.S2.R1.filt.fastq.gz > map/SR.R1.sai
bwa aln -n 0.01 -l 100 -o 1 -d 12 -e 12 -t 8 refs/REFERENCE_GENOME PATH_TO_DATA/DH2018_SR.S2.R2.filt.fastq.gz > map/SR.R2.sai

######

bwa sampe refs/dmel-all-chromosome-r6.25.fasta.gz map/Avg.R1.sai map/Avg.R2.sai data/filtered/DH2018_Avg_S3.R1.filt.fastq.gz data/filtered/DH2018_Avg_S3.R2.filt.fastq.gz > map/Avg.sam
#9010 seconds

bwa sampe refs/dmel-all-chromosome-r6.25.fasta.gz map/Cu.R1.sai map/Cu.R2.sai data/filtered/DH2018_Cu.S1.R1.filt.fastq.gz data/filtered/Cu.S1.R2.filt.fastq.gz > map/Cu.sam

bwa sampe refs/dmel-all-chromosome-r6.25.fasta.gz map/SR.R1.sai map/SR.R2.sai data/filtered/DH2018_SR.S2.R1.filt.fastq.gz data/filtered/SR.S2.R2.filt.fastq.gz > map/SR.sam

```

7. Make the bam file 

```
samtools view -q 20 -bS map/Avg.sam | samtools sort - -T Avg_tmp -o map/Avg.bam
samtools view -q 20 -bS map/Cu.sam | samtools sort - -T Cu_tmp -o map/Cu.bam
samtools view -q 20 -bS map/SR.sam | samtools sort - -T SR_tmp -o map/SR.bam
```

8. Mpileup with java:

```
samtools mpileup -B map/Avg.bam map/Cu.bam > AvgVsCu.mpileup 
samtools mpileup -B map/Avg.bam map/SR.bam > AvgVsSR.mpileup 
samtools mpileup -B map/Cu.bam map/SR.bam > CuVsSR.mpileup 

samtools mpileup -B map/Avg.bam map/Cu.bam map/SR.bam > AvgVsCuVsSR.mpileup 

###

java -ea -Xmx50g -jar <popoolation2-path>/mpileup2sync.jar --input AvgVsCu.mpileup --output AvgVsCu.sync --fastq-type sanger --min-qual 20 --threads 8

java -ea -Xmx50g -jar <popoolation2-path>/mpileup2sync.jar --input AvgVsSR.mpileup --output AvgVsSR.sync --fastq-type sanger --min-qual 20 --threads 8

java -ea -Xmx50g -jar <popoolation2-path>/mpileup2sync.jar --input CuVsSR.mpileup --output CuVsSR.sync --fastq-type sanger --min-qual 20 --threads 8

java -ea -Xmx50g -jar <popoolation2-path>/mpileup2sync.jar --input AvgVsCuVsSR.mpileup --output AvgVsCuVsSR.sync --fastq-type sanger --min-qual 20 --threads 8
```

9. Calculate allele frequency differences with popoolation2:

### Change the prefix next time to something more informative. This labels the files output from this command.
```
# Change the min-coverage to allow for lower frequency snps.
perl <popoolation2-path>/snp-frequency-diff.pl --input AvgVsCu.sync --output-prefix DH2018 --min-count 6 --min-coverage 50 --max-coverage 200

# perl <popoolation2-path>/snp-frequency-diff.pl --input AvgVsCu.sync --output-prefix Avg_Cu --min-count 2 --min-coverage 4 # --max-coverage 500

perl <popoolation2-path>/snp-frequency-diff.pl --input AvgVsSR.sync --output-prefix Avg_SR --min-count 6 --min-coverage 50 --max-coverage 200
perl <popoolation2-path>/snp-frequency-diff.pl --input CuVsSR.sync --output-prefix Cu_SR --min-count 6 --min-coverage 50 --max-coverage 200

perl <popoolation2-path>/snp-frequency-diff.pl --input AvgVsCuVsSR.sync --output-prefix Avg_Cu_SR --min-count 6 --min-coverage 50 --max-coverage 200

# outputs: 
# DH2018.params 
# DH2018_pwc; differences in allele frequencies for ecvery pairwise comparison of the populations
# DH2018_rc; major and minor alleles for every SNP
```

10. Fisher's Exact Test: estimate the significance of allele frequency differences:

### This needs to be done out of the conda environment:
```
source deactivate

perl popoolation2_1201/fisher-test.pl --input AvgVsCu.sync --output Avg_Cu.fet --min-count 6 --min-coverage 50 --max-coverage 200 --suppress-noninformative
```
