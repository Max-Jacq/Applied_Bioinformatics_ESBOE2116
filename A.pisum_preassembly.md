# 0.Requirements
First, we will download the .sra file associated with DRR504715, which is associated with the sequencing (Nanopore) of the *A.pisum* genome harbouring the male-killing *Spiroplasma* endosymbiont.
Here is the NCBI link: 

```
#!/bin/bash
#SBATCH --job-name=DRR504715
#SBATCH --time=48:00:00
#SBATCH --mem=64G
#SBATCH --cpus-per-task=8
#SBATCH --output=DRR504715.out
#SBATCH --error=DRR504715.err

module load releases/2020b
module load SRA-Toolkit/2.10.9-gompi-2020b


prefetch DRR504715

# Fast conversion to FASTQ
fasterq-dump DRR504715 --split-files --threads 8
```

# 1.Quality Check
FastQC produces a visual report (HTML) describing the quality of the reads: quality per base, GC content, presence of adapters, duplication levels, etc. <br>
This report helps you decide whether trimming is necessary and which parameters to use. <br>
You can find more information on the following website: https://olvtools.com/en/documents/fastqc <br>

```
#!/bin/bash
#SBATCH --job-name=fastqc
#SBATCH --time=01:00:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=2
#SBATCH --mem=2G
#SBATCH --partition=batch
#SBATCH --output=fastqc_%j.out
#SBATCH --error=fastqc_%j.err

module purge
module load releases/2022b
module load FastQC/0.11.9-Java-11

# Folders
INPUT_DIR=$PWD

# Results folder
OUTDIR=FastQC_results
mkdir -p "$OUTDIR"

# Run FastQC on all FASTQ files in the folder
fastqc -t 2 -o "$OUTDIR" *.fastq *.fastq.gz
```
