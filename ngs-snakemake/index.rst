.. _ngs-annotation:

Snakemake - automation and reproducability
=================

Preface
-------

At the start of this tutorial, we discussed the importance of making analyses easy to reproduce. That way, you can re-analyse new data using the same methods as you did previously, have other people analyse your data, or just have a clear record of what steps occurred inn your analyses.

To do all of these things, we will begin using a workflow manager, `Snakemake <https://snakemake.readthedocs.io/en/stable/>`_


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
          tree -L 2

.. code:: bash

         assembly/
         data/
         kraken/
         mappings/
         trimmed/
         trimmed-fastqc/
         variants/


Installing the software
-----------------------

.. code:: bash

          # of course you activate your conda env
          conda activate ngs
          
          # then we need snakemake
          conda install -c bioconda snakemake

Structuring the workflow
-------------------------

You will manage our workflow from the primary directory that you have created, in which your ``/data`` subdirectory sits.

|snakemake| works on the principal that all you have to tell it what input files you have and what output files you require, and it will execute all the necessary steps ("rules") to create the output files (and *only* those steps).

For example, if you think back to the workflow for the QC steps that we did in the previous section of the tutorial, we had several input files to perform QC on and several output files that resulted from that QC. Previously we performed all the QC steps separately. While this helps us greatly in understnading what is happening during each step, it makes repeating the entire process quite difficult. The |snakemake| workflow manager will simplify this entire process, making it simple and painless for you, or for anyone, to repeat the process.

Snakemake rules
~~~~~~~~~~~~~~~~

As explained above, |snakemake| tries to produce an output file(s) from an input file(s) using a series of steps or rules. In it's simplest form, a rule requires you to specify an input file, an output file, and a command of some sort telling it what to do with the input file (often so that it can create the output file). For example, this could look like this: 

.. code:: bash

    rule trim_fastq:
      input:
        "myfile.R1.fastq"
      output:
        "myfile.R1.trimmed.fastq"
      shell:
        "fastp -i {input} -o {output}"

This would take an input ``.fastq`` file and use the |fastp| program to create a ``.fastq`` with trimmed reads.


.. only:: html

   .. rubric:: References

.. [SIMAO2015] Simao FA, Waterhouse RM, Ioannidis P, Kriventseva EV and Zdobnov EM. BUSCO: assessing genome assembly and annotation completeness with single-copy orthologs. `Bioinformatics, 2015, Oct 1;31(19):3210-2 <http://doi.org/10.1093/bioinformatics/btv351>`__

.. [STANKE2005] Stanke M and Morgenstern B. AUGUSTUS: a web server for gene prediction in eukaryotes that allows user-defined constraints. `Nucleic Acids Res, 2005, 33(Web Server issue): W465–W467. <https://dx.doi.org/10.1093/nar/gki458>`__
