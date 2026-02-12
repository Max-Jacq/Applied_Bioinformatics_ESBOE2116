# 0.Requirements
First, we will download the .sra file associated with DRR504715, which is associated with the sequencing (Nanopore) of the *A.pisum* genome harbouring the male-killing *Spiroplasma* endosymbiont. <br>
Here is the NCBI link: https://www.ncbi.nlm.nih.gov/sra/DRR504715 <br>

```
#!/bin/bash
#SBATCH --job-name=PRJEB51899
#SBATCH --time=48:00:00
#SBATCH --mem=64G
#SBATCH --cpus-per-task=8
#SBATCH --output=PRJEB51899.out
#SBATCH --error=PRJEB51899.err

# ---Charge les modules nécessaires---
module purge
module load releases/2020b
module load SRA-Toolkit/2.10.9-gompi-2020b

# ---Téléchargement du fichier de séquençage depuis la base de données SRA (Sequence Read Archive)---
prefetch PRJEB51899

# ---Conversion rapide du fichier SRA en fichier .fastq---
fasterq-dump /home/your_eid/PRJEB51899/ --split-files --threads 8
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

# ---Charge les modules nécessaires---
module purge
module load releases/2022b
module load FastQC/0.11.9-Java-11

# ---Création du dossier de sortie---
OUTDIR=/home/maxenjac/results/FastQC/DRR504175
mkdir -p "$OUTDIR"

 # ---Analyse FastQC du fichier .fastq---
fastqc -t 8 -o "$OUTDIR" /home/maxenjac/data/Spiroplasma/DRR504715.fastq
```

# 2.Adapter trimming
## Why trimming is important in NGS data analysis

Raw sequencing reads often contain technical artifacts such as adapter sequences, low-quality bases at the read ends, and sequencing errors. If these issues are not removed, they can negatively affect downstream analyses, including read mapping, genome assembly, and variant calling. Trimming tools like Trimmomatic are used to clean raw FASTQ files by removing adapters and low-quality regions, and by discarding reads that are too short after trimming. This preprocessing step improves the overall quality of the data, increases mapping accuracy, reduces false positives, and leads to more reliable biological results.

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

# ---Aller dans le scratch du job---
cd "$SLURM_TMPDIR" || exit 1

# ---Copier les fichiers ne�cessaires dans  scratch---
cp /home/maxenjac/ESBOE2116/data/rawdata/DRR504715.fastq .

# ---Lancer Porechop---
porechop -i DRR504715.fastq -o DRR504715.trim.fastq -t 2

# ---Copier le résultat final dans le dossier permanetn---
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
