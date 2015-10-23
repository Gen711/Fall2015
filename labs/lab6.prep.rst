========================================
Lab 5: setup for transcriptome assembly
========================================


During this lab, we will acquaint ourselves with digital normalization. You will:

1. Install software and download data

2. Quality and adapter trim data sets.

3. Apply digital normalization to the dataset.

4. Count and compare kmers and kmer distributions in the normalized and un-normalized dataset.

5. Plot in RStudio.


The JellyFish manual: ftp://ftp.genome.umd.edu/pub/jellyfish/JellyfishUserGuide.pdf
The Khmer manual: http://khmer.readthedocs.org/en/v1.1
Skewer: http://www.biomedcentral.com/1471-2105/15/182
Seqtk: https://github.com/lh3/seqtk


> Step 1: Launch and AMI. For this exercise, we will use a c4.2xlarge instance. **ADD A 100GB HARD DRIVE TO YOUR INSTANCE**

::

	ssh -i ~/Downloads/?????.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com


> Update Software

::

	sudo apt-get update && sudo apt-get -y upgrade


> Install other software

::

	sudo apt-get -y install tmux git curl gcc make g++ python-dev unzip default-jre libboost1.55-all python-pip gfortran libreadline-dev

> Install Rcorrector

::

  cd 
  git clone https://github.com/mourisl/Rcorrector.git
  cd Rcorrector
  make && sudo make all install
  PATH:$PATH:$(pwd)

> Install Skewer

::

  cd $HOME
  git clone https://github.com/relipmoc/skewer.git
  cd skewer
  make
  PATH=$PATH:$(pwd)

> Merge --> Trim low quality bases and adapters from dataset  --> count kmers --> make a histogram. Normalize in the 1nd command. Make sure you know what is going on here!

> download reads

::

  mkdir $HOME/reads && cd $HOME/reads
  curl -LO https://s3.amazonaws.com/gen711/1.subsamp_1.fastq
  curl -LO https://s3.amazonaws.com/gen711/1.subsamp_2.fastq

::


  mkdir $HOME/trimming && cd $HOME/trimming
    
::

  seqtk mergepe $HOME/reads/1.subsamp_1.fastq $HOME/reads/1.subsamp_2.fastq \
    | skewer -l 25 -m pe --mean-quality 2 --end-quality 2 -t 8 -x $HOME/TruSeq3-PE.fa - > $HOME/trimming/trim2.interleaved.fastq

::

  perl ~/Rcorrector//run_rcorrector.pl -t 8 -k 25 -i $HOME/trimming/trim2.interleaved.fastq 


