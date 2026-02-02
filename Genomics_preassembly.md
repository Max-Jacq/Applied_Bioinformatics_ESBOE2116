#  🛠 0.Requirements

## Load the commun "releases" environment
ml releases/2022b

## List loaded modules
module list

## Seach for a module (ex: fastqc)
ml spider fastqc

## Short reminder
Type...

```
pwd
```       
For your current file <br>

```
ls -lh      
```
File list (readable size) <br>

```
cd dir
```
To switch files <br>

 ```
zcat file.fastq.gz | head -n 8
```
See the first 2 reads (of fastq file) <br>

```
less file    
```
Read file bit by bit <br>


# 🔍 1.Quality Check
FastQC produces a visual report (HTML) describing the quality of the reads: quality per base, GC content, presence of adapters, duplication levels, etc. <br>
This report helps you decide whether trimming is necessary and which parameters to use. <br>
You can find more information on the following website: https://olvtools.com/en/documents/fastqc <br>

```
ml releases/2022b
ml MultiQC/1.14-foss-2022b   # Then, MultiQC can be usefull
# If module FastQC is available:
ml FastQC/0.11.9

# lauch FastQC (2 paired-end files)
fastqc sample_R1.fastq.gz sample_R2.fastq.gz -o qc/
```

Explore and inspect the FastQC report for mt_reads.fastq.

Q1) Which of the following statements is correct?

The reads passed all of the quality control measures <br>
The reads failed all of the quality control measures <br>
The reads passed some of the quality control measures but failed others <br>

Q2) The per-base sequence quality is lowest at the _____ of the reads. <br>

Q3) Most of the reads were assigned a quality (Phred) score of _____. <br>

Q4) Examine the per-base sequence content. The base composition is unusual/unexpected for position _____ to position _____ of the reads. <br>

Q5) This unexpected composition may be due to the inclusion of _____. <br>

## Job SLURM (script via nano)

Create the run_fastqc.sbatch file:

nano run_fastqc.sbatch

Then, insert

```
#!/bin/bash
#SBATCH --job-name=fastqc_sample
#SBATCH --output=fastqc_sample.%j.out
#SBATCH --error=fastqc_sample.%j.err
#SBATCH --time=00:30:00
#SBATCH --cpus-per-task=4
#SBATCH --mem=8G

ml releases/2022b
ml MultiQC/1.14-foss-2022b
ml FastQC/0.11.9

mkdir -p qc
fastqc -t 4 sample_R1.fastq.gz sample_R2.fastq.gz -o qc/
```

Then, submit the work:

```
sbatch run_fastqc.sbatch
squeue -u $USER    # vérifier la file d'attente
```

# ✂️ 2.Adapter Trimming

Trim Galore delete:
  -Residual adapters,
  -Low-quality bases at the ends,
  -Reads that are too short after trimming.
  
This improves the quality of the inputs for assembly.

```
# Load
ml releases/2022b
# If available :
ml TrimGalore/0.6.7

# execute (paired-end)
trim_galore --paired sample_R1.fastq.gz sample_R2.fastq.gz -o trimmed/
```

Create your nano file and run it:

```
#!/bin/bash
#SBATCH --job-name=trim_sample
#SBATCH --output=trim_sample.%j.out
#SBATCH --error=trim_sample.%j.err
#SBATCH --time=01:00:00
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G

ml releases/2022b 
ml TrimGalore/0.6.7

mkdir -p trimmed
trim_galore --paired --cores 4 sample_R1.fastq.gz sample_R2.fastq.gz -o trimmed/
```
# 3. Post-trimming

```
#FastQC
fastqc trimmed/*val_*.fq.gz -o qc_trimmed/

#MultiQC (aggregates FastQC reports)
multiqc qc_trimmed/ -o qc_trimmed/
```
