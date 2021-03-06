.. -*- mode: rst -*-
.. vi: set ft=rst sts=4 ts=4 sw=4 et tw=79:

.. _chap_customization:

********************************************
Customization and extension of functionality
********************************************

DataLad provides numerous commands that cover many use cases. However, there
will always be a demand for further customization or extensions of built-in
functionality at a particular site, or for an individual user. DataLad
addresses this need by providing a generic plugin interface.

Using plugins
=============

A number of plugins are shipped with DataLad. This includes plugins which
operate on a particular dataset, but also general functionality that can be
used outside the context of a specific dataset. The following table provides an
overview of plugins included in this DataLad release.

.. currentmodule:: datalad.plugin
.. autosummary::
   :toctree: generated

   add_readme
   export_archive
   no_annex
   wtf

Any plugin can be executed via the :ref:`man_datalad-plugin` command, or the
corresponding Python API function. In the simplest case this could look like this:

.. code-block:: shell

   % datalad plugin wtf

Often plugins will perform a task on a given dataset. The ``plugin`` command
will identify a dataset from the current working directory. Otherwise a dataset
can be given via the ``--dataset`` or ``-d`` options. Moreover, plugins can
take any number of arguments (in the command line those can be given via
``<argument>=<value>`` strings after the plugin name). The following commands
are equivalent and export the content of a dataset into a ZIP archive:

.. code-block:: shell

   # dataset in the current working directory
   /path/to/dataset % datalad plugin export_archive compression=gz archivetype=zip
   # dataset given by an explicit path
   % datalad plugin -d /path/to/dataset export_archive compression=gz archivetype=zip

In addition, DataLad can be configured to run any number of plugins prior or
after particular commands. For example, it is possible to execute a plugin
each time DataLad has created a dataset to configure it so that all files
that are added to its ``code/`` subdirectory will always be managed directly
with Git and not be put into the dataset's annex. In order to achieve this,
adjust your Git configuration in the following way:

.. code-block:: shell

   git config --global --add datalad.create.run-after 'no_annex pattern=code/**'

This will cause DataLad to run the ``no_annex`` plugin to add the given pattern
to the dataset's ``.gitattribute`` file, which in turn instructs git annex to
send any matching files directly to Git. The same functionality is available
for ad-hoc adjustments via the ``--run-after`` option supported by most
commands.

Analog to ``--run-after`` DataLad also supports ``--run-before`` to execute
plugins prior a command.


Plugin detection
================

DataLad will discover plugins at three locations:

1. official plugins that are part of the local DataLad installation

2. system-wide plugins, provided by the local admin

   The location where plugins need to be placed depends on the platform.
   On GNU/Linux systems this will be ``/etc/xdg/datalad/plugins``, whereas
   on Windows it will be ``C:\ProgramData\datalad.org\datalad\plugins``.

   This default location can be overridden by setting the
   ``datalad.locations.system-plugins`` configuration variable in the local or
   global Git configuration.

3. user-supplied plugins, customizable by each user

   Again, the location will depend on the platform.  On GNU/Linux systems this
   will be ``$HOME/.config/datalad/plugins``, whereas on Windows it will be
   ``C:\Users\<username>\AppData\Local\datalad.org\datalad\plugins``.

   This default location can be overridden by setting the
   ``datalad.locations.user-plugins`` configuration variable in the local or
   global Git configuration.

Identically named plugins in latter location replace those in locations
searched before. This can be used to alter the behavior of plugins provided
with DataLad, and enables users to adjust a site-wide configuration.


Writing own plugins
===================

The best way to go about writing your own plugin, is to have a look at the
`source code of those include in DataLad
<https://github.com/datalad/datalad/tree/master/datalad/plugin>`_. Writing
a plugin a rather simple when following the following rules.

Language and location
---------------------

Plugins are written in Python. In order for DataLad to be able to find them,
plugins need to be placed in one of the supported locations described above.
Plugin file names have to have a '.py' extensions and must not start with an
underscore ('_').

Naming and conventions
----------------------

Plugin source files must define a function named::

  dlplugin

This function is executed as the plugin. It can have any number of
arguments (positional, or keyword arguments with defaults), or none at
all. All arguments, except ``dataset`` must expect any value to
be a string.

The plugin function must be self-contained, i.e. all needed imports
of definitions must be done within the body of the function.

Documention
-----------

The doc string of the plugin function is displayed when the plugin
documentation is requested. The first line in a plugin file that starts
with triple double-quotes will be used as the plugin short description
(this will typically be the doc string of the module file). This short
description is displayed as the plugin synopsis in the plugin overview
list.


Expected behavior
-----------------

Plugin functions must yield their results as a Python generator. Results are
DataLad status dictionaries. There are no constraints on the number of results,
or the number and nature of result properties. However, conventions exists and
must be followed for compatibility with the result evaluation and rendering
performed by DataLad.

The following property keys must exist:

"status"
    {'ok', 'notneeded', 'impossible', 'error'}

"action"
    label for the action performed by the plugin. In many cases this
    could be the plugin's name.

The following keys should exists if possible:

"path"
    absolute path to a result on the file system

"type"
    label indicating the nature of a result (e.g. 'file', 'dataset',
    'directory', etc.)

"message"
    string message annotating the result, particularly important for
    non-ok results. This can be a tuple with 'logging'-style string
    expansion.
