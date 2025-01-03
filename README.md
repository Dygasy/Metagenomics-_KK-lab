# Welcome to Metagenomics Analysis Workflow

## Workflow Diagram
Below is the workflow diagram for the metagenomics analysis process:

![Workflow Diagram](./Initial%20workflow%20diagram%20_%20MTTSH-2024-12-09-043159%20(1).png)

---

This repository provides scripts and pipelines for analyzing metagenomics data. The workflow integrates tools for preprocessing, assembly, annotation, and visualization, tailored for gut microbiome research.

## Table of Contents
1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Workflow](#workflow)
4. [Preprocessing Stage](#Preprocessing Stage)
5. [Results](#results)
6. [References](#references)
7. [Contributing](#contributing)

---

## Introduction
Metagenomics involves the study of genetic material recovered directly from environmental samples. This workflow is designed to process gut microbiome data, with a focus on **[specific goals, e.g., taxonomic profiling or functional annotation].

## Preprocessing Stage

### Overview
The preprocessing stage ensures raw sequence reads are filtered and prepared for downstream analysis. This includes quality control, host genome removal, and file format processing.

---

### Steps for Preprocessing

#### 1. **System Setup**
- **Operating System**: Linux (Ubuntu)
- **User**: `bulat@495-9H56S54`
- **Environment**: Conda with Python 3.9 (Miniconda recommended)

Commands to set up the environment:
```bash
conda create -n metagenomics_env python=3.9
conda activate metagenomics_env

#### 2. **Verify Raw Sequence Files**
``` bash 
file X401SC24100039-Z01-F001_02.tar or 03.tar
```
To extract the archive: 

``` bash
tar -xvf X401SC24100039-Z01-F001_02.tar
```
#### 3. **Automate Extraction of Nested Archives (folders that are compressed to be extracted)**
```bash
#!/bin/bash
extract() {
    for file in "$@"; do
        case "$file" in
            *.tar.gz|*.tgz) tar -xzvf "$file" ;;
            *.tar.bz2) tar -xjvf "$file" ;;
            *.tar.xz) tar -xJvf "$file" ;;
            *.tar) tar -xvf "$file" ;;
            *.zip) unzip "$file" ;;
            *.gz) gzip -d "$file" ;;
            *.bz2) bzip2 -d "$file" ;;
            *) echo "Unknown file type: $file" ;;
        esac
    done
}
extract *
```

#### 4. **Move the decompress files**

files can be too large and will require a bigger storage, that is why i am moving mine over to storage DATA:(E)

```bash
cd /mnt/e

mv ~/X401SC24100039-Z01-F001_02 /mnt/e
```
Decompress all .gz files in the raw data directory:
```bash
cd /mnt/e/X401SC24100039-Z01-F001_02/01.RawData
for dir in */; do
    find "$dir" -type f -name "*.gz" -exec gzip -d {} +
done
```
#### 5. **Perform Quality control and filtering**

Difference between readfq and Bowtie2

Purpose of readfq is to provide summary statistics for reads (sequence count, total sequence/quality lengths)
Purpose of bowtie2 is to align reads to a reference genome to remove host DNA.

**Processing of raw files with readfq**
```bash
for file in $(find /mnt/e/X401SC24100039-Z01-F001_02/01.RawData -type f -name "*.fq"); do
    output_file="${file%.fq}_processed.fq"
    python3 /mnt/e/X401SC24100039-Z01-F001_02/01.RawData/readfq/readfq.py < "$file" > "$output_file"
    echo "Processed $file -> $output_file"
done
```
#### 6. **Install Bowtie2 and Reference Genome**

```bash
sudo apt update
sudo apt install bowtie2
bowtie2 --version
```
Download GRCh28 no-alt-analysis from the Bowtie2 manual (https://bowtie-bio.sourceforge.net/bowtie2/manual.shtml)

Place the .bt2 index files in the appropriate folder and verify: 
```bash
ls /mnt/e/X401SC24100039-Z01-F001_02/01.RawData
```
#### 7. **Run Bowtie2 for Host Genome Removal**

Test it out on one smaple to test:
```bash
bowtie2 -x /mnt/e/X401SC24100039-Z01-F001_02/01.RawData/bowtie2_index/GRCh38_noalt_as \
-1 /mnt/e/X401SC24100039-Z01-F001_02/01.RawData/AA_0001/AA_0001_L4_1.fq \
-2 /mnt/e/X401SC24100039-Z01-F001_02/01.RawData/AA_0001/AA_0001_L4_2.fq \
-S /mnt/e/X401SC24100039-Z01-F001_02/01.RawData/AA_0001/output.sam
```

## Assembly using MEGAHIT
### Overview
After preprocessing, we proceed to the assemnbly step to reconstruct genomic sequences using **MEGAHIT**

Install MEGAHIT on ubuntu:
```bash
sudo apt install megahit
```
Try running it on single samples (paired-end non-human-cleaned FASTQ files)
```bash
megahit -1 /mnt/e/X401SC24100039-Z01-F001_03/01.RawData/AA_0126/non_human_cleaned_1.fq \
        -2 /mnt/e/X401SC24100039-Z01-F001_03/01.RawData/AA_0126/non_human_cleaned_2.fq \
        -o /mnt/e/X401SC24100039-Z01-F001_03/01.RawData/AA_0126/assembly_output
```
each sample will take approximately 3 hours to process. that is why we are going to process multiple folders in parallel:

**Install GNU Parallel**
```bash
sudo apt-get install parallel
```
**Set up and output directory:**
```bash
mkdir -p /mnt/e/MEGAHIT_results
```
**Generate a job list for MEGAHIT**
```bash
find /mnt/e/X401SC24100039-Z01-F001_02/01.RawData/AA_* \
    -type d | while read folder; do
    sample_name=$(basename $folder)
    fq_file=$(find $folder -type f -name "*.fq" | grep -E "non_human_cleaned.fq")
    if [ -n "$fq_file" ]; then
        echo "megahit -r $fq_file -o /mnt/e/MEGAHIT_results/${sample_name} --num-cpu-threads 4 --memory 28000"
    else
        echo "echo 'No non-human cleaned FASTQ file found in $folder. Skipping...'"
    fi
done > megahit_job_list.txt
```
**Proceeed to run the jobs in parallel**
```bash
parallel -j 4 < megahit_job_list.txt
```
We do face some samples that have missing results. All you need to do is rerun Bowtie2, extract non-human reads, and repeat MEGAHIT process for those folders. 
To avoid overwritign, MEGAHIT skips completed results unless explicitly specified with -o.

## Gene Analysis Stage using MetaGeneMark
### Overview 
The gene analysis process begins with **MetaGeneMark**, a tool for gene prediciton in metagenomic sequences. Below is a step-by-step guide to process your samples using the MetaGeneMark web interface. 

###Steps to Process Samples###
**1. Input the Final Contigs File**
   * Use the final contigs (.fa) file obtained from the assembly step.
   * Navigate to the MetaGeneMark official website interface
   * Under the **Input Sequence** section:
   * upload the file
**2.Configure Options:**
* **Output Format for Gene Prediciton**: Select GFF
GFF format provides detailed annotations, which are useful for downstream analysis. such as visualisation or functional annotation.

* **Output Options:**
* Check the boxes for:
  1. **Protein Sequences (outputs a .faa file)
  2. **Gene Nucleotide Sequences) (output a .fnn file)
**3.Start Processing**
* Click on the **Start GeneMark.hmm** button to begin the analysis.

**4.Download the results:**
* Once processing is complete, the output page will display links to the result files
   * gmhmmp.out: Coordinates of predicted genes
   * gmhmmp.out.faa: Predicted protein sequences
   * gmhmmp.out.gnn: Gene nucleotides sequences
* Manually download each file by clicking the respective links
**5.Organise the results**
 * Save the downloaded files into the appropriate folder (eg,., MEGAHIT_results/AA_0004), ensuring proper organisation for downstream analysis 

Manual downloads ensure better control over the extraction process, avoiding complications with compressed files. tried it and it didnt work. 

### Prerequisites
- Python (version >= 3.9)
- Miniconda
- Docker (optional)
- Tools used:
  - readfq,bowtie2
  - MEGAHIT
  - MetaGeneMark
  - Trimmomatic
  - Kraken2
  - SPAdes
  - Prokka

### Installation
1. Clone the repository:
   ```bash
   git clone https://github.com/dygasy/metagenomics-workflow.git
   cd metagenomics-workflow
   ```
