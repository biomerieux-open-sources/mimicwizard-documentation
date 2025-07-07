Installation
########################

You'll find here information about how to make MIMICWizard work on your computer/infrastructure.

Before starting you should keep in mind that MIMICWizard is divided in 2 part, the **application** run with R-Shiny and the **database** run with PostgreSQL

TL;DR
-----

A. Install the application

   * Install R 4.4 and Rtools (Windows only)
   * Clone the MIMICWizard repository from GitHub
   * Install the required R packages using renv

B. Install the PostgreSQL server and import MIMIC-IV database
   
   * Install PostgreSQL server
   * Import the MIMIC-IV database (demo or full version)
   * Import MIMICWizard internal schema

C. Configure and start MIMICWizard
   
   * Configure the application configuration file
   * Run the application

D. Extend and optimize the application (Optionnal)

   * Add additional index to speed up the application
   * Add microbiology supplementary data to the application

A. Install the application (Run from RStudio or R CLI)
*********************************************************
*Theses instructions are the simplest way to get MIMICWizard running, but it's not the way it is intended to be hosted and distributd to multiple users in your organization*

You can download the MIMICWizard **source code** using git or directly by zip downloading from GitHub.

.. code-block:: bash

   git clone https://github.com/biomerieux-open-sources/mimicwizard.git

In order to install the app, you'll need to have **R 4.4** installed `R official repository (CRAN) <https://cran.r-project.org/mirrors.html>`_ 

For Windows user, you need to install Rtools to compile some of the packages, available on the `Rtools official repository <https://cran.r-project.org/bin/windows/Rtools/>`_

Once R is installed, open a terminal in the project directory and run the following command to install the required packages :

.. code-block:: bash

   R -e "renv::restore()"


This step should take a few minutes, go grab a coffee ☕ 

Your coffee is finished but the package installation is still ongoing ? Don't worry, you can carry to the next step and let the installation finish in the background.

B. Install the PostgreSQL server and import MIMIC-IV database
***************************************************************

Now you need to install the **PostgreSQL server** that will hold the database. 

You can download the latest version of PostgreSQL with your favorite package manager for UNIX user or on the `PostgreSQL official repository <https://www.postgresql.org/download/>`_ for windows user.

.. note::

   Alternatively, Windows 10 users (Windows 11 is not supported) can use `PostgreSQL Portable <https://github.com/rsubr/postgresql-portable>`_ (originally developed by ``garethflowers``, kept up-o-date by ``rsubr``) which, as is name discreetly suggest is a portable implementation of a PostgreSQL database manager.
   *Note that it's possible to host the full MIMIC-IV database in the portable version but it's not recommended due to performance issue*

You should then import the MIMIC-IV database in your PostgreSQL server.

There's now two choices :

* `Use  MIMIC-IV demo <Import MIMIC-IV demo to your PostgreSQL server_>`_ 

* `Use  MIMIC-IV full version <Import MIMIC-IV full version to your PostgreSQL server_>`_ 


Import MIMIC-IV demo to your PostgreSQL server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Download the MIMIC-IV demo database available on the `Physionet Repository - MIMIC-IV Clinical Database demo <https://physionet.org/content/mimic-iv-demo/>`_ (the download button is at the bottom of the page).
* Unzip the database in the demo folder at the MIMICWizard root repository

Your MIMICWizard root repository should now look like that 

.. code-block::

   cache
   demo/
   -- hosp/
   -- icu/
   -- ...
   R/
   renv/
   app.R
   DESCRIPTION
   renv.lock

If it's the case that's perfect, you just have to run ``PostgreSQLPortable.exe`` before launching the app and that's it, your demo database will be automatically populated on MIMICWizard startup.

.. tip:: 

   Use the Init Demo procedure on the application homepage the first time you connect to the database with MIMICWizard. This procedure will use the file in the demo folder to populate your database. Once it has been done one time, you could use the run demo procedure.

Import MIMIC-IV full version to your PostgreSQL server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
In order to host the full database, we recommend you following the process below (adapted from mimic-code repository).

.. code-block:: bash

   # clone repo
   git clone https://github.com/MIT-LCP/mimic-code.git
   cd mimic-code
   # download data
   wget -r -N -c -np --user <USERNAME> --ask-password https://physionet.org/files/mimiciv/2.2/
   mv physionet.org/files/mimiciv mimiciv && rmdir physionet.org/files && rm physionet.org/robots.txt && rmdir physionet.org
   createdb mimiciv
   psql -d mimiciv -f mimic-iv/buildmimic/postgres/create.sql
   psql -d mimiciv -v ON_ERROR_STOP=1 -v mimic_data_dir=mimiciv/2.2 -f mimic-iv/buildmimic/postgres/load_gz.sql
   psql -d mimiciv -v ON_ERROR_STOP=1 -v mimic_data_dir=mimiciv/2.2 -f mimic-iv/buildmimic/postgres/constraint.sql
   psql -d mimiciv -v ON_ERROR_STOP=1 -v mimic_data_dir=mimiciv/2.2 -f mimic-iv/buildmimic/postgres/index.sql
   cd mimic-iv/concepts_postgres/ | psql -d mimiciv -f  postgres-make-concepts.sql

If you can't use wget, you can download the data manually from `Physionet Repository - MIMIC-IV Clinical Database <https://physionet.org/content/mimic-iv/2.2/>`_ and put the data in the mimiciv/2.2 folder.
You may need to adapt this sample code depending on your configuration

Import MIMICWizard internal table to your PostgreSQL server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Last step, **you need to install the internal data tables needed by MIMICWizard** with the script available `here <https://github.com/biomerieux-open-sources/mimicwizard/blob/main/installation/schema.sql>`_

.. code-block:: bash

   psql -d mimiciv -f mimicwizard_internal_init.sql


.. tip:: 

   Windows user will need to install `gzip <https://gnuwin32.sourceforge.net/packages/gzip.htm>`_ and add gzip and postgresql binaries to the PATH environment variable.
   Postgres run command with your windows user as default, you should add the argument `-U postgres` to use the default postgres user.
   If you have any trouble with installation you can refer to the original MIMIC Documentation `Buid MIMIC (from mimic-code) <https://github.com/MIT-LCP/mimic-code/tree/main/mimic-iv/buildmimic/postgres>`_

.. warning::

   The database is quite large and the importation process can take a long time (several hours). 
   Make sure you have enough space on your hard drive and that your computer is plugged in.
   Some command may take a long time to execute and the process may seems blocked, be patient.

C. Configure and start MIMICWizard
***********************************

Now you're database is ready to work with MIMICWizard, configure the correct authentification parameters in the configuration file to make the final link between database and application.

Configuration file
^^^^^^^^^^^^^^^^^^

The configuration file is located at the root of MIMIWizard folder. This file is named ``global.R`` and store all the configuration options.


+----------------------+-------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Configuration option | Possible value                            | Description                                                                                                                                                      |
+======================+===========================================+==================================================================================================================================================================+
| **INTERACTIVE**      | - TRUE                                    | Do you want to activate the application landing page where user can choose if he want to use demo or hosted database. Should be disabled for hosted application. |
|                      | - FALSE                                   |                                                                                                                                                                  |
+----------------------+-------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **IS_ED_LOADED**     | - TRUE                                    | Is MIMICIV ED schema loaded ? This add new information in the patient explorer tab for patient with an emergency admission path                                  |
|                      | - FALSE                                   | Keep this to false f you're using MIMICIV demo                                                                                                                   |
+----------------------+-------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **CACHE_DIR**        | empty string or <path/to/cache/folder>    | Repository where the application cache object are written                                                                                                        |
|                      |                                           | Default "" create a cache folder in the application directory                                                                                                    |
|                      |                                           | Shiny Server should have writing rights in this folder                                                                                                           |
|                      |                                           | Need a closing /                                                                                                                                                 |
|                      |                                           |                                                                                                                                                                  |
+----------------------+-------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **APPLICATION_MODE** | - INIT_DEMO                               | Force application mode, only if interactive is set to FALSE                                                                                                      |
|                      | - DEMO                                    |                                                                                                                                                                  |
|                      | - HOSTED                                  | - INIT_DEMO will regenerate the demo database and override the existing demo database                                                                            |
|                      |                                           | use this mode if you have only one user at the same time and want the database to be clean each time the user start the app.                                     |
|                      |                                           |                                                                                                                                                                  |
|                      |                                           | - DEMO run the application in restricted mode, the application will use the demo database configuration. Some function won't be available.                       |
|                      |                                           |                                                                                                                                                                  |
|                      |                                           | - HOSTED run the application in full mode, the application will use the hosted database configuration.                                                           |
+----------------------+-------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **HOSTED_DBNAME**,   | Your database configuration, if it exists |                                                                                                                                                                  |
| **HOSTED_HOST**,     |                                           |                                                                                                                                                                  |
| **HOSTED_PORT**,     |                                           |                                                                                                                                                                  |
| **HOSTED_USER**,     |                                           |                                                                                                                                                                  |
| **HOSTED_PASSWORD**  |                                           |                                                                                                                                                                  |
+----------------------+-------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **DEMO_DBNAME**,     | The demo database configuration           | If you're using default postgres configuration, you shouldn't have anything to change                                                                            |
| **DEMO_HOST**,       |                                           |                                                                                                                                                                  |
| **DEMO_PORT**,       |                                           |                                                                                                                                                                  |
| **DEMO_USER**,       |                                           |                                                                                                                                                                  |
| **DEMO_PASSWORD**    |                                           |                                                                                                                                                                  |
+----------------------+-------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------+


Once all the packages are downloaded and installed, database is loaded, and configuration file is set-up, **MIMICWizard is ready**. 

**Make sure your database is running**, cd to the app directory and run :

.. code-block:: bash

   R -e "shiny::runApp()"


D. Extend and optimize the application (Optionnal)
**************************************************


Extra SQL index :
^^^^^^^^^^^^^^^^^
SQL index are used to speed up the application by allowing the database to quickly find the data it needs.
You can find additional index to speed up MIMICWizard in a dedicated file `here <https://github.com/biomerieux-open-sources/mimicwizard/blob/main/installation/extra_index.sql>`_.
You can run this script in your database with the command :

.. code-block:: bash

   psql -d mimiciv -f extra_index.sql


Add microbiology supplementary data to the application
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As stated in the original research paper associated with the application, we curated a list of pathogens referenced in the MIMIC-IV database and added their classification in the application.
Theses additional data add information about the type of pathogens (Virus, Bacteria, Fungus or Parasitis) tested for a patient.
It helps to better understand the patient state and the treatment that has been administrated and can be used to stratify or create cohort.

You can find supplementary data in a dedicated file `here <https://github.com/biomerieux-open-sources/mimicwizard/blob/main/installation/additional.sql>` 

Add this supplementary data to your application by running the following command in your database :

.. code-block:: bash

   psql -d mimiciv -f additional.sql


Host the application on your infrastructure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
You can host MIMICWizard using `Posit Shiny Server <https://posit.co/download/shiny-server/>`_ 

They provide a detailed documentation about how to deploy a Shiny Application in their `Administrator Guide <https://docs.posit.co/shiny-server/>`_
The combination of the current page and the Posit documentation should be sufficient to deploy MIMICWizard considering your infrastructure modalities.


.. danger:: MIMICWizard has not been build to be injection-free and without vunerabilities. As a result, **I strongly discourage to distribute this app on a public infrastructure.**
   
   Also, I recommend to give **read-only rights to the database user** you're using in the app **on MIMIC-IV Data**.
   Note that database user should have writing right on public schema as its mandatory for app content to work as intented.