Source: https://github.com/chrisquince/DESMAN#taxa_assignment  

**Important** Make sure you have already formatted your contig fasta headers to a compatible version by using this script from Anvi'o:
```
anvi-script-reformat-fasta contigs_1000_filtered.fa -o contigs_1000_filtered-fixed.fa -l 0 --simplify-names
```

This is a gene based approach.  
Make sure to have Diamond installed (20,000x faster than regular blast). For the binary version:  
```
wget http://github.com/bbuchfink/diamond/releases/download/v0.8.33/diamond-linux64.tar.gz
tar xzf diamond-linux64.tar.gz
```
Also download the NR database from NCBI and format it for diamond
```
wget ftp://ftp.ncbi.nlm.nih.gov/blast/db/FASTA/nr.gz
wget ftp://ftp.ncbi.nih.gov/pub/taxonomy/accession2taxid/prot.accession2taxid.gz
diamond makedb --in nr.gz -d nr --threads 40
```
Next make a taxlist to link to taxonomy.  
Clone [this](https://github.com/zyxue/ncbitax2lin) repository.
Make sure to have <code>python-anaconda2/201607</code> loaded. 
```
cd ncbitax2lin
make
```
Then unzip the file and format it to tsv 
```
gunzip lineages.csv.gz
cat lineages.csv | sed 's/,/\t/g' > lineages.tsv
cut -f1,2,3,4,5,6,7,8 lineages.tsv > lineages_filtered.tsv
sed -i "s/tax_id/0/g" lineages_filtered.tsv
sed -i "s/superkingdom/_0/g" lineages_filtered.tsv
sed -i "s/phylum/_1/g" lineages_filtered.tsv
sed -i "s/class/_2/g" lineages_filtered.tsv
sed -i "s/order/_3/g" lineages_filtered.tsv
sed -i "s/family/_4/g" lineages_filtered.tsv
sed -i "s/genus/_5/g" lineages_filtered.tsv
sed -i "s/species/_6/g" lineages_filtered.tsv
``` 
This file doesn't quite work yet so for now use this one:
```
wget https://desmandatabases.s3.climb.ac.uk/all_taxa_lineage_notnone.tsv
```
Make sure to have these modules loaded
```
module load prodigal python-anaconda2/201607 ncbi-blast R anvio
```

The entire workflow is available in this shell script. In case Vizbin gives an error when using the Vizbin annotation file, open it in excel and retype the label column name, resave and try again. This should fix the issue.  
**Notice:** *I changed the `MIN_FRACTION` parameter in `ClassifyContigNR.py` from `MIN_FRACTION=0.9` to `MIN_FRACTION=0.1` in order to reduce the stringency in classifying contigs.

```
#/bin/bash

# This script classifies selected bins by Diamond (NR database).
# Also outputs an annotation file that can be used in Vizbin
# This is a gene based approach.

# Usage: bash classify_contigs_diamond.sh

# Modules to be loaded:
# module load prodigal python-anaconda2/201607 ncbi-blast R anvio

# created by Ruben Props <ruben.props@ugent.be>

#######################################
##### MAKE PARAMETER CHANGES HERE #####
#######################################

# Name of contig fasta to be classified
fasta=contig

# Filter threshold
filter=3000

#fasta extension
ext=fa

# Number of threads
threads=40

# Paths
database=/nfs/vdenef-lab/Shared/Ruben/databases_metaG/NR_diamond
name_nr=nr_desman.dmnd
DESMAN=~/DESMAN/scripts/
SCRIPTS=/nfs/vdenef-lab/Shared/Ruben/scripts_metaG/Ruben/

# Files
prot_taxid=prot.accession2taxid
all_taxa_lineage=all_taxa_lineage_notnone.tsv
gid_taxid=gi_taxid_prot_desman.dmp
# Classification level
class=4 # 4 = order level, 5 = family level, etc

####################################################
##### DO NOT MAKE ANY CHANGES BEYOND THIS LINE #####
#####     unless you know what you're doing    #####
####################################################

# Begin
echo "[`date`] --- Classifying fasta: ${fasta}.${ext}"
echo "[`date`] --- Only classifying contigs larger than >: ${filter}"

# Make new directory
mkdir classification_${filter}
cd classification_${filter}
cp ../${fasta}.${ext} .

# Format headers
anvi-script-reformat-fasta ${fasta}.${ext} -o ${fasta}-fixed.${ext} -l 0 --simplify-names

# Filter out short contigs
reformat.sh in=${fasta}-fixed.${ext} out=${fasta}-fixed-filtered_${filter}.${ext} minlength=$filter

# Search for genes on contigs
export NR_DMD=${database}/${name_nr}
prodigal -i ${fasta}-fixed-filtered_${filter}.${ext} -a ${fasta}-fixed-filtered_${filter}.faa -d ${fasta}-fixed-filtered_${filter}.fna  -f gff -p meta -o ${fasta}-fixed-filtered_${filter}.gff

# Blastp to NR database
mkdir AssignTaxa
cd AssignTaxa
cp ../${fasta}-fixed-filtered_${filter}.faa .
diamond blastp -p $threads -d $NR_DMD -q ${fasta}-fixed-filtered_${filter}.faa -a ${fasta}-fixed-filtered_${filter} > d.out
diamond view -a ${fasta}-fixed-filtered_${filter}.daa -o ${fasta}-fixed-filtered_${filter}_nr.m8

# Start classification
python ${DESMAN}/Lengths.py -i ${fasta}-fixed-filtered_${filter}.faa > ${fasta}-fixed-filtered_${filter}.len
python ${DESMAN}/ClassifyContigNR.py ${fasta}-fixed-filtered_${filter}_nr.m8 ${fasta}-fixed-filtered_${filter}.len -o ${fasta}-fixed-filtered_${filter}_nr -l ${database}/${all_taxa_lineage} -g ${database}/${gid_taxid}

# Extracting specific classification level
~/DESMAN/scripts/Filter.pl $class < ${fasta}-fixed-filtered_${filter}_nr_contigs.csv > classification_contigs_${class}.csv

# Make tsv
cat classification_contigs_${class}.csv | sed 's/,/\t/g' > classification_contigs_${class}.tsv

# Get contig list
grep ">" ../${fasta}-fixed-filtered_${filter}.${ext} | sed "s/>//g" > contig_ids-fixed-filtered_${filter}.txt

# Format to annotation file for vizbin
Rscript ${SCRIPTS}/diamond2vizbin.R classification_contigs_${class}.tsv contig_ids-fixed-filtered_${filter}.txt

echo "[`date`] --- Done"```
