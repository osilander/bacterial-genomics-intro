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

Lets see what our directory structure looks so far:

.. code:: bash

    # a nice tree-like structure
    # Here we only look two levels down (-L 2)
    # each of yours will look different
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

For example, if you think back to the workflow for the QC steps that we did in the previous section of the tutorial, we had several input files to perform QC on and several output files that resulted from that QC. Previously we performed all the QC steps separately. While this helps us greatly in understnading what is happening during each step, it makes repeating the entire process quite difficult. Not only do you first have to remember the commands and the order in which you typed them, you also have to type the commands several times - once for each set of files you want to analyse.

The |snakemake| workflow manager will simplify this entire process, making it simple and painless for you, or for anyone, to repeat the process.

Snakemake rules
~~~~~~~~~~~~~~~~

As explained above, |snakemake| tries to produce an output file(s) from an input file(s) using a series of steps or rules. In it's simplest form, a rule requires you to specify an input file, an output file, and a command of some sort telling it what to do with the input file (often so that it can create the output file). For example, this could look like this: 

.. code:: bash

    rule trim_illumina:
      input:
        "data/illumina/myfile.R1.fastq"
      output:
        "results/myfile.R1.trimmed.fastq"
      shell:
        "fastp -i {input} -o {output}"

This would take an input ``.fastq`` file and use the |fastp| program to create a ``trimmed.fastq`` with trimmed reads. Note that the |fastp| program must exist (although we will take care of this possible problem later). Note also that ``snakemake`` will only look where you tell it to look (i.e. here it will look for ``data/illumina/myfile.R1.fastq``)

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
Write your first workflow by opening the ``nano`` editor and writing in the two rules discussed above.

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

In this case, it first looks at rule ``all`` and sees that you would like a file called
``results/myfile.R1.trimmed.fastq`` - in other words, a trimmed fastq file that sits in
a directory called ``results`` (which in fact does not yet exist). At this point ``snakemake`` looks around to see if there is another rule that would create this trimmed fastq file (i.e. the output is ``results/myfile.R1.trimmed.fastq``), or if this file already exists (it does not of course). At this point it finds one - your ``rule trim_fastq``. Now it goes there and checks what input is needed. It sees that ``data/myfile.R1.fastq`` is needed. Now, again, it checks if there is a rule to make this file, or if the file already exists.

It should find the file in your ``data`` directory. If it doesn't, and if there is not a rule to create that file, it will error out and try to tell you why.

.. attention::
  You need to make sure that you correctly specify the locations of your input and output files. For example, you should (generally) executing |snakemake| from with the top-level of your analysis directory. If you have used the directory structure specified in the QC section of the tutorial, then your Illumina reads sit in ``data/illumina``. Ensure that you specify this full path. Similarly, you should structure your output. I recommend putting the results of your analysis into a ``results`` directory. |snakemake| *does* have the useful feature that it will create directories that do not exist. Thus, you can ask it to output to the ``results/`` directory without that directory actually existing. |snakemake| will then create that directory.

Now if you are satisfied that the ``snakemake`` dry-run does what you would like, you can go ahead and execute a real run. Note that we need one more argument in this case - the ``-j``. This specifies how many `cores <https://en.wikipedia.org/wiki/Central_processing_unit>`_ to use when snakemake runs. This computer has 24 processors, each of which has two cores, for a total of 48 cores. (You can type ``htop`` to see the cores that are available; type ``q`` to exit ``htop``)

.. code:: bash

    # hope this works
    snakemake -p -j 2

If everything has worked as planned, then you should have a new set of trimmed ``.fastq`` files in your ``results/`` directory.

However, this has resulted in trimming only a single read file. It is *much* more likely that you will actually want to trim multiple read files, and you do not want to have to type each command individually. In this case, you can rely on the power of ``snakemake`` to solve your problem.

Now we will use the ``*`` wildcard character to recognize *all* files that we might want to trim. This will get a little bit tricky at first and require come explanation. First, let's review what the ``*`` character does. Here are a few resources; some might be more iuintuitive than others: `geek university <https://geek-university.com/linux/wildcard/#:~:text=A%20wildcard%20in%20Linux%20is,begin%20with%20the%20letter%20O>`_, `ryans tutorials <https://ryanstutorials.net/linuxtutorial/wildcards.php>`_, `indiana <https://kb.iu.edu/d/ahsf#:~:text=The%20asterisk%20(%20*%20),-The%20asterisk%20represents&text=Use%20it%20when%20searching%20for,you%20have%20only%20partial%20names.&text=For%20most%20web%20search%20engines,documents%20with%20that%20one%20word>`_.



.. only:: html

   .. rubric:: References

.. [SIMAO2015] Simao FA, Waterhouse RM, Ioannidis P, Kriventseva EV and Zdobnov EM. BUSCO: assessing genome assembly and annotation completeness with single-copy orthologs. `Bioinformatics, 2015, Oct 1;31(19):3210-2 <http://doi.org/10.1093/bioinformatics/btv351>`__

.. [STANKE2005] Stanke M and Morgenstern B. AUGUSTUS: a web server for gene prediction in eukaryotes that allows user-defined constraints. `Nucleic Acids Res, 2005, 33(Web Server issue): W465–W467. <https://dx.doi.org/10.1093/nar/gki458>`__
