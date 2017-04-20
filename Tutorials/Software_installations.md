# General software installations

**IMPORTANT:** Make sure to reconnect to the server or <code> source .bash_profile</code> in order to update your path.

### Modules required for quality trimming of reads.
Install trimmomatic in your path (http://www.usadellab.org/cms/?page=trimmomatic). We will use the adapter databases available there to do adater trimming with scythe. Unzip binary (source may give difficulties) in your local directory:
```
unzip Trimmomatic-Src-0.36.zip
```
and add it to your path (.bash_profile):
```
export PATH=/home/yourusername/Trimmomatic-0.36:$PATH
```
Check if permissions allow execution.
```
cd ./Trimmomatic-0.36
ls -l
```
```
module load Scythe/0.993b sickle/1.33.6
```
Scythe is not installed on flux so install it in your home directory and add it to the path similar to Trimmomatic. 
```
cd
git clone https://github.com/vsbuffalo/scythe.git
cd scythe
make all
```
Also install Fastqc binary from here: http://www.bioinformatics.babraham.ac.uk/projects/download.html#fastqc. And add to path (see trimmomatic) and set permissions.
```
cd ./FastQC
chmod 755 fastqc
```
Next to samtools also install bedtools:
```
wget https://github.com/arq5x/bedtools2/releases/download/v2.26.0/bedtools-2.26.0.tar.gz
tar -zxvf bedtools-2.26.0.tar.gz
cd bedtools2
make
```
### Assembler modules
Install Ray assembler (get latest version from https://github.com/sebhtml/Ray-Releases/). Make sure to have <code>openmpi</code> and <code>gcc</code> loaded. And add to path.
```
cd 
tar xjf Ray-2.3.1.tar.bz2
cd Ray-2.3.1
make
cp Ray /home/user/bin
```
**IMPORTANT** Also install IDBA-UD (https://github.com/loneknightpy/idba) locally since the central module on flux will only process reads of up to 128 nt. So if you have 150 bp paired-end reads, a segmentation fault will occur. To circumvent this you will need to compile IDBA-UD locally, change the scr/sequence/short_sequence.h script as recommended [here] (http://i.cs.hku.hk/~alse/hkubrg/projects/idba_ud/)
```
module load gcc
./configure
make
# At this point you can change the scr/sequence/short_sequence.h script
# After you changed it reconfigure and make
./configure
make
```
### Module for random subsampling of fastq files

Install setqk for randomly subsampling fastq files and add to path.
```
cd
git clone https://github.com/lh3/seqtk.git
cd seqtk
make
cp seqtk /home/user/bin
```
You can also install <code>BBmap</code> in case you want to do digital normalization or have issues with co-assemblies (f.e., memory issues): install from [here] (https://sourceforge.net/projects/bbmap/)
### CheckM
```
pip install checkm-genome --user
```
**IMPORTANT:** If it's been a while that you've used CheckM, update data files:
```
checkm data update
```
### Phylosift
Phylosift for taxonomic classification of contigs/bins: find instructions [here] (https://github.com/gjospin/PhyloSift).

**Notice** For large assemblies with long contigs, during the alignment memory limits may be exceeded of cmalign. To avoid this follow [these] (https://github.com/gjospin/PhyloSift/issues/213) guidelines (the current memory limit for v1.0.1 is set at 2500Mb).

### Diamond
Required for fast blast searches.
```
wget http://github.com/bbuchfink/diamond/releases/download/v0.8.33/diamond-linux64.tar.gz
tar xzf diamond-linux64.tar.gz
```

### Centrifuge
```
git clone https://github.com/infphilo/centrifuge
cd centrifuge
git checkout 30e3f06ec35bc83e430b49a052f551a1e3edef42
make
```
Then download databases (9Gb)
```
mkdir p+h+v
cd p+h+v
wget ftp://ftp.ccb.jhu.edu/pub/infphilo/centrifuge/data/p+h+v.tar.gz
tar -zxvf p+h+v.tar.gz && rm -rf p+h+v.tar.gz
```

# Functional annotation
### Prokka
```
cpan Time::Piece XML::Simple Bio::Perl Digest::MD5

```
