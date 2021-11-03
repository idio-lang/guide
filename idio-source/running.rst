.. include:: ./global.rst

************
Running Idio
************

:lname:`Idio` is started like many scripting languages with:

.. parsed-literal::

   .../idio [*Idio-args*] [*script-name* [*script-args*]]

where arguments to :lname:`Idio` must come before any script name or
arguments to the script as, both in essence and in practice, the first
argument that isn't recognised as an :lname:`Idio` argument will be
treated as the name of a script.  ``--hlep`` beware!

The *script* is searched for on :envvar:`IDIOLIB`, a regular Unix
``:``-separated list of directories.  The use of :envvar:`IDIOLIB` is
probably more complicated than it should be, see below.

Idio Arguments
==============

:lname:`Idio` accepts a few arguments with ``--help`` giving an up to
date list.

For *users*, the useful arguments are:

.. option:: --load <NAME>

   evaluate :samp:`load {NAME}` and continue processing arguments

.. option:: --version

   print the version number and quit

.. option:: --help

   print the argument help information and quit

For *developers* (of :lname:`Idio`), the useful arguments are:

.. option:: --debugger

   enable the debugger (if the session is interactive)

   (The debugger is barely useful, even for developers just to get
   machine state.)

.. option:: --vm-reports

   enable some VM-oriented reporting on a clean shutdown

   The reports are created in the current working directory and
   include:

   * :file:`vm-constants` which is the VM's *constants* table referred
     to by various :var:`ci` (constants index) entities

   * :file:`vm-values` which is the VM's *values* table referred to by
     various :var:`vi` (values index) entities

   * :file:`vm-dasm` which is the augmented disassembly of the byte
     code

   * :file:`vm-modules` which is a long list of top-level names per
     module including their constants index (:file:`vm-constants`),
     value index (:file:`vm-values`) and whether the name was exported

   If you were running a "debug" build you will also get

   * :file:`vm-perf.log` which contains a mix of:

     * Garbage Collections stats by type

     * per function usage stats (when freed or at shutdown)

     * instruction throughput rates 

     * memory allocation stats (on shutdown)

   .. warning::

      Debug builds on some systems become chronically slowed as
      calling various time and resource usage interfaces overburdens
      them.

IDIOLIB
=======

Whatever value of :envvar:`IDIOLIB` is supplied in the environment,
two possible prefixes may occur:

#. if the actual :program:`idio` executable looks like
   :file:`.../bin/{name}` then :envvar:`IDIOLIB` will be prefixed with
   :file:`.../lib/idio/{M.N}` (where :file:`{M.N}` are the major and
   minor version numbers of :lname:`Idio`)

   This makes the executable "position independent" (assuming you copy
   the libraries along with it).

#. if the :program:`idio` command run is a symlink and looks like
   :file:`.../bin/{name}` then :envvar:`IDIOLIB` will be prefixed with
   :file:`.../lib/idio/{M.N}` (where :file:`{M.N}` are the major and
   minor version numbers of :lname:`Idio`)

   This allows users to have *virtualenv*-style working areas separate
   from the main distribution.

   .. note::

      There is no other support for virtualenvs at this time.

.. attention::

   :lname:`Idio` allows for both relative directories and empty
   elements (meaning :envvar:`PWD`) in :envvar:`IDIOLIB`.

   At some point these will be either elided or normalized to absolute
   directories (based on the initial :envvar:`PWD`) to avoid to
   traditional problem of running unexpected code.

.. include:: ./commit.rst
