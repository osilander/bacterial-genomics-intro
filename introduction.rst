Introduction
============

This is an introductory tutorial for learning genomics mostly on the Linux command-line.

In this tutorial you will learn how to analyse next-generation sequencing (NGS) data.

You will be given some NGS data from an "experimental evolution" experiment [KAWECKI2012]_, [ELENA2003]_. The NGS data consist of two different types of NGS data, "Illumina" and "Oxford Nanopore" (more on that later). The aim of the tutorial is to use this data to identify the genome changes in these "experimentally evolved" strains of bacteria that can explain the observed biological phenotype.

The experiment
------------
*Escherichia coli* is a bacterial species with pathogenic, commensal (i.e. living with a host) and free-living (i.e. living and growing in soil, water or plants) strains. Until 2010 or so, it was largely considered to be only pathogenic or commensal. However, since then, a large number of "natural isolates" of *E. coli* have been obtained and *phenotyped* (what does this mean?).

In my lab, one topic we focus on how these natural isolates evolve over time in the wild and in the lab.

One aim of our doing laboratory evolution of these strains has been to understand how antibiotic resistance evolves in such "natural isolates" of *E. coli*. We have evolved antibiotic resistant strains of these natural isolatesof *E. coli* hundreds of times, and now we are interested in the *genetic* changes that cause this resistance.

The data here are from one of these antibiotic resistance experiments. In particular, the experimental treatment was a period of laboratory evolution during which the isolates were grown them in ever-increasing concentrations of an antibiotic, `Streptomycin <https://en.wikipedia.org/wiki/Streptomycin>`_. After this, we generated NGS from the genomes of the resistant bacteria. It is your job to find the changes that have occurred in the genome.


The workflow
------------

The tutorial workflow is summarised in :numref:`fig-workflow`.

.. _fig-workflow:
.. figure:: /images/workflow.png

   The tutorial will follow this workflow.


Learning outcomes
-----------------

During this tutorial you will learn to:

- Check the data quality of an NGS experiment
- Create a genome assembly of the ancestor based on NGS data
- Map NGS reads of evolved lines to the created ancestral reference genome
- Call genome variations/mutations in the evolved lines
- Annotate a newly derived reference genome
- Find variants of interest that may be responsible for the observed evolved phenotype

  
.. only:: html

   .. rubric:: References

               
.. [KAWECKI2012] Kawecki TJ et al. Experimental evolution. `Trends in Ecology and Evolution (2012) 27:10 <http://dx.doi.org/10.1016/j.tree.2012.06.001>`__
               
.. [ELENA2003] Elena SF and Lenski RE. Evolution experiments with microorganisms: the dynamics and genetic bases of adaptation `Nature Reviews Genetics (2003) 4: 457–469 <https://www.nature.com/articles/nrg1088>`__

