# alignment_snakemake
Pipeline for aligning pair-end cram files to reference genomes on the Cambridge HPC cluster login-icelake.hpc.cam.ac.uk node using Snakemake.


### Dependencies
- snakemake v7.8.5  (It only seems to work for snakemake v7.8.5 and not any later versions!)

To get this working on rds you might have to
1. install python 3.9.6
```
conda install python=3.9.6
```
3. copy this into your .local folder (be aware that this might break/affect other dependencies!)
```
cd .local
cp -r ../rds/rds-durbin-group-8b3VcZwY7rY/software_RHEL8/Bettina_local/* .
```
(you can ignore the permission issues for the other python shared folders)

3. update your .bashrc file by adding the following paths

PATH=$PATH:~/.local/bin

export BCFTOOLS_PLUGINS=~/rds/rds-durbin-group-8b3VcZwY7rY/software_RHEL8/bcftools_1.20/plugins/

4. run source on the .bashrc to take effect

The following software are also installed in ~/rds/rds-durbin-group-8b3VcZwY7rY/software_RHEL8/bin so should be in your path.
- bwa 0.7.18-r1243-dirty
- samtools 1.20-1-g43da234
- crumble 0.9.1  (this requires to load module load htslib/1.14/gcc/wwc5wqv5)


### profile
Update the cluster.yaml file with the full path to the location of the **profile** directory (I found that relative paths do not always work!). Update the config.yaml account to your groups account.

### Required Files:
see example folder
- fofn = tab delimited file with 3 columns: 1. path to the read cram file; 2. primaryID; 3. sampleID (or primaryID).


This will create the outout filepath for e.g. the fAstCal1.2 mapped files as


output_folder/sampleID/fAstCal1.2/primaryID.mem.crumble.cram


### Reference fasta locations on rds
- /rds/project/rd109/rds-rd109-durbin-group/ref/fish/Astatotilapia_calliptera/fAstCal1.2/GCA_900246225.3_fAstCal1.2_genomic_chromnames_mt.fa
- /rds/project/rd109/rds-rd109-durbin-grou/ref/fish/Aulonocara_stuartgranti/fAulStu2/fAulStu2.mm.asm.FINAL.2.mt.fa
- /rds/project/rd109/rds-rd109-durbin-group/ref/fish/Maylandia_zebra/M_zebra_UMD2a/bwa-mem2_index/GCF_000238955.4_M_zebra_UMD2a_genomic_LG.fa

### Update the Snakemake file:
set the following parameters
- user = crsid
- who_wrote_input_files_info = crsid
- path_to_files_info = path to the fofn file (see above)
- output_folder = path to where to write the aligned files
- remap_flag = "no", this assumes that the cram files are unmapped raw pair-end crams, if "yes" then the script will read name sort the already mapped files before aligning them to a new genome reference
- reference = path to the genome reference fasta
- reference_string = short name of the reference, e.g. fAstCal1.2

### Run
open a screen terminal (take a not of the login node so you can attach back to the same later)
```
screen -RD
module purge
module load rhel8/default-icl
module load htslib/1.14/gcc/wwc5wqv5
module load gcc/11.3.0/gcc/4zpip55j
```

To check how many jobs are required and display the shell commands use:
 ```
snakemake -n
```
This will print a DAG table in yellow which states the number of total jobs this run/stage will require.


To run use the full path to the profile folder, and where n is the maximum number of jobs to be submitted in parallel. Our cichlid data requires ~60Gb at the sam stage so I recommend to limit the number of jobs running in parallel to 20:
```
snakemake --profile full_path_to/profile/ --latency 20 --jobs 20
```


