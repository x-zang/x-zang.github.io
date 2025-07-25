---
layout: post
giscus_comments: true
related_posts: false
title: 'Tutorial: RNA-seq short variant calling using GATK4'
date: 2021-02-05
permalink: /posts/2021/02/gatk-rna-snp/
tags:
  - gatk
  - rna-seq
  - variation
---

GATK is powerful. However, running it may not be as easy. People, especially bioinformatics beginners are often overwhelmed by its powerfulness and complexity. 

This repo is a tutorial of how to locally running the workflow for RNA-seq short variant calling (SNPs & indels) using GATK4. The original workflow is available at [gatk-workflows](https://github.com/gatk-workflows)/**[gatk4-rnaseq-germline-snps-indels](https://github.com/gatk-workflows/gatk4-rnaseq-germline-snps-indels)**, developed by the GATK Team.  This repo is made for my personal interest and record to make it easier to run GATK workflow. Root is not required if this tutorial is followed. The tutorial is made based on the [GATK4 workflow repo](https://github.com/gatk-workflows/gatk4-rnaseq-germline-snps-indels), [its best practice](https://gatk.broadinstitute.org/hc/en-us/articles/360035531192-RNAseq-short-variant-discovery-SNPs-Indels-), and the tutorial on [how to run GATK workflow](https://gatk.broadinstitute.org/hc/en-us/articles/360035530952?id=12521). 

Note: This is for GATK4 and may not be compatible with GATK3.8.

**Last updated: Jul 2, 2020.**



# Install GATK4

**Download the latest release** from GATK4 repository https://github.com/broadinstitute/gatk/releases. Downloading the pre-compiled binary file is the easiest installation. Requirements for GATK4 can be found form https://github.com/broadinstitute/gatk.

```sh
# download gatk
# replace the link with latest gatk release from https://github.com/broadinstitute/gatk/releases
wget https://github.com/broadinstitute/gatk/releases/download/4.1.8.0/gatk-4.1.8.0.zip
unzip gatk-4.1.8.0.zip
cd gatk-4.1.8.0
```



**Install conda environment for GATK**. On [this page](https://gatk.broadinstitute.org/hc/en-us/articles/360035889851--How-to-Install-and-use-Conda-for-GATK4), it is stated that the bioconda GATK installation does not configure the environment correctly. Then, it is necessary to manually create a conda environment and install GATK dependencies.

```shell
# create a conda environment
conda env create -n gatk -f gatkcondaenv.yml
# or 
conda env create --prefix ./path/to/directory/ -f gatkcondaenv.yml
# activate conda environment
conda activate gatk
# or
conda activate ./path/to/directory/
```

You might also want to install `java-jdk` if it is not yet installed (`conda install -c cyclus java-jdk`).

Also, have STAR installed in this environment.

This is a convenient way of installing required conda dependencies of GATK. Sometimes it doesn't work, e.g. conflicts. In case of conda not working, manually install the packages described in this file with `conda` and `pip`.



# Run Docker without root

Docker is required to run GATK4 workflow. Root is needed, but you can also run docker daemon without sudo ([see docker docs for more details](https://docs.docker.com/engine/security/rootless/)). This section works for Ubuntu. Other OS may have some [prerequisites](https://docs.docker.com/engine/security/rootless/#prerequisites).

```sh
curl -fsSL https://get.docker.com/rootless | sh

# At the end of the script, some environment variables are required to be set.
# They would be displayed like the below examples, C&P to run them.
export DOCKER_HOST=unix:///run/user/1001/docker.sock
```
Replace the number in `/run/user/1001/` with your uid, which can be retrieved using `id -u`. For example, if `id -u` outputs `1005`, the command should be `export DOCKER_HOST=unix:///run/user/1005/docker.sock`.

```sh
# start/stop/restart docker without root using the following
systemctl --user (start|stop|restart) docker
```


### Change docker root directory

When running docker, I found it uses `$home/.local/share` as the root dir. This directory is in my `$home`, which is limited in space. I changed docker root dir following [this](https://medium.com/@hsadanuwan/how-to-change-docker-default-data-directory-f884dac76c1f), otherwise for my case, the disk can be full quickly and the process fails.

Run `docker info`. The line starting with `Docker Root Dir` states the root directory of docker. I intend to change it to another directory in the disk. It can be configured by editing `docker.service` file.

```shell
# find the docker.service file
locate docker.service
```

Edit `docker.service` file. Find the `ExecStart` line and add `-g /customized/root/dir` to the end of this line. 

Now for me, this line looks like `ExecStart=/home/userid/bin/dockerd-rootless.sh --experimental --storage-driver=overlay2 -g /disk/userid/tools/docker-tmp`

```shell
# reload dockerd and restart docker
systemctl --user daemon-reload
systemctl --user restart docker
```

Run `docker info  ` again. I can see `Docker Root Dir: /disk/userid/tools/docker-tmp`. Then all's set. Some other options can be found [here](https://github.com/IronicBadger/til/blob/master/docker/change-docker-root.md)



# Download the GATK4 workflow

Set up a working directory.

```shell
mkdir gatk-workflows
cd gatk-workflows
```

Download the latest releases of the workflow from [here](https://github.com/gatk-workflows/gatk4-rnaseq-germline-snps-indels/releases).

```shell
# replace the following link with the latest release
wget https://github.com/gatk-workflows/gatk4-rnaseq-germline-snps-indels/archive/1.0.0.tar.gz 
# change the .tar.gz file name if needed
tar -zxvf 1.0.0.tar.gz
```



# Prepare input files

Set up inputs directory, and put all necessary input files in this directory.

```shell
mkdir inputs
```

Download necessary file from Google Cloud Bucket of Broad Institute https://console.cloud.google.com/storage/browser/gcp-public-data--broad-references/ using browser or [gsutil](https://cloud.google.com/storage/docs/gsutil_install#linux). Most of the required references and databases can be found there. Make sure all of the files uses either one of UCSC names or Ensembl names.

```shell
# This is an example of how to download with gsutil
gsutil -m cp gs://gcp-public-data--broad-references/hg38/v0/Homo_sapiens_assembly38.dbsnp138.vcf ./inputs/
```



The input file format for GATK is **unmapped bam**. GATK `FastqToSam` can be used to convert fastq to unmapped bam.

```shell
gatk FastqToSam \
-F1 reads_R1.fastq \
-F2 reads_R2.fastq \
-O reads.unmapped.bam \
--SAMPLE_NAME sample001 \
--PLATFORM ILLUMINA \
--READ_GROUP_NAME group01 \
--SORT_ORDER unsorted 
# --SORT_ORDER {unsorted, queryname, coordinate, duplicate, unknown}
```



Index and dictionary files (.fai/.dict/.idx) can be generated with samtools and igvtools

```shell
samtools faidx Homo_sapiens_assembly38.genome.fasta
samtools dict Homo_sapiens_assembly38.genome.fasta -o Homo_sapiens_assembly38.genome.dict 
igvtools index file.vcf 
```



**Then edit the `.json` file (in the GATK workflow directory `gatk4-rnaseq-germline-snps-indels`) to replace the corresponding file paths with your local file paths.**

Also replace the GATK path in the `.json` file with the directory where GATK4 is installed.  

```
  "##_COMMENT5": "PATHS",
  "#RNAseq.gatk_path_override": "/path/to/gatk4",
```



#### **Edit `gatk4-rna-best-practices.wdl` file**

I'm not sure if this step is necessary or correct, but it worked for me (version `gatk4-rnaseq-germline-snps-indels-1.0.0`).  Read the following instructions, or directly replace the original one with my edited `.wdl` (version 1.0.0,  see [this](https://github.com/x-zang/gatk4-rnaseq-germline-snps-indels/blob/master/gatk4-rna-best-practices.wdl)).

Search `BedToIntervalList ` in the `gatk4-rna-best-practices.wdl` file. You can see a block of code like the following.

```
        ${gatk_path} \
            BedToIntervalList \
            -I=exome.fixed.bed \
            -O=${output_name} \
            -SD=${ref_dict}
```

This section looks more like GATK3.8 commands but not GATK4's, so I deleted the `=` sign behind `-I`, `-O`, `-SD` arguments. Then this block of code looks like:

            ${gatk_path} \
                BedToIntervalList \
                -I exome.fixed.bed \
                -O ${output_name} \
                -SD ${ref_dict}

Do the same to the `IntervalListTools` code block.

````
        ${gatk_path} --java-options "-Xms1g" \
            IntervalListTools \
            --SCATTER_COUNT ${scatter_count} \
            --SUBDIVISION_MODE BALANCING_WITHOUT_INTERVAL_SUBDIVISION_WITH_OVERFLOW \
            --UNIQUE true \
            --SORT true \
            --INPUT ${interval_list} \
            --OUTPUT out
````

And `MergeVcfs` (line 721):

```
        ${gatk_path} --java-options "-Xms2000m"  \
            MergeVcfs \
            --INPUT ${sep=' --INPUT ' input_vcfs} \
            --OUTPUT ${output_vcf_name}
```



# Execute the workflow

Everything should be all set so far. Download the [cromwell](https://github.com/broadinstitute/cromwell/releases) program which will execute the GATK4 workflow. `cd` to the `gatk-workflows` directory.

```shell
wget https://github.com/broadinstitute/cromwell/releases/download/51/cromwell-51.jar

# execute GATK4 workflow
java -jar cromwell-51.jar run gatk4-rnaseq-germline-snps-indels-1.0.0/gatk4-rna-best-practices.wdl --inputs gatk4-rnaseq-germline-snps-indels-1.0.0/gatk4-rna-germline-variant-calling.inputs.json
```





# References

https://github.com/gatk-workflows/gatk4-rnaseq-germline-snps-indels

https://gatk.broadinstitute.org/hc/en-us/articles/360035531192-RNAseq-short-variant-discovery-SNPs-Indels-

https://gatk.broadinstitute.org/hc/en-us/articles/360035530952?id=12521

https://docs.docker.com/engine/security/rootless/
