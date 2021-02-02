Quick command reference
=======================

Shell commands
--------------

As we have discussed elsewhere, there is one fundamental aspect of using the command line that you **must never forget**. It is perhaps the single most powerful method available to save time. That method is...


**tab-complete**


Tab-complete can be used to auto-complete commands, directory names, and file names. If you are not sure whether your file is named ``results_QC.txt`` or ``results_qc.txt`` then on the command line you can simply type ``results`` *and then tab*, and the computer will auto-complete the name (assuming there is a file or directory or command that begins with ``results``).


Commands for navigating the shell
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    # List the documents and sub-directories in the current directory
    ls

    # A nicer listing with more information
    ls -laF

    # A graphic illustration of the directiories
    # showing downto 2 levels
    tree -L 2

    # Change into your home directory
    cd

    # Change back into the last directory
    cd -

    # Change one directory up in the tree
    cd ..

    # Change explicitly into a directory "temp"
    cd temp

Commands to peek inside files
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

    # Quickly show content of a file "temp.txt"
    # exist the view with "q", navigate line up and down with "k" and "j" and "spacebar"
    less temp.text

    # Show the first ten lines of a file "temp.txt"
    head temp.txt

    # Show the last ten lines of a file "temp.txt"
    tail temp.txt

Commands for moving and copying
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash
    # Copy a file
    cp  myfile.txt mynewfile.txt

    # Copy a directory
    cp -r mydir mynewdir

    # Rename a file
    mv myfile.txt renamed.txt

    # Move a file
    mv myfile.txt newplace/

    # Make a new directory
    mkdir mydir

Commands for deleting
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash
    # Delete a file
    rm myfile.txt

    # Delete a directory
    rmdir mydir

    # Delete a directory with thigns in it (*careful!*)
    rm -r mydir


Commands for when you forget
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash
    # Where in the directory tree am I?
    pwd

    # I don't know a command means
    man unknown_command

    # I can't find my file but I know
    # the file is called "pattern"-something 
    find . -name "pattern"

    # What was that command I used ten minutes ago?
    history

    # I used a command ten minutes ago I can't
    # remember but it's something like "pattern"
    history | grep "pattern"

    # Find a line in a file that has a certain pattern
    grep  "pattern" myfile.txt


General conda commands
----------------------

.. code:: bash

    # List all packages installed
    conda list [-n env]

    # List environments
    conda env list

    # Create a new environment
    conda create -n [env-name] package [package1-name] ...

    # Activate an environment
    conda activate [name]

    # Deavtivate env
    conda deactivate
