========================================
Lab 5: setup for transcriptome assembly
========================================


During this lab, we will acquaint ourselves with digital normalization. You will:

1. Install software and download data

2. Quality and adapter trim data sets.

3. Assembly a transcriptome


The JellyFish manual: ftp://ftp.genome.umd.edu/pub/jellyfish/JellyfishUserGuide.pdf
The Khmer manual: http://khmer.readthedocs.org/en/v1.1
Skewer: http://www.biomedcentral.com/1471-2105/15/182
Seqtk: https://github.com/lh3/seqtk


> Step 1: Launch and AMI. For this exercise, we will use a **c4.4xlarge** instance. **ADD A 100GB HARD DRIVE TO YOUR INSTANCE**

::

	ssh -i ~/Downloads/?????.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com


> Update Software

::

	sudo apt-get update && sudo apt-get -y upgrade


> Install other software

::

	sudo apt-get -y install cmake sparsehash valgrind libboost-atomic1.55-dev libibnetdisc-dev ruby-full gsl-bin \
	libgsl0-dev libgsl0ldbl libboost1.55-all-dev libboost1.55-dbg subversion tmux git curl bowtie libncurses5-dev \
	samtools gcc make g++ python-dev unzip dh-autoreconf default-jre python-pip zlib1g-dev

> Install Rcorrector

::

  cd 
  git clone https://github.com/mourisl/Rcorrector.git
  cd Rcorrector
  make
  PATH=$PATH:$(pwd)

> install seqtk

::

  cd $HOME
  git clone https://github.com/lh3/seqtk.git
  cd seqtk
  make
  PATH=$PATH:$(pwd)

> Install Skewer

::

  cd $HOME
  git clone https://github.com/relipmoc/skewer.git
  cd skewer
  make
  PATH=$PATH:$(pwd)
  curl -LO https://s3.amazonaws.com/gen711/TruSeq3-PE.fa

> install trinity

::

  git clone git clone https://github.com/trinityrnaseq/trinityrnaseq.git
  cd trinityrnaseq/
  make -j8
  PATH=$PATH:$(pwd)
  
> install Khmer

::

  sudo easy_install -U setuptools
  sudo pip khmer

> download reads

::

  mkdir $HOME/reads && cd $HOME/reads
  curl -LO https://s3.amazonaws.com/gen711/1.subsamp_1.fastq
  curl -LO https://s3.amazonaws.com/gen711/1.subsamp_2.fastq

> trimming

::


  mkdir $HOME/trimming && cd $HOME/trimming

  seqtk mergepe $HOME/reads/1.subsamp_1.fastq $HOME/reads/1.subsamp_2.fastq \
    | skewer -l 25 -m pe --mean-quality 2 --end-quality 2 -t 8 -x $HOME/skewer/TruSeq3-PE.fa - -1 > $HOME/trimming/trim2.interleaved.fastq

> error correct

::

  perl ~/Rcorrector//run_rcorrector.pl -t 16 -k 25 -i $HOME/trimming/trim2.interleaved.fastq 


> Assemble with trinity. Even though using `--min_kmer_cov 2` seems reasonable (think about what I told you about unique kmers), this almost always makes the assembly worse, expecially in low coverage RNAseq datasets. 

::

  Trinity --seqType fq --min_kmer_cov 2 --max_memory 10G --left trim2.interleaved.cor.fq.1  --right trim2.interleaved.cor.fq.2 --CPU 16
