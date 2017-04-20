How to:  
“Measurement of bacterial replication rates in microbial communities” - Christopher T. Brown, Matthew R. Olm, Brian C. Thomas, Jillian F. Banfield

[http://dx.doi.org/10.1038/nbt.3704](http://dx.doi.org/10.1038/nbt.3704)

## 1. Map reads to bins
Here is an example for three bins.

a. Index the genome bin fasta files
```
mkdir iRep
cd iRep
mkdir bins
bwa index ./bins/Bin_1-contigs.fa
bwa index ./bins/Bin_2_refined-contigs.fa
bwa index ./bins/Bin_2_Virus-contigs.fa
```
b. Map reads back to genome bins
```
bwa mem -t 5 ./bins/Bin_1-contigs.fa ../data/DNA-Pieter-16/*_R1.fastq ../data/DNA-Pieter-16/*_R2.fastq > ./map-DNA-Pieter-16/Bin_1.sam
bwa mem -t 5 ./bins/Bin_2_refined-contigs.fa ../data/DNA-Pieter-16/*_R1.fastq ../data/DNA-Pieter-16/*_R2.fastq > ./map-DNA-Pieter-16/Bin_2_refined.sam
bwa mem -t 5 ./bins/Bin_2_Virus-contigs.fa ../data/DNA-Pieter-16/*_R1.fastq ../data/DNA-Pieter-16/*_R2.fastq > ./map-DNA-Pieter-16/Bin_2_Virus.sam
bwa mem -t 5 ./bins/Bin_1-contigs.fa ../data/DNA-Pieter-17/*_R1.fastq ../data/DNA-Pieter-17/*_R2.fastq > ./map-DNA-Pieter-17/Bin_1.sam
bwa mem -t 5 ./bins/Bin_2_refined-contigs.fa ../data/DNA-Pieter-17/*_R1.fastq ../data/DNA-Pieter-17/*_R2.fastq > ./map-DNA-Pieter-17/Bin_2_refined.sam
bwa mem -t 5 ./bins/Bin_2_Virus-contigs.fa ../data/DNA-Pieter-17/*_R1.fastq ../data/DNA-Pieter-17/*_R2.fastq > ./map-DNA-Pieter-17/Bin_2_Virus.sam
```

## 2. Run iRep
```
# For sample 17
iRep -f ./bins/Bin_1-contigs.fa -s ./map-DNA-Pieter-17/Bin_1.sam -o ./results-DNA-Pieter-17/iRep-DNA-Pieter-17_bin1 -t 10 -ff --sort

# For sample 16
iRep -f ./bins/Bin_1-contigs.fa -s ./map-DNA-Pieter-16/Bin_1.sam -o ./results-DNA-Pieter-16/iRep-DNA-Pieter-16_bin1 -t 10 -ff --sort
```
iRep normally tries to connect to a remote display, which requires you to go through VNC. To avoid this run the iRep commands like this in a PBS script:

```
xvfb-run -e ./xvfb.${PBS_JOBID}.err -f ./xvfb.${PBS_JOBID}.auth -w 10 -s "-screen 0 1600x1200x24" iRep -f ./bins/Bin_1-contigs.fa -s ./map-DNA-Pieter-16/Bin_1.sam -o ./results-DNA-Pieter-16/iRep-DNA-Pieter-16_bin1 -t 10 -ff --sort
```
