# 0.Requirements
First, we will download the .sra file associated with DRR504715, which is associated with the sequencing (Nanopore) of the *A.pisum* genome harbouring the male-killing *Spiroplasma* endosymbiont. <br>
Here is the NCBI link: https://www.ncbi.nlm.nih.gov/sra/DRR504715 <br>

```
#!/bin/bash
#SBATCH --job-name=SRR11853141
#SBATCH --time=48:00:00
#SBATCH --mem=64G
#SBATCH --cpus-per-task=8
#SBATCH --output=SRR11853141.out
#SBATCH --error=SRR11853141.err

module load releases/2020b
module load SRA-Toolkit/2.10.9-gompi-2020b

# ---SCRATCH---
SCRATCH=/scratch/maxenjac/$SLURM_JOB_ID
mkdir -p "$SCRATCH"
cd "$SCRATCH"

prefetch SRR11853141

# ---Fast conversion to FASTQ---
fasterq-dump SRR11853141 --split-files --threads 8


# ---Copy the final files to the permanent folder already existing via prefetch---
mv SRR11853141*.fastq /home/maxenjac/scripts/SRR11853141/
```
This method also allows downloading, but you would not use it in the current context. <br>
```
wget https://ftp.ncbi.nlm.nih.gov/sra/sra-instant/reads/ByRun/sra/DRR/DRR504/DRR504715/DRR504715.sra
```

# 1.Quality Check
FastQC produces a visual report (HTML) describing the quality of the reads: quality per base, GC content, presence of adapters, duplication levels, etc. <br>
This report helps you decide whether trimming is necessary and which parameters to use. <br>
You can find more information on the following website: https://olvtools.com/en/documents/fastqc <br>

```
#!/bin/bash
#SBATCH --job-name=fastqc_DRR504715
#SBATCH --time=06:00:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=16G
#SBATCH --partition=batch
#SBATCH --output=fastqc_%j.out
#SBATCH --error=fastqc_%j.err
#SBATCH --mail-user=maxence.jacquet@unamur.be
#SBATCH --mail-type=END,FAIL

# ---Load the necessary modules---
module purge
module load releases/2022b
module load FastQC/0.11.9-Java-11

# ---Creating the discharge file---
OUTDIR=/home/maxenjac/results/FastQC/DRR504175
mkdir -p "$OUTDIR"

 # ---FastQC analysis of the .fastq file---
fastqc -t 8 -o "$OUTDIR" /home/maxenjac/data/Spiroplasma/DRR504715.fastq
```

# 2.Adapter trimming
## Why trimming is important in NGS data analysis

Raw sequencing reads often contain technical artifacts such as adapter sequences, low-quality bases at the read ends, and sequencing errors. If these issues are not removed, they can negatively affect downstream analyses, including read mapping, genome assembly, and variant calling. Trimming tools like Porechop are used to clean raw FASTQ files by detecting and removing adapters, trimming low-quality regions, and discarding reads that are too short after processing. Porechop is particularly well suited for long-read sequencing data, such as those produced by Oxford Nanopore or PacBio, where adapters can appear internally or in complex patterns. This preprocessing step improves the overall quality of the data, increases mapping accuracy, reduces false positives, and leads to more reliable biological results.

## 2.1. Porechop
```
#!/bin/bash
#SBATCH --job-name=porechop
#SBATCH --time=12:00:00
#SBATCH --mem=32G
#SBATCH --cpus-per-task=2
#SBATCH --mail-user=maxence.jacquet@unamur.be
#SBATCH --mail-type=ALL


module purge
module load releases/2021b
module load Porechop/0.2.4-GCCcore-11.2.0

# ---Go to the job scratch---
cd "$SLURM_TMPDIR" || exit 1

# ---Copy the necessary files to scratch---
cp /home/maxenjac/ESBOE2116/data/rawdata/DRR504715.fastq .

# ---Launch Porechop---
porechop -i DRR504715.fastq -o DRR504715.trim.fastq -t 2

# ---Copy the final result to the permanent folder---
cp DRR504715.trim.fastq /home/maxenjac/ESBOE2116/data/rawdata/
```
> [!TIP]
> The modules may not load correctly. Type the following commands outside of the scripts. <br>
```
module load releases/2021b

module load Porechop/0.2.4-GCCcore-11.2.0

#The following command will show you a path that confirms porechop has been loaded.
which porechop
```
## 2.2. Skewer
Skewer is a bioinformatics tool used to clean high-throughput sequencing data, particularly short reads from technologies such as Illumina. It detects and removes artificial adapters added during sequencing, filters or truncates low-quality bases, and processes both single-end and paired-end reads. This cleaning step is essential for short reads, which are more susceptible to errors and adapter contamination. By removing these artifacts and improving read quality, Skewer ensures more reliable results for subsequent analyses, such as reference genome alignment, variant detection, expression quantification, or de novo assembly. In summary, Skewer prepares short reads to be clean and of high quality, which is essential for accurate and robust bioinformatic analyses.

### 2.2.1. Installing Skewer in a personal folder (if the tool is not present in the clusters)
**Préparer `~/bin`** <br>
```
mkdir -p ~/bin
echo $PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

**Download Skewer** <br>
```
wget https://github.com/relipmoc/skewer/archive/refs/heads/master.zip
unzip master.zip
cd skewer-master
```

**Load the compatible compiler** <br>
```
module purge
module load GCC/7.1.0-2.28
#Then, do a check
```

**Move Skewer in `~/bin`** <br>
```
cp skewer ~/bin/

#And, check it out
which skewer
skewer --help
```

### 2.2.2. Launch skewer
```
#!/bin/bash
#SBATCH --job-name=Skewer
#SBATCH --time=06:00:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=64G
#SBATCH --mail-user=maxence.jacquet@unamur.be
#SBATCH --mail-type=ALL
#SBATCH --partition=batch
#SBATCH --output=skewer_output.txt
#SBATCH --error=skewer_error.txt

set -euo pipefail

# ---Scratch---
SCRATCH="$HOME/scratch/$SLURM_JOB_ID"
mkdir -p "$SCRATCH"
cd "$SCRATCH"

# ---Skewer is in ~/bin → no modules required--- 
skewer -t 8 -o /home/maxenjac/data/s_kunkelii/ /home/maxenjac/data/s_kunkelii/SRR9495982.fastq
```

# 4.Quality filtering

## Why using fastqc after trimming is important in data analysis
After trimming, it is essential to run FastQC again in order to evaluate the quality of the processed reads and to verify that the trimming step was effective. The second FastQC report allows us to confirm that adapter sequences have been successfully removed, that low-quality bases at the read ends have been reduced, and that overall read quality has improved. By comparing FastQC reports before and after trimming, we can directly assess the impact of trimming and ensure that the cleaned reads are suitable for downstream analyses such as read alignment, genome assembly, or variant calling.

# Optional question (with the `mt_reads.fastq` file or not)

Q1) Which of the following statements is correct?

The reads passed all of the quality control measures <br>
The reads failed all of the quality control measures <br>
The reads passed some of the quality control measures but failed others <br>

Q2) The per-base sequence quality is lowest at the___of the reads.

Q3) Most of the reads were assigned a quality (Phred) score of____.

Q4) Examine the per)base sequence content. The base composition is unusual/unexpected for position____to position_____of the reads.

Q5) This unexpected composition may due to the inclusion of____.
