.. _ngs-voi:

Variants-of-interest
====================

Preface
-------

In this section we will use our genome annotation of our reference and our genome variants in the evolved line to find variants that are interesting in terms of the observed biology.

.. NOTE::

    You will encounter some **To-do** sections at times. Write the solutions and answers into a text-file.   


Overview
--------

The part of the workflow we will work on in this section can be viewed in :numref:`fig-workflow-voi`.

.. _fig-workflow-voi:
.. figure:: images/workflow.png
    
    The part of the workflow we will work on in this section marked in red.
   
     
Learning outcomes
-----------------

After studying this section of the tutorial you should be able to:

#. Identify variants of interests.
#. Understand how the variants might affect the observed biology in the evolved line.


  
General comments for identifying variants-of-interest
-----------------------------------------------------


Things to consider when looking for variants-of-interest:

- The quality score of the variant call.
  
  * Do we call the variant with a higher then normal score?
    
- The mapping quality score.
  
  * How confident are we that the reads were mapped at the position correctly?
    
- The location of the SNP.
  
  * Does the SNP overlap a coding region in the genome annotation?
    
- The type of SNP.

  * substitutions vs. indels (indels are common and often *wrong* for Nanopore assemblies).

Consider all of these factors and then construct some hypotheses about why you observe the change(s) you do (:numref:`fig-hypotheses`)

.. _fig-hypotheses:
.. figure:: images/hypotheses.png
    
    Your hypotheses.


SnpEff
------

We will be using |snpeff| to annotate our identified variants. The tool will tell us 
which mutations might warrant further analyses.


Installing software
~~~~~~~~~~~~~~~~~~~
  
Tools we are going to use in this section and how to install them if you not have done it yet.


.. code:: bash

    # activate the env
    conda activate ngs
          
    # Install these tools into the conda environment
    # if not already installed
    conda install snpeff
    install -c bioconda snpsift
    conda install genometools-genometools
  

Make a directory if you like (e.g. before you integrate these steps into
your ``snakemake``), and change into
the directory:


.. code:: bash

    mkdir voi

    # change into the directory
    cd voi

         
Prepare the SnpEff database
~~~~~~~~~~~~~~~~~~~~~~~

We need to create our own config-file for |snpeff|. Where is the ``snpEff.config``:


.. code:: bash
    
    # look for snpEff.config in the miiniconda directory.
    # specify the /share/ subdirectory
    find ~/miniconda3/share/ -name snpEff.config
    # result should be something like
    # myhome/share/snpeff-5.0-1/snpEff.config
    

This will give you the path to the ``snpEff.config``. It might be looking a bit different then the one shown here, depending on the version of |snpeff| that is installed.

Make a local copy of the ``snpEff.config`` into your current directory
 (e.g. the results directory of your annotation) 
 and then edit it with an editor of your choice:


.. code:: bash

    # make sure this path is to *your* snpEff config
    # we are copying this so that the path is easy
    # to find and that we don't mess up the original
    cp myhome/share/snpeff-5.0-1/snpEff.config .
    nano snpEff.config

          
Make sure the data directory path in the ``snpEff.config`` looks like this (most
 likely it does):


.. code:: bash

    data.dir = ./data/

          
There is a section with databases, which starts like this (around line 130):


.. code:: bash

    #-------------------------------------------------------------------------------
    # Databases & Genomes
    #
    # One entry per genome version. 
    #
    # For genome version 'ZZZ' the entries look like
    #	ZZZ.genome              : Real name for ZZZ (e.g. 'Human')
    #	ZZZ.reference           : [Optional] Comma separated list of URL to site/s Where information for building ZZZ database was extracted.
    #	ZZZ.chrName.codonTable  : [Optional] Define codon table used for chromosome 'chrName' (Default: 'codon.Standard')
    #
    #-------------------------------------------------------------------------------


Add the following two lines in the database section underneath these header lines:


.. code:: bash

    # my E. coli genome
    ecolianc.genome : EcoliAnc

          
And go ahead and save and close the ``snpEff.config`` file.

Now, we need to create a local data folder called ``./data/ecolianc`` (e.g. 
in your ``voi`` directory).


.. code:: bash

    # create folders
    # here -p makes the intermediate directories if needed
    mkdir -p ./data/ecolianc


Copy our genome assembly to the newly created data folder.
The name needs to be ``sequences.fa`` or ``ecolianc.fa`` (not
``assembly.fasta``). Again here we copy so that it is present in 
the ``./data/ecolianc`` directory.


.. code:: bash
    
    # for exxample
    cp assembly.fasta ./data/ecolianc/sequences.fa

    
Lastly, copy your genome annotation to the data folder.
The name needs to be ``genes.gff`` (or ``genes.gtf`` for gtf-files), 
and should be a result of your ``prokka`` analysis.


.. code:: bash

    cp my_prokka_annotation.gff ./data/ecolianc/genes.gff
    #gzip ./data/yeastanc/genes.gff


Now you can build a new |snpeff| database using the ``snpEff build`` command. We need to give
``snpEff`` the ``.gff`` file and the directory with the assembly. We will place the output of the command
into a file for later reference (``snpEff.stdout``). This should take only a few seconds.


.. code:: bash

    snpEff build -c snpEff.config -gff3 -v ecolianc > snpEff.stdout


SNP annotation
~~~~~~~~~~~~~~

Now we can use our new |snpeff| database to annotate some variants. To do this we
invoke the ``snpEff`` command, tell it the folder that contains the reference, gff, and
a newly created snpEff predictor file, and lastly, give it the current ``.vcf`` file. Important:
 use the ``ud 0`` argument to prevent annotation of possible effects on upstream and downstream genes.
  This means that it will look 0 bp upstream and 0 bp downstream for effects.
For example:


.. code:: bash

    snpEff -ud 0 -c snpEff.config ecolianc my_variant_calls.q225.vcf > my_variant_calls.q225.annotated.vcf


|snpeff| adds ``ANN`` fields to the vcf-file entries that explain the effect of the variant.


Example
~~~~~~~

Lets look at one entry from the annotated one.
We are only interested in the 8th column, which contains information regarding the variant.
|snpeff| will add fields here:


.. code:: bash


    # my_variant_calls.q225.annotated.vcf
    1   4935712 .   G   A   4244.62 .   AB=0;ABP=0;AC=2;AF=1;AN=2;AO=142;CIGAR=1X;DP=143;DPB=143;DPRA=0;EPP=18.6694;EPPR=5.18177;GTI=0;LEN=1;MEANALT=1;MQM=60;MQMR=60;NS=1;NUMALT=1;ODDS=192.595;PAIRED=1;PAIREDR=1;PAO=0;PQA=0;PQR=0;PRO=0;QA=4801;QR=32;RO=1;RPL=64;RPP=6.00754;RPPR=5.18177;RPR=78;RUN=1;SAF=67;SAP=3.98899;SAR=75;SRF=0;SRP=5.18177;SRR=1;TYPE=snp;ANN=A|missense_variant|MODERATE|HCBPOPCK_04741|GENE_HCBPOPCK_04741|transcript|TRANSCRIPT_HCBPOPCK_04741|protein_coding|1/1|c.338C>T|p.Thr113Ile|338/702|338/702|113/233||,A|upstream_gene_variant|MODIFIER|HCBPOPCK_04738|GENE_HCBPOPCK_04738|transcript|TRANSCRIPT_HCBPOPCK_04738|protein_coding||c.-4108C>T|||||4108|,A|upstream_gene_variant|MODIFIER|HCBPOPCK_04743|GENE_HCBPOPCK_04743|transcript|TRANSCRIPT_HCBPOPCK_04743|protein_coding||c.-2043G>A|||||2043|,A|downstream_gene_variant|MODIFIER|HCBPOPCK_04739|GENE_HCBPOPCK_04739|transcript|TRANSCRIPT_HCBPOPCK_04739|protein_coding||c.*1729G>A|||||1729|,A|downstream_gene_variant|MODIFIER|HCBPOPCK_04740|GENE_HCBPOPCK_04740|transcript|TRANSCRIPT_HCBPOPCK_04740|protein_coding||c.*556G>A|||||556|,A|downstream_gene_variant|MODIFIER|HCBPOPCK_04742|GENE_HCBPOPCK_04742|transcript|TRANSCRIPT_HCBPOPCK_04742|protein_coding||c.*383C>T|||||383|,A|downstream_gene_variant|MODIFIER|HCBPOPCK_04744|GENE_HCBPOPCK_04744|transcript|TRANSCRIPT_HCBPOPCK_04744|protein_coding||c.*2528C>T|||||2528|,A|downstream_gene_variant|MODIFIER|HCBPOPCK_04745|GENE_HCBPOPCK_04745|transcript|TRANSCRIPT_HCBPOPCK_04745|protein_coding||c.*3352C>T|||||3352|,A|downstream_gene_variant|MODIFIER|HCBPOPCK_04746|GENE_HCBPOPCK_04746|transcript|TRANSCRIPT_HCBPOPCK_04746|protein_coding||c.*4131C>T|||||4131|WARNING_TRANSCRIPT_NO_START_CODON,A|downstream_gene_variant|MODIFIER|HCBPOPCK_04747|GENE_HCBPOPCK_04747|transcript|TRANSCRIPT_HCBPOPCK_04747|protein_coding||c.*4650C>T|||||4650|,A|downstream_gene_variant|MODIFIER|HCBPOPCK_04748|GENE_HCBPOPCK_04748|transcript|TRANSCRIPT_HCBPOPCK_04748|protein_coding||c.*4998C>T|||||4998|,A|intragenic_variant|MODIFIER|HCBPOPCK_00086|null|gene_variant|null|||n.4935712G>A||||||  GT:DP:RO:QR:AO:QA:GL    1/1:143:1:32:142:4801:-429.048,-39.848,0  


Wow.

|snpeff| added annotation information starting with ``ANN=T|missense_variant|...``.
If we look a bit more closely we find that the variant results in a amino acid change from a threonine to a isoleucine (``Thr113Ile``).


False positives
~~~~~~~

There are frequently false positive variants identified when dealing with 
hybrid assemblies (like ``unicycler``). In our experience, we find those are 
often indels. For this reason, it might be useful to screen out variants that 
are indels, or to select *only* variants that are SNPs (e.g. mutations from C to T). 
Using the command line program ``grep``, it is simple to identify those variants, for example:


.. code:: bash
    # grep finds lines in a file that match specific pattern of text
    # here we look for lines that match "TYPE=snp"
    # the syntax of grep is "grep mypattern myfile.txt"
    grep 'TYPE=snp' my_variant_calls.q225.annotated.vcf


You could also look for variants that satisfy two conditions, for example, that
are both SNPs AND which cause missense mutations (rather than synonymous mutations):

.. code:: bash
    # here we look for lines that match "TYPE=snp" AND
    # "missense_variant"
    # the .* in the middle acts as a wildcard
    grep 'TYPE=snp.*missense_variant' my_variant_calls.q225.annotated.vcf

In both cases above you can redirect the output to a file using ``>``.

.. _fig-blast-voi:
.. figure:: images/blast.png
    
    Results of a |blast| search of the CDS.
