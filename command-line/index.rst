.. _tool-installation:

The command line interface
=================

Preliminaries
---------------------------------


Note that throughout this course you may see commands or files or directories that are named something like ``my_awesome_file.tab`` or ``my_home_directory`` or ``myresults.txt``. When you see names like this in the instructions, *this does not mean that you should name your files or directories in this way*. Rather, you should replace these placeholder names with names that are **relevant to you**, or which are descriptive **for you**, or which contain **your** directory names. For example, if you are making a file that contains the results of a quality control analysis of DNA sequences from *E. coli*, you might name the file ``ecoli_qc_results.txt``.


Naming convention
---------------------------------


One important aspect of organising files and directories (folders) is `naming convention <https://en.wikipedia.org/wiki/Naming_convention_(programming)>`_. When working on the command line, your life will become considerably easier if you avoid using spaces in your files and directory names. Thus, **never** name your file ``my awesome file.txt``. Instead, name it ``my_awesome_file.txt`` ("snake case"), or ``myAwesomeFile.txt`` ("camel case") or ``my-awesome-file.txt`` ("kebab case") or ``my.awesome.file.txt`` but probably not ``MY-AWESOME-FILE.txt`` ("screaming snake case"). You should pick one of these at the start of the course, and *stick to that format throughout the course* (i.e. camel case, or kebab case, etc.).


While we are the topic of `naming conventions <https://en.wikipedia.org/wiki/Naming_convention_(programming)>`_, there are certain characters that you should **always** avoid when naming files and folders. Besides spaces, these are (not necessarily exhaustive):


.. code-block:: bash

   : ; ` " ' \ / ! @ # $ % ^ & * ( ) + , ? [ ] { } | > <
  

Directory structure
---------------------------------

In addition to naming conventions, there are good and bad ways to organise your files and directories. Please have a brief read through `this resource, <https://www.oreilly.com/library/view/developing-bioinformatics-computer/1565926641/ch04.html>`_


.. Attention::
   Never forget.

   There is one fundamental aspect of using the command line that you **must never forget**. It is perhaps the single most powerful method available to save time. That method is...
   

   **tab-complete**
   

   `Tab-complete <https://en.wikipedia.org/wiki/Command-line_completion>`_ can be used to auto-complete commands, directory names, and file names. If you are not sure whether your file is named ``results_QC.txt`` or ``results_qc.txt`` then on the command line you can simply type ``results`` *and then tab*, and the computer will auto-complete the name (assuming there is a file or directory or command that begins with ``results``).

   If you type the first part of a file and then press tab, but find that it does not autocomplete *even though you know you have the correct start of the file name*, then try pressing tab twice. This will give you a list of all the files (directories), etc. that begin with the first few letters that you have typed. This becomes important, for example, if you have a file named ``my_awesome_file.txt`` and ``my_awesome_file2.txt`` but you only type ``my_awes`` and then tab-complete.


   
Becoming a better bioinformatician
---------------------------------

Throughout this lab course, *google is your friend*. If you have errors, or if you are not sure how you might do something, or if you forget a command, google it!

Thus, **Step One** as you begin the lab is: Approach the command line with confidence and in a calm manner, assured that whatever goes wrong, you can google your way out of it.

`It's <https://codeahoy.com/2016/04/30/do-experienced-programmers-use-google-frequently/>`_

`what <https://www.reddit.com/r/programming/comments/3bwo68/how_much_does_an_experienced_programmer_use_google/>`_

`all <https://www.hanselman.com/blog/am-i-really-a-developer-or-just-a-good-googler>`_

`good <https://www.freecodecamp.org/news/google-not-learn-not-why-searching-can-be-better-than-knowing-79838f7a0f06/>`_

`programmers <https://fossbytes.com/do-best-programmers-use-google-stack-overflow-time/>`_

`do <https://news.ycombinator.com/item?id=11603078>`_


Navigating via the command line
---------------------------------

Likely the most common method that you use to interact with your computer to do things like change into different folders, open new folders or files, remove folder and files, etc. is via the mouse or trackpad.