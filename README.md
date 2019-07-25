# eQTL_HeadsNotes

192 samples, NextSeq HO-PE37 with 1% PhiX and using 1-mismatch demultiplexing.

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

4. Run generate_JOBARRAY_input.sh

```
sh generate_JOBARRAY_input.sh <PROJECTNAME> <PATH_TO_FILES>
```

5. fastp quality filtering for paired end data.
 * used ARRAY method.
 * check that the first job runs, then do the rest.
 
```
sbatch --array [1] PATH_TO/Heads_ARRAY_sjmac.sh

sbatch --array [2-3] PATH_TO/Heads_ARRAY_sjmac.sh
```


