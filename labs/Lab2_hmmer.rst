===============
Lab3 : HMMER
===============

During this lab, we will acquaint ourselves with the the software package HMMER. Your objectives are:


1. Familiarize yourself with the software, how to execute it, how to visualize results.

2. Regarding your dataset. Characterize a few conserved domains.

The HMMER manual ftp://selab.janelia.org/pub/software/hmmer3/3.1b1/Userguide.pdf

The HMMER webpage: http://hmmer.janelia.org/


Step 1: Launch and AMI. For this exercise, we will use a c4.xlarge instance.

::

  ssh -i ~/Downloads/your.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com


Update your machine

::

  sudo apt-get update && sudo apt-get -y upgrade

Install new software

::

  sudo apt-get -y install tmux git curl gcc make g++ python-dev unzip default-jre


Ok, for this lab we are going to use HMMER

::

  cd $HOME
  curl -LO http://selab.janelia.org/software/hmmer3/3.1b1/hmmer-3.1b1-linux-intel-x86_64.tar.gz
  tar -zxf hmmer-3.1b1-linux-intel-x86_64.tar.gz
  cd hmmer-3.1b1-linux-intel-x86_64/
  ./configure
  make && sudo make all install
  make check
  cd


You will download the dataset. Also, download Swissprot and Pfam-A

::

  sudo chown ubuntu /mnt
  cd /mnt
  curl -LO https://s3.amazonaws.com/gen711/dataset1.pep

  #download the SwissProt database

  curl -LO https://s3.amazonaws.com/gen711/uniprot.pep

  #download the Pfam-A database

  curl -LO ftp://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.hmm.gz


We are going to run HMMER to identify conserved protein domains. This will take a little while, and we'll use ``tmux`` to allow us to do this in the background, and continue to work on other things.

::

  gzip -d Pfam-A.hmm.gz
  tmux new -s pfam
  hmmpress Pfam-A.hmm #this is analogous to 'makeblastdb'
  hmmscan -E 1e-3 --domtblout dataset.pfam --cpu 4 Pfam-A.hmm dataset1.pep
  ctl-b d
  top -c #see that hmmscan is running..


The neat thing about HMMER is that it can be used as a replacement for blastP or PSI-blast.

::

  #blastp-like HBB-HUMAN is a Hemoglobin B protein sequence. 
    
  phmmer --noali --domtblout hbb.phmmer -E 1e-5 /home/ubuntu/hmmer-3.1b1-linux-intel-x86_64/tutorial/HBB_HUMAN uniprot.pep
  
look at phmmer results

::

  more hbb.phmmer

PSI-blast-like

::
    
  jackhmmer --noali --domtblout hbb.jackhammer -E 1e-5 /home/ubuntu/hmmer-3.1b1-linux-intel-x86_64/tutorial/HBB_HUMAN uniprot.pep
    
look at jackhammer results

::

  more hbb.phmmer
    
Now let's look at the ``hmmscan``  results. This analyses may still be running, but we can look at it while it's still in progress.

::

    more dataset.pfam
    #There are a bunch of columns in this table - what do they mean?
    
    #Try to extract all the hits to a specific domain. Google a few domains (column 1) to see if any seem interesting. 
    
    #for instance, find all occurrences of ABC_tran
    grep ABC_tran dataset.pfam
    
    #use grep to count the number of matches. Copy this number down.
    
    grep -c ABC_tran dataset.pfam
    
    #Find all the contigs that have a ABC_tran domain. 
    
    grep ABC_tran dataset.pfam | awk '{print $4}' | sort -u
    

> Just for fun, check on the Pfam search to see what it is doing... 

::

    tmux attach -t pfam
    ctl-b d

