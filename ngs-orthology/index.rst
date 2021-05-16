.. _ngs-orthology:

Orthology and Phylogeny
=======================


Preface
-------

In this section you will use some software to find orthologue genes and do phylogenetic reconstructions.


Learning outcomes
-----------------

After studying this tutorial you should be able to:

#. Use bioinformatics software to find orthologues in the NCBI database.
#. Use bioinformatics software to perform sequence alignment.
#. Use bioinformatics software to perform phylogenetic reconstructions.

Installing the software
-----------------------
Today we will be building an alignment and phylogeny. As this part of 
the lab does not require automation, there is no need to (necessarily) intergate it 
into your ``snakemake`` pipeline. However, if you wish to do so, 
that is fine.
         
Installing the software
-----------------------
We will use several pieces of software today to find open reading frames
that are similar to our own, to perform
alignments of different open reading frames and 
infer phylogenies. The first program we will install (if you 
do not have this already) is ``blast``. This program (as expected) 
can be installed via ``conda`` using the ``install`` command (``conda install``).

This will install a |blast| executable that you can use to remotely query the NCBI database.
The second program we need is one that will align homologous nucleotide sequences. This program is ``muscle``,  installable using ``conda``.

Finally, we will install |raxml|, a phylogenetic tree inference tool, which uses
maximum-likelihood (ML) optimality criterion. This program can also be installed using
 ``conda``.


Selecting a gene to build a phylogeny with the software
-----------------------
The first thing we need to do is select a gene that we will 
use to build a *gene tree* to infer the phylogenetic relatedness
of different *E. coli* isolates. Today, we will use *gyrB* (DNA gyrase B), 
a gene involved in the supercoiling of DNA, and which is necessary of DNA replication. 
This gene is commonly used for building phylogenies of closely-related bacteria.
We will use your ``unicycler`` annotation to find this gene. First, you will need to search your annotation for this gene. To do this, we will use the command-line tool
``grep``, which can locate text that matches your text-of-interest. In this 
case, the text-of-interest is *gyrase*. Try the following:

.. code:: bash
         
         # find which genes are named gyrase
         # note that we will look in the
         # annotated nucleotide file .ffn
         grep "gyrase" my_prokka_annotation.ffn

This should yield a result in which several genes are listed, one of which
should be *gyrase B*, for example something similar to *HCBPOPCK_00004 DNA gyrase subunit B*.
We can now use the name of this gene and ``seqtk`` to make a new ``fasta`` file consisting only of the nucleotide sequence of this gene. We can do this in the following way:

.. code:: bash
         
         # make a text file with the name
         # of the gyrB gene
         echo "HCBPOPCK_00004 DNA gyrase subunit B" > gyrB.txt
         # use seqtk to get the sequence of this gene
         seqtk subseq my_prokka_annotation.ffn gyrB.txt

This should yield a ``fasta`` file containing your gyrB sequence only. Check that the file does contain the expected sequence use ``cat``.


Finding orthologues using BLAST
-------------------------------

We will first make a |blast| database of our current assembly so that we can
find the orthologous sequence of the *S. cerevisiae* gene.
To do this, we run the command ``makeblastdb``:


.. code:: bash
          
          # create blast db
          makeblastdb –in ../assembly/spades_final/scaffolds.fasta –dbtype nucl


To run |blast|, we give it:

- ``-db``: The name of the database that we are BLASTing
- ``-query``: A fasta format input file
- A name for the output files
- Some notes about the format we want

  
First, we blast without any formatting:


.. code:: bash

          blastn –db ../assembly/spades_final/scaffolds.fasta –query s_cerev_tef2.fas > blast.out


This should output a file with a set of |blast| hits similar to what you might
see on the |blast| web site.

Read through the output (e.g. using ``nano``) to see what the results of your |blast| run was.

   
Next we will format the output a little so that it is easier to deal with.

.. code:: bash
          
          blastn –db ../assembly/spades_final/scaffolds.fasta –query s_cerev_tef2.fas –evalue 1e-100 –outfmt “6 length sseq” > blast_formatted.out

          
This will yield a file that has only the sequences of the subject, so that we can later add those to other fasta files.
However, the formatting is not perfect.
To adjust the format such that it is fasta format, open the file in an editor (e.g. ``nano``) and edit the first line so that it has a name for your sequence.
You should know the general format of a fasta-file (e.g. the first line start with a “>”).


.. hint::

   To edit in ``vi`` editor, you will need to press the escape key and “a” or “e”.
   To save in ``vi``, you will need to press the escape key and “w” (write).
   To quit ``vi``, you will need to press the escape key and “q” (quit).

   
Next, you have to replace the dashes (signifying indels in the |blast| result).
This can easily be done in ``vi``:
Press the escape key, followed by: ``:%s/\-//g``

Now we will |blast| a remote database to get a list of hits that are already in the NCBI database.


.. note::

   It turns out you may not be able to access this database from within BioLinux. In such a case, download the file named ``blast.fas`` and place it into your ``~/analysis/phylogeny/`` directory.


.. code:: bash

           curl -O http://compbio.massey.ac.nz/data/203341/blast_u.fas
           
           
Append the fasta file of your yeast sequence to this file, using whatever set of commands you wish/know.


.. note::

   Should the download fail, download manually from :ref:`downloads`.


Performing an alignment
-----------------------

We will use |muscle| to perform our alignment on all the sequences in the |blast| fasta file.
This syntax is very simple (change the filenames accordingly):


.. code:: bash

          muscle –in infile.fas –out your_alignment.aln


Building a phylogeny
--------------------

We will use |raxml| to build our phylogeny.
This uses a maximum likelihood method to infer parameters of evolution and the topology of the tree.
Again, the syntx of the command is fairly simple, except you must make sure that you are using the directory in which |raxml| sits.


The arguments are:

- ``-s``: an alignment file
- ``-m``: a model of evolution. In this case we will use a general time reversible model with gamma distributed rates (GTR+GAMMA)
- ``-n``: outfile-name
- ``-p``: specify a random number seed for the parsimony inferences

  
.. code:: bash

          raxmlHPC -s your_alignment.aln -m GTRGAMMA –n yeast_tree –p 12345


Visualizing the phylogeny
-------------------------

We will use the online software `Interactive Tree of Life (iTOL) <http://itol.embl.de/upload.cgi>`__ to visualize the tree.
Navigate to this homepage.
Open the file containing your tree (``*bestTree.out``), copy the contents, and paste into the web page (in the Tree text box).

You should then be able to zoom in and out to see where your yeast taxa is.
To find out the closest relative, you will have to use the `NCBI taxa page <https://www.ncbi.nlm.nih.gov/Taxonomy/TaxIdentifier/tax_identifier.cgi>`__.

Phylogeny ToDo
~~~~~~~~~~~~~~~~~~~~~~

.. todo::

   Are you certain that the **E. coli** are related in the way that the phylogeny suggests? Why might the topology of this phylogeny not truly reflect the evolutionary history of these **E. coli** species? 
