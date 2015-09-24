================================
Lab3: Processing fastQ and fastA
================================

During this lab, we will acquaint ourselves with the the software packages FastQC and JellyFish. Your objectives are:


1. Familiarize yourself with the software, how to execute it, how to visualize results.

2. Regarding your dataset. Characterize sequence quality.

The FastQC manual:http://solexaqa.sourceforge.net/

The JellyFish manual: ftp://ftp.genome.umd.edu/pub/jellyfish/JellyfishUserGuide.pdf


> Step 1: Launch and AMI. For this exercise, we will use a c3.xlarge instance. Remember to change the permission of your key code `chmod 400 ~/Downloads/????.pem` (change ????.pem to whatever you named it)

::

	ssh -i ~/Downloads/?????.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com


> Update Software

::

  sudo apt-get update && sudo apt-get -y upgrade


> Install other software

::

  sudo apt-get -y install tmux git curl gcc make g++ python-dev unzip default-jre libboost1.55-all python-pip


> Ok, for this lab we are going to use FastQC. There is a version available on apt-get, but it is an old version and we want to make sure that we have the most updated version.. **Make sure you know what each of these commands does, rather than blindly copying and pasting..**

::

  cd $HOME
  curl -LO https://cran.r-project.org/src/base/R-3/R-3.2.2.tar.gz
  tar -zxf R-3.2.2.tar.gz
  cd R-3.2.2/
  ./configure --with-x=no
  make -j4
  sudo make all install

::

    cd $HOME
    git clone
    cd solexaqa-code
    make
    PATH=$PATH:$(pwd)


> Download data, and uncompress them.. What does the `-cd` flag mean WRT gzip??

::

  mkdir reads && cd reads
  curl -L https://s3.amazonaws.com/NYGC_August2015/raw_data/382-Kidney_ACTTGA_BC6PR5ANXX_L008_001.R1.fastq.gz > kidney.1.fq.gz 
  curl -L https://s3.amazonaws.com/NYGC_August2015/raw_data/382-Kidney_ACTTGA_BC6PR5ANXX_L008_001.R2.fastq.gz > kidney.2.fq.gz  


> Start SolexaQA running

::

  tmux new -s solexa
  cd $HOME && mkdir read_analysis && cd read_analysis 
  SolexaQA++ analysis -p 0.1  ~/reads/kidney.1.fq.gz ~/reads/kidney.2.fq.gz
  ctl-b d


> While SolexaQA is working, lets install khmer.. Again, make sure you know what each of these commands does, rather than just copying and pasting..

::

  cd $HOME
  sudo pip install --upgrade setuptools
  git clone https://github.com/dib-lab/khmer.git
  cd khmer
  make -j4
  sudo make all install


> Run khmer. Make sure to look at the manual.

::

  tmux new -s khmer
  mkdir khmer && cd khmer
  
  interleave-reads.py ~/reads/kidney.1.fq.gz ~/reads/kidney.2.fq.gz \
  | abundance-dist-single.py --threads 8 -M 2000000000 -k 25 - reads.hist

  ctl-b d


> Wait for these things to be done.. Use ``top -c`` to do this.. Remember ``q`` gets you outta ``top``.

> Open up a new terminal window using the buttons command-t

::

    scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:~/read_analysis/*pdf ~/Downloads/
    scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:/read_analysis/*quality ~/Downloads/
    scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:/read_analysis/*hist ~/Downloads/


> Now, on your MAC, find the PDF files you just downloaded.. 


> Now look at the ``.quality`` and ``.hist`` file.  which is the plot of quality containing both the mean quality as well as that for each tile. I want you to plot the distribution using R and RStudio.



> OPEN RSTUDIO

::

    #Import Data
    histo <- read.table("~/Downloads/Pero360B.histo", quote="\"")
    head(histo)
    
    #Plot
    plot(histo$V2 ~ histo$V1, type='h')
    
    #That one sucks, but what does it tell you about the kmer distribution?
    
    #Maybe this one is better?
    plot(histo$V2 ~ histo$V1, type='h', xlim=c(0,100))
    
    #Better. what is xlim? Maybe we can still improve? 
    
    plot(histo$V2 ~ histo$V1, type='h', xlim=c(0,500), ylim=c(0,1000000))
    
    #Final plot
    
    plot(histo$V2 ~ histo$V1, type='h', xlim=c(0,500), ylim=c(0,1000000),
            col='blue', frame.plot=F, xlab='25-mer frequency', ylab='Count',
            main='Kmer distribution in brain sample before quality trimming')



> Done?
