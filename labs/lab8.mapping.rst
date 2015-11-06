===================
Lab 8: Read Mapping
===================

During this lab, we will acquaint ourselves with de novo transcriptome assembly using Trinity. You will:

1. Install software and download data

2. Use sra-toolkit to extract fastQ reads

3. Map reads to dataset

4. look at mapping quality


The BWA manual: http://bio-bwa.sourceforge.net/ 

Flag info: http://broadinstitute.github.io/picard/explain-flags.html</a>


> Step 1: Launch and AMI. For this exercise, we will use a c4.2xlarge machine. 

::

	ssh -i ~/Downloads/?????.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com



> Update Software


::

	sudo apt-get update && sudo apt-get -y upgrade

> Install other software

::

	sudo apt-get -y install subversion tmux git curl samtools gcc make g++ python-dev unzip dh-autoreconf default-jre zlib1g-dev


> INSTALL BWA

::

    cd $HOME
    git clone https://github.com/lh3/bwa.git
    cd bwa
    make -j4
    PATH=$PATH:$(pwd)
    echo 'PATH=$PATH:$(pwd)' >> ~/.profile


>INSTALL SRATOOLKIT

::


    cd $HOME
    wget http://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/2.4.2/sratoolkit.2.4.2-ubuntu64.tar.gz
    tar -zxf sratoolkit.2.4.2-ubuntu64.tar.gz
    PATH=$PATH:/home/ubuntu/sratoolkit.2.4.2-ubuntu64/bin
    echo '$PATH:/home/ubuntu/sratoolkit.2.4.2-ubuntu64/bin' >> ~/.profile



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


> Map reads!! (20 minutes)

::

    mkdir $HOME/mapping
    cd $HOME/mapping
    tmux new -s mapping
    bwa index -p index $HOME/data/brain.final.fasta
    time bwa mem -t8 index $HOME/data/SRR1575395_1.fastq $HOME/data/SRR1575395_2.fastq > brain.sam


> Look at SAM file. 


::

    #Take a quick general look.

    head brain.sam
    tail brain.sam
    
    #Count how many reads in fastq files. `grep -c` counts the number of occurances of the pattern, which in this case is `^@`. I am looking for lines that begin with (specified by `^`) the @ character. 
    
    grep -c ^@ ../data/SRR1575395_1.fastq ../data/SRR1575395_2.fastq
    
    #count number of reads mapping with Flag 65/67. The 1st part of this command `awk`, pulls out the second column of the files, and counts everthing that has either 65 or 67. What do these flags correspond to?   
    
    awk '{print $2}' brain.sam | grep ^6 | grep -c '65\|67'
    
    #why do we need the `grep ^6` thing in there... try `awk '{print $2}' brain.sam | grep '65\|67' | wc -l`
    
    #what about this??
    
    awk '{print $2}' brain.sam | grep '^65\|^67' | wc -l


> Can you pull out the number of mismatches targeting the NM tag in column 12?

::

       #I'm giving you the last bit of the awk code. You have to figure out the 1st awk command and the 1st grep command. This will send the number of mismatches to a file `mismatches.txt`. Can you download it to your usb or HD and plot the results, find the mean number of mismatches, etc??

	awk | grep | awk -F ":" '{print $3}' > mismatches.txt

=======================
TERMINATE YOUR INSTANCE
=======================
