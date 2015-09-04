============
Lab 0: BLAST
============

During this lab, we will acquaint ourselves with the the software package BLAST. Your objectives are:


1. Familiarize yourself with the software, how to execute it, how to visualize results.



**Step 1:** Launch and AMI. For this exercise, we will use a c3.xlarge instance.

::

  ssh -i ~/Downloads/your.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com



The machine you are using is Linux Ubuntu: Ubuntu is an operating system you can use (I do) on your laptop or desktop. One of the nice things about this OS is the ability to update the software, easily.  The command `sudo apt-get update` checks a server for updates to existing software.


**The upgrade command** actually installs any of the required updates.

::

  sudo apt-get update
  sudo apt-get -y upgrade


OK, what are these commands?  ``sudo`` is the command that tells the computer that we have admin privileges. Try running the commands without the sudo -- it will complain that you don't have admin privileges or something like that. *Careful here, using sudo means that you can do something really bad to your own computer -- like delete everything*, so use with caution. It's not a big worry when using AWS, as this is a virtual machine- fixing your worst mistake is as easy as just terminating the instance and restarting.



**INSTALL SOFTWARE:** So now that we have updates the software, lets see how to add new software. Same basic command, but instead of the ``update`` or ``upgrade`` command, we're using ``install``. EASY!!

::

  sudo apt-get -y install tmux git curl gcc make g++ python-dev unzip \
        default-jre


**INSTALL BLAST:** ok, for this lab we are going to use BLAST, which is available as a package entitled ``ncbi-blast+``

::

  sudo apt-get -y install ???


to get a feel for the different options, type ``blastp -help``. Which type of blast does this correspond to? Look at the help info for blastp and tblastx



**DOWNLOAD DATA:**  remember, for blasting we need both some data (a query) and a database. Lets start with the data 1st. You will have one of the 5 different datasets. Do you remember how to use the `wget` and `gzip` commands from last week?

::

  cd /mnt
  curl -LO https://s3.amazonaws.com/gen711/Vicugna_pacos.vicPac1.pep.all.fa
  curl -LO ftp://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_sprot.fasta.gz
  gzip -d *gz


Make blast database and blast
--

::

  makeblastdb -in uniprot_sprot.fasta -out uniprot -dbtype prot

Now we are ready to blast.

::

  head -6 Vicugna_pacos.vicPac1.pep.all.fa > test.fasta
  blastp -evalue 1e-10 -num_threads 4 -db uniprot -query test.fasta -outfmt 6

You will see the results in a table with 12 columns. Use ``blastp -help`` to see what the results mean.

Test out some of the blast options. Try changing the word size ``-word_size``, scoring matrix, evalue, cost to open or extend a gap. See how these changes affect the results.

So great, you can blast 1 sequence, ut you could have done that on the web.. What about 100 sequences

::

  head -1008 Vicugna_pacos.vicPac1.pep.all.fa > larger.fasta
  blastp -evalue 1e-10 -num_threads 4 -db uniprot -query larger.fasta -outfmt 6

This might take a while.. but you can go have a coffee or whatever.. **The point is, that using the command line it's just as easy for you to blast 1 sequence as it is 100 or 100000**

========================
TERMINATE YOUR INSTANCE
========================
