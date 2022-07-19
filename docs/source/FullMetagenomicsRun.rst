A Full Metagenomics Run 
=========================
Here is a full metagenomics run example. It will have code both using and not using the pipline.
Have tips of where to compress and move outputs. And show how to work with multiple assemblies and 
bins sets. It will also have notes on what to be thinking about as you move through each step.

To use the pipeline simply module load singularity and conda activate /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/metaG_pipeline/


1. Create your working directory
----------------------------------
This is the root of your project. Here you will create directories for each analysis you perform. This makes
it easy to compress and move folders when you complete your analysis and no longer need the intermediate files.
It also allows you to move pieces of your analysis to avoid the purge.

.. code-block:: bash

  mkdir your_project_name

once you make your directory you can move into it

.. code-block:: bash

  cd your_project_name

2. Move a copy of your raw reads to your working directory
----------------------------------------------------------
Create a directory in your working directory for your raw reads
.. code-block:: bash

  mkdir raw_reads

Once created you can copy all of your fastq files to the directory
.. code-block:: bash
  cp path/to/fastqs/*.fastq.gz raw_reads

3. Run QC on each set of your reads
------------------------------------
a. steps run in the pipeline
.. code-block:: bash

  1. fastqc to check reads before qc
  2. trim and filter reads for quality
  3. remove adapters
  4. remove exact duplicates
  5. screen for contamination
  6. fastqc qced reads to make sure qc was a success
  7. removed intermediate files
   
b. To use the pipeline on a set of samples
 1. Create a csv file with a header for each set of reads and the path to its corresponding r1 and r2 files like below:
   
   .. code-block:: csv 

    sample,fq1,fq2
    sample_1,raw_reads/sample_1/R1.fq,raw_reads/sample_1/R2.fq
    sample_2,raw_reads/sample_1/R1.fq,raw_reads/sample_1/R2.fq

 The sample name you choose here will become the key connected to the files paths for all read files, so this will be the name your reference in later step when a set of reads is needed.

 2. Once the yaml is made simply run the QC module of the pipeline: 
  
  .. code-block:: bash

      mgjss qc fastqs.csv qc_output --account vdenef1

 Here you supply the csv you made above, output directory name and the account you wish to submit jobs under. This will create the directory for qc_output, and a directory for each Sample named with all generated files for that sample inside. One this step is completed you will need to update the 
csv file with the columns qced_fq1 and qced_fq2. These columns will be the reads used in all of the following steps that require reads (assembly, mapping, etc) will also be updated with the final qced_r1 and qced_r2 files that will be used for assembly and mappings. It will also produce a multiqc report containing all samples to check before moving on to assembly

4. Assembly
------------
 a. steps in the assembly module
 
 .. code-block:: bash

     1. Run megahit
     2. Run Assembly stats

 b. To assemble many samples with the pipeline
  1. Create the assembly_scheme.yml file. The assembly scheme is a yaml file where the header with the the name of the output assembly and a list of all sample names to include in the assembly. This allows for both single and coassemblies in the
  same run.
     
     .. code-block:: yaml 

      #make a single assembly using reads from sample 1
      sample_1:
       - sample_1
      # make a single assembly using reads from sample 2
      sample_2:
       - sample_2
      # make a coassembly named coassembly with reads from both sample1 and sample 2
      coassembly:
       - sample_1
       - sample_2

  2. run the assembly module
    .. code-block:: bash

        mgjss assemble fastqs.csv assembly_scheme.yml assembly_output --account vdenef1

  Similarly to above you provide the path to your fastq files, the assembly scheme, the output directory, and the account to run under. The pipeline will make your output directory, and a directory inside of it for each assembly with their outputs inside. Once the assembly is finished, the pipeline will also run stats.sh from bbtools to generate assembly stats for each assembly. Once you check the assembly stats, you need to create a csv file for the 
fasta files made by the assembly step

   .. code-block:: csv 

    assembly,path
    sample_1,assembly_output/sample_1/final.contigs.fa
    sample_2,assembly_output/sample_2/final.comtigs.fa
    
  c. run the renaming module
    Make sure to load the anvio module on great lake (e.g., module load Bioinformatics anvio/7)
    
    .. code-block:: bash

        mgjss rename-contigs assemblies_set1.csv assemblies_to_rename.txt renamed_assemblies --account vdenef1
  
  In the code, assemblies_set1.csv is exactly the same file you make after assembly that points to the paths to the fasta files
assemblies_to_rename.txt is a list of assemblies you want to rename, one per line (the first column of the above csv if you want them all excluding the column label from the file (assembly))
renamed_assemblies is the output dir that will be made


5. All vs All mapping for differential coverage for binning
------------------------------------------------------------
  a. to map a sample to a ref
   
   .. code-block:: bash

       1. index ref
       2. map reads and convert output to sorted bam
       3. index sorted bam
  
  b. to map many samples to many refs with the pipeline
    1. Create a mapping_scheme.yml file where each header is an assembly name you made in the previous step and a list of all samples you want to map to it.
      
      .. code-block:: yaml

          # map reads from sample_1 and sample_2 to both assembly sample_1 and assembly sample_2
          sample_1:
           - sample_1
           - sample_2
          sample_2:
           - sample_1
           - sample-2
  
    2. Run the mapping module of the pipeline using the mapping scheme and csv file with the paths to your assemblies
      
      .. code-block:: bash

          mgjss map fastqs.csv assembly_paths.csv mapping_scheme.yml mapping_output --account vdenef1

    This will create your mapping output directory with a directory for each assembly. In each assembly directory there will be the sorted bam files and bam indexes produced by the pipeline. Once the mapping is complete, you will need to create a csv file with the sample that was mapped, the ref it was mapped to, and the path to the bam file

    .. code-block:: csv

         sample,ref,path 
         sample_1,assem_1,path/to/bam
         sample_2,assem_2,path/to/bam

6. Binning with concoct
-------------------------
a. to bin a single sample
   
   .. code-block:: bash

       1. cut up fasta
       2. generate coverage profile
       3. run concoct
       4. merge cut up contigs
       5. create bin fastas
  
  b. to bin many samples using many mappings using the pipeline
    1. Create a binning_scheme.yml file where each header is an assembly name you made in the previous step and a list of all sample mappings you want to include in the coverage profile.
      
      .. code-block:: yaml

          # bin both assembly sample_1 and assembly sample_2 using the bams from mapping sample_1 and sample_2 to them 
          sample_1:
           - sample_1
           - sample_2
          sample_2:
           - sample_1
           - sample-2
    

    2. Run the concoct module of the pipeline
      
      .. code-block:: bash

          mgjss concoct assembly_paths.csv bam_paths.csv binning_scheme.yaml binning_output --account vdenef1

    This will create your binning output directory with a directory for each assembly binned. In each assembly directory there will be the binlist file and a directory of fasta files for each bin made. It will also run an initial checkm on these bins created. Once done you will need to create a csv file
that points to the binlists created by concoct to use in anvio.

    .. code-block:: csv

          # bin both assembly sample_1 and assembly sample_2 using the bams from mapping sample_1 and sample_2 to them 
          assembly,binlist
          sample_1,path/to/clustering_merged.csv
          sample_2,path/to/clustering_merged.csv

7. Create ANVIO Databases for Manual Refinement
------------------------------------------------
    1. Create a anvio_scheme.yml file where each header is an assembly name you have binned and a list of all sample mappings you want to include in the coverage profile for anvio. you will also need to module load samtools and anvio.
      
      .. code-block:: yaml

          # bin both assembly sample_1 and assembly sample_2 using the bams from mapping sample_1 and sample_2 to them 
          sample_1:
           - sample_1
           - sample_2
          sample_2:
           - sample_1
           - sample-2
    

    2. Run the concoct module of the pipeline
      
      .. code-block:: bash

          mgjss anvio assembly_paths.csv bam_paths.csv bin_paths.csv anvio_scheme.yml anvio_output --account vdenef1

    This will create your binning output directory with a directory for each assembly binned. In each assembly directory there will be the binlist file and a directory of fasta files for each bin made.
    It will also run an initial checkm on these bins created and create a bin_paths.yml file where each header is the assembly binned and it is followd by the path to the binlist file from concoct.

8. Manually refine bins in anvio
----------------------------------

9.  Merge and dereplicate bin sets
----------------------------------
Once you have a set of bins for each assembly you manually refined. simply add the sample name to the bin fasta name (you can do this as you refine and export the bins in anvio) and copy them all into one directory.
Then do the following:
   
   .. code-block:: bash

       #run pyani on all the bins
       #combine checkm tables
       #convert the pyani tables using the convert_table script from https://github.com/jtevns/Pairwise_Dereplication
       python convert_table.py ANI.tab COV.tab
       #run the dereplication code
       python Select_Unique_Genomes.py pairwise_long.txt binstats.txt ANI_thresh COV_thresh

you will recieve a list of genomes with the best checkm stats

At this point you have a set of bins to do downstream analysis with. This could be a pangenome analysis, annotation, etc. Much of this can be done in anvio.

10. Competitively map to dereplicated bins (an example of using the pipeline for other steps)
----------------------------------------------------------------------------------------------

Once you have a set of bins to do downstream analysis simply concatenate them all into a single fasta file. make an assembly csv file like above with the path to that fasta file, and make a mapping scheme file with the new "assembly" and all the samples you would like to map to the bins. Finally run mapping like above.


11. Some extra notes
----------------------
Any files that are not put into a csv file used by the pipeline are not required to move forward and can be moved once the step is finished. If you want to perform extra steps such as normalization of reads, just do that on the qced output and then use the normalized read files in the qced_fq1 and qced_fq2 columns of the fastq.csv files. File names do not matter.

