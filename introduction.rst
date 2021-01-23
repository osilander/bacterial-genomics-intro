Introduction
============

This is an introductory tutorial for learning genomics mostly on the Linux command-line.

In this tutorial you will learn how to analyse next-generation sequencing (NGS) data. The data you will be using is actual research data from an "experimental evolution" experiment [KAWECKI2012]_, [ELENA2003]_. The aim of the tutorial is to identify the genome changes in experimentally evolved lines of wild bacteria that can explain the observed biological phenotype.

The experiment
------------
*Escherichia coli* is a bacterial species with pathogenic, commensal (i.e. living with a host) and free-living (i.e. living and growing in soil, water or plants) strains. Until 2010 or so, it was considered to be only pathogenic or commensal. However, since then, a large number of "natural isolates" have been obtained and *phenotyped* (what does this mean?). In my lab, one topic we focus on is the role of these natural isolates in microbial ecology, as well as how they evolve over time.

One line of investigation in our lab has been aimed at trying to undertstnad the frequency and mechansism fo antibiotic resistance  iin theser "natural isolates" of *E. coli*.

In my lab, we conducted an experiment using four natural isolates of E. coli: A5, H5, H7, and H8.


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

