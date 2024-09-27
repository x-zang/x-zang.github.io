---
title: 'Check strandness of RNA-seq'
date: 2024-09-26
permalink: /posts/2024/09/check-strandess/
tags:
  - pipline
  - tutorial
  - gatk
---



Two posts by [Griffith Lab](https://rnabio.org/module-09-appendix/0009/12/01/StrandSettings/) and by [Hong Zheng](https://littlebitofdata.com/en/2017/08/strandness_in_rnaseq/) are already pretty comprehensive in describing strandness of many tools. Here I make some notes for my own records. 

Two tools, [infer_experiment.py](https://rseqc.sourceforge.net/#infer-experiment-py) from RSeQC and [check_strandness](https://github.com/signalbash/how_are_we_stranded_here) are easy to use tools for checking strandness.



Assuming all reads are paired and sequenced inwards. 

| RNA                 | 5' --> 3'                                 | 3' --> 5'                                 |                                                              |
| ------------------- | ----------------------------------------- | ----------------------------------------- | ------------------------------------------------------------ |
| DNA                 | coding / sense  strand                    | noncoding/ anti-sense                     |                                                              |
| Reads               | R1 is a sub-string of RNA, R2 is rev-comp | R2 is a sub-string of RNA, R1 is rev-comp | Both R1 and R2 may be the same or rev-comp of a sub-string of RNA |
| infer_experiment.py | 1++,1--,2+-,2-+                           | 1+-,1-+,2++,2--                           |                                                              |
| check_strandness    | FR/fr-secondstrand                        | RF/fr-firststrand                         |                                                              |
| Kallisto            | `--fr-stranded`                           | `--rf-stranded`                           |                                                              |
| Salmon              | `--libType ISF`                           | `--libType ISR`                           | `--libType IU`                                               |
| Scallop             | `--library_type second`                   | `--library_type first`                    | `--library_type unstranded`                                  |
| Stringtie           | `--fr`                                    | `--rf`                                    |                                                              |

