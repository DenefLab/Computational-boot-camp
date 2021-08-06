=============================================
Shared Resources
=============================================

The shared resources is a collection of commonly used databases,
containers, conda envs, workflows, and scripts. It serves to create
a common location for resources that most people will use and remove
the complexity of installing things on your own and keep multiple copies
of databases from existing. 

This shared directory is read only for most users, but feel free to request 
that things be added that you think others will benefit from. This includes 
a database you made, or scripts you write.

The directory can be found here:
.. code-block:: bash

    conda activate /nfs/turbo/lsa-dudelabs/

Conda envs
============
If you would like, you can edit your bashrc file to use this conda installation
as your default and then you can activate the envs by name alone. Doing this means
that you will be unable to make conda envs though (i think) and you can only use 
ones that exist in the shared directory all ready. 

How to activate
----------------
If you would like to have your own conda installation you can also do that and 
simply activate the conda envs as follows:
.. code-block:: bash

    conda activate /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/ENV_NAME/

Currently available envs
------------------------
The current conda envs available to use are:

- base                     /nfs/turbo/lsa-dudelabs/conda_envs/miniconda
- anvio                    /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/anvio
- anvio-7                  /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/anvio-7
- aspera                   /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/aspera
- checkv                   /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/checkv
- concoct                  /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/concoct
- globus                   /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/globus
- mamba                    /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/mamba
- metaG_pipeline        *  /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/metaG_pipeline
- nextflow                 /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/nextflow
- oligotyping              /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/oligotyping
- operams                  /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/operams
- pyani_env                /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/pyani_env
- quast                    /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/quast
- snakemake                /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/snakemake
- snakemake2               /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/snakemake2
- vcontact2                /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/vcontact2
- vibrant                  /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/vibrant
- virsorter                /nfs/turbo/lsa-dudelabs/conda_envs/miniconda/envs/virsorter
  
Databases
=========
The current databases are split up into 16S, functional, genomes, nt, proteins. It is also the location of data
stored for software such as checkv and virsorter. 

The current available databases and the directories they are under are.

- 16S 
    - Freshwater
    - green_genes
    - silva  
- functional (functional databases have hmm, blast, and diamond versions)  
    - COGs 
    - eggNOGs
    - Tigrfam
    - pfam
    - kegg
    - kofam 
- genomes 
    - refseq  
- nt 
- proteins
    - NR  
- centrifuge_indexes
    - p+h+v
- checkv
- virsorter

Containers
==========
Most software is stored in singularity containers. THis includes the container image as well as the definition
file used to create the container.

See the container section for more information on running these containers on your own.

Available containers:

- bbtools
- bwa
- checkm
- checkv
- concoct
- fastqc
- kofamscan
- megahit
- mothur
- nanopore
- subreads
- vcontact2
- vibrant
- virfinder
- virsorter
