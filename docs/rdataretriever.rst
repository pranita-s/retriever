==================================
Using the Data Retriever from R
==================================

rdataretriever
~~~~~~~~~~~~~~

The `Data Retriever`_ provides an R interface to the Data Retriever so
that the Retriever's data handling can easily be integrated into R workflows.

Installation
~~~~~~~~~~~~

To use the R package ``rdataretriever``, you first need to `install the Retriever <introduction.html#installing-binaries>`_.

The ``rdataretriever`` can then be installed using
``install.packages("rdataretriever")``

To install the development version, use ``devtools``

::

  # install.packages("devtools")
  library(devtools)
  install_github("ropensci/rdataretriever")

Note: The R package takes advantage of the Data Retriever's command line
interface, which must be available in the path. This path is given to the 
``rdataretriever`` using the function ``use_RetrieverPath()``. The location of 
``Retriever`` is dependent on the Python installation (Python.exe, Anaconda, Miniconda),
the operating system and the presence of virtual environments in the system. Following instances
exemplify this reliance and how to find Retriever's path.

Ubuntu OS with default Python:
*******************************
If ``Retriever`` is installed in default Python, it can be found out in the system with the help
of ``which`` command in the terminal. For example:

::

  $ which retriever
  /home/<system_name>/.local/bin/retriever

The path to be given as input to ``use_RetrieverPath()`` function is */home/<system_name>/.local/bin/*
as shown below:

:: 

  library(rdataretriever)
  use_RetrieverPath("/home/<system_name>/.local/bin/")

The ``which`` command in the terminal finds the exact location of ``Retriever``, but the path
required by the function is actually the directory which contains ``Retriever``. Therefore, the
final folder from the path (*retriever* in this case) needs to be removed before using it.

Ubuntu OS with Anaconda environment:
*************************************
When ``Retriever`` is installed in an virtual environment, the user can track its location only
when that particular environment is activated. To illustrate, assume the virtual environment is *py27*:

::

  $ conda activate py27
  (py27) $ which retriever
  /home/<system_name>/anaconda2/envs/py27/bin/retriever

After the location has been found out, it can be used for ``rdataretriever`` after removing last directory
from the path as follows:

::

  library(rdataretriever)
  use_RetrieverPath("/home/<system_name>/anaconda2/envs/py27/bin/")

Note: ``rdataretriever`` will be able to locate ``Retriever`` even if the virtual environment is
deactivated.

RDataRetriever functions:
~~~~~~~~~~~~~~~~~~~~~~~~~

datasets():
***********

**Arguments** : No arguments needed.

**Description** : The function returns a list of available datasets.

**Example** :

::

  rdataretriever::datasets()

fetch():
********

**Arguments** :
* dataset - name of dataset to be downloaded
* quiet - decides if warnings need to be displayed (TRUE/FALSE)
* data_names - name assigned to dataset once it is downloaded

**Description** : Each datafile in a given dataset is downloaded to a temporary directory and then imported as a
data.frame as a member of a named list.

**Example** :

::

  rdataretriever :: fetch(dataset = 'portal')

Examples
~~~~~~~~

::

 library(rdataretriever)
 
 # List the datasets available via the Retriever
 rdataretriever::datasets()
 
 # Install the Gentry forest transects dataset into csv files in your working directory
 rdataretriever::install('gentry-forest-transects', 'csv')
 
 # Download the raw Gentry dataset files without any processing to the 
 # subdirectory named data
 rdataretriever::download('gentry-forest-transects', './data/')
 
 # Install and load a dataset as a list
 Gentry = rdataretriever::fetch('gentry-forest-transects')
 names(gentry-forest-transects)
 head(gentry-forest-transects$counts)


To get citation information for the ``rdataretriever`` in R use ``citation(package = 'rdataretriever')``:


.. _Data Retriever: http://data-retriever.org



===============================
RDataRetriever with PostgreSQL
===============================

RDataRetriever supports different DBMS for storing of downloaded datasets through the Retriever R API. It supports PostgreSQL, MySQL, SQLite and MS Access.


Installation of PostgreSQL in Ubuntu
====================================

PostgreSQL installation command differs on the basis of Ubuntu version (Trusty 14.04, Xenial 16.04, Zesty 17.04). The following are the steps:

- Create the file /etc/apt/sources.list.d/pgdg.list, and add a line for the repository 

>>> deb http://apt.postgresql.org/pub/repos/apt/ <ubuntu_version>-pgdg main

For example:

>>> deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main

- Import the repository signing key, and update the package lists 

>>> wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

>>> sudo apt-get update 

- To install PostgreSQL on Ubuntu, use the apt-get (or other apt-driving) command: 

>>> apt-get install postgresql-<version>

Once it is installed, we will setup PostgreSQL by first running the postgres client as root:

>>> sudo -u postgres psql

Now, a role will be created using CREATE ROLE. A role is an entity that can own database objects and have database privileges. It can be given CREATEDB and CREATEROLE privileges. For example, the following command creates a role named ``newadmin`` with password ``abcdefgh``. It is equipped with privileges of creating a new role and creating a new database.

>>> create role newadmin with createrole createdb login password 'abcdefgh';

Next, exit the PostgreSQL client by the following command:

>>> \q

We will need to change the pg_hba.conf file to indicate that users will authenticate by md5 as opposed to peer authentication. To do this, first open the pg_hba.conf file by running the command:

>>> sudo nano /etc/postgresql/x.x/main/pg_hba.conf

Where x.x is the version installed, in this case x.x = 9.6

Change the line ``local all postgres peer`` to ``local all postgres md5``. Also, change the line ``local all all peer`` to ``local all all md5``. After making these changes, make sure to save the file.

Restart the postgresql client:

>>> sudo service postgresql restart

Now, we will be able to login to the postgres client with the newadmin user that we created by running the following command:

>>> psql -U newadmin -d postgres -h localhost -W

You will be prompted to enter a password, which is ``abcdefgh``. You can create a database (for instance: newdb) with the following command:

>>> createdb -U vmsadmin vms;

You will be again prompted for password. After the successful setup of PostgreSQL, it can now be used with R API for Retriever.

====================================
Using PostgreSQL with RDataRetriever
====================================

While using PostgreSQL as a connection type for ``rdataretriever::install`` function, a file named ``postgres.conn`` is needed. It contains information for establishing connection with the requested DBMS, in this case, PostgreSQL. Default location of the file is the directory, through which RStudio is running. If it saved in some other location, its path needs to be given to the install function.
In the above example, ``postgres.conn`` will look like below:


.. code-block:: python

  host localhost 
  port 5432 
  user newadmin
  password abcdefgh

Assuming it is saved in default directory, ``install`` function for ``airports`` dataset can be executed as follows:

>>> rdataretriever::install('airports',connection = 'postgres')





