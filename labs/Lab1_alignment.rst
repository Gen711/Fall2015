===================
Lab 2: Alignment
===================

During this lab, we will acquaint ourselves with alignment. Your objectives are:



1. Familiarize yourself with the software, how to execute it, how to visualize results.

2. Regarding your dataset, tell me how some of these genes are related to their homologous copies.



Step 1: Launch and AMI. For this exercise, we will use a c4.2xlarge instance.

::

	ssh -i ~/Downloads/your.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com



The machine you are using is Linux Ubuntu: Ubuntu is an operating system you can use (I do) on your laptop or desktop. One of the nice things about this OS is the ability to update the software, easily.  The command `sudo apt-get update` checks a server for updates to existing software.

::


	sudo apt-get update && sudo apt-get -y upgrade


OK, what are these commands?  `sudo` is the command that tells the computer that we have admin privileges. Try running the commands without the sudo -- it will complain that you don't have admin privileges or something like that. *Careful here, using sudo means that you can do something really bad to your own computer -- like delete everything*, so use with caution. It's not a big worry when using AWS, as this is a virtual machine- fixing your worst mistake is as easy as just terminating the instance and restarting.



> So now that we have updates the software, lets see how to add new software. Same basic command, but instead of the ``update`` or ``upgrade`` command, we're using ``install``. EASY!!

::

	sudo apt-get -y install tmux git curl gcc make g++ python-dev unzip \
        default-jre ncbi-blast+



=======================
Install mafft and RAxML
=======================

Let's install mafft so that we can do an alignment (<a href="http://mafft.cbrc.jp/alignment/software/">http://mafft.cbrc.jp/alignment/software/</a>)

::

    cd $HOME
    curl -LO http://mafft.cbrc.jp/alignment/software/mafft-7.245-without-extensions-src.tgz
    tar -zxf mafft-7.245-without-extensions-src.tgz
    cd mafft-7.245-without-extensions/core
    sudo make && sudo make install
    cd

Now lets install RAxML so that we can make a phylogeny. ()


    cd $HOME
    git clone https://github.com/stamatak/standard-RAxML.git
    cd standard-RAxML/
    make -f Makefile.PTHREADS.gcc
    PATH=$PATH:$(pwd)



remember, for blasting we need both some data (a query) and a database. Lets start with the data 1st. You will have one of the 5 different datasets. Do you remember how to use the `wget` and `gzip` commands from last week?

::

    sudo chown ubuntu /mnt
    cd /mnt
    curl -LO https://s3.amazonaws.com/gen711/dataset1.pep



> Now let's download the database. For this exercise we will use Swissprot: 

::

  curl -LO https://s3.amazonaws.com/gen711/uniprot.pep


Make blast database and blast
--

-

> make a blast database

::

  makeblastdb -in uniprot.pep -out uniprot -dbtype prot

> Now we are ready to blast.

::

  grep -A1 'ENSPTRP00000032491\|ENSPTRP00000032494' dataset1.pep > query.fa
  blastp -evalue 8e-8 -num_threads 8 -db uniprot -query query.fa -max_target_seqs 3 -outfmt "6 qseqid pident evalue stitle"

> You will see the results in a table with 4 columns. Use `blastp -help` to see what the results mean.

> Test out some of the blast options. Try changing the word size `-word_size`, scoring matrix, evalue, cost to open or extend a gap. See how these changes affect the results.

> After you've done this, you should make a file containing the query and the hits.

::

  grep --no-group-separator -A1 'HXA2_HUMAN\|HXA2_BOVIN\|HXA2_PAPAN\|HXA3_HUMAN\|HXA3_MOUSE\|HXA3_BOVIN' uniprot.pep > results.pep


Make a file that contains the query sequences and the sequences we found by blasting.

::

  cat query.pep results.pep > for_alignment.pep

=====
mafft
=====

> Align the proteins using mafft

::

  mafft --reorder --bl 80 --localpair --thread 8 for_alignment.pep > for.tree

=====
RAxML
=====

> Make a phylogeny

::

  raxmlHPC-PTHREADS -f a -m PROTCATBLOSUM62 -T 4 -x 34 -N 100 -n tree -s for.tree -p 35

> Copy phylogeny and view online.

::

	more RAxML_bipartitionsBranchLabels.tree

	#copy this info.

> Visualize tree on website: http://iubio.bio.indiana.edu/treeapp/treeprint-form.html
