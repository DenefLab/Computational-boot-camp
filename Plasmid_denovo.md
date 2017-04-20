Paper can be found [here](https://academic.oup.com/bioinformatics/article/33/4/475/2623362/Recycler-an-algorithm-for-detecting-plasmids-from)  
Install (requires python-anaconda2/201607 loaded)
```
git clone https://github.com/rozovr/Recycler.git
cd Recycler
python setup.py install --user
```
Can only be used with **Megahit** or **SPAdes** assemblies. E.g., `k99.contigs.fa`. This batch script will assemble your reads with `megahit`, format the output to a `recycler` compatible one, run `recycler` and annotate the ORFs with `prokka`.
Its dependencies are:
```
samtools
bwa
recycler
megahit
prokka # requires BioPerl v1.6.9 or higher
```

```
#/bin/bash

# This script assembles plasmids from denovo assembly graphs

# created by Ruben Props <ruben.props@ugent.be>

#######################################
##### MAKE PARAMETER CHANGES HERE #####
#######################################

# Make sure you named the binning folder according to the
# parameterization (e.g., k4_L3000 for kmer=4 and length threshold=3000)
folder=$1

# extension
ext1=$2
ext2=$3

# Number of threads
threads=20

####################################################
##### DO NOT MAKE ANY CHANGES BEYOND THIS LINE #####
#####     unless you know what you're doing    #####
####################################################

# Enter directory
cd $1
echo "[`date`] Running recycler pipeline in directory: ${1}"

# Create files Read lists
ls *${ext1}.fastq | head -c -1 | tr '\n' ',' > R1.csv
ls *${ext2}.fastq | head -c -1 | tr '\n' ',' > R2.csv

# Start assemblies
megahit -1 $(<R1.csv) -2 $(<R2.csv) -t 20 -o megahit_sensitive_assembly --presets meta-sensitive > megahit.out

# Create graph file
mkdir recyler_output
megahit_toolkit contig2fastg 99 megahit_sensitive_assembly/intermediate_contigs/k99.contigs.fa > recyler_output/k99.fastg

# Turn to node fasta
cd recyler_output
make_fasta_from_fastg.py -g k99.fastg 

# Map reads + format into specific bam
bwa index k99.nodes.fasta

bwa mem k99.nodes.fasta ../*${ext1}.fastq ../*${ext2}.fastq | samtools view -buS - > reads_pe.bam
samtools view -bF 0x0800 reads_pe.bam > reads_pe_primary.bam
samtools sort reads_pe_primary.bam > reads_pe_primary.sort.bam
samtools index reads_pe_primary.sort.bam

# Run recycler
recycle.py -g k99.fastg -k 99 -b reads_pe_primary.sort.bam -i False

# Annotate with prokka
prokka k99.cycs.fasta --outdir prokka --norrna --notrna --metagenome --cpus 20

# Get back
cd ../..
```

Example usage:
```
bash recycler.sh Sample-16 fastq_pairs_R1 fastq_pairs_R2
```
with `Sample-16` the sample directory which contains the `fastq` files, `fastq_pairs_R1` and `fastq_pairs_R2` are the patterns in front of `.fastq` that indicate the read `fastq` files.
