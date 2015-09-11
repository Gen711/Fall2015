===================
Lab 1: Alignment
===================

During this lab, we will acquaint ourselves with alignment. Your objectives are:



1. Familiarize yourself with the software, how to execute it, how to visualize results.

2. Regarding your dataset, tell me how some of these genes are related to their homologous copies.



Step 1: Launch and AMI. For this exercise, we will use a c4.2xlarge instance.

::

	ssh -i ~/Downloads/your.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com



The machine you are using is Linux Ubuntu: Ubuntu is an operating system you can use (I do) on your laptop or desktop. One of the nice things about this OS is the ability to update the software, easily.  The command ``sudo apt-get update`` checks a server for updates to existing software.

::

	sudo apt-get update && sudo apt-get -y upgrade


OK, what are these commands?  ``sudo`` is the command that tells the computer that we have admin privileges. Try running the commands without the sudo -- it will complain that you don't have admin privileges or something like that. *Careful here, using sudo means that you can do something really bad to your own computer -- like delete everything*, so use with caution. It's not a big worry when using AWS, as this is a virtual machine- fixing your worst mistake is as easy as just terminating the instance and restarting.



> So now that we have updates the software, lets see how to add new software. Same basic command, but instead of the ``update`` or ``upgrade`` command, we're using ``install``. EASY!!

::

  sudo apt-get -y install tmux git curl gcc make g++ python-dev unzip default-jre ncbi-blast+

======================
Finish up with GitHub
======================

**Go to the github repo you just created**, and find the box on the middle right side of the screen labelled ``HTTPS clone URL``. Copy the text that is in that box, then go back to your terminal. 

::

  git clone https://github.com/macmanes-lab/fri_lecture.git
  cd fri_lecture  #you're going to ``cd`` into some other directory, whatever you named your repo. 

  ##lets make a file, for fun

  nano test_file.txt


  git add test_file.txt
  git commit -m 'i am adding my first new file, YAY'
  git push  #You're going to have to type in your Github user name and password

  ##Go to github, you might have to reload teh page, and see that your file has been added. 

  ##now lets edit that file ``test_file.txt``

  nano test_file.txt

  git commit -m 'whoa, version control FTW!!'
  git push
 
  ##Go to github, you might have to reload the page, click on the filename, then history. 


=======================
Install mafft and RAxML
=======================

**Let's install mafft** so that we can do an alignment (http://mafft.cbrc.jp/alignment/software/)

::

    cd $HOME
    curl -LO http://mafft.cbrc.jp/alignment/software/mafft-7.245-without-extensions-src.tgz
    tar -zxf mafft-7.245-without-extensions-src.tgz
    cd mafft-7.245-without-extensions/core
    sudo make && sudo make install
    cd

**Now lets install RAxML** so that we can make a phylogeny. (http://sco.h-its.org/exelixis/web/software/raxml/index.html)

::

    cd $HOME
    git clone https://github.com/stamatak/standard-RAxML.git
    cd standard-RAxML/
    make -f Makefile.PTHREADS.gcc
    PATH=$PATH:$(pwd)
    cd



remember, for blasting we need both some data (a query) and a database. Lets start with the data 1st. You will have one of the 5 different datasets. Do you remember how to use the ``curl`` command from last week?

::

    sudo chown ubuntu /mnt
    cd /mnt
    curl -LO https://s3.amazonaws.com/gen711/dataset1.pep



**Download the database.** For this exercise we will use Swissprot, which is the same curated database we used last week 

::

  curl -LO https://s3.amazonaws.com/gen711/uniprot.pep


=============================
Make blast database and blast
=============================



make a blast database

::

  makeblastdb -in uniprot.pep -out uniprot -dbtype prot

**For teh query sequnces, I am going to pull out 2 specific sequences from the larger dataset ``dataset1.pep``. These are both HOX genes that I have identified ahead of time. The command ``grep`` finds words/numbers/symbols in files. Remember that the ``>`` sends the results of the command to a file, in this case names ``query.pep``.   

::

  grep -A1 'ENSPTRP00000032491\|ENSPTRP00000032494' dataset1.pep > query.pep

Now that I have a couple of query sequences, we are ready to blast.

::

  blastp -evalue 8e-8 -num_threads 8 -db uniprot -query query.pep -max_target_seqs 3 -outfmt "6 qseqid pident evalue stitle"

You will see the results in a table with 4 columns. Use `blastp -help` to see what the results mean. Test out some of the blast options. Try changing the word size ``-word_size``, scoring matrix, evalue, cost to open or extend a gap. See how these changes affect the results.

**Make a file that contains the sequences for the blast hits you just discovered**


::

  grep --no-group-separator -A1 'HXA2_HUMAN\|HXA2_BOVIN\|HXA2_PAPAN\|HXA3_HUMAN\|HXA3_MOUSE\|HXA3_BOVIN\|HXA9_HUMAN' uniprot.pep > results.pep


**Now, make a file that containsBOTH  the query sequences AND the sequences we found by blasting.

::

  cat query.pep results.pep > for_alignment.pep

=====
mafft
=====

Align the proteins using mafft

::

  mafft --reorder --bl 80 --localpair --thread 8 for_alignment.pep > for.tree

=====
RAxML
=====

Make a phylogeny

::

  raxmlHPC-PTHREADS -f a -m PROTCATBLOSUM62 -T 8 -x 34 -N 100 -n tree -s for.tree -p 35

Copy phylogeny and view online.

::

	more RAxML_bipartitionsBranchLabels.tree

	#copy this info.

Visualize tree on website: http://www.evolgenius.info/evolview/

1. Click on the folder icon in teh top-left part of the page. 
2. Paste in the code from your terminal. **FYI, this is the NEWICK tree format**, yes, named after Newick's just down the road from us!!
3. Find the HOX9 gene - this is the outgroup sequence we will use to rood the tree. Hower over teh branch and it will turn red - a box will open, click "reroot here"

===============================================
TERMINATE YOUR INSTANCE
===============================================
