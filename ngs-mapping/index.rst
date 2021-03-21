.. _ngs-mapping:

Read mapping
============

Preface
-------

In this section we will use our skill on the command-line interface to map our
reads from the evolved line to our ancestral reference genome.

.. The first part of the following lecture is of importance to this tutorial (`ChIP - An Introduction <https://doi.org/10.6084/m9.figshare.1554130.v1>`__).

.. NOTE::

   You will encounter some **To-do** sections at times. Write the solutions and answers into a text-file.


Overview
--------

The part of the workflow we will work on in this section can be viewed in :numref:`fig-workflow-map`.

.. _fig-workflow-map:
.. figure:: images/workflow.png

   The part of the workflow we will work on in this section marked in red.


Learning outcomes
-----------------

After studying this section of the tutorial you should be able to:

#. Explain the process of sequence read mapping.
#. Use bioinformatics tools to map sequencing reads to a reference genome.
#. Filter mapped reads based on quality.


Before we start
---------------

Lets see what our directory structure looks like so far.

.. code:: bash

          tree -L 2

          # if we have used snakemake the structure
          # should look similar to below
          # with a data and results folder, and in the
          # results folder there are trimmed reads and
          # and assemblies
          ├── data
          │   ├── illumina
          │   └── nanopore
          ├── results
          │   ├── H8_anc
          │   ├── H8_anc_fastp.html
          │   ├── H8_anc_fastp.json
          │   ├── H8_anc.hiqual.fastq
          │   ├── H8_anc_R1.trimmed.fastq
          │   ├── H8_anc_R1.trimmed.sub.fastq
          │   ├── H8_anc_R2.trimmed.fastq
          │   └── H8_anc_R2.trimmed.sub.fastq
          └── Snakefile


Mapping sequence reads to a reference genome
--------------------------------------------

Now that we have assembled a reference genome for our *ancestral* clone, we want to identify the changes that have occurred in the *evolved* clone. There are at lteast two possible ways to do this. One option would be to assemble a second genome for our evolved clone and compare this to the ancestral genome. However, this would be the wrong approach for two reasons. First, it is more computationally difficult to peform another assembly. Thus, we would be wasting time and effort and computational resources. Second, we would not get any measure of how *sure* we could be that a change occurred. For these reasons we will instead **map** our reads onto the genome that we have assembled in the section :ref:`ngs-assembly`. We will then figure out which mutations have occurred. This process is often denoted *calling vraiants*.

To map reads and call variants we will now use the second set of Illumina reads that you have, those from the evolved clone. Make sure that those are the ones you are dealing with today. First, we will go through the process of mapping them, and then we will add a rule to our ``snakefile`` that spceifies the input and output files tht we need, and the method used to create them.

First, you need to make sure that the reads you are using have been trimmed using |fastp|. If they are not, please go ahead and do that. For reference on how to use ``fastp``, see QC section :ref:`ngs-qc` or refer to your ``snakefile``.

We are going to use the quality trimmed forward and backward DNA sequences of the evolved line and use a program called |bwa| to map the reads.

.. todo::

   #. Discuss briefly why we are using the ancestral genome as a reference genome as opposed to a genome for the evolved line.


Installing the software
~~~~~~~~~~~~~~~~~~~~~~~

We are going to use a program called |bwa| to map our reads to our genome. Please install it now using ``conda``.


BWA
---


Overview
~~~~~~~~

|bwa| is a versatile read aligner that can take a reference genome and map single- or paired-end data to it [LI2009]_. The method that it uses for this is the Burrows-Wheeler transform, and it was one of the first read aligners to adopt this strategy (with ``bowtie``).

|bwa| first requires an indexing step for which you need to supply the reference genome. In subsequent steps this index will be used for aligning the reads to the reference genome. The general command structure of the |bwa| tools we are going to use are shown below:

.. code:: bash

   # bwa index help
   bwa index

   # indexing
   bwa index path/to/reference-genome.fa

   # bwa mem help
   bwa mem

   # single-end mapping, general command structure, adjust to your case
   bwa mem path/to/reference-genome.fa path/to/reads.fq.gz > path/to/aln-se.sam

   # paired-end mapping, general command structure, adjust to your case
   bwa mem path/to/reference-genome.fa path/to/read1.fq.gz path/to/read2.fq.gz > path/to/aln-pe.sam



Create an |bwa| index for your reference genome assembly now using the ``bwa index`` command. Attention! Remember which file you need to submit to |bwa|.


Mapping reads in a paired-end manner
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now that we have created our index, it is time to map the filtered and trimmed sequencing reads of our evolved line to the reference genome. Use the correct ``bwa mem`` command structure from above and map the reads of the evolved line to the reference genome.


.. _sam-file-format:

The sam mapping file-format
---------------------------

|bwa| will produce a mapping file in ``sam`` format (Sequence Alignment/Map). Have a look into the sam-file that was created by either program.
A quick overview of the ``sam`` format can be found `here <http://bio-bwa.sourceforge.net/bwa.shtml#4>`__.
Briefly, first there are a set of header lines for each file detailing what information is contained in the file. Then, for each read, that mapped to the reference, there is one line with information about the read in 12 different columns.

The columns of such a line in the mapping file are described in :numref:`table-sam`.

.. _table-sam:
.. table:: The sam-file format fields.

   +-----+---------+-----------------------------------------------------------+
   | Col |  Field  | Description                                               |
   +=====+=========+===========================================================+
   | 1   | QNAME   | Query (pair) NAME                                         |
   +-----+---------+-----------------------------------------------------------+
   | 2   | FLAG    | bitwise FLAG                                              |
   +-----+---------+-----------------------------------------------------------+
   | 3   | RNAME   | Reference sequence NAME                                   |
   +-----+---------+-----------------------------------------------------------+
   | 4   | POS     | 1-based leftmost POSition/coordinate of clipped sequence  |
   +-----+---------+-----------------------------------------------------------+
   | 5   | MAPQ    | MAPping Quality (Phred-scaled)                            |
   +-----+---------+-----------------------------------------------------------+
   | 6   | CIAGR   | extended CIGAR string                                     |
   +-----+---------+-----------------------------------------------------------+
   | 7   | MRNM    | Mate Reference sequence NaMe (‘=’ if same as RNAME)       |
   +-----+---------+-----------------------------------------------------------+
   | 8   | MPOS    | 1-based Mate POSition                                     |
   +-----+---------+-----------------------------------------------------------+
   | 9   | ISIZE   | Inferred insert SIZE                                      |
   +-----+---------+-----------------------------------------------------------+
   | 10  | SEQ     | query SEQuence on the same strand as the reference        |
   +-----+---------+-----------------------------------------------------------+
   | 11  | QUAL    | query QUALity (ASCII-33 gives the Phred base quality)     |
   +-----+---------+-----------------------------------------------------------+
   | 12  | OPT     | variable OPTional fields in the format TAG\:VTYPE\:VALUE  |
   +-----+---------+-----------------------------------------------------------+

One line of a mapped read can be seen here:

.. code:: bash

    M02810:197:000000000-AV55U:1:1101:10000:11540   83      NODE_1_length_1419525_cov_15.3898       607378  60      151M    =       607100  -429    TATGGTATCACTTATGGTATCACTTATGGCTATCACTAATGGCTATCACTTATGGTATCACTTATGACTATCAGACGTTATTACTATCAGACGATAACTATCAGACTTTATTACTATCACTTTCATATTACCCACTATCATCCCTTCTTTA FHGHHHHHGGGHHHHHHHHHHHHHHHHHHGHHHHHHHHHHHGHHHHHGHHHHHHHHGDHHHHHHHHGHHHHGHHHGHHHHHHFHHHHGHHHHIHHHHHHHHHHHHHHHHHHHGHHHHHGHGHHHHHHHHEGGGGGGGGGFBCFFFFCCCCC NM:i:0  MD:Z:151        AS:i:151        XS:i:0

Mpst importantly, this line defines the read name, the position in the reference genome where the read maps, and the quality of the mapping.


Mapping post-processing
-----------------------

Fix mates and compress
~~~~~~~~~~~~~~~~~~~~~~

Because aligners can sometimes leave unusual `SAM flag <http://bio-bwa.sourceforge.net/bwa.shtml#4>`__ information on SAM records, it is helpful when working with many tools to first clean up read pairing information and flags with |samtools|.
We are going to produce also compressed bam output for efficient storing of and access to the mapped reads. To understand why we are going to compress the file, take a look at the size of your original ``fastq`` files that you used for mapping, and the size of the ``sam`` file that resulted. Along the way toward compressing, we will also sort our reads for easier access. This simply means we will order the reads by the position in the genoome that they map to. 

To perform all of these steps, we will rely on a powerful quite of software tools that are implemented in ``samtools``. The first of these, then is ``sort``. One very important aspect of ``samtools`` that you should always remember is that in almost all cases **the default behaviour of ``samtools`` is to output to the terminal (standard out)**. For that reason, we will be using the redirect arrow ``>`` quite a bit. In other cases, we will use the "pipe" operator ``|``. We use the pipe operator so that we do not have to deal with intermediate files.

First, we use ``samtools fixmate``, which according to its documentation, can be used to: "Fill in mate coordinates, ISIZE and mate related flags from a name-sorted or name-collated alignment." Here, ``ISIZE`` refers to insert size.


Note, ``samtools fixmate`` expects **name-sorted** input files, which we can achieve with ``samtools sort -n``.


.. code:: bash

   # -n sorts by name
   # -O sam outputs sam format
   samtools sort -n -O sam my_mapped_file.sam > my_mapped_sort.sam

Next, we need to take this name-sorted by and give it to ``samtools fixmate`` . This will fill in our extra fields. We will also output in compressed ``.bam`` format.

- ``-m``: Add ms (mate score) tags. These are used by markdup (below) to select the best reads to keep.
- ``-O bam``: specifies that we want compressed bam output from fixmate.

.. code:: bash

   # -n sorts by name
   # -O sam outputs sam format
   samtools fixmate -m -O bam my_mapped_sort.sam > my_mapped_fixmate.bam

.. attention::

   Make sure that you are following the file naming conventions for your suffixes. Simple mapped files will be in ``sam`` format and should be denoted by that suffix. The *compressed* version will be in ``bam`` format, and be denoted by that suffix.

Once we have this fixmate bam-file, delete the ``.sam`` files as they take up a significant amount of space. Use ``rm`` to do this, but **be careful because ``rm`` is forever**.

We will be using the `SAM flag <http://bio-bwa.sourceforge.net/bwa.shtml#4>`__ information later below to extract specific alignments.

.. hint::

   A very useful tools to explain samtools flags can be found `here <http://broadinstitute.github.io/picard/explain-flags.html>`__.


Sorting by location
~~~~~~~

We are going to use |samtools| again to sort the ``.bam`` file into **coordinate order**:


.. code:: bash

    # sort by location
    # -O indicates bam output again
    # note the redirect > arrow
    samtools sort -O bam my_mapped_fixmate.bam > my_mapped_soprted.bam


Remove duplicates
~~~~~~~~~~~~~~~~~

In this step we remove duplicate reads. The main purpose of removing duplicates is to mitigate the effects of PCR amplification bias introduced during library construction.
**It should be noted that this step is not always recommended.**
It depends on the research question.
In SNP calling it is a good idea to remove duplicates, as the statistics used in the tools that call SNPs sub-sequently expect this (most tools anyways).
However, for other research questions that use mapping, you might not want to remove duplicates, e.g. RNA-seq.

.. code:: bash

    # Markdup can simply *marks* the duplicate reads
    # But the -r option tells it to remove those reads.
    # This works on a very simple principal that we will discuss
    samtools markdup -r -S mappings/evolved-6.sorted.bam mappings/evolved-6.sorted.dedup.bam

.. todo::

   Exlpain what "PCR amplification bias" means and discuss why it might not be used for RNA-seq experiments.


Mapping statistics
------------------

Stats with SAMtools
~~~~~~~~~~~~~~~~~~~

Lets get an mapping overview:


.. code:: bash

    samtools flagstat mappings/evolved-6.sorted.dedup.bam


.. todo::

   Look at the mapping statistics and understand `their meaning
   <https://www.biostars.org/p/12475/>`__. Discuss your results.
   Explain why we may find mapped reads that have their mate mapped to a different chromosome/contig?
   Can they be used for something?


For the sorted bam-file we can get read depth for at all positions of the reference genome, e.g. how many reads are overlapping the genomic position.


.. code:: bash

    samtools depth mappings/evolved-6.sorted.dedup.bam | gzip > mappings/evolved-6.depth.txt.gz


.. todo::

   Extract the depth values for contig 20 and load the data into R, calculate some statistics of our scaffold.


.. code:: bash

   zcat mappings/evolved-6.depth.txt.gz | egrep '^NODE_20_' | gzip >  mappings/NODE_20.depth.txt.gz


Now we quickly use some |R| to make a coverage plot for contig NODE20.
Open a |R| shell by typing ``R`` on the command-line of the shell.

.. code:: R

   x <- read.table('mappings/NODE_20.depth.txt.gz', sep='\t', header=FALSE,  strip.white=TRUE)

   # Look at the beginning of x
   head(x)

   # calculate average depth
   mean(x[,3])
   # std dev
   sqrt(var(x[,3]))

   # mark areas that have a coverage below 20 in red
   plot(x[,2], x[,3], col = ifelse(x[,3] < 20,'red','black'), pch=19, xlab='postion', ylab='coverage')

   # to save a plot
   png('mappings/covNODE20.png', width = 1200, height = 500)
   plot(x[,2], x[,3], col = ifelse(x[,3] < 20,'red','black'), pch=19, xlab='postion', ylab='coverage')
   dev.off()


The result plot will be looking similar to the one in :numref:`coverage`

.. _coverage:
.. figure:: images/covNODE20.png

   A example coverage plot for a contig with highlighted in red regions with a coverage below 20 reads.


.. todo::

   Look at the created plot. Explain why it makes sense that you find relatively bad coverage at the beginning and the end of the contig.


Stats with QualiMap
~~~~~~~~~~~~~~~~~~~

For a more in depth analysis of the mappings, one can use |qualimap| [OKO2015]_.

|qualimap| examines sequencing alignment data in SAM/BAM files according to the features of the mapped reads and provides an overall view of the data that helps to the detect biases in the sequencing and/or mapping of the data and eases decision-making for further analysis.

Installation:


.. code::

   conda install qualimap


Run |qualimap| with:


.. code:: bash

   qualimap bamqc -bam mappings/evolved-6.sorted.dedup.bam


This will create a report in the mapping folder.
See this `webpage <http://qualimap.bioinfo.cipf.es/doc_html/analysis.html#output>`__ to get help on the sections in the report.


.. todo::

   Install |qualimap| and investigate the mapping of the evolved sample. Write
   down your observations.



Sub-selecting reads
-------------------

It is important to remember that the mapping commands we used above, without additional parameters to sub-select specific alignments (e.g. for |bowtie| there are options like ``--no-mixed``, which suppresses unpaired alignments for paired reads or ``--no-discordant``, which suppresses discordant alignments for paired reads, etc.), are going to output all reads, including unmapped reads, multi-mapping reads, unpaired reads, discordant read pairs, etc. in one file.
We can sub-select from the output reads we want to analyse further using |samtools|.

.. todo::

   Explain what concordant and discordant read pairs are? Look at the |bowtie| manual.


Concordant reads
~~~~~~~~~~~~~~~~

We can select read-pair that have been mapped in a correct manner (same chromosome/contig, correct orientation to each other, distance between reads is not stupid).


.. code:: bash

   samtools view -h -b -f 3 mappings/evolved-6.sorted.dedup.bam > mappings/evolved-6.sorted.dedup.concordant.bam

- ``-h``: Include the sam header
- ``-b``: Output will be bam-format
- ``-f 3``: Only extract correctly paired reads. ``-f`` extracts alignments with the specified `SAM flag <http://bio-bwa.sourceforge.net/bwa.shtml#4>`__ set.


.. todo::

   Our final aim is to identify variants. For a particular class of variants, it is not the best idea to only focus on concordant reads. Why is that?


Quality-based sub-selection
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In this section we want to sub-select reads based on the quality of the mapping.
It seems a reasonable idea to only keep good mapping reads.
As the SAM-format contains at column 5 the :math:`MAPQ` value, which we established earlier is the "MAPping Quality" in Phred-scaled, this seems easily achieved.
The formula to calculate the :math:`MAPQ` value is: :math:`MAPQ=-10*log10(p)`, where :math:`p` is the probability that the read is mapped wrongly.
However, there is a problem!
**While the MAPQ information would be very helpful indeed, the way that various tools implement this value differs.**
A good overview can be found `here <https://sequencing.qcfail.com/articles/mapq-values-are-really-useful-but-their-implementation-is-a-mess/>`__.
Bottom-line is that we need to be aware that different tools use this value in different ways and the it is good to know the information that is encoded in the value.
Once you dig deeper into the mechanics of the :math:`MAPQ` implementation it becomes clear that this is not an easy topic.
If you want to know more about the :math:`MAPQ` topic, please follow the link above.

For the sake of going forward, we will sub-select reads with at least medium quality as defined by |bowtie|:

.. code:: bash

   samtools view -h -b -q 20 mappings/evolved-6.sorted.dedup.bam > mappings/evolved-6.sorted.dedup.q20.bam

- ``-h``: Include the sam header
- ``-q 20``: Only extract reads with mapping quality >= 20


.. hint::

   I will repeat here a recommendation given at the source `link <https://sequencing.qcfail.com/articles/mapq-values-are-really-useful-but-their-implementation-is-a-mess/>`__ above, as it is a good one: If you unsure what :math:`MAPQ` scoring scheme is being used in your own data then you can plot out the :math:`MAPQ` distribution in a BAM file using programs like the mentioned |qualimap| or similar programs.
   This will at least show you the range and frequency with which different :math:`MAPQ` values appear and may help identify a suitable threshold you may want to use.


Unmapped reads
~~~~~~~~~~~~~~

We could decide to use |kraken| like in section :ref:`taxonomic-investigation` to classify all unmapped sequence reads and identify the species they are coming from and test for contamination.

Lets see how we can get the unmapped portion of the reads from the bam-file:


.. code:: bash

    samtools view -b -f 4 mappings/evolved-6.sorted.dedup.bam > mappings/evolved-6.sorted.unmapped.bam

    # count them
    samtools view -c mappings/evolved-6.sorted.unmapped.bam


- ``-b``: indicates that the output is BAM.
- ``-f INT``: only include reads with this `SAM flag <http://bio-bwa.sourceforge.net/bwa.shtml#4>`__ set. You can also use the command ``samtools flags`` to get an overview of the flags.
- ``-c``: count the reads


Lets extract the fastq sequence of the unmapped reads for read1 and read2.


.. code:: bash

    bamToFastq -i mappings/evolved-6.sorted.unmapped.bam -fq mappings/evolved-6.sorted.unmapped.R1.fastq -fq2  mappings/evolved-6.sorted.unmapped.R2.fastq


.. only:: html

   .. rubric:: References


.. [TRAPNELL2009] Trapnell C, Salzberg SL. How to map billions of short reads onto genomes. `Nat Biotechnol. (2009) 27(5):455-7. doi: 10.1038/nbt0509-455. <http://doi.org/10.1038/nbt0509-455>`__

.. [LI2009] Li H, Durbin R. (2009). Fast and accurate short read alignment with Burrows-Wheeler transform. `Bioinformatics. 25 (14): 1754–1760. <https://doi.org/10.1093%2Fbioinformatics%2Fbtp324>`__

.. [OKO2015] Okonechnikov K, Conesa A, García-Alcalde F.  Qualimap 2: advanced multi-sample quality control for high-throughput sequencing data. `Bioinformatics (2015), 32, 2:292–294. <https://doi.org/10.1093/bioinformatics/btv566>`__
