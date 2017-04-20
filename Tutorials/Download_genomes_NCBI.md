
# Download genomes from NCBI
**From: https://www.biostars.org/p/61081/**

#### Get the list of assemblies:
```
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/genbank/bacteria/assembly_summary.txt
```
#### Parse the addresses of **complete** genomes from it:
```
awk -F '\t' '{if($12=="Complete Genome") print $20}' assembly_summary.txt > assembly_summary_complete_genomes.txt
```
#### Make a dir for data
```
mkdir GbBac
```
#### Fetch data
```
for next in $(cat assembly_summary_complete_genomes.txt); do wget -P GbBac "$next"/*genomic.fna.gz; done
```
#### Extract data
```
gunzip GbBac/*.gz
```
#### Concatenate data
```
cat GbBac/*.fna > all_complete_Gb_bac.fasta
```

Bash script to selectively download genomes from NCBI, for example based on taxon IDs selected from IMG.
```
#/bin/bash

# This script downloads all genomes from NCBI based on a list of NCBI taxon IDs (extracted from IMG for example).

# created by Ruben Props <ruben.props@ugent.be>

#######################################
##### MAKE PARAMETER CHANGES HERE #####
#######################################

# provide name of list which has the NCBI taxon IDs in the first column with
# one ID on each row
input_list=taxid_tax.txt


####################################################
##### DO NOT MAKE ANY CHANGES BEYOND THIS LINE #####
#####     unless you know what you're doing    #####
####################################################

awk '{FS="\t"} !/^#/ {print $1} ' $input_list > list
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/genbank/bacteria/assembly_summary.txt
awk '{FS="\t"} !/^#/ {print $6} ' assembly_summary.txt > tmp
tail -n +3 assembly_summary.txt > tmp2
paste tmp tmp2 > assembly_summary_r.txt
while read pat; do grep -w "^$pat" assembly_summary_r.txt; done < list > list2
rm assembly_summary_r.txt tmp2 tmp list
awk '{FS="\t"} !/^#/ {print $21} ' list2 > assembly_summary_complete_genomes.txt
rm list2
for next in $(cat assembly_summary_complete_genomes.txt); do wget -P GbBac "$next"/*genomic.fna.gz; done
cd GbBac
# remove rna and cds data
rm *cds*
rm *rna*
cd ..
```
