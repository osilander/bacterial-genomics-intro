.. _ngs-snakemake:

Snakemake - automation and reproducibility
=================

Preface
-------

At the start of this tutorial, we discussed the importance of making analyses easy to reproduce. That way, you can re-analyse new data using the same methods as you did previously, have other people analyse your data, or just have a clear record of what steps occurred inn your analyses.

To do all of these things, we will begin using a workflow manager, `Snakemake <https://snakemake.readthedocs.io/en/stable/>`_

Overview
--------

The part of the workflow we will work on in this section can be viewed in :numref:`fig-workflow-voi`.

.. _fig-workflow-voi:
.. figure:: images/workflow.png
    
    The part of the workflow we will work on in this section marked in red.


Learning outcomes
-----------------

After studying this section of the tutorial you should be able to:

#. Explain the advantages of using a workflow manager
#. Use a workflow manager (|snakemake|) to manage your workflows
#. Explain how to utilise workflow "rules" to execute certain workflow steps


Before we start
---------------

Lets see what our directory structure looks like so far:

.. code:: bash

    # a nice tree-like structure
    # Here we only look two levels down (-L 2)
    # each of yours will look different
    # but should have a data/ directory
    tree -L 2


Installing the software
-----------------------

.. code:: bash

          # of course you activate your conda env
          conda activate ngs
          
          # then we need snakemake
          conda install -c bioconda snakemake

Structuring the workflow
-------------------------

You will manage your workflow from the primary directory that you have created, in which your ``/data`` subdirectory sits. For example, this could be ``genome_analysis``.

|snakemake| works on the principal that all you have to tell it is what input files you have and what output files you would like, and it will execute all the necessary steps ("rules") to create the output files (and *only* those steps). Furthermore, it will only execute those steps if your input files are *newer* than your output files. If your output files have been made *after* your input files, |snakemake| will assume that you have already done the analysis, and it will thus do nothing.

For example, if you think back to the workflow for the QC steps that we did in the previous section of the tutorial, we had several input files to perform QC on and several output files that resulted from that QC. Previously we performed all the QC steps separately. While this helps us considerably in understanding what is happening during each step, it makes repeating the entire process quite difficult. Not only do you first have to remember the commands and the order in which you typed them, you also have to type the commands several times - once for each set of files you want to analyse.

The |snakemake| workflow manager will simplify this entire process, making it simple and painless for you, or for anyone, to repeat the process.

Snakemake rules
~~~~~~~~~~~~~~~~

As explained above, you tell Snakemake what input you *have* (i.e. what files) and what output you *need* (i.e. what files), and |snakemake| tries to produce the output file(s) from the input file(s) using a series of steps or rules. In its simplest form, a rule requires you to specify an input file, an output file, and a command of some sort telling it what to do with the input file (usually so that it can create the output file). For example, this could look like this: 

.. code:: bash

    # we need a name for our rule and it should make sense
    rule trim_illumina:
      # and here's the input for the rule (what we are providing)
      input:
        "data/illumina/myfile.R1.fastq"
      # and here's the output (what we expexct to get)
      output:
        "results/myfile.R1.trimmed.fastq"
      # and here's the command - how we can go from the input file 
      # to the output file
      shell:
        "fastp -i {input} -o {output}"

This would take an input ``.fastq`` file and use the |fastp| program to create a ``trimmed.fastq`` with trimmed reads. Note that the |fastp| program must exist (although we will take care of this possible problem later). Note also that ``snakemake`` will only look where you tell it to look for the input file (i.e. here it will look for ``data/illumina/myfile.R1.fastq``) - more on that later.

One rule to rule them all
~~~~~~~~~~~~~~~~~~~~~~~~~

The first thing |snakemake| does when trying to figure out what it needs to do is look for a rule called ``all``. In this rule, you need to define all the output files that you require. |snakemake| will then try to create these files by searching through and executing other rules. For the QC steps, your all rule might look something like this:

.. code:: bash

    rule all:
      input:
        "results/myfile.R1.trimmed.fastq"

|snakemake| will then search your ``Snakefile`` for another rule that has as an output ``results/myfile.R1.trimmed.fastq``. But look! We have already written such a rule above! Creating a workflow is as simple as writing these two rules into a single ``Snakefile``.

A simple snakemake workflow
~~~~~~~~~~~~~~~~~~~~~~~~~
Write your first workflow by opening the ``nano`` editor and writing in the two rules discussed above. I have put a lot of comments below(``#``) - you do not need all of these.

.. code:: bash

    # open nano
    nano

    # add the rules, with the "all" rules at the top
    # and the trim rule next. Make sure that you follow
    # the structure above, and indent properly.
    # A good rule to follow is to use four spaces when you indent.
    # At the end, save and exit, naming your file "Snakefile" (no extension)
    rule all:
      input:
        "results/myfile.R1.trimmed.fastq"

    rule trim_illumina:
      input:
        "data/illumina/myfile.R1.fastq"
      output:
        "results/myfile.R1.trimmed.fastq"
      shell:
        "fastp -i {input} -o {output}"

Let's now see what our workflow will do (or, *attempt* to do). To dry-run |snakemake|, simply type ``snakemake -np``. |snakemake| will look for a file called ``Snakefile`` and tell you the rules that it will execute (if any).

In this case, it first looks at the rule ``all`` and sees that you would like a file called
``results/myfile.R1.trimmed.fastq`` - in other words, a trimmed fastq file that sits in
a directory called ``results`` (a directory which in fact does not yet exist). At this point ``snakemake`` looks around to see if this file already exists (it does not of course), or if there is another rule that would create this trimmed fastq file (i.e. the output is ``results/myfile.R1.trimmed.fastq``). At this point it finds a rule that would create this file - your ``rule trim_fastq``. Now it goes there and checks what input is needed. It sees that ``data/myfile.R1.fastq`` is needed. Now, again, it checks if the file already exists, or if there is a rule to make this file.

It should find that this file *does* exist - in your ``data`` directory. If the file doesn't exist, and if there is no rule to create that file, it will error out and try to tell you why.

.. attention::
  You need to make sure that you correctly specify the locations of your input and output files. For example, you should (generally) execute |snakemake| from with the top-level of your analysis directory. If you have used the directory structure specified in the QC section of the tutorial, then your Illumina reads sit in ``data/illumina``. Ensure that you specify this full path. Similarly, you should structure your output. I recommend putting the results of your analysis into a ``results`` directory. |snakemake| *does* have the useful feature that it will create directories that do not exist. Thus, you can ask it to output to the ``results/`` directory without that directory actually existing. |snakemake| will then create that directory.

Now if you are satisfied that the ``snakemake`` dry-run does what you would like, you can go ahead and execute a real run. Note that we need one more argument in this case - the ``-j``. This specifies how many `cores <https://en.wikipedia.org/wiki/Central_processing_unit>`_ to use when snakemake runs. This computer that you *happen* to be using has 24 processors, each of which has two cores, for a total of 48 cores. (You can type ``htop`` to see the cores that are available; type ``q`` to exit ``htop``)

.. code:: bash

    # hope this works
    snakemake -p -j 2

If everything has worked as planned, then you should have a new set of trimmed ``.fastq`` files in your ``results/`` directory.

Generalising snakemake workflows
~~~~~~~~~~~~~~~~~~~~~~~~~

However, the above instructions have resulted in trimming only a single read file. It is *much* more likely that you will actually want to trim multiple read files, and you do not want to have to type each command individually. In this case, you can *once again* rely on the power of ``snakemake`` to solve your problem.

Now we will use a file matching strategy that is identical to using a ``*`` wildcard character to recognize *all* files that we might want to trim.

This will get a little bit tricky at first and require some explanation. First, let's review what the ``*`` character does. Here are a few resources; some might be more intuitive than others: `geek university <https://geek-university.com/linux/wildcard/#:~:text=A%20wildcard%20in%20Linux%20is,begin%20with%20the%20letter%20O>`_, `ryans tutorials <https://ryanstutorials.net/linuxtutorial/wildcards.php>`_, `indiana <https://kb.iu.edu/d/ahsf#:~:text=The%20asterisk%20(%20*%20),-The%20asterisk%20represents&text=Use%20it%20when%20searching%20for,you%20have%20only%20partial%20names.&text=For%20most%20web%20search%20engines,documents%20with%20that%20one%20word>`_.

On a basic level: the ``*`` character will match *any* number of *unknown* letters or numbers when you are looking for a file or a directory on the command line. For example:

.. code:: bash

    # list all files in the directory
    ls -lh

    # output
    total 4.9G
    -rw-rw-r-- 1 olin olin 518K Mar  1 11:04 H8_anc.fastp.html
    -rw-rw-r-- 1 olin olin 158K Mar  1 11:04 H8_anc.fastp.json
    -rwxrwxr-x 1 olin olin 597M Mar  1 10:08 H8_anc_R1.fastq
    -rw-rw-r-- 1 olin olin 597M Mar  1 11:04 H8_anc_R1_trimmed.fastq
    -rwxrwxr-x 1 olin olin 484M Mar  1 10:09 H8_anc_R2.fastq
    -rw-rw-r-- 1 olin olin 483M Mar  1 11:04 H8_anc_R2_trimmed.fastq
    -rw-rw-r-- 1 olin olin 477K Mar  1 11:15 H8_evolved.fastp.html
    -rw-rw-r-- 1 olin olin 133K Mar  1 11:15 H8_evolved.fastp.json
    -rwxrwxr-x 1 olin olin 709M Mar  1 11:10 H8_evolved_R1.fastq
    -rw-rw-r-- 1 olin olin 696M Mar  1 11:15 H8_evolved_R1_trimmed.fastq
    -rwxrwxr-x 1 olin olin 709M Mar  1 11:10 H8_evolved_R2.fastq
    -rw-rw-r-- 1 olin olin 696M Mar  1 11:15 H8_evolved_R2_trimmed.fastq
    drwxrwxr-x 2 olin olin 4.0K Mar  1 11:22 multiqc_data
    -rw-rw-r-- 1 olin olin 1.1M Mar  1 11:22 multiqc_report.html

.. code:: bash

    # List ONLY files that have "R1" at the start OR end
    # Here we use the wildcard * twice (once at the start and
    # once at the end) to match any start or end characters
    # Note that here you cannot tab complete the name
    ls -lh *R1*
    -rwxrwxr-x 1 olin olin 597M Mar  1 10:08 H8_anc_R1.fastq
    -rw-rw-r-- 1 olin olin 597M Mar  1 11:04 H8_anc_R1_trimmed.fastq
    -rwxrwxr-x 1 olin olin 709M Mar  1 11:10 H8_evolved_R1.fastq
    -rw-rw-r-- 1 olin olin 696M Mar  1 11:15 H8_evolved_R1_trimmed.fastq

.. code:: bash

    # list ONLY files that have "fastq" at the END.
    # Here we use a * at the START to match any 
    # letters / numbers at the  beginning
    # Again, you cannot tab complete the name
    ls -lh *fastq
    -rwxrwxr-x 1 olin olin 597M Mar  1 10:08 H8_anc_R1.fastq
    -rw-rw-r-- 1 olin olin 597M Mar  1 11:04 H8_anc_R1_trimmed.fastq
    -rwxrwxr-x 1 olin olin 484M Mar  1 10:09 H8_anc_R2.fastq
    -rw-rw-r-- 1 olin olin 483M Mar  1 11:04 H8_anc_R2_trimmed.fastq
    -rwxrwxr-x 1 olin olin 709M Mar  1 11:10 H8_evolved_R1.fastq
    -rw-rw-r-- 1 olin olin 696M Mar  1 11:15 H8_evolved_R1_trimmed.fastq
    -rwxrwxr-x 1 olin olin 709M Mar  1 11:10 H8_evolved_R2.fastq
    -rw-rw-r-- 1 olin olin 696M Mar  1 11:15 H8_evolved_R2_trimmed.fastq

.. code:: bash

    # list ONLY files that have "H8_evol" at the START
    # Here we use a * at the END to match any letters/numbers
    # at the end
    # Again, you cannot tab complete the name
    ls -lh H8_evol*
    -rw-rw-r-- 1 olin olin 477K Mar  1 11:15 H8_evolved.fastp.html
    -rw-rw-r-- 1 olin olin 133K Mar  1 11:15 H8_evolved.fastp.json
    -rwxrwxr-x 1 olin olin 709M Mar  1 11:10 H8_evolved_R1.fastq
    -rw-rw-r-- 1 olin olin 696M Mar  1 11:15 H8_evolved_R1_trimmed.fastq
    -rwxrwxr-x 1 olin olin 709M Mar  1 11:10 H8_evolved_R2.fastq
    -rw-rw-r-- 1 olin olin 696M Mar  1 11:15 H8_evolved_R2_trimmed.fastq

We are now going to use the ``*`` to our advantage by adding a line to your ``Snakefile``. However, instead of writing it as an asterisk ``*``, you are going to immediately assign the matches that it finds to a new variable. Here, we name this variable ``sample``, and designate it as a variable with the ``{}`` curly brackets. To do this, you need to add a line at the very start of your Snakefile: ``STRAIN, = glob_wildcards("./data/illumina/{sample}_R1.fastq")``.

Do this now by editing your ``Snakefile`` using the ``nano`` text editor.

Explanation: in this case, the bracketed portion, ``{sample}``, is acting as a wildcard, and is matching *any* file that is located in the ``./data/illumina/`` directory and which *ends* in ``_R1.fastq``. Why are we doing this? Well, we know that all Illumina data that we are dealing with is paired end. And we also know that this is the data we would like to qc - but you *don't* want to separately qc Read1 and Read2. So you will find all the samples to qc by *only* matching the Read1 (R1) samples. Let's in fact check what files these are. Return to the command line and try typing ``ls -lh ./data/illumina/*_R1.fastq``. You should find that it lists all the samples that you want to qc and nothing more - namely one ancestor file and one evolved file (in *your* case). You could imagine, however, that this would also be possible if you had fifty files in the directory, and all of these files had different names or sample dentifiers.

The second thing we have done is to assign the list of these ``{sample}`` variables to a list ofall variables. This is the ``STRAIN`` name, and we have performed (well...I have suggested) capitalisation because it is a list of all important variables. We are using a specific *function* in python to do so, the `glob_wildcards <https://snakemake.readthedocs.io/en/stable/project_info/faq.html#how-do-i-run-my-rule-on-all-files-of-a-certain-directory>`_ function.

Now what you have this, we can proceed with the rest of the Snakefile and workflow.

.. only:: html

   .. rubric:: References

.. [SIMAO2015] Simao FA, Waterhouse RM, Ioannidis P, Kriventseva EV and Zdobnov EM. BUSCO: assessing genome assembly and annotation completeness with single-copy orthologs. `Bioinformatics, 2015, Oct 1;31(19):3210-2 <http://doi.org/10.1093/bioinformatics/btv351>`__

.. [STANKE2005] Stanke M and Morgenstern B. AUGUSTUS: a web server for gene prediction in eukaryotes that allows user-defined constraints. `Nucleic Acids Res, 2005, 33(Web Server issue): W465–W467. <https://dx.doi.org/10.1093/nar/gki458>`__
