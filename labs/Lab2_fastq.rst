================================
Lab4: Processing fastQ
================================

During this lab, we will acquaint ourselves with the the software packages SolexaQA and khmer. Your objectives are:


1. Familiarize yourself with the software, how to execute it, how to visualize results.

2. Regarding your dataset. Characterize sequence quality.

The SolexaQA manual:http://solexaqa.sourceforge.net/

The khmer manual: http://khmer.readthedocs.org/en/v2.0/


> Step 1: Launch and AMI. For this exercise, we will use a c4.2xlarge instance. **IMPORTANT DETAIL!!!!!!!!!** We need to have more hard drive space for this exercise. So when you select the machine type, don't click ``Review and Launch`` like you normally do. Instead, go to near the top of teh page and click ``4. Add Storage``. You'll see a column labelled ``Size (GiB)`` with a 8 under that.. Change that 8 to 100. Now click ``Review and Launch`` like normal. You now have a computer with a hard drive of size 100Gb. 

If you have to make a new ``pem`` code, remember to change the permission of your key code `chmod 400 ~/Downloads/????.pem` (change ????.pem to whatever you named it)

::

	ssh -i ~/Downloads/?????.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com


> Update Software

::

  sudo apt-get update && sudo apt-get -y upgrade


> Install other software

::

  sudo apt-get -y install tmux git curl gcc make g++ python-dev unzip default-jre libboost1.55-all python-pip gfortran libreadline-dev


> Install R. R is a stats program, better than SPSS or JMP or whatever other software. There is somewhat of a steep learning curve, however.

::

  cd $HOME
  curl -LO https://cran.r-project.org/src/base/R-3/R-3.2.2.tar.gz
  tar -zxf R-3.2.2.tar.gz
  cd R-3.2.2/
  ./configure --with-x=no
  make -j4
  sudo make all install
  cd $HOME

> Ok, for this lab we are going to use SolexaQA. There is a version available on apt-get, but it is an old version and we want to make sure that we have the most updated version.. **Make sure you know what each of these commands does, rather than blindly copying and pasting..**


::

    cd $HOME
    git clone git://git.code.sf.net/p/solexaqa/code solexaqa-code
    cd solexaqa-code
    make
    PATH=$PATH:$(pwd)


> Download data, and uncompress them..

::

  cd $HOME && mkdir reads && cd reads
  curl -L https://s3.amazonaws.com/Mc_Transcriptome/Thomas_McBr1_R1.PF.fastq.gz > kidney.1.fq.gz 
  curl -L https://s3.amazonaws.com/Mc_Transcriptome/Thomas_McBr1_R2.PF.fastq.gz > kidney.2.fq.gz  


> Start SolexaQA running in a tmux window, then close it so we can work on other things. 

::

  tmux new -s solexa
  cd $HOME && mkdir read_analysis && cd read_analysis 
  SolexaQA++ analysis -p 0.1  ~/reads/kidney.1.fq.gz ~/reads/kidney.2.fq.gz
  ctl-b d


> While SolexaQA is working, lets install khmer, a Python package for working with kmers. Again, make sure you know what each of these commands does, rather than just copying and pasting..

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
  cd $HOME && mkdir khmer_analysis && cd khmer_analysis
  
  interleave-reads.py ~/reads/kidney.1.fq.gz ~/reads/kidney.2.fq.gz > kidney.fq
  abundance-dist-single.py --threads 8 -M 2000000000 -k 25 kidney.fq reads.hist


  ctl-b d


> Wait for these things to be done.. Use ``top -c`` to do this.. Remember ``q`` gets you outta ``top``.

> Open up a new terminal (tab) or window using the buttons command-t. You're going to download the files you created on teh AWS machine to the MAC your using in the lab. 

::

    scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:~/reads/*pdf ~/Downloads/
    scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:~/reads/*quality ~/Downloads/
    scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:~/khmer_analysis/reads.hist ~/Downloads/


> Now, on your MAC, find the PDF files you just downloaded.. Open them up and see what they look like. Can you figure out what they mean? 


> Now look at the ``.quality`` and ``.hist`` file.  which is the plot of quality containing both the mean quality as well as that for each tile. I want you to plot the distribution using R and RStudio.



> OPEN RSTUDIO - this should be instaled on your Mac. 

::

    #Import Data
    histo <- read.csv("~/Downloads/reads.hist", quote="\"")
    head(histo)
    
    #Plot
    plot (histo$cumulative_fraction ~ histo$abundance)
    
    #That one sucks, but what does it tell you about the kmer distribution?
    
    #Maybe this one is better?
    plot (histo$cumulative_fraction[1:10] ~ histo$abundance[1:10])
    
    #Final plot
    
    plot(histo$cumulative_fraction[1:10] ~ histo$abundance[1:10], type='l', lwd=5,
            col='blue', frame.plot=F, xlab='25-mer frequency', ylab='Cumulative Fraction',
            main='Kmer distribution in brain sample before quality trimming')

