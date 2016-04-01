.. _Replacing CSMH:

########################################################
Replacing the ``courseware_studentmodulehistory`` Table
########################################################

This section describes a change to the ``courseware_studentmodulehistory``
database table. This change requires a new database configuration in all Open
edX instances, and offers an optional data migration. This change will be
released to the edx-platform repository on TBD, and will affect all Open edX
installations that follow master on that date.

.. important:: If you are responsible for maintaining an Open edX instance,
 including a devstack, fullstack, or production installation, you must prepare
 for this change before you upgrade to the TBD version of master.

This section presents an overview of the change followed by options for
completing the required and optional procedures. EdX recommends that you review
all of these options before you choose an option for your Open edX
installation.

.. toctree::
   :maxdepth: 2

   CSMHE_overview
   migration_options
   migration_procedures

No changes are required or supported at this time for Open edX installations
that use the **Dogwood** release. For those installations, the changes
described in this section will be a part of the upgrade to the next Open edX
release, Eucalyptus.

.. include:: ../../../links/links.rst
