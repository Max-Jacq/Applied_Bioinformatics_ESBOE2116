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

## 1.1.Expected results

At the end of the process, Flye generates several files in the output directory: <br>

  -`assembly.fasta`: main file containing the assembled contigs
  
  -`graph.gfa`: repetition graph used by Flye
  
  -`repeat_graph` and `assembly_info.txt`: information on assembly and detected repetitions

# 2.Verification of assembly quality

## 2.1.Local installation of Seqkit
```
#From your first path, we will create the bin folder.
mkdir -p ~/bin
wget https://github.com/shenwei356/seqkit/releases/download/v2.6.1/seqkit_linux_amd64.tar.gz
tar -xzf seqkit_linux_amd64.tar.gz
mv seqkit ~/bin/
export PATH=$HOME/bin:$PATH
```
## 2.2.Flye assembly stats
```
seqkit stats assembly.fasta
```

## 2.3.Expected results
```
file            format  type  num_seqs      sum_len  min_len    avg_len    max_len
assembly.fasta  FASTA   DNA      2,793  527,410,710      514  188,833.1  7,185,291
```
Biological interpretation
  -**num_seqs = 2,793**. Flye generated a large number of contigs. For a eukaryotic genome (such as A. pisum, 500–540 Mb), this is not surprising with unpolished long reads and imperfect coverage. Small contigs (<10 kb) are often repetitive fragments or errors.

  -**sum_len = 527 Mb**. Very close to the expected size of the aphid genome (~517–540 Mb depending on the strain). So Flye has assembled almost the entire genome.

  -**min_len = 514 bp**. Many small fragments, typical of repeats, transposons, or assembly errors.

  -**avg_len = 188 kb**. The majority of contigs are still quite long, which is good for a long-read assembly.

  -**max_len = 7.18 Mb**. A very long contig, probably a large part of a chromosome or a contig containing several scaffolds concatenated by Flye.

  # 3.Assessment of genome completeness with BUSCO
After assembling a genome, it is essential to evaluate its quality and completeness. To do this, we use BUSCO (**Benchmarking Universal Single-Copy Orthologs**), a tool that measures the presence of universal and conserved genes in a given taxonomic group.

BUSCO provides an estimate of genome completeness by categorizing the universal genes detected as:

  -**Complete** (C): gene present and complete
  
  -**Single-Copy** (S): present only once
  
  -**Duplicated** (D): present multiple times
  
  -**Fragmented** (F): gene partially found
  
  -**Missing** (M): gene absent

```
#!/bin/bash
#SBATCH --job-name=busco_spiro
#SBATCH --time=12:00:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --partition=batch
#SBATCH --output=busco_%j.out
#SBATCH --error=busco_%j.err
#SBATCH --mail-user=maxence.jacquet@unamur.be

## --- Clean environment ---
module purge
module load releases/2022b
module load BUSCO

# --- Input assembly ---
ASSEMBLY=/home/maxenjac/results/flye/spiroplasma/assembly.fasta

# --- Output directory ---
OUTDIR=/home/maxenjac/results/busco/spiroplasma

# --- BUSCO parameter ---
THREADS=8
LINEAGE=insecta_odb10

# LINEAGE = un ensemble de gène universels et conserves qu'on s'attend à rtrouver une seule fois dans tous les génomes d'un groupe donné.

# --- Create output directory ---
mkdir -p ${OUTDIR}

busco -i ${ASSEMBLY} -o busco_spiroplasma -l ${LINEAGE} -m genome -c ${THREADS} --out_path ${OUTDIR}
```

## 3.1.Expected results

At the end of the run, BUSCO produces several files in the output directory:

  -`short_summary.*.txt`: quick summary with percentages of complete, fragmented, and missing genes
  
  -`full_table.*.tsv`: complete list of BUSCO genes found, with their status
  
  -`run_*/`: folder containing intermediate results

## 3.2.Our results
```
# BUSCO version is: 5.4.7
# The lineage dataset is: insecta_odb10 (Creation date: 2024-01-08, number of genomes: 75, number of BUSCOs: 1367)
# Summarized benchmarking in BUSCO notation for file /home/maxenjac/results/flye/spiroplasma/assembly.fasta
# BUSCO was run in mode: euk_genome_met
# Gene predictor used: metaeuk

        ***** Results: *****

        C:91.2%[S:87.0%,D:4.2%],F:3.1%,M:5.7%,n:1367
        1246    Complete BUSCOs (C)
        1189    Complete and single-copy BUSCOs (S)
        57      Complete and duplicated BUSCOs (D)
        43      Fragmented BUSCOs (F)
        78      Missing BUSCOs (M)
        1367    Total BUSCO groups searched

Assembly Statistics:
        2793    Number of scaffolds
        2793    Number of contigs
        527410710       Total length
        0.000%  Percent gaps
        819 KB  Scaffold N50
        819 KB  Contigs N50


Dependencies and versions:
        hmmsearch: 3.3
        bbtools: 39.01
        metaeuk: GITDIR-NOTFOUND
        busco: 5.4.7

```
