By: Karin Isaev 

Started June 17th, 2020

### HPC tutorial 

Please go through this tutorial before proceeding with cluster instructions:
https://hpc-carpentry.github.io/hpc-intro/

###### Jobs submission

- Keep in mind how much memory your job may require and which node you should request

- Some tasks require a lot of temporary memory (sorting FASTQ files for example), in these cases make sure to set your own temporary folder rather than using the deafult which is shared by cluster users, you can do this by adding these lines in your code:

```bash
export TMPDIR=/cluster/some_directory
export TEMP=/cluster/some_directory
export TMP=/cluster/some_directory
```

- Usually, you will submit a ".sh" script to the cluster that will do some job. These types of scripts have these lines in the very beginning:

```bash
#!/bin/bash
#
#SBATCH -N 1 # Ensure that all cores are on one machine
#SBATCH -p himem #<-- here using the himem node because using 60GB memory
#SBATCH -c 6
#SBATCH --mem=60440M #<-- memory we want to allocate for this job
#SBATCH -t 5-00:00 # Runtime in D-HH:MM
#SBATCH -J index # <-- name of job
```

- Once you have a script with these lines in the very beginning, you can submit it

- Choose directory where you want to submit it

- Submit it:

  -  `sbatch path_to_code`
  - in terminal you can then write `squeue` to view your job's status

- What if you have 100 VCF files and you want to apply the same script to each VCF file?

  - Ideally you want to submit one job per VCF file so they run in parallel

  - There are several options for doing this

  - I have found the easiest way to do this is to use 'job arrays'. What does this mean? You first make a file that lists all your VCF files

  - For example let's say you are in a directory with your vcf files, you can do this

    -  `ls *.vcf > all_vcf_files.txt`

  - You can submit your job on every entry in the list of VCF files

  - This can look like this:

```bash
#!/bin/bash
#
#SBATCH -N 1 # Ensure that all cores are on one machine
#SBATCH -p himem
#SBATCH -c 6
#SBATCH --mem=20000M
#SBATCH -t 5-00:00 # Runtime in D-HH:MM
#SBATCH -J annovar #<-- name of the job
#SBATCH -o %j.out			# File to which standard output will be written
#SBATCH -e %j.err			 # File to which standard errors will be written
#SBATCH --array=0-135 # job array index - number of jobs = number of samples in your list of VCF files for example (total number of jobs that will be run)

#load all modules required for analysis

module load annovar
module load bam-readcount
module load gcc
module load STAR

#go into directory with your VCF files

cd /cluster/projects/directory_w_vcf_files

ls *.vcf > all_vcf_files.txt

samples=all_vcf_files.txt
names=($(cat $samples)) #this will save a list of all VCF files
echo ${names[${SLURM_ARRAY_TASK_ID}]} #this will show one sample from the list

sample=${names[${SLURM_ARRAY_TASK_ID}]} #save that sample name as the variable for this script on which everything will be run
echo $sample

#first keep only variants that passed baseline encoded variants already done
awk -F '\t' '{if($0 ~ /\#/) print; else if($7 == "PASS") print}' ${sample}.vcf > ${sample}.clean.vcf
```

  - Once you submit this job, it will actually initate a job for each sample in the list and you will end up submitting 136 jobs in this case (first job is index 0)

  - Alternatively you can write a script with a for-loop that iterates through each sample and submits a job on it

###### Managing raw sequencing data

- Data from sequencing facilities gets uploaded to our cluster project space
- It is important that raw data such as FASTQ/BAM files remain as "read only"
- Symlinks can be useful
- Make sure to note the genome build that you used for a given project or that was provied by sequencing facilities

### Code/Github

- You can make a personal repository and keep your code up-to-date there or join an existing repository for a particular project
- [Team Collaboration with Github](https://code.tutsplus.com/articles/team-collaboration-with-github--net-29876)
- [Reproducible Programming for Biologists Who Code](https://ben-heil.github.io/2020-06-16-mustdo/)
- [More documentation on how to use git](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
- The list of tools I like to use together are:
  - iTerm + Atom text editor + github integration
- [Getting started with Unix](https://seankross.com/the-unix-workbench/)

### R resources

- [Computational Genomics with R](https://compgenomr.github.io/book/)
- [Bayesian Statistics with R](https://statswithr.github.io/book/the-basics-of-bayesian-statistics.html#bayes-rule)
- Useful R packages you should check out 
  - [ggpubr](https://github.com/kassambara/ggpubr) - makes plotting life so much easier for basic tasks

### Statistics

- [Science Forum: Ten common statistical mistakes to watch out for when writing or reviewing a manuscript](https://elifesciences.org/articles/48175?utm_source=twitter&utm_medium=social)
- [Avoiding common pitfalls when clustering biological data](https://stke.sciencemag.org/content/9/432/re6)
- [Common statistical tests are linear models (or: how to teach stats)](https://lindeloev.github.io/tests-as-linear/)

### Additional links

- [Next generation sequencing blog](http://enseqlopedia.com/)
