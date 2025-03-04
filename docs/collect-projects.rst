.. _collect-projects:

Managing Projects in Collect
============================

.. versionadded:: v2021.2

Starting with Collect v2021.2, Collect can be set up with multiple Projects. Each Project in Collect has its own set of :doc:`Forms <collect-forms>` and :doc:`Settings <collect-settings>`.

Some examples of scenarios where having more than one project might be useful:

- An enumerator working on different data collection efforts across different servers
- Multiple enumerators using one device for the same data collection project but with different user accounts on the server
- Form designers testing forms on different staging/development servers

Each Project is identified by a name, icon and color. These are shown at the top of the Main Menu:

.. image:: /img/collect-projects/main-menu-with-project.png
  :alt: Collect's Main Menu with an example Project
  :class: device-screen-vertical

Tapping the Project icon allows you to add, manage, and switch Projects:

.. image:: /img/collect-projects/project-settings-dialog.png
  :alt: Collect's Project settings dialog
  :class: device-screen-vertical

.. _collect-add-project:

Adding a new Project
~~~~~~~~~~~~~~~~~~~~

#. Tap the :ref:`Project <collect-projects>` icon on the upper right of the Main Menu screen
#. Tap :guilabel:`Add project`
#. Scan a :doc:`QR code <collect-import-export>` or tap :guilabel:`Manually add project details` to set the server for the new project

.. warning::
  If a Project is added with the same URL as an existing one, Collect will warn the user before proceeding. This is to prevent users from accidentally adding the same Project multiple times.

.. _collect-switch-project:

Switching between Projects
~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Tap the :ref:`Project <collect-projects>` icon on the upper right of the Main Menu screen
#. Select the Project you want to switch to

.. _collect-delete-project:

Deleting a Project
~~~~~~~~~~~~~~~~~~

#. Tap the :ref:`Project <collect-projects>` icon on the upper right of the Main Menu screen
#. Select the Project you want to delete if it is not already the current Project
#. Tap :guilabel:`Settings` and then :guilabel:`Project Management`
#. Tap :guilabel:`Delete` and confirm that you want to delete the Project
