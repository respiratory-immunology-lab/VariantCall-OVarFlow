Using the VariantCall-OVarFlow pipeline for variant calling
===========================================================

This pipeline is based on OVarFlow pipeline, which provides a Snakemake-based automated pipeline derived from the GATK Best Practice workflow for “Germline short variant discovery”. This pipeline has been optimised to work on M3 cluster. Check the original [paper](https://www.biorxiv.org/content/10.1101/2021.05.12.443585v1) for more information.

## Installing OVarFlow on the cluster

```
# Create a new conda environment
conda create -n OVarFlow
conda activate OVarFlow

# Get dependencies and add to environment
wget https://gitlab.com/computational-biology/ovarflow/-/raw/master/OVarFlow_src/OVarFlow_dependencies_mini.yml
conda env update --file OVarFlow_dependencies_mini.yml
rm -rf  OVarFlow_dependencies_mini.yml

# Create a new project directory and download files
mkdir [var_calling_project]
cd [var_calling_project]
wget https://gitlab.com/computational-biology/ovarflow/-/raw/master/OVarFlow_src/Snakefile
wget https://gitlab.com/computational-biology/ovarflow/-/raw/master/OVarFlow_src/samples_and_read_groups.csv
wget https://gitlab.com/computational-biology/ovarflow/-/raw/master/OVarFlow_src/config.yaml

# Create a scripts directory and download scripts for execution
mkdir scripts
cd scripts 
wget https://gitlab.com/computational-biology/ovarflow/-/raw/master/OVarFlow_src/scripts/average_coverage.awk
wget https://gitlab.com/computational-biology/ovarflow/-/raw/master/OVarFlow_src/scripts/createIntervalLists.py
chmod +x average_coverage.awk
chmod +x createIntervalLists.py
```

## Databases

A FASTA reference genome as well as an annotation file are required for the pipeline. They need to be placed in the `REFERENCE_INPUT_DIR`.

```
cd REFERENCE_INPUT_DIR
wget http://ftp.ensembl.org/pub/release-98/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa.gz
wget http://ftp.ensembl.org/pub/release-98/gff3/homo_sapiens/Homo_sapiens.GRCh38.98.gff3.gz
```

## Preparing your project

The first step is to create a CSV file with some information about the samples and reference files.

```
# Activate environment
source activate OVarFlow

# Create CSV template
snakemake --configfile config.yaml --cluster "sbatch --job-name=OVarFlow --account=of33 --time=01:00:00 --mem-per-cpu=1G --ntasks=1 --cpus-per-task=1 --partition=genomics --qos=genomics" -j 2 -w 30 -p create_CSV_template

# To get sample name quickly
for f in *_R1.fastq.gz; do Basename=${f%_merged_*}; echo $Basename; done
```

The second step is to prepare the reference genome database.

```
snakemake --configfile config.yaml --cluster "sbatch --job-name=OVarFlow --account=of33 --time=04:00:00 --mem-per-cpu=24G --ntasks=1 --cpus-per-task=6 --partition=genomics --qos=genomics" -j 2 -w 30 -p create_snpEff_db
```

## Running OVarFlow

These are the parameters for using the `--partition=genomics --qos=genomics` partition on the cluster.

```
# Step 1: QC
snakemake --configfile config.yaml --cluster "sbatch --job-name=OVarFlow_QC --account=of33 --time=04:00:00 --mem=30G --ntasks=1 --cpus-per-task=4 --partition=genomics --qos=genomics" -j 50 -w 30 -p QC_all

# Step 2: Mapping
snakemake --configfile config.yaml --cluster "sbatch --job-name=OVarFlow_mapping --account=of33 --time=04:00:00 --mem=120G --nodes=1 --ntasks=1 --cpus-per-task=24 --partition=genomics --qos=genomics" -j 10 -w 30 -p mapping_all

# Step 3: Sorting and duplicates marking
snakemake --configfile config.yaml --cluster "sbatch --job-name=OVarFlow_sortingmarking --account=of33 --time=04:00:00 --mem=120G --nodes=1 --ntasks=1 --cpus-per-task=4 --partition=genomics --qos=genomics" -j 10 -w 50 -p-p sortingmarking_all
```

## Citation

If you used this repository in a publication, please mention its url.

In addition, you may cite the tools used by this pipeline:

* **OVarFlow:** OVarFlow: a resource optimized GATK 4 based Open source Variant calling workFlow
Jochen Bathke, Gesine Lühken
bioRxiv 2021.05.12.443585; doi: https://doi.org/10.1101/2021.05.12.443585

## Rights

* Copyright (c) 2021 Respiratory Immunology lab, Monash University, Melbourne, Australia.
* Authors: C. Pattaroni
