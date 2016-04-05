.. _CSMHE Procedures:

############################################################
Procedures for Replacing ``courseware_studentmodulehistory``
############################################################

This topic provides procedures for updating to the new database and table
configuration required by the ``courseware_studentmodulehistory`` change. It
also includes the optional procedure for migrating all data from
``courseware_studentmodulehistory`` to
``coursewarehistoryextended_studentmodulehistoryextended``.

.. contents::
   :local:
   :depth: 1

Before you follow these procedures for your Open edX instance, be sure to
review the different :ref:`options for updating
``courseware_studentmodulehistory``<Options for Updating Your Open edX
Instances>`.

***************************************
Configure the New Database and Table
***************************************

.. note:: This procedure is required for all fullstack and production
  instances.

#. Create a mysql database. For the edx.org and edX Edge instances, edX named
   this database ``edxapp_csmh``.  You will need to modify this example for your
   database users and naming schemes.

   .. code-block:: sql

     mysql> create database edxapp_csmh DEFAULT CHARACTER SET utf8;
     mysql> grant SELECT,INSERT,UPDATE,DELETE on edxapp_csmh.* to 'edxapp001@hosts'
     mysql> grant SELECT,INSERT,UPDATE,DELETE,ALTER,CREATE,DROP,INDEX on edxapp_csmh.* to 'migrate@hosts'

   Alternatively, you can use the `create_db_and_users.yml playbook`_
   to create this databse and user automatically.


#. If you are using the `edxapp.yml playbook`_
   to install your edxapp instances, then master after TBD will properly populate DATABASES
   for you by setting ``edxapp_databases``.  This code expects ``EDXAPP_MYSQL_CSMH_DB_NAME``,
   ``EDXAPP_MYSQL_CSMH_USER``, ``EDXAPP_MYSQL_CSMH_PASSWORD``, ``EDXAPP_MYSQL_CSMH_HOST``,
   ``EDXAPP_MYSQL_CSMH_PORT`` to be populated in the same way the ``EDXAPP_MYSQL_...``
   variables are populated in your ansible overrides.

   If you have another method of updating ``lms.auth.json``, you will want to add
   a clause similar to the following to the DATABASES section.

   .. code-block:: bash

     "student_module_history": {
            "ENGINE": "django.db.backends.mysql",
            "HOST": "localhost",
            "NAME": "edxapp_csmh",
            "PASSWORD": "password",
            "PORT": "3306",
            "USER": "edxapp001"
        },


#. Edit the ``lms.env.json`` file to set the ``ENABLE_CSMH_EXTENDED`` feature
   flag.

   .. code-block:: bash

    ``"ENABLE_CSMH_EXTENDED": true``

#. Run Django migrations to create the new
   ``coursewarehistoryextended_studentmodulehistoryextended`` table.  There
   are two scripts, ``/edx/bin/edxapp-migrate-lms`` and ``/edx/bin/edxapp-migrate-cms``
   which are what the ``edxapp.yml`` play uses to run migrations.  If you wish to run
   migrations by hand, the commands you want to run are on the last few lines of those
   scripts.  In particular, you need to run lms and cms migrations, and against both
   the default and student_module_history database.  These scripts hide that complexity.

When you bring your servers back online with this configuration, the system
only writes records for interactions with problems to
``coursewarehistoryextended_studentmodulehistoryextended``.

Optionally, you can now migrate all data from
``courseware_studentmodulehistory`` to
``coursewarehistoryextended_studentmodulehistoryextended``.

*************************************************************
Migrate Data to ``coursewarehistoryextended_studentmodulehistoryextended``
*************************************************************

.. note:: This procedure only applies large production instances which need
   the operational benefits listed in :ref:`Why Is A New Database Needed`.

#. Follow the deployment setps above so that you are running a deploy of Open edX which is
   writing only to ``coursewarehistoryextended_studentmodulehistoryextended``.

#. Make use of the `migration scripts`_ we have written.

    #. ``migrate-separate-database-instances.sh`` assumes you have split your databases
       into separate database servers.  We did this by creating a read replica and then severing it from
       production.  This ensures you have a mostly up to date ``courseware_studentmodulehistory`` which
       can be copied into ``coursewarehistoryextended_studentmodulehistoryextended``.  If you go this route,
       you will want to do a final mysqldump from the first database server to the second database server.
       After this is done and ``courseware_studentmodulehistory`` is caught up, you can run
       ``migrate-seterate-database-instances.sh`` to slowly copy data.  Be sure to monitor your progress
       to ensure that you don't copy too quickly and cause disk contention or other performance issues
       on this new database instance.

       .. code-block:: bash

          mysqldump --skip-add-drop-table --no-create-info -u migrate -p -h dbhost db courseware_studentmodulehistory --where='id > LAST_ID' --result-file=catchup.sql
          mysql -u migrate -p -h newdbhost db2 < catchup.sql

    #. ``migrate-same-database-instance.sh`` assumes you have created a new database in the same
       database server.  This is simpler than setting up a separate database server, but gains a
       separate set of operational benefits.

    #. If you need to restart either migration, you can use the following command to
       find the largest ID value that was successfully inserted into the new table.
       You can then rerun with MINID set to the result of the following query.

       .. code-block:: bash

         select max(id) from wwc.courseware_studentmodulehistory where id < MAXID

#. Edit the ``lms.env.json`` file to set the
   ``ENABLE_READING_FROM_MULTIPLE_HISTORY_TABLES`` feature flag.

   .. code-block:: bash

    "ENABLE_READING_FROM_MULTIPLE_HISTORY_TABLES": false

When you bring your servers back online with this configuration, the system
only writes to and queries from
``coursewarehistoryextended_studentmodulehistoryextended``.

#. Truncate your ``courseware_studentmodulehistory`` tables.  If you have a very large
   table, you may need to use our ``slow-delete.sh`` script, or one of the many other
   techniques for slowly draining a mysql table.  ``TRUNCATE TABLE courseware_studentmodulehistory``
   is the preferred technique, but can cause a lot of disk activity.

.. _migration scripts: https://github.com/edx/configuration/blob/master/util/csmh-extended
.. _edxapp playbook: https://github.com/edx/configuration/blob/master/playbooks/edx-east/edxapp.yml
.. _create_db_and_users.yml playbook:   https://github.com/edx/configuration/blob/master/playbooks/edx-east/create_db_and_users.yml
