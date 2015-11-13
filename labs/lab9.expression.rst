======================
Lab 9: Gene Expression
======================

During this lab, we will acquaint ourselves with de novo transcriptome assembly using Trinity. You will:

1. Install software and download data

2. Use sra-toolkit to extract fastQ reads

3. Map reads to dataset

4. look at mapping quality





> Step 1: Launch and AMI. For this exercise, we will use a c4.2xlarge machine. Add 100Gb storage.

::

	ssh -i ~/Downloads/?????.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com



> Update Software


::

	sudo apt-get update && sudo apt-get -y upgrade

> Install other software

::

	sudo apt-get -y install subversion tmux git curl samtools gcc make g++ python-dev unzip dh-autoreconf default-jre zlib1g-dev cmake libhdf5-dev libboost1.55-all-dev libboost1.55-dbg


> INSTALL Kallisto

::

    cd $HOME
    git clone https://github.com/pachterlab/kallisto.git
    cd kallisto
    mkdir build
    cd build
    cmake ..
    make
    sudo make all install

> Install Salmon

::

  cd $HOME
  curl -LO https://github.com/COMBINE-lab/salmon/archive/v0.5.1.tar.gz
  tar -zxf v0.5.1.tar.gz
  cd salmon-0.5.1/
  mkdir build
  cd build
  cmake ..
  make -j6
  sudo make all install  

>INSTALL SRATOOLKIT - this tool lets you work with SRA files

::

    cd $HOME
    wget http://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/2.4.2/sratoolkit.2.4.2-ubuntu64.tar.gz
    tar -zxf sratoolkit.2.4.2-ubuntu64.tar.gz
    PATH=$PATH:/home/ubuntu/sratoolkit.2.4.2-ubuntu64/bin
    echo "PATH=$PATH:/home/ubuntu/sratoolkit.2.4.2-ubuntu64/bin" >> ~/.profile

> Download data

::

    mkdir $HOME/data
    cd $HOME/data
    curl -LO http://datadryad.org/bitstream/handle/10255/dryad.72141/brain.final.fasta
    curl -LO ftp://ftp-trace.ncbi.nlm.nih.gov/sra/sra-instant/reads/ByRun/sra/SRR/SRR157/SRR1575395/SRR1575395.sra


> Convert SRA format into fastQ (takes a few minutes)

::

	cd $HOME/data
	fastq-dump --split-files --split-spot SRR1575395.sra


> Use Kallisto to count

::

    mkdir $HOME/quant
    cd $HOME/quant
    kallisto index -i transcripts.idx $HOME/data/brain.final.fasta
    kallisto quant -t 8 -i transcripts.idx -o kallisto_output -b 100 $HOME/data/SRR1575395_1.fastq $HOME/data/SRR1575395_2.fastq

> Using Salmon to count


::

  cd $HOME/quant
  ~/salmon-0.5.1/bin/salmon index -t $HOME/data/brain.final.fasta -i transcripts_index --type quasi -k 31
  ~/salmon-0.5.1/bin/salmon quant -p 8 -i transcripts_index -l MSR -1 $HOME/data/SRR1575395_1.fastq -2 $HOME/data/SRR1575395_1.fastq -o salmon_output


> look at estimates of TPM 

::

  cd $HOME/quant
  cat salmon_output/quant.sf | sed -e 1,11d | sort -nk1 > salmon.quant
  cat kallisto_output/abundance.tsv | sed -e 1d | sort -nk1 > kallisto.quant
  paste salmon.quant kallisto.quant  | column -s $'\t' -t | awk '{print $1 "\t" ($3-$9)/((($3+$9)/2)+.001)}' > expression.diffs


> Download and plot- open up a new terminal window

::

  scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:/home/ubuntu/quant/expression.diffs ~/Downloads/

> Open RStudio

::

  diffs <- read.table("~/Downloads/expression.diffs", quote="\"")
  hist(diffs$V2, col='light blue', xlab='percent different', main='Different in estimate of TPM: Kallisto versus Salmon')


=======================
TERMINATE YOUR INSTANCE
=======================
