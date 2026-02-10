# 1.Assembly with Flye
Assembling a genome from long reads (Nanopore or PacBio) is a crucial step in genomic analysis. Here, we use `Flye`, an assembler optimized for long reads, capable of producing high-quality assemblies even with repetitive or fragmented genomes.
`Flye` uses a repeat graph to assemble reads, corrects errors inherent in long-read sequencing technologies, and generates one or more contigs representing the genome of the organism under study. 
In this project, we aim to assemble the genome of *A. pisum* from Nanopore reads pre-processed with Porechop.

```
#!/bin/bash
#SBATCH --job-name=flye_trim_spiro
#SBATCH --time=36:00:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=64G
#SBATCH --partition=batch
#SBATCH --output=flye_%j.out
#SBATCH --error=flye_%j.err
#SBATCH --mail-user=maxence.jacquet@unamur.be

module purge
module load releases/2022b
module load Flye

READS=/home/maxenjac/results/trim_galore/spiroplasma/DRR504715_trimmed.fq
OUTDIR=/home/maxenjac/results/flye/spiroplasma
THREADS=8
GENOME_SIZE=1.8m

mkdir -p ${OUTDIR}

flye --nano-raw ${READS} --genome-size ${GENOME_SIZE} --out-dir ${OUTDIR} --threads ${THREADS}
```

# 1.1.Expected results

At the end of the process, Flye generates several files in the output directory: <br>

  -`assembly.fasta`: main file containing the assembled contigs
  
  -`graph.gfa`: repetition graph used by Flye
  
  -`repeat_graph` and `assembly_info.txt`: information on assembly and detected repetitions
  

  
