# Quality Control
```
Once your samples have been sequenced some quality control measures must be taken.

The major steps are:
    - adapter trimming
    - low quality base trimming
    - dereplication of raw reads
```
# Basic Example Workflow
```
raw reads -> finding adapters -> adapter trimming -> poor base trimming -> dereplication -> read pairing (if paired end)
```

# What is the Purpose of Each Step?
The following introduction to steps will use a metagenomic context and fastqc visualizations of sequencing data to understand what is happening in qc and what you should look for once the qc steps are done.

## Trimming Adapters
```
Adapters are synthetic sequences added to the beginning and ends of reads for sequencing. Because tehse sequences are not part of your experimental sample it is imiperitive that they are removed. If they are not completely removed they can contaminate you assemblies and appear downstream in contigs making results from you analysis not make sense and also useless.
```

## Removing poor quality reads
```
A result of the sequencing technology is that reads towards the beginning and ends of sequences suffer from a loss of quality. Quality in this cacse refers to the confidence of the base call. The quantitative measure of the quality of a base is its phred score. 

A general overview of phred scores:
![aslk](./images/screenshot_4.png)

```

## Removing replicate reads
```

```
## Visualization of Read Quality
```

```