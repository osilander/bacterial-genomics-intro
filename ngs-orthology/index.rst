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
into your ``snakemake`` pipeline, and indeed it will not be sinple to do. However, if you wish to do so, 
that is fine.
         
Installing the software
-----------------------
We will use several pieces of software today to find open reading frames
that are similar to our own, to perform
alignments of different open reading frames and 
infer phylogenies.

The first program we need is one that will align homologous nucleotide sequences. This program is ``muscle``,  installable using ``conda``.

Second, we will install |raxml|, a phylogenetic tree inference tool, which uses
maximum-likelihood (ML) optimality criterion. This program can also be installed using
 ``conda``.


Selecting a gene to build a phylogeny with the software
-----------------------
The first thing we need to do is select a gene that we will 
use to build a *gene tree* to infer the phylogenetic relatedness
of different *E. coli* isolates. Today, we will use *gnd* (6-phosphogluconate dehydrogenase), 
a gene involved in O-antigen synthesis, and which is highly polymorphic in *E. coli*.
We will use your ``unicycler`` annotation to find this gene. First, you will need to search your annotation for this gene. To do this, we will use the command-line tool
``grep``, which can locate text that matches your text-of-interest. In this 
case, the text-of-interest is *6-phosphogluconate*. Try the following:

.. code:: bash
         
         # find which genes are named 6-phosphogluconate
         # note that we will look in the
         # annotated nucleotide file .ffn
         grep "6-phosphogluconate" my_prokka_annotation.ffn

This should yield a result in which several genes are listed, one of which
should be *6-phosphogluconate dehydrogenase*, for example something similar to *HCBPOPCK_01798 6-phosphogluconate dehydrogenase*.
We can now use the name of this gene (*HCBPOPCK_01798*) and ``seqtk`` to make a new ``fasta`` file consisting only of the nucleotide sequence of this gene. We can do this in the following way:

.. code:: bash
         
         # make a text file with the name
         # of the gnd gene
         echo "HCBPOPCK_00004"  > gnd.txt
         # use seqtk to get the sequence of this gene
         seqtk subseq my_prokka_annotation.ffn gnd.txt > mygnd.fas

This should yield a ``fasta`` file containing your gnd sequence only. Check that the file does contain the expected sequence use ``cat``.


Finding orthologues using BLAST
-------------------------------

We would usually use |blast| to find matches to your sequence in the NCBI database by requesting a ``remote`` search using the *gnd* gene as the query. However, the remote |blast| service is not currently available. Instead, I have made a ``fasta`` file of *E. coli* *gnd* sequences available in the ``data`` directory. You will need to copy this file into your own directory.


You next need to append the sequence of the *gnd* from your own *E. coli* strain to this file, using whatever set of commands you wish/know. Most likely, this will be ``cat`` and the redirect ``>>`` (do *not* use ``>`` as it will overwrite the whole file).


Performing an alignment
-----------------------

We will use |muscle| to perform our alignment on all the sequences in the |blast| fasta file.
This syntax is very simple (change the filenames accordingly):


.. code:: bash

          muscle -in infile.fas -out your_alignment.aln


This will take a couple of minutes to complete, so you may want to execute it in a ``tmux`` window.

Building a phylogeny
--------------------

We will use |raxml| to build our phylogeny.
This uses a maximum likelihood method to infer parameters of evolution and the topology of the tree.
Again, the syntx of the command is fairly simple.


The arguments are:

- ``-s``: an alignment file
- ``-m``: a model of evolution. In this case we will use a general time reversible model with gamma distributed rates (GTR+GAMMA)
- ``-n``: outfile-name
- ``-p``: specify a random number seed for the parsimony inferences

  
.. code:: bash

          raxmlHPC -s your_alignment.aln -m GTRGAMMA -n ecoli_tree -p 12345


Visualizing the phylogeny
-------------------------

We will use the online software `Interactive Tree of Life (iTOL) <http://itol.embl.de/upload.cgi>`__ to visualize the tree.
Navigate to this homepage. Because you will need to do this in the browser, you will need to download
(using ``rsync`` or ``scp``) your phylogeny (``RAxML_bestTree``). Do so now.

Once you have done that, open the file containing your tree, copy the contents, and paste into the web page (in the Tree text box).

You should then be able to zoom in and out to see where your *E. coli* isolate is located relative to
other *E. coli*. 
To find out the closest relative, you will have to use the `NCBI taxa page <https://www.ncbi.nlm.nih.gov/Taxonomy/TaxIdentifier/tax_identifier.cgi>`__.

Phylogeny ToDo
~~~~~~~~~~~~~~~~~~~~~~

.. todo::

   Are you certain that the other *E. coli* you have found are related in the way that the phylogeny suggests? Why might the topology of this phylogeny not truly reflect the evolutionary history of these *E. coli* species? 
