=======================
Development Environment
=======================

These instructions are purposefully opinionated and terse to help you get your development environment up and running as quickly as possible! Please note that familiarity with running commands from the terminal is assumed.

.. contents:: |toctitle|
	:local:

Quick Start
-----------

The quickest way to get the Dataverse Software running is to use Vagrant as described in the :doc:`tools` section, but for day to day development work, we recommended the following setup.

Set Up Dependencies
-------------------

Supported Operating Systems
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Mac OS X or Linux is required because the setup scripts assume the presence of standard Unix utilities.

Windows is not well supported, unfortunately, but Vagrant and Minishift environments are described in the :doc:`windows` section.

Install Java
~~~~~~~~~~~~

The Dataverse Software requires Java 11.

We suggest downloading OpenJDK from https://adoptopenjdk.net

On Linux, you are welcome to use the OpenJDK available from package managers.

Install Netbeans or Maven
~~~~~~~~~~~~~~~~~~~~~~~~~

NetBeans IDE is recommended, and can be downloaded from http://netbeans.org . Developers may use any editor or IDE. We recommend NetBeans because it is free, works cross platform, has good support for Jakarta EE projects, and includes a required build tool, Maven.

Below we describe how to build the Dataverse Software war file with Netbeans but if you prefer to use only Maven, you can find installation instructions in the :doc:`tools` section.

Install Homebrew (Mac Only)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

On Mac, install Homebrew to simplify the steps below: https://brew.sh

Clone the Dataverse Software Git Repo
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Fork https://github.com/IQSS/dataverse and then clone your fork like this:

``git clone git@github.com:[YOUR GITHUB USERNAME]/dataverse.git``

Build the Dataverse Software War File
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you installed Netbeans, follow these steps:

- Launch Netbeans and click "File" and then "Open Project". Navigate to where you put the Dataverse Software code and double-click "Dataverse" to open the project.
- If you see "resolve project problems," go ahead and let Netbeans try to resolve them. This will probably including downloading dependencies, which can take a while.
- Allow Netbeans to install nb-javac (required for Java 8 and below).
- Select "Dataverse" under Projects and click "Run" in the menu and then "Build Project (Dataverse)". Check back for "BUILD SUCCESS" at the end.

If you installed Maven instead of Netbeans, run ``mvn package``. Check for "BUILD SUCCESS" at the end.

NOTE: Do you use a locale different than ``en_US.UTF-8`` on your development machine? Are you in a different timezone
than Harvard (Eastern Time)? You might experience issues while running tests that were written with these settings
in mind. The Maven  ``pom.xml`` tries to handle this for you by setting the locale to ``en_US.UTF-8`` and timezone
``UTC``, but more, not yet discovered building or testing problems might lurk in the shadows.

Install jq
~~~~~~~~~~

On Mac, run this command:

``brew install jq``

On Linux, install ``jq`` from your package manager or download a binary from http://stedolan.github.io/jq/

Install Payara
~~~~~~~~~~~~~~

Payara 5.201 or higher is required.

To install Payara, run the following commands:

``cd /usr/local``

``sudo curl -O -L https://s3-eu-west-1.amazonaws.com/payara.fish/Payara+Downloads/5.2020.6/payara-5.2020.6.zip``

``sudo unzip payara-5.2020.6.zip``

``sudo chown -R $USER /usr/local/payara5``

Install PostgreSQL
~~~~~~~~~~~~~~~~~~

For the past few release cycles much of the development has been done under PostgreSQL 9.6. While that version is known to be very stable, it is nearing its end-of-life (in Nov. 2021). The Dataverse Software has now been tested with versions up to 13 (13.2 is the latest released version as of writing this).  

On Mac, go to https://www.postgresql.org/download/macosx/ and choose "Interactive installer by EDB" option. Note that version 9.6 is used in the command line examples below, but the process will be identical for any version up to 13. When prompted to set a password for the "database superuser (postgres)" just enter "password".

After installation is complete, make a backup of the ``pg_hba.conf`` file like this:

``sudo cp /Library/PostgreSQL/9.6/data/pg_hba.conf /Library/PostgreSQL/9.6/data/pg_hba.conf.orig``

Then edit ``pg_hba.conf`` with an editor such as vi:

``sudo vi /Library/PostgreSQL/9.6/data/pg_hba.conf``

In the "METHOD" column, change all instances of "md5" to "trust". This will make it so PostgreSQL doesn't require a password.

In the Finder, click "Applications" then "PostgreSQL 9.6" and launch the "Reload Configuration" app. Click "OK" after you see "server signaled".

Next, to confirm the edit worked, launch the "pgAdmin" application from the same folder. Under "Browser", expand "Servers" and double click "PostgreSQL 9.6". When you are prompted for a password, leave it blank and click "OK". If you have successfully edited "pg_hba.conf", you can get in without a password.

On Linux, you should just install PostgreSQL using your favorite package manager, such as ``yum``. (Consult the PostgreSQL section of :doc:`/installation/prerequisites` in the main Installation guide for more info and command line examples). Find ``pg_hba.conf`` and set the authentication method to "trust" and restart PostgreSQL.

Install Solr
~~~~~~~~~~~~

`Solr <http://lucene.apache.org/solr/>`_ 8.8.1 is required.

To install Solr, execute the following commands:

``sudo mkdir /usr/local/solr``

``sudo chown $USER /usr/local/solr``

``cd /usr/local/solr``

``curl -O http://archive.apache.org/dist/lucene/solr/8.8.1/solr-8.8.1.tgz``

``tar xvfz solr-8.8.1.tgz``

``cd solr-8.8.1/server/solr``

``cp -r configsets/_default collection1``

``curl -O https://raw.githubusercontent.com/IQSS/dataverse/develop/conf/solr/8.8.1/schema.xml``

``curl -O https://raw.githubusercontent.com/IQSS/dataverse/develop/conf/solr/8.8.1/schema_dv_mdb_fields.xml``

``mv schema*.xml collection1/conf``

``curl -O https://raw.githubusercontent.com/IQSS/dataverse/develop/conf/solr/8.8.1/solrconfig.xml``

``mv solrconfig.xml collection1/conf/solrconfig.xml``

``cd /usr/local/solr/solr-8.8.1``

(Please note that the extra jetty argument below is a security measure to limit connections to Solr to only your computer. For extra security, run a firewall.)

``bin/solr start -j "-Djetty.host=127.0.0.1"``

``bin/solr create_core -c collection1 -d server/solr/collection1/conf``

Run the Dataverse Software Installer Script
-------------------------------------------

Navigate to the directory where you cloned the Dataverse Software git repo change directories to the ``scripts/installer`` directory like this:

``cd scripts/installer``

Create a Python virtual environment, activate it, then install dependencies:

``python3 -m venv venv``

``source venv/bin/activate``

``pip install psycopg2-binary``

The installer will try to connect to the SMTP server you tell it to use. If you don't have a mail server handy you can run ``nc -l 25`` in another terminal and choose "localhost" (the default) to get past this check.

Finally, run the installer (see also :download:`README_python.txt <../../../../scripts/installer/README_python.txt>` if necessary):

``python3 install.py``

Verify the Dataverse Software is Running
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After the script has finished, you should be able to log into your Dataverse installation with the following credentials:

- http://localhost:8080
- username: dataverseAdmin
- password: admin

Configure Your Development Environment for Publishing
-----------------------------------------------------

Run the following command:

``curl http://localhost:8080/api/admin/settings/:DoiProvider -X PUT -d FAKE``

This will disable DOI registration by using a fake (in-code) DOI provider. Please note that this feature is only available in Dataverse Software 4.10+ and that at present, the UI will give no indication that the DOIs thus minted are fake.

Next Steps
----------

If you can log in to the Dataverse installation, great! If not, please see the :doc:`troubleshooting` section. For further assistance, please see "Getting Help" in the :doc:`intro` section.

You're almost ready to start hacking on code. Now that the installer script has you up and running, you need to continue on to the :doc:`tips` section to get set up to deploy code from your IDE or the command line.

----

Previous: :doc:`intro` | Next: :doc:`tips`
