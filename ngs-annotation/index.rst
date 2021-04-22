.. _ngs-annotation:

Genome annotation
=================

Preface
-------

In this section you will predict genes and assess your assembly using ``prokka``, which employs several gene prediction algorithms as well as databases for functional annotation |augustus| and |busco|.


.. NOTE::

   You will encounter some **To-do** sections at times. Write the solutions and answers into a text-file.   


Overview
--------

The part of the workflow we will work on in this section can be viewed in :numref:`fig-workflow-anno`.

.. _fig-workflow-anno:
.. figure:: images/workflow.png

   The part of the workflow we will work on in this section marked in red.


Learning outcomes
-----------------

After studying this section of the tutorial you should be able to:

#. Explain how annotation completeness is assessed using orthologues
#. Use bioinformatics tools to perform gene prediction
#. Use genome-viewing software to graphically explore genome annotations and NGS data overlays 


Installing the software
-----------------------

We will use two pieces of software today, |busco| and |prokka|. Both are available on the ``bioconda`` channel, named as expected (``prokka`` and ``busco``). Go ahead and install them now. The ``prokka`` installation will take a couple of minutes.

If you would like, you can now also make a new directory for the annotation results.


Genome Annotation
---------------------------------------------

``prokka`` uses a number of methods to find open reading frames, tRNA, rRNAs, tmRNAs, and signal peptides.

The majority of the program that are used implement a hidden markov model (HMM) to infer where these elements lie in the assembly you have made.

``prokka`` is very simple to run, but has a very wide range of `options <https://github.com/tseemann/prokka#crazy-person>`_. Minimally, to run the program you need to give it one argument - the location of the genome assembly as a ``.fasta`` file. However, you will probably find that it is more useful to give it the location of the ``.fasta`` and an output directory (``--outdir``). A prefix for your organism may also be useful (``--prefix``). For more options, you can simply type ``prokka``:

.. code:: bash
  
      Name:
      Prokka 1.13 by Torsten Seemann <torsten.seemann@gmail.com>
      Synopsis:
        rapid bacterial genome annotation
      Usage:
        prokka [options] <contigs.fasta>
      General:
        --help            This help
        --version         Print version and exit
        --docs            Show full manual/documentation
        --citation        Print citation for referencing Prokka
        --quiet           No screen output (default OFF)
        --debug           Debug mode: keep all temporary files (default OFF)


The annotation will take around five minutes, so you can run it in a ``tmux`` terminal if you like.

.. Attention::

   You may find that you need to downgrade blast. If so you can install it using

.. code:: bash
    conda install blast=2.2


Assessment of orthologue presence and absence
---------------------------------------------

|busco| will assess orthologue presence absence using |blastn|, a rapid method of finding close matches in large databases (we will discuss this in lecture).
It uses |blastn| to make sure that it does not miss any part of any possible coding sequences. To run the program, we give it

- A fasta format input file
- A name for the output files
- The name of the lineage database against which we are assessing orthologue presence absence (that we downloaded above)
- An indication of the type of annotation we are doing (genomic, as opposed to transcriptomic or previously annotated protein files).

.. code:: bash
  
          busco -i ../assembly/spades_final/scaffolds.fasta -o file_name_of_your_choice -l ./saccharomycetales_odb9 -m geno

          
.. NOTE::

   This should take about 90 minutes to run. In the meantime you can run the next step.

          

Interactive viewing
-------------------

We will use the software |igv| to view the assembly, the gene predictions you have made, and the variants that you have called, all in one window. 

Installing |igv|
----------------

We will not install this software using |conda|.
Instead, make a new directory in your home directory entitled “software”, and change into this directory.
You will have to download the software from the Broad Institute:

.. code:: bash

          mkdir software
          cd software
          wget http://data.broadinstitute.org/igv/projects/downloads/2.4/IGV_2.4.10.zip

          # unzip the software:
          unzip IGV_2.4.10.zip

          # and change into that directory.
          cd IGV_2.4.10.zip
          
          # To run the interactive GUI, you will need to run the bash script in that directory:
          bash igv.sh



This will open up a new window.
Navigate to that window and open up your genome assembly:

- Genome -> load Genome from File
- Load your assembly, not your gff file.

Load the tracks:

- File -> Load from file
- Load your ``vcf`` file from last week
- Load your ``gff`` file from this week.

  
At this point you should be able to zoom in and out to see regions in which there are SNPs or other types of variants.
You can also see the predicted genes.
If you zoom in far enough, you can see the sequence (DNA and protein).

If you have time and interest, you can right click on the sequence and copy it.
Open a new browser window and go to the blastn homepage.
There, you can blast your gene of interest (GOI) and see if blast can assign a function to it.

The end goal of this lab will be for you to select a variant that you feel is interesting (e.g. due to the gene it falls near or within), and hypothesize as to why that mutation might have increased in frequency in these evolving yeast populations.


Assessment of orthologue presence and absence (2)
-------------------------------------------------

Hopefully your |busco| analysis will have finished by this time.
Navigate into the output directory you created.
There are many directories and files in there containing information on the orthologues that were found, but here we are only really interested in one: the summary statistics.
This is located in the ``short_summary*.txt`` file.
Look at this file.
It will note the total number of orthologues found, the number expected, and the number missing.
This gives an indication of your genome completeness.

.. TODO::

   Is it necessarily true that your assembly is incomplete if it is missing some orthologues? Why or why not?



.. only:: html

   .. rubric:: References

.. [SIMAO2015] Simao FA, Waterhouse RM, Ioannidis P, Kriventseva EV and Zdobnov EM. BUSCO: assessing genome assembly and annotation completeness with single-copy orthologs. `Bioinformatics, 2015, Oct 1;31(19):3210-2 <http://doi.org/10.1093/bioinformatics/btv351>`__

.. [STANKE2005] Stanke M and Morgenstern B. AUGUSTUS: a web server for gene prediction in eukaryotes that allows user-defined constraints. `Nucleic Acids Res, 2005, 33(Web Server issue): W465–W467. <https://dx.doi.org/10.1093/nar/gki458>`__
