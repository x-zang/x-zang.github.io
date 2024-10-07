---
title: 'Check strandness of RNA-seq'
date: 2024-09-26
permalink: /posts/2024/09/check-strandess/
tags:
  - pipline
  - tutorial
  - gatk
---



Two posts by [Griffith Lab](https://rnabio.org/module-09-appendix/0009/12/01/StrandSettings/) and by [Hong Zheng](https://littlebitofdata.com/en/2017/08/strandness_in_rnaseq/) are already pretty comprehensive in describing the strandness parameters of many tools. Here I make some notes for my own records. 

Two tools, [infer_experiment.py](https://rseqc.sourceforge.net/#infer-experiment-py) from RSeQC and [check_strandness](https://github.com/signalbash/how_are_we_stranded_here) are easy to use tools for checking strandness.



Assuming all reads are *paired* and sequenced *inwards*. 

| RNA                 | 5' --> 3'                                 | 3' --> 5'                                 |                                                              |
| ------------------- | ----------------------------------------- | ----------------------------------------- | ------------------------------------------------------------ |
| DNA                 | coding/ sense  strand                     | noncoding/ anti-sense                     |                                                              |
| Reads               | R1 is a sub-string of RNA, R2 is rev-comp | R2 is a sub-string of RNA, R1 is rev-comp | Both R1 and R2 may be the same or rev-comp of a sub-string of RNA |
| infer_experiment.py | 1++,1--,2+-,2-+                           | 1+-,1-+,2++,2--                           |                                                              |
| check_strandness    | FR/fr-secondstrand                        | RF/fr-firststrand                         |                                                              |
| Kallisto            | `--fr-stranded`                           | `--rf-stranded`                           |                                                              |
| Salmon              | `--libType ISF`                           | `--libType ISR`                           | `--libType IU`                                               |
| Scallop             | `--library_type second`                   | `--library_type first`                    | `--library_type unstranded`                                  |
| Stringtie           | `--fr`                                    | `--rf`                                    |                                                              |



## Naming conventions 

Most of the parameters above fall into two types: `rf/fr` for `strandness` and `first/second` for `library type`.

**RF**: The first read is **R**everse sequence of RNA and the second read is **F**orward sequence of RNA.

**FR**: The first read is **F**orward sequence of RNA and the second read is **R**everse sequence of RNA.



The `first/second` library types can be somewhat confusing (at least I got confused a few times...). R1 of `first` stranded libraries are the same as the *anti-sense* strand and vice versa. This is because the sequenced substrate in RNA-seq technologies are (for most of the time) cDNAs. Since DNAs are double-stranded, this further divides technologies by whether:

**first**: sequencing the first cDNA strand (anti-sense)

**second**: sequencing the second cDNA strand (sense). 

Note that the first cDNA strand is reverse-complementary to the RNA molecule.
