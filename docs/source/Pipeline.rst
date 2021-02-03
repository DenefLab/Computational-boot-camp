=============================================
Metagenomics Pipeline, From raw reads to bins
=============================================

Basic Pipeline Structure
========================
Currently, this pipeline covers:

- Quality control
 - adapter trimming
 - quality trimming
- Assembly with MEGAHIT
 - single sample assemblies
 - co-assemblies
- Mapping with bwa
 - can map multiple samples to one reference
 - generates sorted bams
- Binning with concoct
 - runs the multithreaded version of concoct
 - checkm (BROKEN)
 
The pipeline is a large Snakemake workflow that calls on different sets of rules for each of the above steps based on teh requested output files. It aims to be modular so that any step can be run without performing the step that comes before it (ie. run mapping on something that was not QCed with this pipeline). To acomplish this the pipeline is controlled by config files and a few csv files that keep track of important information like Sample names and their corresponding files. The pipeline will also submit a single slurm job for each rule run without the need to write your own slurm scripts.
 
The Files
=========
The Main Config File
----------------
The config file is the main file for control. Here you tell the pipeline what mode you want to run in, information for your slurm account, where the inportant csv files are, how you want things run, etc. 

The CSV Files
-------------
There are three csv files that are used by the pipeline to supply what files you want to run through it. These include the fastq, assembly, and mapping csv files. It is important to note that the names you use for samples and assemblies are what they will be known as to the pipeline and will be what the outfiles use in their naming scheme. It is also important to note that the column names in your file must match the names in these examples. LETTER CASE MATTERS.

The Scheme Files
----------------
The assembly, mapping, and binning workflows require these scheme files. They are used to include multiple samples for co-assembly, mapping multiple samples to the same reference, and indicating what mappings to include when computing coverage for binning.


