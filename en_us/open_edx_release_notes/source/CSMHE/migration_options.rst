.. _Options for Updating Your Open edX Instances:

##########################################################
Options for Updating ``courseware_studentmodulehistory``
##########################################################

This topic outlines the options for updating to the new database and
table configuration required by the ``courseware_studentmodulehistory`` change.

.. contents::
   :local:
   :depth: 1

Because each Open edX installation has a unique set of constraints
and requirements, edX recommends that you review all of these options before
selecting one for your instance or instances.

*******************************
Keep, and Query, Both Tables
*******************************

This option is suitable for many instances with small databases, such as a
fullstack or small production instance, that do not have the performance
considerations or other operational needs described in the :ref:`overview<CSMH
Overview>`.

For instances with relatively small databases, you set up the new database and
table and then configure the system to read from both tables, without migrating
data. An outline of the steps you need to complete follows.

#. Create the ``edxapp_csmh`` database.

#. Update ``lms.auth.json`` with a new entry in the DATABASES section.

#. Update ``lms.env.json`` to set ``"ENABLE_CSMH_EXTENDED": true``. Leave
   ``"ENABLE_READING_FROM_MULTIPLE_HISTORY_TABLES": true``.

#. Run django migrations to generate the new table.

A system with this configuration writes only to the new
``coursewarehistoryextended_studentmodulehistoryextended`` table, but it reads
from both that table and the ``courseware_studentmodulehistory`` table in the
``edxapp`` database.

For more information, see :ref:`CSMHE Procedures`.

***********************
Reinstall Your Devstack
***********************

.. note:: This option is suitable only for devstacks.

To set up devstack with the new database and SQL table, you can reprovision.
If you choose this option for updating devstack, no further configuration
or migration procedures are required.

   .. code-block:: bash

     cd devstack
     vagrant provision

Reprovisioning adds the ``edxapp_csmh`` database and its
``coursewarehistoryextended_studentmodulehistoryextended`` table. The
``lms.env.json`` feature flags that control use of the
``coursewarehistoryextended_studentmodulehistoryextended`` table have the
following settings.

   .. code-block:: bash

     "ENABLE_CSMH_EXTENDED": true
     "ENABLE_READING_FROM_MULTIPLE_HISTORY_TABLES": true

A system with this configuration writes to the new
``coursewarehistoryextended_studentmodulehistoryextended`` table only, but
queries both tables.

.. note:: Only set ``ENABLE_READING_FROM_MULTIPLE_HISTORY_TABLES`` to
  ``true``  for an existing site that does not choose the data migration
  option. For new installations, this setting should be ``false``.

.. _Migrate All Data to One Table:

******************************
Migrate All Data to One Table
******************************

This option is suitable for installations that have a large number of records
in the ``coursewarehistoryextended_studentmodulehistoryextended`` table, such
as large production instances.

If you select this option, you set up the new database and table and then
migrate all existing data to the new table. When the process is complete, the
system uses only the new table. This is the procedure that edX followed for
edx.org and edX Edge.

For more information, see :ref:`Why Is A New Database Needed`.

An outline of the steps you complete follows.

#. Create the ``edxapp_csmh`` database.

#. Update ``lms.auth.json`` with a new entry in the DATABASES section.

   If you use the edxapp ansible role to update ``lms.auth.json``, the system
   automatically merges an update to the ``edxapp_databases`` dictionary in
   `edxapp/defaults/main.yml`_.

#. Update ``lms.env.json`` to set ``"ENABLE_CSMH_EXTENDED": true``.

#. Run migrations to create the new database table.

#. Deploy so that all new data is being written to the new
   ``coursewarehistoryextended_studentmodulehistoryextended`` table.

#. Migrate all data from ``courseware_studentmodulehistory`` to
   ``coursewarehistoryextended_studentmodulehistoryextended``.

#. Update ``lms.env.json`` to set
   ``"ENABLE_READING_FROM_MULTIPLE_HISTORY_TABLES": false``.

#. Truncate ``courseware_studentmodulehistory``.

As soon as you deploy a system with ``ENABLE_CSMH_EXTENDED`` enabled, the
system writes only to the
``coursewarehistoryextended_studentmodulehistoryextended`` table, but it reads
from both that table and the ``courseware_studentmodulehistory`` table. To
reduce the overhead of querying two tables in two databases, you migrate data
and then set ``"ENABLE_READING_FROM_MULTIPLE_HISTORY_TABLES": false``.

For more information, see :ref:`CSMHE Procedures`.

.. include:: ../../../links/links.rst

.. _edxapp/defaults/main.yml: https://github.com/edx/configuration/blob/master/playbooks/roles/edxapp/defaults/main.yml#L635
