==========================
COMPUTER EXAM 1
==========================


Name _________________________________________


I will give you points for each section. If I have to help you get past a particular section, you lose those points, but if you can do the rest of the steps, you can still get credit for that. 



1. Launch a ``c4.xlarge`` instance.  _______ 5 points

|
|

2. Update your machine and install the software using ``apt-get`` just like we did in lab 3. Type ``which gcc`` in the terminal and write in the results of that command here, for 5 points

|
|

3. Install BLAST, and download uniprot.pep and like you did in previous labs. Type ``which blastp`` and write in the results here, for 10 points. 

|
|

4. Download the dataset located here: ``https://s3.amazonaws.com/gen711/test.dataset3.fa``. The query you will use in ``blastp`` is called ``test.dataset3.fa``.  _________________ 5 points

|
|

5. Make the blast database using ``makeblastdb`` and start ``blastp`` like we did in lab 0. Write the commands you used for each for 10 points (``blastp`` should run for <3 minutes.)

|
|
|
|

6. Search for the ``ENSPTRP00000000180`` transcript in your results file. ``ENSPTRP00000000180`` is a specific gene present in your query dataset. Column 2 is the Uniprot ID of the hit, column 3 is the percent ID, and column 11 is the e-value. You'll note that the there are several different genes listed in column 2. Which of these is the best match to the query transcript and why? ::




     Which uniprot ID is the best match to ``ENSPTRP00000000180``  _______________ 5 points




     Why is this gene the best match  _______________ 10 points

