# Genome Mapping Using Bowtie2 and Samtools

Suppose we have obtained single-end or paired-end reads from an experiment, which are basically short fragments of DNA. We need to map them back to the original genome. How do we do that? Through genome mapping techniques.

First, we need the single/paired-end reads (usually stored in `.gz` files). We also need a reference genome (usually in FASTA or GFF format).

Let’s say we have an organism called *E. coli*, and we also have reads tracing back to it. You might ask why it's necessary. There are several reasons:

1. **Localization**: Sometimes we know the reads belong to a certain organism, but we don't know where those reads exactly come from. It's like searching for a specific sentence (reads) in a book (genome).
2. **Variation Detection**: Sometimes we're looking for differences. Tracing the reads back to the genome may help us detect if there's been a mutation (insertion/deletion/etc.) in the organism.
3. **Functional Insight**: GFF files are the annotated version of FASTA files. They determine which parts are coding regions, genes, RNAs, and such. Using GFF files, we can determine if the reads overlap with genes, exons, or regulatory regions.

---

## Requirements

Install the necessary tools:

```bash
sudo apt install bowtie2 igv samtools openjdk-11-jdk
```

---

## Downloading Files

### Reads:

[ENA SRX012992](https://www.ebi.ac.uk/ena/browser/view/SRX012992?show=reads)

### Reference Genome:

[NCBI NC\_012967](https://www.ncbi.nlm.nih.gov/nuccore/NC_012967)

---

## Setting Up Directories

```bash
mkdir mapping
# Place your reads here
ls mapping
# Output:
SRR030257_1.fastq.gz SRR030257_2.fastq.gz

mkdir ref
# Place your reference genome files here
ls ref
# Output:
Ecoli.fasta Ecoli.gff3

mkdir index
```

---

## Indexing the Reference Genome

```bash
bowtie2-build ref/Ecoli.fasta index/Ecoli
```

---

## Mapping Reads to the Genome

The next step is to map the paired-end reads to the reference genome and save the output in SAM format. SAM format is human-readable and includes important information such as alignment quality and mismatches.

**Example SAM line:**

```
read1    0    NC_000913.3    1    60    5M    *    0    0    AGCTG    *    NM:i:0
```

```bash
bowtie2 -x index/Ecoli -1 SRR030257_1.fastq.gz -2 SRR030257_2.fastq.gz -S mapped/Ecoli.sam --threads 8
```
-x: Prefix of index files.

-1, -2: Paired-end read files.

-S: Output SAM file.

--threads: Number of CPU threads to use.

---

## Converting and Sorting

After producing the SAM files, we convert convert them to BAM format. BAM files are the binary version of SAM files and contain the same information but are compressed for faster storage and access. The format is not human-readable:

samtools view -b -o mapped/Ecoli.bam mapped/Ecoli.sam

Convert SAM to BAM (binary format):

```bash
samtools view -b -o mapped/Ecoli.bam mapped/Ecoli.sam
```
Sorting the BAM file by genomic position improves downstream performance and efficient processing:

```bash
samtools sort -@ 8 -o mapped/Ecoli.sorted.bam mapped/Ecoli.bam
```

Example (unsorted → sorted):

```
read3, mapped to position 100
read1, mapped to position 50
read2, mapped to position 75

# becomes:
read1, mapped to position 50
read2, mapped to position 75
read3, mapped to position 100
```

---

## Indexing the Sorted BAM File

Indexing allows quick access to specific genomic regions without having to access the entire BAM file:

```bash
samtools index mapped/Ecoli.sorted.bam
```

## Visualization
The last step is to visualize the BAM file by running igv in the terminal.

```bash
igv.sh
```

Once igv opens, in the top pannel, we select 'Genomes' and load the Ecoli.fasta file. Then, from the 'File' tab we load the Ecoli.sorted.bam file. You can identify the following factors through visualization in igv:

![IGV](igv(1).png)
![IGV](igv(2).png)

- **Gray bars**: mapped reads.

- **Colored letters**: mismatches between reads and reference.

- **Asterisks (*)**: protein-level changes due to mutations.

- **Coverage bars on top**: height indicates read depth.
