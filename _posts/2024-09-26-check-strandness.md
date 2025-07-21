---
layout: post
giscus_comments: true
related_posts: false
title: 'Strandness of RNA-seq and Transcripts Explained'
date: 2024-09-26
last_updated: 2025-02-28
tags:
  - rna-seq
  - transcriptome
  - sequencing method 
category:
  - note
---



Two posts by [Griffith Lab](https://rnabio.org/module-09-appendix/0009/12/01/StrandSettings/) and by [Hong Zheng](https://littlebitofdata.com/en/2017/08/strandness_in_rnaseq/) are already pretty comprehensive in describing the strandness parameters of many tools. Here I make some notes for my own records. 

# Strandness of RNA-seq

Imagine we collected some RNA molecules, performed the sequencing experiments, and obtained a bunch of paired-end RNA-seq reads (respectively R1 and R2 for each end of one pair). Now we want to know whether R1 has the same sequence as the original RNA or R2 has the same sequence as the original RNA (but of course, the reads are shorter and erroneous compared to an RNA molecule). This is called **strandness** of RNA-seq.

As of today, the mainstream of NGS (e.g. Illumina) technology is sequence-by-synthesis (SBS). Consequently, R1 and R2 must be from different strands of the RNA molecule, because they are synthesized from different directions! That means we have only 2 possibilities of strandness: (1) R1 is from sense-strand and R2 is form anti-sense-strand; or (2) R1 is from anti-sense strand and R2 is from sense-strand. Here we call the strand with the *same* sequences of the RNA as *sense* strand. Likewise, the strand that is reverse-complementary to the RNA is called *anti-sense* strand.

## Naming conventions 

Most of the name conventions fall into two types: `rf/fr` for `strandness` and `first/second` for `library type`.

**RF**: The first read is **R**everse sequence of RNA and the second read is **F**orward sequence of RNA.

**FR**: The first read is **F**orward sequence of RNA and the second read is **R**everse sequence of RNA.



The `first/second` library types can be somewhat confusing (at least I got confused a few times...). R1 of `first` stranded libraries are the same as the *anti-sense* strand and vice versa. This is because the sequenced substrates in RNA-seq technologies are (for most of the time) cDNAs. The first cDNA strand, which uses the RNA as template, is rev-comp to the RNA sequence. Then cDNA molecules can be synthesized to be double-stranded. Since cDNAs are double-stranded, this further divides technologies by whether:

**first**: sequencing the first cDNA strand (anti-sense)

**second**: sequencing the second cDNA strand (sense). 

Note that the first cDNA strand is reverse-complementary to the RNA molecule.



## Parameters for some bioinformatics tools

Two tools, [infer_experiment.py](https://rseqc.sourceforge.net/#infer-experiment-py) from RSeQC and [check_strandness](https://github.com/signalbash/how_are_we_stranded_here) are easy-to-use tools for checking strandness.

Assuming all reads are *paired* and sequenced *inwards*. 

| RNA                 | 5' --> 3'                                 | 3' --> 5'                                 | unstranded                                                   |
| ------------------- | ----------------------------------------- | ----------------------------------------- | ------------------------------------------------------------ |
|                     | coding/ sense  strand                     | noncoding/ anti-sense                     | Mixed of both strands                                        |
| Reads               | R1 is a sub-string of RNA, R2 is rev-comp | R2 is a sub-string of RNA, R1 is rev-comp | Both R1 and R2 may be the same or rev-comp of a sub-string of RNA |
| infer_experiment.py | 1++,1--,2+-,2-+                           | 1+-,1-+,2++,2--                           |                                                              |
| check_strandness    | FR/fr-secondstrand                        | RF/fr-firststrand                         |                                                              |
| Kallisto            | `--fr-stranded`                           | `--rf-stranded`                           |                                                              |
| Salmon              | `--libType ISF`                           | `--libType ISR`                           | `--libType IU`                                               |
| Scallop             | `--library_type second`                   | `--library_type first`                    | `--library_type unstranded`                                  |
| Stringtie           | `--fr`                                    | `--rf`                                    |                                                              |



## Loss of strand-specifity

Sometimes the sequencing protocol *per se* is strand-specific, but due to various reasons, the reads may lose the strand-specifity and become unstranded. Strand specificity requires us to know which strand is being sequenced/synthesized. Is this strand our direct RNA, the 1st cDNA, the 2nd cDNA, ..., etc. 

A common senario to lost strand-specifity is that the sequenced molecule is not single-strand RNA. For example, some experiments may amplify their target transcript using PCR. Then the actual molecules in the sequencer, are double-stranded cDNAs after many aplificaiton cycles. Hence, we won't know which strand of the cDNA is sense and which is anti-sense. 



# Strand of transcripts

Transcripts/RNAs also have "strand" information (e.g. the 7th column `strand` in a gtf file). It needs to be clarified that:

- strandness of RNA-seq: we are talking about which **strand of RNA/cDNA** the **seq reads are from**, the cRNA strand (i.e. rev-comp to RNA) or the direct-RNA strand (i.e. same sequence as RNA). The second strand of cDNA has the same sequence as the direct RNA, so computationally direct-RNA seq has the same strandness as "second-stranded"
- strand of transcripts: we are talking about which **strand of the genome** the **gene/transcript is from**. In other words, whether the RNA aligns to the forward sequence of the genome (`+` strand, same sequence as genome.fa file) or the RNA aligns to the reverse-complementary sequence of the genome (`-` strand, rev-comp to sequence of genome.fa file).

The strand of a transcript can be inferred by using read alignment and strandness of reads, and vice versa. 

## SAM tags `ts`, `tx` 

Some splice-aware aligner outputs [SAM tags](https://samtools.github.io/hts-specs/SAMtags.pdf) `ts` that indicates *which transcript strand the read is from*. This flag is usually inferred by checking the canonical intron splice motif of the reads without prior knowledge of the transcript information. Namely, `+` ts-tag means the read is from the same strand as the mRNA, and  `-` ts-tag means the read if from the first cDNA strand (rev-comp to mRNA). Likewise, R1 of a paired and *fr-stranded* RNA-seq sample is supposed to have all *positive* ts tag, while R2 of the same sample has all *negative* ts tag. Reads from an unstranded sample may have roughly half reads assigned positive ts and half assigned negative ts.

Some splice-aware aligners (e.g. [STAR](https://physiology.med.cornell.edu/faculty/skrabanek/lab/angsd/lecture_notes/STARmanual.pdf) by setting `--outSAMstrandField intronMotif`) output SAM tags `xs` that indicate *strand of the RNA transcript*. Namely, this xs tag should be the same as the `strand` information in a gtf file of the same transcript. The information of ts and xs can be inferred by examining the read alignments. Obviously, if a read aligns to the positive-strand of the genome and it is from the positive strand of the RNA (positive ts), then the transcript should be from the positive strand of the genome (positive xs, positive strand in gtf). If a read aligns to the negative-strand of the genome and it is from the negative strand of the RNA (negative ts), then the transcript should still be from the positive strand of the genome (positive xs, positive strand in gtf), i.e. the double negation cancels out.

In SAM/BAM format, bit `16` in a [SAM FLAG](https://broadinstitute.github.io/picard/explain-flags.html) indicates whether a read aligns to the reverse strand of the genome (bit 16 set iff `-` strand; bit 16 unset iff `+` strand). Hence, considering all the tags and flags as Boolean values, `tx` is negative iff bit 16 and `ts` flag are the same; `tx` is positive iff bit 16 and `ts` flag are different.



