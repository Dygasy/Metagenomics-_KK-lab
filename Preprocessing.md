# Welcome to Metagenomics Analysis Workflow

## Workflow Diagram
Below is the workflow diagram for the metagenomics analysis process:

![Workflow Diagram](Initial%20Workflow%20Diagram.png)


---

This repository provides scripts and pipelines for analyzing metagenomics data. The workflow integrates tools for preprocessing, assembly, annotation, and visualization, tailored for gut microbiome research.

## Table of Contents
1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Workflow](#workflow)
4. [Preprocessing Stage](#Preprocessing-stage)
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
```

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

```bash
base_dir="/mnt/e/X401SC24100039-Z01-F001_01/01.RawData"

for folder in "$base_dir"/AA_*; do
    echo "Processing folder: $folder"
    
    # Find FASTQ or gzipped FASTQ files
    find "$folder" -type f \( -name "*.fq" -o -name "*.fastq" -o -name "*.fq.gz" -o -name "*.fastq.gz" \) | while read -r fastq_file; do
        echo "Found: $fastq_file"

        # If file is gzipped, decompress it
        if [[ "$fastq_file" == *.gz ]]; then
            echo "Decompressing: $fastq_file"
            gunzip "$fastq_file"
        fi

        # Optionally convert to FASTA
        decompressed_file="${fastq_file%.gz}"  # Remove .gz extension
        echo "Processing: $decompressed_file"
        seqkit fq2fa "$decompressed_file" -o "${decompressed_file%.fq}.fa"
    done
done

echo "All FASTQ files processed!"
```

#### 4. **Move the decompress files**

files can be too large and will require a bigger storage, that is why i am moving mine over to storage DATA:(E) that has 10tb storage

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
Download readfq.py 
```bash
 wget -O /mnt/e/X401SC24100039-Z01-F001_01/01.RawData/readfq/readfq.py https://raw.githubusercontent.com/lh3/readfq/master/readfq.py
```
Open the file using text editor:
```bash
nano /mnt/e/X401SC24100039-Z01-F001_01/01.RawData/readfq/readfq.py
```
Ensure the first line reads:
def readfq(fp):  # this is a generator function

If any unexpected characters are present (e.g., / before def), remove them.
Locate all lines with print statements (e.g., print n, '\t', slen, '\t', qlen) and modify them like this:
```bash
print(n, '\t', slen, '\t', qlen)
```
Make the script executable again:
```bash
chmod +x /mnt/e/X401SC24100039-Z01-F001_01/01.RawData/readfq/readfq.py
```
The following script ensures only the original .fq files are processed and skips those already processed:
```bash

#!/bin/bash

# Base directory containing the .fq files
base_dir="/mnt/e/X401SC24100039-Z01-F001_01/01.RawData"

# Path to the readfq script
readfq_script="$base_dir/readfq/readfq.py"

# Iterate through each .fq file in the directory
find "$base_dir" -type f -name "*.fq" | grep -v "_processed.fq" | while read -r fq_file; do
    # Define the output file
    output_file="${fq_file%.fq}_processed.fq"

    # Skip if the output file already exists
    if [[ -f "$output_file" ]]; then
        echo "Skipping $fq_file as $output_file already exists."
        continue
    fi

    echo "Processing $fq_file -> $output_file"

    # Run the Python script
    python3 "$readfq_script" < "$fq_file" > "$output_file"

    # Check for errors
    if [[ $? -ne 0 ]]; then
        echo "Error processing $fq_file. Exiting."
        exit 1
    fi
done

echo "All files processed!"
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

Test it out on one smaple to test for bowtie2 alignment:
```bash
bowtie2 -x /mnt/e/X401SC24100039-Z01-F001_01/01.RawData/bowtie2_index/GRCh38_noalt_as \
    -1 /mnt/e/X401SC24100039-Z01-F001_01/01.RawData/AA_0141/AA_0141_DKDN240043855-1A_22TMJFLT3_L1_1.fq \
    -2 /mnt/e/X401SC24100039-Z01-F001_01/01.RawData/AA_0141/AA_0141_DKDN240043855-1A_22TMJFLT3_L1_2.fq \
    -S /mnt/e/X401SC24100039-Z01-F001_01/01.RawData/AA_0141/output.sam
```

Convert and Sort BAM
If output.sam is successfully generated, proceed to convert it to BAM and sort:

```bash
samtools view -bS /mnt/e/X401SC24100039-Z01-F001_02/01.RawData/AA_0008/output.sam | \
samtools sort -o /mnt/e/X401SC24100039-Z01-F001_02/01.RawData/AA_0008/sorted_output.bam
```
Index the BAM File

```bash
samtools index /mnt/e/X401SC24100039-Z01-F001_02/01.RawData/AA_0008/sorted_output.bam
```
Extract Non-Human Reads by generating the non_human_cleaned.fq file:

```bash
samtools fastq -f 12 -F 256 /mnt/e/X401SC24100039-Z01-F001_02/01.RawData/AA_0008/sorted_output.bam > \
/mnt/e/X401SC24100039-Z01-F001_02/01.RawData/AA_0008/non_human_cleaned.fq
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

**3.Post-Processing using DIAMOND**
We will be using NCBI's nr database, input the metagenomic reads to produce an output DIAMOND results. Filter the weak matches and report only the best match.

Download the nr directly from NCBI FTP.
```bash
  wget -c ftp://ftp.ncbi.nlm.nih.gov/blast/db/FASTA/nr.gz -P /mnt/e/Krona_results/
gunzip -c /mnt/e/Krona_results/nr.gz > /mnt/e/Krona_results/nr.fasta
diamond makedb --in nr -d nr 
```
third step for diamond makedb --in nr -d nr --> this is to convert to Diamond-compatible database, generating nr.dmnd

The codes above downloads the latest nr database from NCBI, decompresses the file and then converts nr into a DIAMOND compatible format.

however, generating the NCBI nr database is not enough as it only contains the protein sequences. it does not contain taxonomy information like taxonomic identifiers (TaxIDs). DIAMOND requires taxonomic data to assign taxonomic classifcations to your sequences. 
 
```bash
mkdir -p /mnt/e/Krona_results/nr_db/taxonomy
cd /mnt/e/Krona_results/nr_db/taxonomy
```
## Download taxonomy nodes, names, and taxon mapping
```bash
wget ftp://ftp.ncbi.nih.gov/pub/taxonomy/accession2taxid/prot.accession2taxid.gz
wget ftp://ftp.ncbi.nlm.nih.gov/pub/taxonomy/taxdump.tar.gz
```
prot.accession is for DIAMOND to link protein sequences with TaxIDs.
taxdump is for DIAMOND to interpret the taxonomic relationships. 

## Extract the files
```bash
gunzip prot.accession2taxid.gz
tar -xvf taxdump.tar.gz
```
DIAMOND requires your taxonomy files and database to be together during database creation. Hence, you need to rebuild your nr.dmnd with taxonomy information to generate: nr.fasta

## Next up: Run DIAMOND with Taxonomy
DIAMOND BLASTX to generate diamond_output.daa but first you will convert nr.fasta into diamond database **with taxonomy**
```bash
diamond makedb \
    --in /mnt/e/Krona_results/nr.fasta \
    -d /mnt/e/Krona_results/nr/nr_tax.dmnd \
    --taxonmap /mnt/e/Krona_results/nr_db/taxonomy/prot.accession2taxid \
    --taxonnodes /mnt/e/Krona_results/nr_db/taxonomy/nodes.dmp \
    --taxonnames /mnt/e/Krona_results/nr_db/taxonomy/names.dmp
```

Clean all FASTA Files in both groups (eg: above_ids_group): this is to avoid possibility of tab characters (\t) in the seqeunce titles (headers) of your FASTA file. DIAMOND expects FASTA headers to be free from special characters like tabs, as this can cause parsing issues. We do not want it to affect any accuracy of downstream analysis, especially when linking taxonomic information. 

```bash
for fasta in /mnt/e/Krona_results/above_ids_group/AA_*/input_above.fasta; do
    sed -i 's/\t/ /g' "$fasta"
    echo "Cleaned tabs in $fasta"
done
```


Now run DIAMOND BLASTX once you've created nr_tax.dmnd
```bash
#!/bin/bash
for dir in /mnt/e/Krona_results/above_ids_group/AA_*; do
    sample=$(basename $dir)
    diamond blastx -d /mnt/e/Krona_results/nr/nr_tax.dmnd \
        -q $dir/input_above.fasta \
        -o $dir/diamond_output_above.daa \
        --evalue 0.001 --max-target-seqs 1 --outfmt 100
done
```

Now process the DIAMOND output for taxonomic classification, producing an .rma6 file needed for MEGAN:
```bash
daa-meganizer -i /mnt/e/Krona_results/above_ids_group/AA_0008/diamond_output_above.daa \
              --mapDB /mnt/e/Krona_results/nr_db/megan/megan-map-Feb2022.db/megan-map-Feb2022.db \
              -t 100
```
OR 
```bash
mnt/e/Krona_results/nr_db/megan/tools$ /mnt/e/Krona_results/nr_db/megan/tools/daa2rma \
  -i /mnt/e/Krona_results/above/AA_0009/diamond_output_above.daa \
  -o /mnt/e/Krona_results/above/AA_0009/diamond_output_above.rma6 \
  --mapDB /mnt/e/Krona_results/nr_db/megan/megan-map-Feb2022.db \
  -t 100
```
OR
```bash
daa2rma -i /mnt/e/Krona_results/above/AA_0009/diamond_output_above.daa \
              -o /mnt/e/Krona_results/above/AA_0009/diamond_output_above.rma6 \
              --mapDB /mnt/e/Krona_results/nr_db/megan/megan-map-Feb2022.db/megan-map-Feb2022.db \
              -t 100
```

Visualise in MEGAN and you can explore the taxonomic assignments
Load the .rma6 file into MEGAN for visualisation:
```bash
megan -i /mnt/e/Krona_results/nr_db/diamond_output.rma6
```

Click on Tree --> Taxonomy to see the hierarchical classification of your sequences. You can expand nodes to see how many reads are assigned to different taxa(phylum, genus, species).
Adjust and set the minimum support filter to only include taxa with significant read counts. Fine tune your LCA Parameters to improve taxonomic resolution.

Do Functional database through the MEGANization step (like KEGGm eggNOG) 
Click on Tree --> Functional to explore functions such as metabolic pathways, enzyme activities


Use of GTDB: produce more consistency across genus/species, allow to do Krona plots, able to integrate with tools like PICRUSt2, GTDB-tk or QIIME2.
- based on genome-resolved taxonomy, highly curated, consistent, modern classification
-ideal for genome-centric metagenomics 
-Works well with Kraken2 + Bracken + MetaPhlAn, of GTDB-tk outputs



## Generate Krona HTML Plots 
import os
import re

# === Input & Output Paths ===
input_file = "comparison_cleaned_absolute.txt"
output_dir = "krona_inputs_with_environmental"
os.makedirs(output_dir, exist_ok=True)

# === Read and Prepare ===
with open(input_file, "r", encoding="utf-8") as f:
    lines = f.readlines()

header = lines[0].strip().split("\t")
sample_ids = header[1:]

# === Create Output Files ===
output_files = {
    sample: open(os.path.join(output_dir, f"{sample}.krona.txt"), "w")
    for sample in sample_ids
}

# === Process Each Line ===
for line in lines[1:]:
    parts = line.strip().split("\t")

    # Restore full taxonomy (keep environmental and angle-bracket tags)
    taxonomy_raw = parts[0]
    taxonomy_clean = taxonomy_raw.replace(",", " ")  # ONLY replace commas (Krona can't parse tabs inside fields)

    raw_tax = taxonomy_clean.strip().split(";")
    taxonomy = raw_tax[:8]  # keep up to 8 ranks (strain-level), no padding if fewer

    # Write Krona-compatible output per sample
    for i, count in enumerate(parts[1:]):
        sample = sample_ids[i]
        try:
            c = float(count)
            if c > 0:
                output_files[sample].write(f"{c}\t" + "\t".join(taxonomy) + "\n")
        except ValueError:
            continue

# === Close All Files ===
for f in output_files.values():
    f.close()

print(f"✅ Krona input files saved in: {output_dir}")
