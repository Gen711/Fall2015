===================
Lab 8: Read Mapping
===================

During this lab, we will acquaint ourselves with de novo transcriptome assembly using Trinity. You will:

1. Install software and download data

2. Use sra-toolkit to extract fastQ reads

3. Map reads to dataset

4. look at mapping quality


The BWA manual: http://bio-bwa.sourceforge.net/ 

Flag info: http://broadinstitute.github.io/picard/explain-flags.html


> Step 1: Launch and AMI. For this exercise, we will use a c4.2xlarge machine. 

::

	ssh -i ~/Downloads/?????.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com



> Update Software


::

	sudo apt-get update && sudo apt-get -y upgrade

> Install other software

::

	sudo apt-get -y install subversion tmux git curl samtools gcc make g++ python-dev unzip dh-autoreconf default-jre zlib1g-dev


> INSTALL BWA

::

    cd $HOME
    git clone https://github.com/lh3/bwa.git
    cd bwa
    make -j4
    PATH=$PATH:$(pwd)
    echo 'PATH=$PATH:$(pwd)' >> ~/.profile


>INSTALL SRATOOLKIT

::


    cd $HOME
    wget http://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/2.4.2/sratoolkit.2.4.2-ubuntu64.tar.gz
    tar -zxf sratoolkit.2.4.2-ubuntu64.tar.gz
    PATH=$PATH:/home/ubuntu/sratoolkit.2.4.2-ubuntu64/bin
    echo '$PATH:/home/ubuntu/sratoolkit.2.4.2-ubuntu64/bin' >> ~/.profile

> Install SAMBAMBA

::

  cd $HOME
  curl -LO https://github.com/lomereiter/sambamba/releases/download/v0.5.8/sambamba_v0.5.8_linux.tar.bz2
  tar -jxf sambamba_v0.5.8_linux.tar.bz2
  sudo mv sambamba_v0.5.8 /usr/bin/sambamba_v0.5.8
  chmod a+x /usr/bin/sambamba_v0.5.8

> Download data

::

    mkdir $HOME/data
    cd $HOME/data
    curl -LO http://datadryad.org/bitstream/handle/10255/dryad.72141/brain.final.fasta
    curl -LO ftp://ftp-trace.ncbi.nlm.nih.gov/sra/sra-instant/reads/ByRun/sra/SRR/SRR157/SRR1575395/SRR1575395.sra


> Convert SRA format into fastQ (takes a few minutes)

::

	cd $HOME/data
	fastq-dump --split-files --split-spot SRR1575395.sra


> Map reads!! (20 minutes)

::

    mkdir $HOME/mapping
    cd $HOME/mapping
    tmux new -s mapping
    bwa index -p index $HOME/data/brain.final.fasta
    time bwa mem -t8 index $HOME/data/SRR1575395_1.fastq $HOME/data/SRR1575395_2.fastq \
    | sambamba_v0.5.8 view -l 0 -t 8 -S -f bam -o /dev/stdout /dev/stdin \
    | sambamba_v0.5.8 sort -l 0 -t 8 -m 15G -o brain.bam /dev/stdin


> Look at SAM file. 


::

    #Take a quick general look.

    sambamba_v0.5.8 view brain.bam | head
    


> look at mapping stats

::

  sambamba_v0.5.8 flagstat brain.bam

=======================
TERMINATE YOUR INSTANCE
=======================
