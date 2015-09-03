Getting Started with UNIX
--

During this lab, we will acquaint ourselves with the Unix terminal, learn how to access data, install software, and  find things. *it is absolutely critical that you master these skills*, so please ask questions if confused.

Important UNIX rules
--

* Everything is case sensitive. Gen711 is not the same as gen711
* Spaces in file names should be avoided
* The unix $PATH is the collection of locations where the computer looks for executables (programs)
* Folders and Files are all you have. If you want to access one of these, you need to tell the computer *EXACTLY* where it is. `/home/macmanes/gen711/exam1_key.txt` will work (assuming you've spelled things correctly, and that the file really exists in that location), but `exam1_key.txt` may not.

* Lines that begin with a `#` are comments.

Basic shell commands 
--


> `pwd` print working directory

> `ls` list directory contents

> `nano` open up a text editor

> `mkdir` make a directory (folder)

> `cd` Change directories

> `rm` delete a file

> `mv` move a file 

> `cp` copy a file

*What does this command do?*

	pwd


*the `ls` command lists the files and folders present in your current directory.  Try `ls -lt` and `ls -lth`. *What is the difference between these commands?*

	ls -l


>create a file

    nano hello.txt
    #The nano text editor will appear -> type something
    This is my 1st unix file
    CTL-x
    y
    #typing n would get rid of the text you just wrote.



>look at the file, there are several ways to look at the file

	head -5 hello.txt #this shows you the 1st 5 lines of the file
	more hello.txt #this shows you the whole file, 1 screen at a time. Space bar to advance, q to quit



>make a copy of the file, using a different name, then remove it.

	cp hello.txt bye.txt
	ls -lth
	rm bye.txt
	ls -lth

-

>move the file (or rename it). What is the difference between `mv` and `cp`???

	mv hello.txt bye.txt
	ls -lth



>make a folder (directory), make a file inside a folder.

    mkdir testfolder
    ls -lth
    #make a folder inside that folder
    mkdir testfolder/inside_test
    #make a file
    nano testfolder/inside_test/inside.txt
    head testfolder/inside_test/inside.txt
    rm testfolder/inside_test/inside.txt

>there are a few other commands that you should be familiar with: `sort`, `cat`, `clear`, `tail`, `history`. Try googling and using `man` to figure them out.


>There is some info that is complementary to this material found here: <a href="http://swcarpentry.github.io/2014-08-21-upenn/novice/ref/01-shell.html">http://swcarpentry.github.io/2014-08-21-upenn/novice/ref/01-shell.html</a>


Launch and EC2 Instance -- Go to PPT BEFORE PROCEEDING
--

> Step 1: Launch and AMI. For this exercise, a t1.micro will be sufficient.


	ssh -i ~/Downloads/gen711.pem ubuntu@ec2-???-???-???-???.compute-1.amazonaws.com



> The machine you are using is Linux Ubuntu: Ubuntu is an operating system you can use (I do) on your laptop or desktop. One of the nice things about this OS is the ability to update the software, easily.  The command `sudo apt-get update` checks a server for updates to existing software.


	sudo apt-get update


>The upgrade command actually installs any of the required updates.

	sudo apt-get upgrade

>OK, what are these commands?  `sudo` is the command that tells the computer that we have admin privileges. Try running the commands without the sudo -- it will complain that you don't have admin privileges or something like that. *Careful here, using sudo means that you can do something really bad to your own computer -- like delete everything*, so use with caution. It's not a big worry when using AWS, as this is a virtual machine- fixing your worst mistake is as easy as just terminating the instance and restarting.

-

> So now that we have updates the software, lets see how to add new software. Same basic command, but instead of the `update` or `upgrade` command, we're using `install`. EASY!!


	sudo apt-get -y install tmux git curl gcc make g++ python-dev unzip \
        default-jre
 


>After you run this command, try something else - try to install something else. R (a stats package - more on this wonderful software later). The package is named `r-base-core`. See if you can install it!! Installing software on Linux is easy (so long as there is a downloadable package - more on when no such package exists later in lab)



>BTW, did you notice the `\` at the end of line 1 in the above code snippett?? That is a special character we use to break up a single line of code over 2 or more lines. You'll see me use this a lot!
