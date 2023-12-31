
## **Standardizing and harmonizing NGS analysis workflows**

This workshop is a part of GCB 2023. It is planned to be 3 hours of an overview of standardization and harmonizing NGS analysis strategies in GHGA. We will explore how FAIR principles enable the standardization and harmonization of nf-core-based NGS analysis workflows within GHGA. We will  demonstrate the adaptability of nf-core workflows and discuss the importance of standardization of workflows. Finally, we will show how to make workflows scalable, robust, and automated using a small subset of a public dataset. 


- *A* 9:00am  - 9:15am  : Introduction to the tutorial: What is GHGA? What are our workflow objectives? What is FAIR data
- *B* 9:15am  - 9:45am  : Reproducibility, adaptability, and portability of workflows
- *C* 9:45am  - 10:15am : Standardization of workflows using Workflow Managers
- *D* 10:15am - 10:30am : Q&A and Break
- *E* 10:30am - 11:00am : Accurate analysis and benchmarking
- *F* 11:00am - 12:00am : Hands-on experience (with minimal test cases provided)


### Learning Objectives for Tutorial

- FAIR principles and their relevance for workflow standardization and harmonization
- The adaptability, portability, and scalability of workflows within nf-core 
- Automating robust workflows to ensure reproducibility and the highest quality of code

### Overview

We will walk through the following steps:
1. Learning how to collaborate using GitHub.
2. Constructing an efficient workstation through Visual Studio Code. 
3. Build and use Docker containers to encapsulate workflow dependencies. 
4. Set up a development environment to run Nextflow
5. Running Sarek nf-core workflow on different data and config files
6. Covering different deployment scenarios
7. Using NCBench to benchmark sarek results
8. Exploring the results

### Requirements

- Bash
- [Java11](https://www.oracle.com/java/technologies/downloads/) (or later, up to 18)
- [Nextflow](https://www.nextflow.io/docs/latest/getstarted.html#installation)
- [nf-core](https://nf-co.re/)
- [GitHub id](https://github.com/)
- [Docker](https://www.oracle.com/java/technologies/downloads/)
- Raw files can be downloaded [here](https://drive.google.com/drive/folders/1OXGIx9RHioH1QB65SK75m_liP_fygxYH?usp=drive_link)

# Construction of a simple alignment and variant calling pipeline using nf-core tools

1. What is **GitHub**, how we can use it?

- fork https://github.com/ghga-de/gcb_workshop
- fork https://github.com/nf-core/testpipeline
  
2.  How to build and use **Docker** containers:

- Download and install Docker [here]([url](https://docs.docker.com/get-docker/))

```
docker login
```

- The pull command lets you download a Docker image without running it. For example

```
docker pull ubuntu:16.04
```

 - Launching a BASH shell in the container allows you to operate in an interactive mode in the containerized operating system. For example

```
docker run -ti ubuntu:16.04
```

- Lets build an image to download and install *bcftools*

```
# Update the package inside the container
apt-get update
# Install the tools we need to download and compile Samtools
apt-get install wget build-essential liblzma-dev libbz2-dev libncurses5-dev zlib1g-dev libcurl4-openssl-dev
# Download Samtools
wget https://github.com/samtools/bcftools/releases/download/1.17/bcftools-1.17.tar.bz2

# Unpack the archive
tar jxf bcftools-1.17.tar.bz2
cd bcftools-1.17
# Compile the code
make
# Install the resulting binaries
make install
#check if everything is properly installed
bcftools --version
#If complete exit docker
exit
```

- Save and upload the docker image.
An example image name could be: kubran/bcftools:v1.17

```
docker commit <containerid> dockerid/imagename:version_tag
```

- Push the image to Dockerhub:

```
docker push dockerid/imagename:version_tag
```

3.  Getting familiar with **Nextflow** and **nf-core**

Complete the installations following the links
- [Nextflow]([url](https://www.nextflow.io/docs/latest/getstarted.html#installation))
- [nf-core]([url](https://nf-co.re/))

```
nextflow info
```

**nf-core** is a community effort to optimize and collect a set of nextflow pipelines. Currently, they have **86** pipelines!

- let's explore **nf-core** tool:

```
nf-core --help
```

- Listing available pipelines:

```
nf-core list
```

- Listing available modules through nf-core:

```
nf-core module list remote
```

4.  Using nf-core modules to build a simple variant calling pipeline

- Let's use **nf-core/testpipeline** as a template. We can use the local fork of the pipeline. 

```
  git clone https://github.com/*userid*/testpipeline.git
```

- We will perform _bwa-mem_ alignment, which requires indexed fasta genome, using _bwa-index_. Thus we need bwa-mem and bwa-index modules. Luckily, nf-core provides bwa modules and we can directly install them!

```
nf-core modules install bwa/mem
nf-core modules install bwa/index

```
Now both modules should be located in **modules/nf-core** directory. 

- In order to use those modules, we will add descriptions to the workflow. 

Add three lines of code to workflows/testpipeline.nf

```Nextflow
/*
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    IMPORT NF-CORE MODULES
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*/

//
// MODULE: Installed directly from nf-core/modules
//
include { BWA_MEM                } from '../modules/nf-core/bwa/mem/main'
include { BWA_INDEX              } from '../modules/nf-core/bwa/index/main'
include { BCFTOOLS_MPILEUP       } from '../modules/nf-core/bcftools/mpileup/main'
```

- This pipeline comes with a ready subworkflow in order to check the input files and create an input channel with them (subworkflows/input_check.nf). It automatically checks and validates the header, sample names, and sample directories. This module is implemented for both pair-end and single-end fastq file processing simultaneously. Since we will also use the same format, we won't change the module and make use of it directly. But, still we need to prepare our samplesheet accordingly to be able to input our files:

Place fastq files (reads) into testpipeline directory and create mysamplesheet.csv file: 


```console
sample,fastq1,fastq2
sample_paired_end,reads/NA12878_75M_Agilent_1.merged.fastq.gz,reads/NA12878_75M_Agilent_2.merged.fastq.gz
sample_single_end,reads/NA12878_75M_Agilent_1.merged.fastq.gz,
```

- In order to perform alignment, we need a fasta file to be indexed. testpipeline includes igenome.config template ready to use. Therefore, we can directly use one of the provided fasta files readily.

Note: [IGenomes](https://emea.support.illumina.com/sequencing/sequencing_software/igenome.html) is a source providing collections of references and annotations supported by AWS.

igenome.config includes parameters for the available sources for the pipeline. Yet, to be able to use them, we need to create a channel for them. Let's create a genome channel for the usage of BWA_INDEX module. 
Below, a channel was already created for input samplesheet.csv file. 

```Nextflow
// Check mandatory parameters
if (params.input) { ch_input = file(params.input) } else { exit 1, 'Input samplesheet not specified!' }

ch_genome_fasta = Channel.fromPath(params.fasta).map { it -> [[id:it[0].simpleName], it] }.collect()
```

- Now, we would only need to add our first module! Checking out BWA_INDEX, the only input file is fasta file to be able to create bwa index directory.

The output index directory will be saved into ch_index channel for further usage. 

```Nextflow
    //
    // MODULE: BWA_INDEX
    //
    BWA_INDEX(
        ch_genome_fasta
    )
    ch_index = BWA_INDEX.out.index
    ch_versions = ch_versions.mix(BWA_INDEX.out.versions)
```

- Next task will be adding BWA_MEM alignment module:
BWA_MEM module requires fastq reads which were already prepared through INPUT_CHECK.
We prepared ch_index channel in the previous step.
"true" statement is a value in order to activate samtools sort for BAM file.

```Nextflow
    //
    // SUBWORKFLOW: Read in samplesheet, validate, and stage input files
    //
    INPUT_CHECK (
        ch_input
    )
    ch_versions = ch_versions.mix(INPUT_CHECK.out.versions)

    //
    // MODULE: BWA_MEM
    //
    BWA_MEM(
        INPUT_CHECK.out.reads,
        ch_index,
        "true"
    )
    ch_bam = BWA_MEM.out.bam
    ch_versions = ch_versions.mix(BWA_MEM.out.versions)
```
- Final task will be calling variants using the bam files generated with BWA_MEM. We will use bcftools mpileup together with bcftools call and bcftools view. Let's create our first module using bcftools container that we created!

A draft BCFTOOLS_MPILEUP module will look like this: We will need to define input and output files together with bcftools commands that will process variant calling. 

Now, save this file as bcftools_mpileup.nf and place under modules/local folder. 

``` Nextflow
process BCFTOOLS_MPILEUP {
    tag "$meta.id"
    label 'process_medium'

    conda ""
    container "${ workflow.containerEngine == 'singularity' && !task.ext.singularity_pull_docker_container ?
        'singularity_container':
        'docker_container' }"

    input:
    INPUT_FILES

    output:
    OUTPUT_FILES
    path  "versions.yml"                 , emit: versions

    when:
    task.ext.when == null || task.ext.when

    script:
    def args = task.ext.args ?: ''
    def prefix = task.ext.prefix ?: "${meta.id}"
    """
    CMD

    cat <<-END_VERSIONS > versions.yml
    "${task.process}":
        CMD_VERSION
    END_VERSIONS
    """
}

```  
Now, we need to construct the simple CMD for variant calling: 

```Nextflow
    bcftools \\
        mpileup \\
        --fasta-ref $fasta \\
        -Oz \\
        $bam \\
        | bcftools call --output-type v -mv -Oz \\
        | bcftools view --output-file ${prefix}.vcf.gz --output-type z

    tabix -p vcf -f ${prefix}.vcf.gz
    bcftools stats ${prefix}.vcf.gz > ${prefix}.bcftools_stats.txt

```
bctools mpileup will produce a mpileup file including genotypes, then we will use bcftools call to actually call variant sites and save as gzipped vcf file. As a plus, we will use bcftools stats to examine the number of variants. 

We will need the alignment bam file and reference fasta file to run bcftools mpileup and this argument will produce an indexed vcf.gz file and a statistic file in txt format. Therefore, input and output definitions will be: 

```Nextflow
    input:
    tuple val(meta), path(bam)
    tuple val(meta2), path (fasta)

    output:
    tuple val(meta), path("*vcf.gz")     , emit: vcf
    tuple val(meta), path("*vcf.gz.tbi") , emit: tbi
    tuple val(meta), path("*stats.txt")  , emit: stats
    path  "versions.yml"                 , emit: versions

```

 We will also emit versions file to keep track of bcftool versioning.


```Nextflow
    input:
    cat <<-END_VERSIONS > versions.yml
    "${task.process}":
        bcftools: \$(bcftools --version 2>&1 | head -n1 | sed 's/^.*bcftools //; s/ .*\$//')
    END_VERSIONS
```

 We can add either our docker container or search for the proper bcftool version on[ biocontainers registry](https://quay.io/repository/biocontainers/bcftools?tab=tags). 

```Nextflow
    conda "bioconda::bcftools=1.17"
    container "${ workflow.containerEngine == 'singularity' && !task.ext.singularity_pull_docker_container ?
        'https://depot.galaxyproject.org/singularity/bcftools:1.17--haef29d1_0':
        'quay.io/biocontainers/bcftools:1.17--haef29d1_0' }"
```

 Final module should be like this:


```Nextflow
process BCFTOOLS_MPILEUP {
    tag "$meta.id"
    label 'process_medium'

    conda "bioconda::bcftools=1.17"
    container "${ workflow.containerEngine == 'singularity' && !task.ext.singularity_pull_docker_container ?
        'https://depot.galaxyproject.org/singularity/bcftools:1.17--haef29d1_0':
        'quay.io/biocontainers/bcftools:1.17--haef29d1_0' }"

    input:
    tuple val(meta), path(bam)
    tuple val(meta2), path (fasta)

    output:
    tuple val(meta), path("*vcf.gz")     , emit: vcf
    tuple val(meta), path("*vcf.gz.tbi") , emit: tbi
    tuple val(meta), path("*stats.txt")  , emit: stats
    path  "versions.yml"                 , emit: versions

    when:
    task.ext.when == null || task.ext.when

    script:
    def prefix = task.ext.prefix ?: "${meta.id}"
    """
    bcftools \\
        mpileup \\
        --fasta-ref $fasta \\
        -Oz \\
        $bam \\
        | bcftools call --output-type v -mv -Oz \\
        | bcftools view --output-file ${prefix}.vcf.gz --output-type z

    tabix -p vcf -f ${prefix}.vcf.gz

    bcftools stats ${prefix}.vcf.gz > ${prefix}.bcftools_stats.txt

    cat <<-END_VERSIONS > versions.yml
    "${task.process}":
        bcftools: \$(bcftools --version 2>&1 | head -n1 | sed 's/^.*bcftools //; s/ .*\$//')
    END_VERSIONS
    """
}
```

- Let's save this file and add the module to our pipeline!

``` Nextflow
    //
    // MODULE: BCFTOOLS_MPILEUP
    //
    BCFTOOLS_MPILEUP(
        ch_bam,
        ch_genome_fasta
    )
    ch_versions = ch_versions.mix(BCFTOOLS_MPILEUP.out.versions)
```
- Our simple pipeline, providing parallel alignments for both paired-end and single-end fastq files is ready! Now, we need to create a config file to describe the parameters needed for the run.

The config file needs to include minimal information about the run. 

Open a text file and create this file and name to mytest.config


``` Nextflow
/*
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    Nextflow config file for running minimal tests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    Defines input files and everything required to run a fast and simple pipeline test.

    Use as follows:
        nextflow run nf-core/testpipeline -profile mytest,<docker/singularity> --outdir <OUTDIR>

----------------------------------------------------------------------------------------
*/

params {
    config_profile_name        = 'Test profile'
    config_profile_description = 'Minimal test dataset to check pipeline function'

    // Limit resources so that this can run on GitHub Actions
    max_cpus   = 2
    max_memory = '6.GB'
    max_time   = '6.h'

    // Input data
    // TODO nf-core: Specify the paths to your test data on nf-core/test-datasets
    // TODO nf-core: Give any required params for the test so that command line flags are not needed
    input  = 'assets/mysamplesheet.csv' 

    // Genome references
    genome = 'R64-1-1'
}
```

Then, we should add the path of our config into nextflow.config file to be able to use directly. 

``` Nextflow
profiles{
    mytest    { includeConfig 'conf/mytest.config'    }
}
```


6.  Our first full-functioning pipeline is ready! and we can directly run it!

```
nf-core run main.nf -profile mytest,docker --outdir results --input mysamplesheet.csv
```

We can actually test and debug our pipeline using this command. What is really cool and helpful is using **-resume** tag in order to resume previously finished jobs! Moreover, don't forget to check out .nextflow.log files in case of an error. All of the runs will be saved into _work_ directory. 

7. Analyzing the results:

 - Collection of versions is a vital process in order to keep track of software history. In nf-core pipelines, each tool version is collected in a channel and then processed using _CUSTOM_DUMPSOFTWAREVERSIONS_ module and represented through MultiQC tool.

- MultiQC tool also aggregates logs and reports from the analysis. In our analysis, FASTQC analysis was already included. In this example file, you can both see FASTQC report and also the software versions together with workflow summary. 

[Uploading multiqc_report.html…]()




**NOTE:** 

In case of any installation problems, a preconfigured Nextflow development environment is available using Gitpot: 

To run Gitpod:

- Click the following URL: https://gitpod.io/#https://github.com/nextflow-io/training
  -- This is nextflows GitHub repository URL, prefixed with https://gitpod.io/#
- Log in to your GitHub account (and allow authorization).
- Once you have signed in, Gitpod should load (skip prebuild if asked).


### Sources

Documentation and reference material for nextflow:
- Nextflow homepage: https://www.nextflow.io/
- Nextflow training material: https://training.nextflow.io
- Pipeline examples: https://www.nextflow.io/example1.html
- Main documentation: https://www.nextflow.io/docs/latest/index.html
- Common implementation patterns for developers: http://nextflow-io.github.io/patterns/index.html


