
## **Standardizing and harmonizing NGS analysis workflows**

This tutorial will explore how FAIR principles enable the standardization and harmonization of nf-core-based NGS analysis workflows within GHGA. We will  demonstrate the adaptability of nf-core workflows and discuss the importance of standardization of workflows. Finally, we will show how to make workflows scalable, robust, and automated for continuous benchmarks with hands-on exercises using a subset of a public dataset with various configurations like local and cloud settings.


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
- [Java11]([url](https://www.oracle.com/java/technologies/downloads/)) (or later, up to 18)
- [Nextflow]([url](https://www.nextflow.io/docs/latest/getstarted.html#installation))
- [nf-core]([url](https://nf-co.re/))
- [GitHub id]([url](https://github.com/))
- [Docker]([url](https://www.oracle.com/java/technologies/downloads/))
- [Zenado id]([url](https://zenodo.org/)) 

# **An example benchmark analysis using NCBench and nf-core/sarek**

- Benchmark plan:
-   Data: link to zenado
-   Comparison of aligners
-   Comparison of variant callers

1. What is **GitHub**, how we can use it?

- Open your first repository and fork https://github.com/ghga-de/gcb_workshop

  
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

- Lets build an image to download and install *samtools*

```
# Update the package inside the container
apt-get update
# Install the tools we need to download and compile Samtools
apt-get install wget build-essential liblzma-dev libbz2-dev libncurses5-dev zlib1g-dev
# Download Samtools
wget https://github.com/samtools/samtools/releases/download/1.18/samtools-1.18.tar.bz2
# Unpack the archive
tar jxf samtools-1.18.tar.bz2
cd samtools-1.18
# Compile the code
make
# Install the resulting binaries
make install
#check if everything is properly installed
samtools --version
#If complete exit docker
exit
```

- Save and upload the docker image.
An example image name could be: kubran/samtools:v0

```
docker commit <containerid> dockerid/imagename:version_tag
```

- Push the image to Dockerhub:

```
docker push kubran/samtools:v0
```

3.  Getting familiar with **Nextflow** and **nf-core**

Complete the installations following the links
- [Nextflow]([url](https://www.nextflow.io/docs/latest/getstarted.html#installation))
- [nf-core]([url](https://nf-co.re/))

```
nextflow info
```

**nf-core** is a community effort to optimize and collect a set of nextflow pipelines. Currently, they have **86** pipelines!

- Let's have a look at their famous [sarek pipeline](https://nf-co.re/sarek/3.2.3)
  

### Sources

Documentation and reference material for nextflow:
- Nextflow homepage: https://www.nextflow.io/
- Pipeline examples: https://www.nextflow.io/example1.html
- Main documentation: https://www.nextflow.io/docs/latest/index.html
- Common implementation patterns for developers: http://nextflow-io.github.io/patterns/index.html


