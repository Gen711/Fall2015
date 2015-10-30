========================================
Lab 7: Analyze transcriptome
========================================



BUSCO: http://busco.ezlab.org/


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

> Install busco

::

  cd 
  curl -LO http://busco.ezlab.org/files/BUSCO_v1.1b1.tar.gz
  tar -zxf BUSCO_v1.1b1.tar.gz
  cd BUSCO_v1.1b1
  PATH=$PATH:(pwd) 
  curl -LO http://busco.ezlab.org/files/metazoa_buscos.tar.gz
  tar -zxf metazoa_buscos.tar.gz

> Download assemblies from last week.
::

  mkdir $HOME/assembly && cd $HOME/assembly
  curl -LO https://s3.amazonaws.com/gen711/1.subsamp_1.fastq
  curl -LO https://s3.amazonaws.com/gen711/1.subsamp_2.fastq

> Run BUSCO

::


  mkdir $HOME/busco && cd $HOME/busco
  ln -s $HOME/BUSCO_v1.1b1/metazoa .



=======================
TERMINATE YOUR INSTANCE
=======================
