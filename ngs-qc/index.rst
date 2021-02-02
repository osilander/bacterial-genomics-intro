.. _ngs-qc:

Quality control
===============

.. warning::

  Since 2020, none of the internal links are functioning. Please use the Dropbox links in the :ref:`downloads` section.

Preface
-------

There are many sources of errors that can influence the quality of your sequencing run [ROBASKY2014]_.
In this quality control section we will use our skill on the
command-line interface to deal with the task of investigating the quality and cleaning sequencing data [KIRCHNER2014]_.


.. There is an accompanying lectures for this tutorial (`Next-generation sequencing and quality control: An introduction <https://dx.doi.org/10.6084/m9.figshare.2972320.v1>`__).

.. NOTE::

   You will encounter some **To-do** sections at times. Write the solutions and answers into a text-file, with the laboratory sections and dates noted for all entries. At the end of the Semester, you will need to submit this file for marking.

   
Overview
--------

The part of the workflow we will work on in this section can be viewed in :numref:`fig-workflow-qc`.

.. _fig-workflow-qc:
.. figure:: images/workflow.png

   The part of the workflow we will work on in this section marked in red.
   

Learning outcomes
-----------------

After studying this tutorial you should be able to:

#. Describe the steps involved in pre-processing/cleaning sequencing
   data, including both short-read (Illumina) and long-read (Oxford Nanopore).
#. Distinguish between a good and a bad sequencing data.
#. Compute, investigate and evaluate the quality of sequence data from a
   sequencing experiment.
   

Structuring your directories
--------
Remember from previously that it is critical to maintain a well organised and logical framework for doing your work in. Let's first check which directory you are currently sitting in. Type: ``pwd`` (*print working directory*). You should be in your home directory. If not, change to your home directory by typing ``cd``.

It would be a good idea to create a new directory for the analysis of the data in this course. Make that directory now. The name should be something sensible (and ideally obvious), perhaps ``genome_analysis``. If you cannot remember how to make a directory, refer to the section on *The command line interface*, or google it.

Change into this new directory that you have created. If you cannot remember how to change into a directory, refer to the section on *The command line interface*.

This will be the directory that you do all your analyses in for this class.

The analysis we are doing now will be quality control of our sequence data. We will fetch this data in the next section, but first we should further organise our directories. Make a new directory called ``data`` or something similar. Change into that directory. Now we have an organised project with a data directory.

.. Attention::
    We are missing something, no?
    
    Change back to your ``genome_analysis`` directory and make a README text file. This file should contain information on the project, and could also include (for example) that fact that the first step in your data analysis will be Quality Control. From the command line, there are only a few basic "text editors" that can be used to make a text file. Some of the most common are ``vim``, ``emacs``, and ``nano``. Unless you are well-acquainted with ``vim`` or ``emacs`` I recommend trying ``nano``. To do so, simply type ``nano`` on the command line, and a barebones editor will appear. Use this to write your README.txt file.



The short-read Illumina data
--------

First, we are going to download the short-read Illumina data we will analyse.


.. code-block:: bash

   # create a directory you work in
   mkdir analysis

   # change into the directory
   cd analysis

   # download the data
   curl -O http://compbio.massey.ac.nz/data/203341/data.tar.gz

   # uncompress it
   tar -xvzf data.tar.gz

   
.. note::

   Should the download fail, download manually from :ref:`downloads`.


   
The data is from a paired-end sequencing run data (see :numref:`fig-pairedend`) from an |illumina| MiSeq [GLENN2011]_.
Thus, we have two files, one for each end of the read. 

.. _fig-pairedend:
.. figure:: images/pairedend.png

   Illustration of single-end (SE) versus paired-end (PE) sequencing.

We have covered the basics of this sequencing technology in lecture, but if you need a refresher on how |illumina| paired-end sequencing works have a
look at the `Illumina
technology webpage <http://www.illumina.com/technology/next-generation-sequencing/paired-end-sequencing_assay.html>`__
and this `video <https://youtu.be/HMyCqWhwB8E>`__. 

.. attention::

   The data we are using is "almost" raw data as it came from the machine. However, this data has been post-processed in two ways already. First, all sequences that were identified as belonging to the PhiX174 bacteriophage genome have been removed. This process requires some skills we will learn in later sections. Second, the |illumina| sequencing adapters have been removed as well already! The process is explained below but we are **not** going to do it.


Investigate the data
~~~~~~~~~~~~~~~~~~~~

Make use of your newly developed skills on the command-line to
investigate the files in your ``data`` folder.

.. todo::

   Use the command-line to get some ideas about the file.
   #. What kind of files are we dealing with?
   #. How many sequence reads are in the file (try using the ``wc`` command)?
   #. Assume that your bacteria has a genome size of 5 Mbp. Calculate the coverage based on this formula: ``C = L*N / G``

    - ``C``: Coverage
    - ``G``: is the haploid genome length in bp
    - ``L``: is the read length in bp (e.g. 2x100 paired-end = 200)
    - ``N``: is the number of reads sequenced
  
This leads us to:    

The fastq file format
---------------------

The data we receive from the sequencing is in ``fastq`` format. To remind us what this format entails, we can revisit the `fastq wikipedia-page <https://en.wikipedia.org/wiki/FASTQ_format>`__!

A useful tool to decode base qualities can be found `here <http://broadinstitute.github.io/picard/explain-qualities.html>`__.

What do the sequences in your ``fastq`` file look like? The easiest and fastest way to see is **not** to open the file, but to peek inside of it. There are several ways to do this. Perhaps you just want to see the first few lines of the file. In this case you could use:

.. code-block:: bash
    head myfile.fastq

Or maybe you would like to see the first 20 lines:

.. code-block:: bash
    head -20 myfile.fastq

Or maybe you would like to see the last few lines:

.. code-block:: bash
    tail myfile.fastq

Or perhaps the whole file in screen-sized chunks:

.. code-block:: bash
    less myfile.fastq
(type ``q`` to exit ``less``)

.. todo::

   Explain briefly what the quality value represents.


The short-read QC process
--------------

There are a few steps one need to do when getting the raw sequencing data from the sequencing facility:

#. Remove PhiX sequences
#. Adapter trimming
#. Quality trimming of reads
#. Quality assessment
   

Watch out: PhiX174 DNA
-----------

`PhiX174 <https://en.wikipedia.org/wiki/Phi_X_174>`_ (PhiX for short) is a nontailed bacteriophage with a single-stranded DNA genome with 5386 nucleotides.
Please take a minute to read this page describing how PhiX is used as a quality and calibration control for `sequencing runs <http://www.illumina.com/products/by-type/sequencing-kits/cluster-gen-sequencing-reagents/phix-control-v3.html>`__. Briefly,
PhiX is often added at a low known concentration, spiked in the same lane along with the sample or used as a separate lane.
As the concentration of the genome is known, one can calibrate the instruments, which is required for collecting accurate data. The PhiX DNA also serves as a positive control (we know the DNA is of high quality).


However, this means that after sequencing, PhiX genomic sequences need to be removed before processing your data further as this constitutes a deliberate contamination [MUKHERJEE2015]_.
The steps involve mapping all reads to the "known" PhiX genome, and removing all of those sequence reads from the data.

However, your sequencing provider might not have used PhiX. Thus you should read the protocol carefully, or just do this step in any case.


.. attention::

   We are **not** going to do this step here, as this has been already done. Please see the :ref:`ngs-mapping` section on how to map reads against a reference genome.


Adapter and read trimming
----------------

The process of sequencing DNA via |illumina| technology requires the addition of some adapters to the sequences.
These get sequenced as well and need to be removed as they are artificial and do not belong to the species we try to sequence.
Generally speaking adapter trimming takes time.


.. attention::

   The process of how to do this is explained here, however we are **not** going to do this as our sequences have been adapter-trimmed already.
   

First, we need to know the adapter sequences that were used during the sequencing of our samples.
Normally, you  might ask your sequencing provider, who should be providing this information to you.
|illumina| itself provides a `document <https://support.illumina.com/downloads/illumina-customer-sequence-letter.html>`__ that describes the adapters used for their different technologies.

However, many quality control software programs will automatically search for a range of adapters, which simplifies the process for us. Also the |fastp| tool that we will be using `does exactly this <https://github.com/OpenGene/fastp#adaptersp>`__. So let us begin the QC process.


.. code-block:: bash
    fastp blah blah


.. todo::
 
	#. Run |fastp| also on the evolved samples. 


.. hint::

   Should you not get the command togeter to trim the evolved samples, have a look at the coding solutions at :ref:`code-sickle`. Should you be unable to run |sickle| at all to trim the data. You can download the trimmed dataset `here <http://compbio.massey.ac.nz/data/203341/trimmed.tar.gz>`__. Unarchive and uncompress the files with ``tar -xvzf trimmed.tar.gz``.




Visualising the results of the QC process 
---------------------------

To understand in more detail what the data look like and the results of the trimming process we will view and compare the reports produced by fastp. The tool we will do this with is called |multiqc|, and it is available on the ``bioconda`` channel as ``multiqc``. Install it now. We will use MultiQC later in the course to understand the results of various tools we apply.


.. code-block:: bash
    multiqc --help
    Usage: multiqc [OPTIONS] <analysis directory>

    Main MultiQC run command for use with the click command line, complete
    with all click function decorators. To make it easy to use MultiQC within
    notebooks and other locations that don't need click, we simply pass the
    parsed variables on to a vanilla python function.

    Options:
      -f, --force                     Overwrite any existing reports
      -d, --dirs                      Prepend directory to sample names
      -dd, --dirs-depth INTEGER       Prepend [INT] directories to sample names.
                                      Negative number to take from start of path.

      -s, --fullnames                 Do not clean the sample names (leave as full
                                      file name)

      -i, --title TEXT                Report title. Printed as page header, used
                                      for filename if not otherwise specified.

      -b, --comment TEXT              Custom comment, will be printed at the top
                                      of the report.

      -n, --filename TEXT             Report filename. Use 'stdout' to print to
                                      standard out.

      -o, --outdir TEXT               Create report in the specified output
                                      directory.

Run FastQC on the untrimmed and trimmed data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. todo::

   #. Create a directory for the results --> **trimmed-fastqc**

      
.. hint::

   Should you not get it right, try the commands in :ref:`code-qc1`.

.. only:: html

   .. rubric:: References

               

.. [GLENN2011] Glenn T. Field guide to next-generation DNA sequencers. `Molecular Ecology Resources (2011) 11, 759–769 doi: 10.1111/j.1755-0998.2011.03024.x <http://doi.org/10.1111/j.1755-0998.2011.03024.x>`__

.. [KIRCHNER2014] Kirchner et al. Addressing challenges in the production and analysis of Illumina sequencing data. `BMC Genomics (2011) 12:382 <http://doi.org/10.1186/1471-2164-12-382>`__

.. [MUKHERJEE2015] Mukherjee S, Huntemann M, Ivanova N, Kyrpides NC and Pati A. Large-scale contamination of microbial isolate genomes by Illumina PhiX control. `Standards in Genomic Sciences, 2015, 10:18. DOI: 10.1186/1944-3277-10-18 <http://doi.org/10.1186/1944-3277-10-18>`__

.. [ROBASKY2014] Robasky et al. The role of replicates for error mitigation in next-generation sequencing. `Nature Reviews Genetics (2014) 15, 56-62 <http://doi.org/10.1038/nrg3655>`__
