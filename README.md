# eQTL_HeadsNotes

Started with 192 genotypes. 94 samples (188 files) from plate 1; 96 samples from plate 2, NextSeq HO-PE37 with 1% PhiX and using 1-mismatch demultiplexing.

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
    sample=${theName%_S*_*.fastq.gz}
    # show me the files that will be concatenated
    echo $(ls ERE-TruSeq-P2_NextSeq{HO,MO}_*/${sample}*_${RN}_001.fastq.gz)
    # construct the name of the combined file
    newName="${sample}_${RN}_comb.fastq.gz"
    # show me the name of the combined file
    echo "${newName}"
    # show me the command that will be run to combine the files
    zcat ERE-TruSeq-P2_NextSeq{HO,MO}_*/${sample}_*_${RN}_001.fastq.gz >> data/combined/${newName}
    echo "---------" # this is just to makew it more readable in terminal
  done
done

```

5. Move the files from the other run to the combined directory:

```
cp cp ERE-TruSeq-P1/*.gz data/combined/

```

6. Check the number of files in the combined directory:

```
ls | wc -l

# should be 380
```

7. Run generate_JOBARRAY_input.sh from main eQTL_Heads directory

```
sh generate_JOBARRAY_input.sh <PROJECTNAME> <PATH_TO_FILES>

# Check the file with nano

nano Heads_sample_file_paths_ARRAY.txt

# Ctrl + X to exit
```

8. fastp quality filtering for paired end data followed by Kallisto pseudoalignment
 * used ARRAY method.
 * check that the first job runs, then do the rest.
 
```
sbatch --array [1] PATH_TO/Heads_ARRAY_sjmac.sh

# Check queue

sbatch --array [2-3] PATH_TO/Heads_ARRAY_sjmac.sh
```


