Basic Singularity usage
========================

The singularity documentation can be found here:
https://sylabs.io/guides/3.2/user-guide/index.html

Running software in existing containers
----------------------------------------
To run software in an existing container, simply 

.. code-block:: bash
    
    module load Singularity

Then add singularity exec to the front of the normal command. For Example:

.. code-block:: bash

    # normal megahit command 
    megahit -1 $r1 -2 $r2 -t 40 --presets meta-sensitive -o Megahit_meta-sensitive_out/
    # singularity megahit command 
    singularity path/to/container/megahit.sif megahit -1 $r1 -2 $r2 -t 40 --presets meta-sensitive -o Megahit_meta-sensitive_out/

To build a Container with singularity on Greatlakes
-----------------------------------------------------
1. Set up remote building with singularity
    - create account at cloud.sylabs.io
    - click on your profile in the top right corner
    - click on access token
    - add a name to the create new access token section (something that tells you where you are going to run from i name mine greatlakes)
    - click create
    - copy the resulting token
    - login to greatlakes
    - module load singularity
    - then run the following and follow the prompt:
     
     .. code-block:: bash
         singularity remote login 
    
    you must update this key every 30 days

2. Building a container from a conda install

Building a singularity container requires a def file. The def file contains the instructions to make this file and the following template will work for any software that can be installed using conda

    .. code-block:: bash

        Bootstrap: shub
        From: scleveland/centos7-base-singularity

        %environment
        export PATH=/opt/conda/bin:$PATH

        %post
        yum update -y
        yum  install -y @"Development Tools"
        yum install -y git curl which python3 python3-devel vim htop wget tar bzip2 gzip lz4 lzma mesa-libGL mesa-libGLU

        # install anaconda
        wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
        bash Miniconda3-latest-Linux-x86_64.sh -b -p /opt/conda
        export PATH=/opt/conda/bin:$PATH
        conda update -y conda
        conda init

        # install Bioinformatics tools through conda
        conda install -y -c bioconda -c conda-forge megahit

All you have to do is replace the conda install line with your conda install command. Makes sure to include the -y flag though. THis makes the install not require user input.