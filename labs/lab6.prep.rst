========================================
Lab 6: setup for transcriptome assembly
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

  cd
  git clone https://github.com/trinityrnaseq/trinityrnaseq.git
  cd trinityrnaseq/
  make -j8
  PATH=$PATH:$(pwd)
  
> install Khmer

::

  sudo easy_install -U setuptools
  sudo pip install khmer

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

  seqtk mergepe $HOME/reads/1.subsamp_1.fastq $HOME/reads/1.subsamp_2.fastq \
    | skewer -l 25 -m pe --mean-quality 20 --end-quality 20 -t 8 -x $HOME/skewer/TruSeq3-PE.fa - -1 > $HOME/trimming/trim20.interleaved.fastq

> error correct

::

  perl ~/Rcorrector//run_rcorrector.pl -t 16 -k 25 -i $HOME/trimming/trim2.interleaved.fastq 
  perl ~/Rcorrector//run_rcorrector.pl -t 16 -k 25 -i $HOME/trimming/trim20.interleaved.fastq 

> split the interleaved files back into R and L. 

::

  split-paired-reads.py -0 /dev/null trim2.interleaved.fastq
  split-paired-reads.py -0 /dev/null trim2.interleaved.cor.fq
  split-paired-reads.py -0 /dev/null trim20.interleaved.cor.fq

> Assemble with trinity. **IMPORTANT DETAIL** You're going to be doing 3 different assemblies, each of which takes ~1 hour. You could do this 1 at a time, checking back each hour to start the next, but I'm going to show you a new and better way.  

- To do this, copy the 3 commmands into a file you name ``run.sh``. Do this using ``nano``.
- once you have the commands copied into the file, exit out of nano and get into a tmux window. 
- once in the tmux window, execute the commands using this: ``sh run.sh``. They will run one after the other and finish in about 3 hours. 

::

  Trinity --output trim2.corr.trinity --full_cleanup --seqType fq --max_memory 20G --left trim2.interleaved.cor.fq.1  --right trim2.interleaved.cor.fq.2 --CPU 16
  Trinity --output trim20.corr.trinity --full_cleanup --seqType fq --max_memory 20G --left trim20.interleaved.cor.fq.1  --right trim20.interleaved.cor.fq.2 --CPU 16
  Trinity --output trim2.trinity --full_cleanup --seqType fq --max_memory 20G --left trim2.interleaved.fastq.1  --right trim2.interleaved.fastq.2 --CPU 16

> Fast forward 3 hours, things should be done.

> IN YOUR TMUX WINDOW, install Transrate

::

  cd
  curl -LO https://bintray.com/artifact/download/blahah/generic/transrate-1.0.1-linux-x86_64.tar.gz
  tar -zxf transrate-1.0.1-linux-x86_64.tar.gz
  PATH=$PATH:/home/ubuntu/transrate-1.0.1-linux-x86_64
  
> run transrate

::

  mkdir ~/transrate && cd transrate
  
  
  transrate --install-deps=ref
  transrate --left ../reads/1.subsamp_1.fastq --right ../reads/1.subsamp_2.fastq \
  --threads 16 \
  --assembly ../trimming/trim2.trinity.Trinity.fasta,../trimming/trim20.corr.trinity.Trinity.fasta,../trimming/trim2.corr.trinity.Trinity.fasta
 
> Download results, we'll use them next week

::

  scp -i your.pem ubuntu@---:/home/ubuntu/transrate/transrate_results/*csv ~/Downloads/
  scp -i your.pem ubuntu@---:/home/ubuntu/trimming/*fasta ~/Downloads/
