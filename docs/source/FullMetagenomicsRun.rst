A Full Metagenomics Run 
=========================
Here is a full metagenomics run example. It will have code both using and not using the pipline.
Have tips of where to compress and move outputs. And show how to work with multiple assemblies and 
bins sets. It will also have notes on what to be thinking about as you move through each step.

1. Create your working directory
----------------------------------
This is the root of your project. Here you will create directories for each analysis you perform. This makes
it easy to compress and move folders when you complete your analysis and no longer need the intermediate files.
It also allows you to move pieces of your analysis to avoid the purge.

::
  mkdir your_project_name

once you make your directory you can move into it

::
  cd your_project_name

2. Move a copy of your raw reads to your working directory
----------------------------------------------------------
Create a directory in your working directory for your raw reads
::
  mkdir raw_reads

Once created you can copy all of your fastq files to the directory
::
  cp path/to/fastqs/*.fastq.gz raw_reads

3. Run QC on each set of your reads
------------------------------------
a. To QC one sample
::
  1. (optional) fastqc to check reads before qc
  fastqc raw_reads/*R1*
  fastqc raw_reads/*R2*
  2. trim and filter reads for quality
  3. remove adapters
  4. remove exact duplicates
  5. screen for contamination
  6. fastqc qced reads to make sure qc was a success
  7. removed intermediate files
   
b. To use the pipeline on a set of samples
 1. Create a yaml file with a header for each set of reads and the path to its corresponding r1 and r2 files like below:
   
   .. code-block:: yaml 
    sample_1:
     raw_r1: raw_reads/sample_1/*R1*
     raw_r2: raw_reads/sample_1/*R2*
    sample_2:
     raw_r1: raw_reads/sample_1/*R1*
     raw_r2: raw_reads/sample_1/*R2*

 The header you choose here will become the key connected to the files paths for all read files, so this will be the name your reference in later step when a set of reads is needed.

 2. Once the yaml is made simply run the QC module of the pipeline: 
  
  ::
      mgjss qc fastqs.yml qc_output --account vdenef1

 Here you supply the yaml you made above, output directory name and the account you wish to submit jobs under. This will create the directory for qc_output, and a directory for each Sample named with all generated files for that sample inside.
 The fastqs.yml will also be updated with the final qced_r1 and qced_r2 files that will be used for assembly and mappings. It will also produce a multiqc report containing all samples to check before moving on to assembly

4. Assembly
------------
 a. To assembly one sample
 
 ::
     1. Run megahit
     2. Run Assembly stats

 b. To assemble many samples with the pipeline
  1. Create the assembly_scheme.yml file. The assembly scheme is another yaml file where the header with the the name of the output assembly and a list of all sample names to include in the assembly. This allows for both single and coassemblies in the
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
        mgjss assemble fastqs.yml assembly_scheme.yml assembly_output --account vdenef1

  Similarly to above you provide the path to your fastq files, the assembly scheme, the output directory, and the account to run under. The pipeline will make your output directory, and a directory inside of it for each assembly with their outputs inside.
  once the assembly is finished, the pipeline will also run stats.sh from bbtools to generate assembly stats for each assembly. Once you check the assembly stats, you can move on the mapping 

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
    
    2. Create an assembly paths yaml file where the header is an assembly name and under that is the path to the assembly
       .. code-block:: yaml
          # map reads from sample_1 and sample_2 to both assembly sample_1 and assembly sample_2
          sample_1:
           - path/to/assembly/final.contigs.fa
          sample_2:
           - path/to/assembly/final.contigs.fa
  
    3. Run the mapping module of the pipeline
      
      .. code-block:: bash
          mgjss map fastqs.yml assembly_paths.yml mapping_scheme.yml mapping_output --account vdenef1

    This will create your mapping output directory with a directory for each assembly. In each assembly directory there will be the sorted bam files and bam indexes produced by the pipeline. It will also create a bam_paths.yml file where each header is an assembly you mapped to followed by a list of
    samples you mapped to the ref with the associated bam file path like below:

    .. code-block:: yaml
          sample_1:
           sample_1: path/to/bam/
          sample_2:
           sample_2: path/to/bam/

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
          mgjss concoct assembly_paths.yml bam_paths.yml binning_scheme.yaml binning_output --account vdenef1

    This will create your binning output directory with a directory for each assembly binned. In each assembly directory there will be the binlist file and a directory of fasta files for each bin made.
    It will also run an initial checkm on these bins created and create a bin_paths.yml file where each header is the assembly binned and it is followd by the path to the binlist file from concoct.

    .. code-block:: yaml
          sample_1:
           - path/to/binlist/
          sample_2:
           - path/to/binlist/

7. Create ANVIO Databases for Manual Refinement
------------------------------------------------
    1. Create a anvio_scheme.yml file where each header is an assembly name you have binned and a list of all sample mappings you want to include in the coverage profile for anvio.
      
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
          mgjss assembly_paths.yml bam_paths.yml bin_paths.yml anvio_scheme.yml anvio_output --rename_contigs --account vdenef1

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

10. Competitively map to dereplicated bins
-------------------------------------------

ONce you have a set of unique bins, you can do the mapping the same as above. the difference here is you will merge all of the bins into one fasta file.
Make an assembly_paths file like above, but with the path to this new merged fasta. Make a mapping scheme as well where the header is the new merged fasta and
the following list is all the samples you want to map to it.

11. Create final merged anvio Databases
-------------------------------------------

