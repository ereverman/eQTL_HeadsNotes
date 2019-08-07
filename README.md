# eQTL_HeadsNotes, also followed for eQTL_Midgut

Heads: Started with 94 samples (188 files) from plate 1; 96 samples from plate 2, NextSeq HO-PE37 with 1% PhiX and using 1-mismatch demultiplexing. Plate 2 was also sequenced MO because of underclustering issues.

Midgut: 96 samples from plates 3 and 4.

## Project was initiated with Atom: eQTLCuAdult

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
  * Heads_config.sh: establishes the directory structure
  * generate_JOBARRAY_input.sh: genetates a file with path to data
  * Heads_ARRAY_sjmac.sh: array file that references config and path file.

3. In terminal, on hpc,run config file in existing eQTL_Heads directory.

```
sh Heads_config.sh
```
  * move the Reference file to the /refs/ directory.
  * move data to the data combined directory
  
```
# from teminal on local computer

scp ./*.gz hpc:/panfs/pfs.local/scratch/sjmac/e284e911/eQTL_Midgut/data/combined
```

4. Combine data files from the run and re-runs of plate 2

There are two runs of plate 2 because of an underclustering issue on the original HO run. There are two resulting directories have exactly the same named 192 files that need to simply be concatenated and sent to a new directory. 

```
# This is from Boryana--didn't work because the files were contatenated twice
# The code has echo training wheels on (lol)

for file in ERE-TruSeq-P2_NextSeqHO_Run1/*_001.fastq.gz
do
  for RN in "R1" "R2"
  do
    theName=$(basename ${file})
    sample=${theName%_S*_*.fastq.gz}
    # show me the files that will be concatenated
    echo $(ls ERE-TruSeq-P2_NextSeq{HO,MO}_*/${sample}*_${RN}_001.fastq.gz)
    # construct the name of the combined file
    newName="${sample}_${RN}_comb.fastq.gz"
    # show me the name of the combined file
    echo "${newName}"
    # show me the command that will be run to combine the files
    echo "zcat ERE-TruSeq-P2_NextSeq{HO,MO}_*/${sample}_*_${RN}_001.fastq.gz >> data/combined/${newName} "
    echo "---------" # this is just to makew it more readable in terminal
  done
done

# This is the version that ended up working:

for file in ERE-TruSeq-P2_NextSeqHO_Run1/*R1_001.fastq.gz
do
  for RN in "R1" "R2"
  do
    theName=$(basename ${file})
    sample=${theName%_R*_001.fastq.gz}
    # show me the files that will be concatenated
    echo $(ls ERE-TruSeq-P2_NextSeq{HO,MO}_*/${sample}*_${RN}_001.fastq.gz)
    # construct the name of the combined file
    newName="${sample}_${RN}_009.fastq.gz"
    # show me the name of the combined file
    echo "${newName}"
    # show me the command that will be run to combine the files
    zcat ERE-TruSeq-P2_NextSeq{HO,MO}_*/${sample}_${RN}_001.fastq.gz >> data/combined/${newName}
    echo "---------" # this is just to makew it more readable in terminal
  done
done


```

5. Move the files from the other run to the combined directory:

```
cp ERE-TruSeq-P1/*.gz data/combined/

```

6. Check the number of files in the combined directory:

```
ls | wc -l

# should be 380
```

7. Run generate_JOBARRAY_input.sh from main eQTL_Heads directory
  * Remember that if running PE data, there should only be one line per SAMPLE, not FILE in this. So, just pull out all of the R1 files for example.

```
sh generate_JOBARRAY_input.sh <PROJECTNAME> <PATH_TO_FILES>

# Check the file with nano or less

nano Heads_sample_file_paths_ARRAY.txt

# Ctrl + X to exit
```

8. fastp quality filtering for paired end data followed by Kallisto pseudoalignment
 * used ARRAY method.
 * check that the first job runs, then do the rest.
 * needed to update fastp. To do this, I activated the conda environment for fastp and updated. Didn't work the first time, did the second time for unknown reasons.
 
 ```
 source activate eQTL_Heads_fastp
 
 conda update fastp
 # to check version
 fastp --version
 ```
 
```
sbatch --array [1] PATH_TO/Heads_ARRAY_sjmac.sh

# Check queue
sq

sbatch --array [2-190] PATH_TO/Heads_ARRAY_sjmac.sh
```

## Clust Analysis:

1. Data was analyzed using sleuth in R. R script file is RNAseqDE_Heads.R in the ML_eQTLCuAdult project directory.

2. To analyze kallisto data with Clust, we need the target_id, sample, and TPM data output using the kal.table function. These data are already normalized, but they will be normalized again in clust.

3. Open terminal and run

```
# create conda environment if not already done:
# conda create -c bioconda -n ClustEnv clust

# Go to environment in the directory with tpm data
source activate ClustEnv

clust tpm.data.txt -n 101 3 4 -o clust_output_Heads

```

4. Make nicer plots in R.


