.. _tool-installation:

Tool installation
=================

**Reminder**: You **must never forget** tab-complete.


**Reminder**: *google is your friend*.


Install the conda package manager
---------------------------------


Software **packages** and tools are pieces of software that have been developed to perform specific jobs, or are used to implement specific methods. Your general view of a software package may be something like Excel or Chrome or TikTok. More fundamentally, software is simply a group of instructions used to perform a specific task. In bioinformatics, for example, this could be a set of instructions telling the computer how to interpret and display the quality scores from a ``.fastq`` file.


*However*, software packages and tools often have **dependencies**, which are other pieces of software or tools that are necessary to run the software you would like to install. For example, to use Instagram, you also need software that controls your phone's camera. This reliance of Instagram on camera-controlling software is known as a **dependency**. Importantly, software like Instagram is designed to be **user-friendly**, and during installation will usually check that such camera-controlling software exists, and if it does not, may try to install it.

Despite the existence of  dependencies, many bioinformatics software programs, much of which is written by inexperienced computer scientists (or worse, biologists) do not check for dependencies. This can create significant issues if you try to run a piece of software but are missing dependencies (the other pieces of software that are also required :numref:`fig-dependencies`).

.. _fig-dependencies:
.. figure:: images/dependencies.jpg

  They're everywhere so we have to be careful.


To make sure that we resolve all these dependency issues, we will use a package/tool managing system. This managing system is called |conda|, and it is perhaps the most common package manager used in bioinformatics.

The process of installing a software package is called a *recipe*, and these recipes are contained in places called *channels*. Most recipes for bioinformatic software is contained in the `bioconda <https://bioconda.github.io/>`_ channel, which currently has recipes for more than 7000 software packages. |conda| is not installed by default, thus you need to install it first to be able to use it.

The installation of this tool is perhaps the most complicated installation we will do in this course. However, after the installation of |conda|, your life will become far easier and you will be on your way to becoming a seasoned bioinformatician (`binfie <https://soundcloud.com/microbinfie>`_). Note that in the code sample below, you will *not* be able to use tab-complete because the file does not yet exist on your computer.


.. code-block:: bash

    # download latest conda installer
    curl -O https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh

**Explanation**: ``curl`` is a program that is used to transfer data to or from a server on the command line. Thus, this command is simply using this program to find the file at the location indicated. This file (with the extension ``.sh``) is a ``bash`` file, which is usually run using the command line program ``bash``.


.. Attention::
   As alluded to previously, noting the *extension* of a file can be very helpful in figuring out what is in it, or what it does. For example, you should never end a ``bash`` file with ``.txt`` as that suggests it is a simple text file, when in fact it is not. Similarly, you would never end a Microsoft Word file with ``.xlsx``, you would end it with ``.doc`` or ``.docx``

Let's now actually install ``conda`` (in our case we install a miniature version of it with less bloat, ``miniconda``)

.. code-block:: bash

    # run the installer
    # note: now you can use tab-complete
    bash Miniconda3-latest-Linux-x86_64.sh
    
    # delete the installer after successful run
    rm Miniconda3-latest-Linux-x86_64.sh


.. Tip::
   #. Why should you be careful when using ``rm`` in the above command?


.. Note::
   Should the conda installer download fail. Please find links to alternative locations on the
   :doc:`../downloads` page.

    
Update your ``.bashrc`` and ``.zshrc`` config-files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before you are able to use |conda| you need to tell our shell where it can find the program.
We add the right path to the |conda| installation to our shell config files:

.. code::
   
   echo 'export PATH="/home/manager/miniconda3/bin:$PATH"' >> ~/.bashrc
   echo 'export PATH="/home/manager/miniconda3/bin:$PATH"' >> ~/.zshrc


.. Attention::
   The above assumes that your username is "manager", which is the default on a Biolinux install.
   Replace "manager" with your actual username.
   Find out with ``whoami``. (What does the ``whoami`` command do?)
   
.. Tip::
   #. What does ``echo`` mean in the above command?
   #. What does the ``>>`` do in the above command?
   #. What is inside of the "shell config files" (e.g. ``.bashrc``)?
   #. Why are the shell configuration files preceeded by a ``.``? What effect does this have? (hint: google "hidden file") 

**Explanation**: So what is actually happening here? We are appending a line to a file (either ``.bashrc`` or ``.zshrc``).
If you are starting a new command-line shell, either file gets executed first (depending on which shell you are using, either bash or zsh shells).
What this line does is to put permanently the directory ``~/miniconda3/bin`` first on your ``PATH`` variable. The little ``~`` (tilde) at the start is short-hand for your home directory. **Why** do we need to append this? Read on:

The ``PATH`` variable contains places (directories) in which your computer looks for  programs. These directories are listed one after the other. The computer will search these in the order they are listed until the program you requested is found (or not, then it will complain). For example, you might have a ``PATH`` variable that says: first look in my home directory (``~/``), and then in the ``/usr/bin/`` directory, and then in my friend's directory (``friends_dir/sneaky_files_i_saved_there/``). However, those are *the only* places the computer will look. If you want the computer to look in more places, you have to add those locations to the ``PATH`` variable. The ``$`` indicates that it is a *variable*.


Through the addition of the above line you have now told the computer to also look in ``/home/manager/miniconda3/bin`` so that the program ``conda`` can be found anytime you open a new shell.


Finally, close the shell/terminal and open a **new** shell/terminal.
Now, you should be able to use the |conda| command. One useful way to check that |conda| (*or any other command line program*) is to ask what the program does. This is **almost always** done by typing ``--help`` or ``-h`` after the command. For example try:


.. code-block:: bash

    conda --help

This will bring up a list of sub-commands that |conda| can do. Try it.


Finally, make sure you have the current version of |conda|:


.. code-block:: bash

    conda update conda


Configure conda channels to make tools available
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The methods to install different tools are called recipes, and these are stored in what |conda| calls channels (as noted above). To make sure |conda| looks in the right places for these recipes, we need to tell it what channels to look in, and in what order to search them. This will make the bioinformatics and genomics tools easily find-able for installation:


.. code-block:: bash
    
    # Install some conda channels
    # A channel is where conda looks for recipes to install pakcages
    conda config --add channels defaults
    conda config --add channels bioconda 
    conda config --add channels conda-forge     

   
Create environments
-------------------

Now that we have a method to manage the installation of software packages (the |conda| *package manager*), there may be times that we want to have multiple different versions of a software tools installed (e.g. both ``python 2.7`` and ``python 3.7``). In addition, there may be some software tools that *conflict* with other software tools. This creates a new problem for us. However, we can solve this by creating different |conda| environments. You can imagine these as independent rooms in a larger conda house. In these environments (rooms) we can install only certain versions of a software tool, or only certain pieces of software. So if you want to have a set of specific software tools for performing QC, you can put those in the QC room (environment), and they will stay in there and not interfere with tools you have installed in other rooms (environments).


.. code-block:: bash

    # make a new environment with version 3.7 of python
    # think of a nifty memorable name
    # here we use ngs ("next generation sequencing")
    conda create -n ngs python=3.7
    
    # activate the environment
    conda activate ngs

    
So what is happening when you type ``conda activate ngs`` in a shell?
The ``PATH`` variable (mentioned above) gets temporarily manipulated like so:

.. code-block:: bash

    # make a new environment with version 3.7 of python
    # (we did this in the last code block using the
    # conda create command)

    # in the line below the $ indicates that you are
    # at the command line prompt
    $ conda activate ngs

    # Lets look at the content of the PATH variable
    # Note that the command line prompt now has (ngs)
    # Note also that we prefix PATH with a $ as it is a variable
    (ngs) $ echo $PATH
    /home/manager/miniconda3/envs/ngs/bin:/home/manager/miniconda3/bin:/usr/local/bin: ...


Note that the colons (``:``) in the above text indicate separations between the directory listings.

Now it will look first in your specific |conda| environment's ``bin/`` directory but afterwards in the **general** conda ``bin/`` (``/home/manager/miniconda3/bin``).
So basically, everything you install generally with conda (without being in an environment) is also available to you, but gets overshadowed if a similar program is in ``/home/manager/miniconda3/envs/ngs/bin`` and you are in the ``ngs`` environment.

The **huge** additional advantage of making separate |conda| environments in which you do your work is that it makes your work **reproducible**, as you can easily re-create the entire tool-set with exactly the same software versions numbers later on (e.g. years later, when the functionality of the current software version may have changed completely).

.. Tip::
   Extra-credit reading: `What are <https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard#Directory_structure>`_ all these ``bin/`` directories, and why are they called "bin"?


Install software
----------------

To install software into the activated environment, use the command ``conda install``.

.. code-block:: bash
         
    # install more tools into the environment
    conda install cool-new-package

.. Tip::
   Does this instruction *really* mean that you install all packages using the phrase "cool-new-package"?

.. note::
   To tell if you are in the correct conda environment, look at the command-prompt.
   Do you see the name of the environment in round brackets at the very beginning of the prompt, e.g. ``(ngs)``?
   If not, activate the ``ngs`` environment with ``conda activate ngs`` before installing the tools.

    
                
General conda commands
----------------------

.. code-block:: bash

    # to search for packages
    conda search [package]
    
    # To update all packages
    conda update --all --yes

    # List all packages installed
    conda list [-n env]

    # conda list environments
    conda env list

    # create new env
    conda create -n [environment-name] package [package] ...

    # activate env
    conda activate [environment-name]

    # deavtivate env
    conda deactivate
