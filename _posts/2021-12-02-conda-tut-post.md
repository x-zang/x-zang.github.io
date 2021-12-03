---
title: 'Anaconda getting started'
date: 2021-12-02
permalink: /posts/2021/12/conda-post/
tags:
  - tool
  - tutorial
  - conda
---

# Intro

This is a (oversimplified) tutorial on getting started of `conda`. It works for linux and macOS. All scripts are in bash.

#### What is and why conda?

Since we are going to discuss issues of anaconda (conda, hereafter), why we are interested in `conda`? OK, first of all, what is conda? Here is what I cited from [wiki](https://en.wikipedia.org/wiki/Anaconda_(Python_distribution)).

> Anaconda is a distribution of the Python and R programming languages for scientific computing (data science, machine learning applications, large-scale data processing, predictive analytics, etc.), that aims to simplify package management and deployment."

In short words, the first and formost reason for me to turn to conda is to **download and install packages** (and their dependencies and dependencies of dependencies and depend... ) in a single easy step. A second but equally important reason is to **manage environments for different projects**. For example, I frequently use some tools on python 2, julia 0.5 or R 2.X. Obviously I don't want those tools and their packages to mess up my current important project, so I can use `conda` to create different environments to separate  those tools and side projects. Hence, I avoid conflicts and confusions. 

# Initiation

You need to do initiation only once. 

Type `conda info` in your terminal. If you see any errors instead of normal information about your conda environment, you need initiation. If you see `conda: command not found`, you need initiation.

Depends on what computers you are using, you may do the followings:

#### If on an administred server or supercomputer

It is likely that your administrator have installed `conda` for all users. Try to run the load the module. For example, on Penn State [Roar Supercomputer](https://www.icds.psu.edu/computing-services/roar-user-guide/), do `module load anaconda3`. Consult your administrator for how to load the module.

```sh
# load conda module
module load anaconda3
# init for your $SHELL
conda init bash
```

#### If installed locally or on personal computer

First, install conda from https://www.anaconda.com/ (or it might have been installed by your labmate). 

```sh
# go to the conda directory
cd /path/to/conda/dir
cd bin
# init
./conda init bash
```



After proper `init`, try `conda info` again. If no error pops, you are done. You need to do initialization only once.

# Configure default pkg path

This step is optional but probably saves you from future troubles.

For most of time, the `$HOME` directory has limited space, e.g. around 100M-10G. If your environment builds up, it runs out of space quickly. I would recommend to avoid storing any data, performing any experiments, or doing anything with large files in the `$HOME` directory. Only small configure files should be in `$HOME`.

However, the default path of conda package cache is in `$HOME/.conda/pkg`. BAD! I recommend change the default package cache directory to somewhere else on the disk.  See https://docs.anaconda.com/anaconda/user-guide/tasks/shared-pkg-cache/

Open and edit the conda configuration file (`$HOME/.condarc` ) to add the following lines:

> pkgs_dirs:
>
> ​		- /path/to/somewhere/on/disk



Optionally, you can add your favorite channels to the configuration file.

>channels:
>
>​	- bioconda
>
>​	- conda-forge
>
>​	- defaults



# Create an environment

You can create a new environment as needed, e.g. for a new project, for a tool with conflicting version of dependencies, or for a older version of a package. 

Command to create an environment: (not recommended)

```sh
conda create -n env_name
```

This command creates a new environment named `env_name` (`-n` means name)  in the default directory `$HOME/.conda/envs`. As mentioned, this might quickly exhaust all the space in your `$HOME` dir. I don't like it.

It is more recommended to create a new environment somewhere else (`-p` means prefix):

```sh
conda create -p /some/other/dir/env_name
```



Optionally, you can specific the version of your packages during creation, e.g. python 2.7 or R 3.4:

```sh
conda create -p /some/other/dir/env_name_py2 	 python=2.7
conda create -p /some/other/dir/env_name_R34	 R=3.4
```



# Activate & Deactivate

You need to activate an environment before using it.

```sh
# activate
conda activate env_name
# or equivalently
conda activate /some/other/dir/env_name
```

To exit an environment, deactivate it:

```sh
conda deactivate
```



# Install packages

Google for commands to install your packages, most commands are like `conda install pkg_name`. Sometimes there is a `-c channel_name` argument to specify which channel to use.

For example, to install `scallop` transcriptome assembler, you can use:

```sh
conda install -c bioconda scallop
```



# Summary

```sh
# create new environment
conda create -p /some/other/dir/env_name

# use environment
conda activate env_name
conda install pkg_name
conda deactivate
```

Read more about using conda https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html.

