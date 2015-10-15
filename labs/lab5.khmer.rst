============
Lab 5: khmer
============


During this lab, we will acquaint ourselves with digital normalization. You will:

1. Install software and download data

2. Quality and adapter trim data sets.

3. Apply digital normalization to the dataset.

4. Count and compare kmers and kmer distributions in the normalized and un-normalized dataset.

5. Plot in RStudio.


The JellyFish manual: ftp://ftp.genome.umd.edu/pub/jellyfish/JellyfishUserGuide.pdf

The Khmer manual: http://khmer.readthedocs.org/en/v1.1


> Step 1: Launch and AMI. For this exercise, we will use a c3.xlarge instance. Remember to change the permission of your key code `chmod 400 ~/Downloads/????.pem` (change ????.pem to whatever you named it)

::

	ssh -i ~/Downloads/?????.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com

> Update Software

::

	sudo apt-get update && sudo apt-get -y upgrade


> Install other software

::

	sudo apt-get -y install tmux git curl gcc make g++ python-dev unzip default-jre libboost1.55-all python-pip gfortran libreadline-dev


> Install Skewer

::

  cd $HOME
  git clone https://github.com/relipmoc/skewer.git
  cd skewer
  make
  PATH=$PATH:$(pwd)

> Install Jellyfish

::

    cd $HOME
    wget ftp://ftp.genome.umd.edu/pub/jellyfish/jellyfish-2.1.3.tar.gz
    tar -zxf jellyfish-2.1.3.tar.gz
    cd jellyfish-2.1.3/
    ./configure
    make -j4
    PATH=$PATH:$(pwd)

> Install seqtk

::

  cd $HOME
  git clone https://github.com/lh3/seqtk.git
  cd seqtk
  make -j4
  PATH=$PATH:$(pwd)

> Install Khmer

::

    cd $HOME
    sudo pip install screed pysam
    sudo easy_install -U setuptools
    git clone https://github.com/ged-lab/khmer.git
    cd khmer
    make -j4
    sudo make install


> Download data and a file with the Illumina adapters, ``TruSeq3-PE.fa``

::

  cd $HOME
  curl -LO https://s3.amazonaws.com/gen711/TruSeq3-PE.fa
  mkdir -p $HOME/reads && cd $HOME/reads
  curl -L https://s3.amazonaws.com/Mc_Transcriptome/Thomas_McBr1_R1.PF.fastq.gz > kidney.1.fq.gz &
  curl -L https://s3.amazonaws.com/Mc_Transcriptome/Thomas_McBr1_R2.PF.fastq.gz > kidney.2.fq.gz


> Trim low quality bases and adapters from dataset. These files will form the basis of all out subsequent analyses.

::


  mkdir $HOME/trimming && cd $HOME/trimming
    
    
  trim=2
  norm=30

  #paste the below lines together as 1 command

  seqtk mergepe $HOME/reads/kidney.1.fq.gz $HOME/reads/kidney.2.fq.gz \
    | skewer -l 25 -m pe --mean-quality $trim --end-quality $trim -t 8 -x $HOME/TruSeq3-PE.fa - -1 \
    | jellyfish count -m 25 -F2 -s 700M -t 8 -C -o /dev/stdout /dev/stdin \
    | jellyfish histo /dev/stdin -o trimmed.no.normalize.histo

  #and

  #paste the below lines together as 1 command

  seqtk mergepe $HOME/reads/kidney.1.fq.gz $HOME/reads/kidney.2.fq.gz \
    | skewer -l 25 -m pe --mean-quality $trim --end-quality $trim -t 8 -x $HOME/TruSeq3-PE.fa - -1 \
    | normalize-by-median.py --max-memory-usage 2e9 -C 30 -o - - \
    | jellyfish count -m 25 -F2 -s 700M -t 8 -C -o /dev/stdout /dev/stdin \
    | jellyfish histo /dev/stdin -o trimmed.yes.normalize.histo


> Open up a new terminal window using the buttons command-t

::

	scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:/mnt/jelly/*histo ~/Downloads/


> Now, on your MAC, find the files you just downloaded - for the zip files - double click and that should unzip them.. Click on the `html` file, which will open up your browser. Look at the results. Try to figure out what each plot means.


> Now look at the `.histo` file, which is a kmer distribution. I want you to plot the distribution using R and RStudio.


> OPEN RSTUDIO

::

    #Import all 2 histogram datasets: this is the code for importing 1 of them..
    
    khmer <- read.table("~/Downloads/khmer.histo", quote="\"")
    trim <- read.table("~/Downloads/trimmed.histo", quote="\"")
    
    #What does this plot show you?? 
    
    barplot(c(trim$V2[1],khmer$V2[1]),
        names=c('Non-normalized', 'C50 Normalized'),
        main='Number of unique kmers')
    
    # plot differences between non-unique kmers
    
    plot(khmer$V2[10:300] - trim$V2[10:300], type='l',
        xlim=c(10,300), xaxs="i", yaxs="i", frame.plot=F,
        ylim=c(-10000,60000), col='red', xlab='kmer frequency',
        lwd=4, ylab='count',
        main='Diff in 25mer counts of \n normalized vs. un-normalized datasets')
    abline(h=0)



> What do the analyses of kmer counts tell you?

=======================
TERMINATE YOUR INSTANCE
=======================
