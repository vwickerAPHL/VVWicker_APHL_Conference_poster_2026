# GSU Viral Pathogen Detection NextFlow Pipeline


<!-- 
[![GitHub Actions CI Status](https://github.com/VVwickerAPHL/vsp/actions/workflows/nf-test.yml/badge.svg)](https://github.com/VVwickerAPHL/vsp/actions/workflows/nf-test.yml) -->
[![GitHub Actions Linting Status](https://github.com/VVwickerAPHL/vsp/actions/workflows/linting.yml/badge.svg)](https://github.com/VVwickerAPHL/vsp/actions/workflows/linting.yml)[![Cite with Zenodo](http://img.shields.io/badge/DOI-10.5281/zenodo.XXXXXXX-1073c8?labelColor=000000)](https://doi.org/10.5281/zenodo.XXXXXXX)
<!-- [![nf-test](https://img.shields.io/badge/unit_tests-nf--test-337ab7.svg)](https://www.nf-test.com) -->

[![Nextflow](https://img.shields.io/badge/version-%E2%89%A524.10.5-green?style=flat&logo=nextflow&logoColor=white&color=%230DC09D&link=https%3A%2F%2Fnextflow.io)](https://www.nextflow.io/)
[![nf-core template version](https://img.shields.io/badge/nf--core_template-3.3.2-green?style=flat&logo=nfcore&logoColor=white&color=%2324B064&link=https%3A%2F%2Fnf-co.re)](https://github.com/nf-core/tools/releases/tag/3.3.2)
[![run with conda](http://img.shields.io/badge/run%20with-conda-3EB049?labelColor=000000&logo=anaconda)](https://docs.conda.io/en/latest/)
[![run with docker](https://img.shields.io/badge/run%20with-docker-0db7ed?labelColor=000000&logo=docker)](https://www.docker.com/)
[![run with singularity](https://img.shields.io/badge/run%20with-singularity-1d355c.svg?labelColor=000000)](https://sylabs.io/docs/)
<!-- [![Launch on Seqera Platform](https://img.shields.io/badge/Launch%20%F0%9F%9A%80-Seqera%20Platform-%234256e7)](https://cloud.seqera.io/launch?pipeline=https://github.com/VVwickerAPHL/vsp) -->

## Interested in using this pipeline?
Hello! This page is linked to Vaughn Wicker's poster at the 2026 APHL conference. If you are interested in using this pipeline, please email him at Vaughn.Wicker@dallascounty.org for access to the full repository. Thank you!

## Introduction

**GSU Viral Pathogen Detection** is a bioinformatics nf-core style NextFlow pipeline developed by Vaughn Wicker, an APHL Bioinformatics Research Fellow hosted by the Genome Sequencing Unit (GSU) at the Dallas County Health and Human Services (DCHHS) Public Health Laboratory. This pipeline analyzes clinical NGS data from Illumina and Oxford Nanopore Technologies (ONT) protocols to detect viral pathogens, generate assemblies, and perform phylogenetic analysis. At the end of the pipeline the outputs are summarized in a standard MultiQC report and a custom RMarkdown report. 

<!-- TODO nf-core:
   Complete this sentence with a 2-3 sentence summary of what types of data the pipeline ingests, a brief overview of the
   major pipeline sections and the types of output it produces. You're giving an overview to someone new
   to nf-core here, in 15-20 seconds. For an example, see https://github.com/nf-core/rnaseq/blob/master/README.md#introduction
-->
![GSU-Viral-Pathogen-Detection](./docs/images/NF-GSU-Render.png) 

<!-- TODO nf-core: Include a figure that guides the user through the major workflow steps. Many nf-core
     workflows use the "tube map" design for that. See https://nf-co.re/docs/guidelines/graphic_design/workflow_diagrams#examples for examples.   -->
<!-- TODO nf-core: Fill in short bullet-pointed list of the default steps in the pipeline -->
The following are the major tools used for each phase of the pipeline:
1. **Quality Control, Filtering, and Taxonomic Classification**
   1. Fastp
   2. Kraken2/KrakenTools
   3. Seqkit
   4. Custom Module - Pull references from NCBI
   
2. **Assembly**
   1. *de novo* Assembly
      1. MEGAHIT
      2. BWA
   2. Reference Mapping
      1. Minimap2
      2. Bamtools
      2. iVar

3. **Phylogenetic Analysis and Reporting**
   1. Pilon
   2. Nextclade
   3. MultiQC
   4. Custom Module - R Markdown Report

## Quickstart
For users already familar with Nextflow, clone the repository, download the kraken2 database (MinusB),and extract the default kraken2 database into the pipeline directory (GSU-Viral-Pathogen-Detection/viral_db/minusB): 
```bash
git clone https://github.com/vwickerAPHL/GSU-Viral-Pathogen-Detection.git
cd GSU-Viral-Pathogen-Detection/viral_db
wget https://genome-idx.s3.amazonaws.com/kraken/k2_minusb_20260226.tar.gz -O minusB.tar.gz
tar -xvzf minusB.tar.gz
```
Then, proceed to usage instructions below.


## Detailed Setup and Installation Instructions

Before downloading the pipeline, please note the following dependencies and requirements for running the pipeline: 
1. **Command Line Interface (CLI)**

   >Set up your prefferred Command Line Interface and ensure you are using a linux-like environment (e.g. for Windows, have WSL activated)
   >*Recommended*: Install a package manager to handle dependencies, such as [conda](https://docs.conda.io/en/latest/)

2. **(Optional): Create a conda environment with the provided [.yml](docs/GSU-VPD.yaml) file**
   > The following command can be used to create the conda environment:
   ```bash
   conda env create -f GSU-VPD.yaml -n <ENV_NAME>
   ```

2. **Install Docker (or Singularity)**
   >Docker is recommended because the pipeline was developed and tested using a Docker profile. Singularity *should* work, but please document an issue if you encounter any difficulty.

3. **Install NextFlow and nf-core**
   >*Note:* If you are new to Nextflow and nf-core, please refer to [this page](https://nf-co.re/docs/usage/installation) on how to set-up Nextflow. Make sure to [test your setup](https://nf-co.re/docs/usage/introduction#how-to-run-a-pipeline) with `-profile test` before running the workflow on actual data.

4. **Clone the repository**
```bash
git clone https://github.com/vwickerAPHL/GSU-Viral-Pathogen-Detection.git
```

5. **Kraken2 Database**
   > This pipeline utilizes Kraken2 for taxonomic classification. Note, the default pipeline uses the [MinusB database](https://benlangmead.github.io/aws-indexes/k2), maintained by Ben Lanmead. **After cloning the repository**, get the tar.gz archive and extract the archive. Ensure that the datapath is correct (./GSU-Viral-Pathogen-Detection/viral_db/minusB).

```bash
cd GSU-Viral-Pathogen-Detection
wget https://genome-idx.s3.amazonaws.com/kraken/k2_minusb_20260226.tar.gz -O viral_db/minusB.tar.gz
tar -xvzf viral_db/minusB.tar.gz
```
   >*Note:* The user may supply a local kraken2 database in the run command (see advanced usage below). Utilizing a different database will **dramatically** affect the behavior of the pipeline. It is recommended to run the pipeline with the MinusB database first, then use other databases if there is a **large proportion of unclassified reads** in the final report. 


## Usage

First, prepare a samplesheet with your input data that is structured as follows:

Paired End Reads:
`samplesheet.csv`:

```csv
sample,fastq_1,fastq_2
SAMPLE-ID,SAMPLE-ID_S1_L002_R1_001.fastq.gz,SAMPLE-ID_S1_L002_R2_001.fastq.gz
```

Single End Reads:
`samplesheet.csv`:

```csv
sample,fastq
SAMPLE-ID,SAMPLE-ID.fastq.gz
```

Each row represents a fastq file (single-end) or a pair of fastq files (paired end).



Now, you can run the pipeline using:

<!-- TODO nf-core: update the following command to include all required parameters for a minimal example -->

```bash
nextflow run GSU-Viral-Pathogen-Detection/ \
   -profile <docker/singularity/.../institute> \
   --input samplesheet.csv \
   --outdir <OUTDIR>
```

> [!WARNING]
> Please provide pipeline parameters via the CLI or Nextflow `-params-file` option. Custom config files including those provided by the `-c` Nextflow option can be used to provide any configuration _**except for parameters**_; see [docs](https://nf-co.re/docs/usage/getting_started/configuration#custom-configuration-files).

## Common Usage
Additional Parameters are outlined below to adjust the behavior of the pipeline

|Parameter|Description|Accepted Values|
|:---|:---|:---|
|Single-End Read Analysis (ONT)| By default, the pipeline expects paired-end reads (Illumina data, or similar). Turn on Single-End Read analysis with this parameter | --ont_reads
|Classification Database|A kraken2 database, either a custom or [standard database](https://benlangmead.github.io/aws-indexes/k2)| --kraken2_db <*db_path*>|
|References|One .fastq file with all references to map reads to (e.g. one .fastq file with Multiple Viral pathogen genomes)|--reference *<ref.fastq>*|
|Nextclade Analysis| Specifically for HIV and Mpox (all clades). These parameters run pathogen-specific Nextclade analysis and, in some cases, additional custom analysis. *To add additional pathogens, initate a pull request or open an issue on this repository.* | --mpxv <br>--hiv

## Glossary

## Credits

The GSU Viral Pathogen Detection Pipeline was originally written by Vaughn Wicker.

<!-- We thank the following people for their extensive assistance in the development of this pipeline: -->

<!-- TODO nf-core: If applicable, make list of people who have also contributed -->

## Contributions and Support
The development of this pipeline was supported by the APHL Fellowship Program in conjuction with the Dallas County Health and Human Services (DCHHS) Public Health Laboratory.


If you would like to contribute to this pipeline, please see the [contributing guidelines](.github/CONTRIBUTING.md).

## Citations

<!-- TODO nf-core: Add citation for pipeline after first release. Uncomment lines below and update Zenodo doi and badge at the top of this file. -->
<!-- If you use VVwickerAPHL/vsp for your analysis, please cite it using the following doi: [10.5281/zenodo.XXXXXX](https://doi.org/10.5281/zenodo.XXXXXX) -->

<!-- TODO nf-core: Add bibliography of tools and data used in your pipeline -->

An extensive list of references for the tools used by the pipeline can be found in the [`CITATIONS.md`](CITATIONS.md) file.

This pipeline uses code and infrastructure developed and maintained by the [nf-core](https://nf-co.re) community, reused here under the [MIT license](https://github.com/nf-core/tools/blob/main/LICENSE).

<!-- > **The nf-core framework for community-curated bioinformatics pipelines.**
>
> Philip Ewels, Alexander Peltzer, Sven Fillinger, Harshil Patel, Johannes Alneberg, Andreas Wilm, Maxime Ulysse Garcia, Paolo Di Tommaso & Sven Nahnsen.
>
> _Nat Biotechnol._ 2020 Feb 13. doi: [10.1038/s41587-020-0439-x](https://dx.doi.org/10.1038/s41587-020-0439-x). -->
