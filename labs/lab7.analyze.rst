====================================
Lab 7: Analyze transcriptome
====================================



BUSCO: http://busco.ezlab.org/ and http://bioinformatics.oxfordjournals.org/content/31/19/3210


> Step 1: Launch and AMI. For this exercise, we will use a **c4.2xlarge** instance. **ADD A 100GB HARD DRIVE TO YOUR INSTANCE**

::

	ssh -i ~/Downloads/?????.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com


> Update Software

::

	sudo apt-get update && sudo apt-get -y upgrade


> Install other software

::

	sudo apt-get -y install hmmer cmake tmux git curl bowtie libncurses5-dev samtools gcc make ncbi-blast+ g++ python-dev libboost-iostreams-dev libboost-system-dev libboost-filesystem-dev

> Install busco

::

  cd 
  curl -LO http://busco.ezlab.org/files/BUSCO_v1.1b1.tar.gz
  tar -zxf BUSCO_v1.1b1.tar.gz
  cd BUSCO_v1.1b1
  PATH=$PATH:$(pwd) 
  curl -LO http://busco.ezlab.org/files/metazoa_buscos.tar.gz
  tar -zxf metazoa_buscos.tar.gz

> install augustus

::

  cd
  curl -LO http://bioinf.uni-greifswald.de/augustus/binaries/old/augustus-3.0.2.tar.gz
  tar -zxf augustus-3.0.2.tar.gz
  cd augustus-3.0.2/
  make
  PATH=$PATH:$(pwd)/bin:$(pwd)/scripts
  export AUGUSTUS_CONFIG_PATH=/home/ubuntu/augustus-3.0.2/config/


> Download assemblies from last week.
::

  mkdir $HOME/assembly && cd $HOME/assembly
  curl -LO https://s3.amazonaws.com/gen711/trim2.corr.trinity.Trinity.fasta
  curl -LO https://s3.amazonaws.com/gen711/trim20.corr.trinity.Trinity.fasta

> Run BUSCO: do ONE of these.. coordinate with your neighbor so that s/he is doing the other. 

::


  mkdir $HOME/busco && cd $HOME/busco
  ln -s $HOME/BUSCO_v1.1b1/metazoa .

  #do this one

  python3 ~/BUSCO_v1.1b1/BUSCO_v1.1b1.py -g ../assembly/trim20.corr.trinity.Trinity.fasta -m Trans --cpu 8 -o trim20 -l metazoa

  #or this one

  python3 ~/BUSCO_v1.1b1/BUSCO_v1.1b1.py -g ../assembly/trim2.corr.trinity.Trinity.fasta -m Trans --cpu 8 -o trim2 -l metazoa

> look at BUSCO results

::

  #do whichever of these commands that pertain to the BUSCO run you chose to do above. 

  more run_trim20/short*
  more run_trim2/short*

> What is a BUSCO anyway - see http://bioinformatics.oxfordjournals.org/content/31/19/3210. How do the assemblies differ?

=======================
TERMINATE YOUR INSTANCE
=======================
