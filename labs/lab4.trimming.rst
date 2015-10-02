================================
Lab5: Trimming fastQ
================================


> Step 1: Launch and AMI. For this exercise, we will use a c4.2xlarge instance. **IMPORTANT DETAIL!!!!!!!!!** We need to have more hard drive space for this exercise. So when you select the machine type, don't click ``Review and Launch`` like you normally do. Instead, go to near the top of teh page and click ``4. Add Storage``. You'll see a column labelled ``Size (GiB)`` with a 8 under that.. Change that 8 to 100. Now click ``Review and Launch`` like normal. You now have a computer with a hard drive of size 100Gb. 

::

	ssh -i ~/Downloads/?????.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com

> Update Software

::

  sudo apt-get update && sudo apt-get -y upgrade

> Install other software

::

  sudo apt-get -y install tmux git curl gcc make g++ python-dev unzip default-jre libboost1.55-all python-pip gfortran libreadline-dev

> Download data, and uncompress them.. Let's put this in a tmux window so we can get to doing other things.. remember you need paste the tmux relevant commands one at a time. 

::

  tmux new -s download
  cd $HOME && mkdir reads && cd reads
  curl -LO https://s3.amazonaws.com/gen711/TruSeq3-PE.fa
  curl -L https://s3.amazonaws.com/Mc_Transcriptome/Thomas_McBr1_R1.PF.fastq.gz > kidney.1.fq.gz &
  curl -L https://s3.amazonaws.com/Mc_Transcriptome/Thomas_McBr1_R2.PF.fastq.gz > kidney.2.fq.gz

  #then this to get out of tmux

  ctl-b d

> Install khmer, a Python package for working with kmers. Again, make sure you know what each of these commands does, rather than just copying and pasting..

::

  cd $HOME
  sudo pip install --upgrade setuptools
  git clone https://github.com/dib-lab/khmer.git
  cd khmer
  make -j4
  sudo make all install

> Install Skewer, a trimming tool

::
  
  cd $HOME
  git clone https://github.com/relipmoc/skewer.git
  cd skewer
  make
  PATH=$PATH:$(pwd)

>Install seqtk

::

  cd $HOME
  git clone https://github.com/lh3/seqtk.git
  cd seqtk
  make -j4
  PATH=$PATH:$(pwd)

> now go back to the download tmux window to see if//wait for the download

::

  tmux at -t download

  #IMPORTANT - when it is done downloading, kill the window

  ctl-d

> Run khmer. Make sure to look at the manual.

::

  tmux new -s khmer
  cd $HOME && mkdir khmer_analysis && cd khmer_analysis
  
  #IMPORTANT DETAIL: usually pasting things in 1 at a time is fine - except here... When you see ``\`` at the end of lines, this means copy the 2 (or 3 or 4) lines together. 


  #Important detail #2: Dont do the trimming for trim=2, only to the trimming at trim=30. You can download the trim=2 histogram here: https://s3.amazonaws.com/gen711/P2.reads.hist

  #do trimming at P2

  trim=2
  seqtk mergepe ~/reads/kidney.1.fq.gz ~/reads/kidney.2.fq.gz \
  | skewer -l 25 -m pe --mean-quality $trim --end-quality $trim -t 8 -x $HOME/reads/TruSeq3-PE.fa - -o P$trim.kidney

  abundance-dist-single.py --threads 8 -M 6e9 -k 25 P$trim.kidney-trimmed.fastq P$trim.reads.hist


  #do trimming at P30

  trim=30
  seqtk mergepe ~/reads/kidney.1.fq.gz ~/reads/kidney.2.fq.gz \
  | skewer -l 25 -m pe --mean-quality $trim --end-quality $trim -t 8 -x $HOME/reads/TruSeq3-PE.fa - -o P$trim.kidney

  abundance-dist-single.py --threads 8 -M 6e9 -k 25 P$trim.kidney-trimmed.fastq P$trim.reads.hist

  # to exit out of the tmux window, if you want to. 

  ctl-b d


> Wait for these things to be done.. Use ``top -c`` to do this.. Remember ``q`` gets you outta ``top``.

> Open up a new terminal (tab) or window using the buttons command-t. You're going to download the files you created on teh AWS machine to the MAC your using in the lab. 

::

    scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:~/khmer_analysis/*hist ~/Downloads/

    #also, download the Phred2 trimmed histogram here: https://s3.amazonaws.com/gen711/P2.reads.hist

    curl -L https://s3.amazonaws.com/gen711/P2.reads.hist > ~/Downloads/P2.reads.hist

> Now, on your MAC, find the PDF files you just downloaded.. Open them up and see what they look like. Can you figure out what they mean? 


> Now look at the ``.quality`` and ``.hist`` file.  which is the plot of quality containing both the mean quality as well as that for each tile. I want you to plot the distribution using R and RStudio.



> OPEN RSTUDIO - this should be instaled on your Mac. These commands you'll type into RStudio, NOT the terminal.

::

    #Import Data
    p2 <- read.csv("~/Downloads/P2.reads.hist", quote="\"")
    p30 <- read.csv("~/Downloads/P30.reads.hist", quote="\"")
    
    par(mfcol=c(2,1))
    
    plot(p2$count[1:10] ~ p2$count[1:10], type='l', lwd=5,
            col='blue', frame.plot=F, xlab='25-mer frequency', ylab='Cumulative Fraction',
            main='Kmer distribution in sample with different trimming thresholds')

    lines(p30$count[1:10] ~ p30$count[1:10], type='l', lwd=5,
            col='red')

    plot(p2$count[2:30] - p30$count[2:30], type='l',
        xlim=c(2,20), xaxs="i", yaxs="i", frame.plot=F,
        ylim=c(0,2000000), col='red', xlab='kmer frequency',
        lwd=4, ylab='count',
        main='Diff in 25mer counts of freq 2 to 20 \n Phred2 vs. Phred30')


