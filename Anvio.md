
**For detailed documentation go [here](http://merenlab.org/2016/06/22/anvio-tutorial-v2/)**

# Basic visualization
The best way of doing this for smaller assemblies is to run everything in an interactive job.
```
qsub -I -V -A vdenef_fluxm -q fluxm -l nodes=1:ppn=20,mem=500gb,walltime=24:00:00,qos=flux
```
First make sure that the headers of the assembly fasta are formatted correctly
```
anvi-script-reformat-fasta contig.fa -o contigs-fixed.fa -l 0 --simplify-names
```
Then make an anvio database from your contigs 
```
anvi-gen-contigs-database -f contigs-fixed.fa -o contigs.db
```
Run HMMER for future potential analyses
```
anvi-run-hmms -c contigs.db --num-threads 20
```
Sort and index your mapped read files (can skip if you already did this before).
```
anvi-init-bam SAMPLE-raw_16.bam -o SAMPLE_16.bam
anvi-init-bam SAMPLE-raw_17.bam -o SAMPLE_17.bam
```
Profile your mapped reads
```
anvi-profile -i SAMPLE_17.bam -c contigs.db --min-contig-length 1000 --output-dir SAMPLE_17 --sample-name SAMPLE_17 --min-coverage-for-variability 5
anvi-profile -i SAMPLE_16.bam -c contigs.db --min-contig-length 1000 --output-dir SAMPLE_16 --sample-name SAMPLE_16 --min-coverage-for-variability 5
```
Merge all your samples in a single database
```
anvi-merge SAMPLE_16/RUNINFO.cp SAMPLE_17/RUNINFO.cp -o SAMPLES-MERGED -c contigs.db
```

# Classification with centrifuge
The downside of this approach is that the classification will be provided at the deepest taxonomic level, and thus not allowing you to choose the taxonomic level in the visualization. Have made issue about this at their Github.
```
anvi-get-dna-sequences-for-gene-calls -c contigs.db -o gene-calls.fa
centrifuge -f -x ~/centrifuge/p+h+v/p+h+v gene-calls.fa -S centrifuge_hits.tsv -p 40
```

Then import the output files into the contig database
```
anvi-import-taxonomy -c contigs.db -i centrifuge_report.tsv centrifuge_hits.tsv -p centrifuge
```

Run anvi'o in interactive mode. This will automatically cluster your contigs with CONCOCT.
```
anvi-interactive -p SAMPLES-MERGED/PROFILE.db -c contigs.db --taxonomic-level t_order
```

You can then output the summaries of each bin:
```
anvi-summarize -p SAMPLES-MERGED/PROFILE.db -c contigs.db -o SAMPLES-SUMMARY -C CONCOCT
```
# Refine bin
```
anvi-refine -p SAMPLES-MERGED/PROFILE.db -c contigs.db -C CONCOCT_first3 -b Bin_2
```

# SNV analysis
Download this script for visualization:
```
wget https://raw.githubusercontent.com/meren/anvio-methods-paper-analyses/a57b0cee07e9dd6fc59892114f2ad5bb9df78215/SHARON_et_al/VARIABILITY_REPORTS/02_GEN_FIGURE_SUMMARY.R \
        -O visualize-SNVs.R
chmod +x visualize-SNVs.R
```
The initial SNV characterisation for each sample has been performed in the <code>anvi-profile</code> step. We just need to evaluate these patterns across samples. We use the <code>--report-variability-full</code> flag to report all positions of variability and not just the ones that exceed a certain threshold. In this case we specify a bin of interest to do this analysis on.
```
anvi-gen-variability-profile -p SAMPLES-MERGED/PROFILE.db -c contigs.db -C CONCOCT_first3 -b Bin_2  --min-coverage-in-each-sample 20 --quince-mode --include-split-names -o bin2-SNVs.txt
anvi-gen-variability-profile -c contigs.db -p SAMPLES-MERGED/PROFILE.db -C CONCOCT_first3 -b Bin_2 -o bin2-SNV-density.txt
```
Visualize results
```
/nfs/vdenef-lab/Shared/Ruben/scripts_metaG/Ruben/visualize-SNVs.R bin2-SNVs.txt bin2-SNV-density.txt 150 3740000
```

Or you can enter interactive mode
```
anvi-script-snvs-to-interactive bin2-SNVs.txt -o bin2-SNVs
anvi-interactive -d bin2-SNVs/view_data.txt -s bin2-SNVs/samples.db -t bin2-SNVs/tree.txt -p bin2-SNVs/profile.db -A bin2-SNVs/additional_view_data.txt --title "SNV Profile for the Lhab-A4 bin" --manual
```

When you export figures during the interactive mode as SVG it is best to convert them to png for convenience:
```
inkscape --without-gui -f input.svg --export-png output.png -d 300 -D
```

# Importing binning from other software into anvi'o
Use this shell script to extract the contig names from each fasta file in the directory and combine them with the bin name (fasta file name) in one file (`bin_list.tsv`)
```
for file in *.fa
do
fasta=${file%.fa}
grep ">" ${fasta}.fa | sed "s/>//g" > contig_list_${fasta}
dos2unix contig_list_${fasta}
awk -F ' ' -v samname=$fasta '{$(NF+1)=samname;}1' OFS="\t" contig_list_${fasta} > contig_list_${fasta}_binned
cat *_binned >> bin_list.tsv
rm *_binned 
rm contig_list_*
done
```

Then import this information into anvi'o:
```
anvi-import-collection bin_list.tsv -p SAMPLES-MERGED/PROFILE.db -c contigs.db -C "VIZBIN" --contigs-mode
```

```
 anvi-script-merge-collections -c contigs.db -i vizbins/bin_list.tsv -o collections.tsv
```

# Pangenome analysis

1. Make list file for internal genomes (i.e. genomes that are present in an anvi'o database)

```
name	bin_id	collection_id	profile_db_path	contigs_db_path
bacIa_vizbin1	bacIa_vizbin1	CONCOCT_first3	/SAMPLES-MERGED/PROFILE.db	contigs.db
bacIa_vizbin2	bacIa_vizbin2	CONCOCT_first3	/SAMPLES-MERGED/PROFILE.db	contigs.db
```

2. Create `h5` storage file
```
anvi-gen-genomes-storage -o PAN-GENOMES.h5 -i internal_genome_list

```

3. Run pangenome analysis
```
anvi-pan-genome -g PAN-GENOMES.h5 -J TEST_PANGENOME --num-threads 20
```
