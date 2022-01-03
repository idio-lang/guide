.. include:: ./global.rst

**********
Idio Shell
**********

This is where :lname:`Idio` stops behaving like a programming language
and starts behaving like a shell.

We already have the fundamental behaviour, an *unbound* symbol
evaluates to itself.  That's going to occur for any of the elements of
:samp:`{cmd} {args}` where we are particularly interested in
:samp:`{cmd}`.

We don't get everything our own way, though, as there is a bit of a
semantic discrepancy between a programming language and a shell.  The
programming language is going to win so we need to bring our own hoop
to jump through for a corner case of the shell aspect to work.

If you have a symbol *on its own*, it looks much like, say, a value
being returned from a block.  It's perfectly reasonable to return a
symbol from a expression so that's what happens.

Of course, as a shell, we want to execute the external program called
``ls`` (or ``env`` or ``make`` or ...) for which we are forced to wrap
the single symbol in parentheses.

If you pass any arguments or use an operator the right thing happens
but a single element needs to be wrapped:

.. code-block:: idio

   make			; gets the symbol make
   (make)		; runs the command /usr/bin/make (or wherever)
   make -k		; runs the command /usr/bin/make -k
   make > "/dev/null"	; runs the command /usr/bin/make

Annoying.

Command Execution
=================

If the VM sees a symbol (or a string) in functional position,
ie. :samp:`{cmd}` then it will try to find and execute the external
command.

cmd
---

If :samp:`{cmd}` contains a ``/`` then it is assumed to be the
executable path as is, otherwise :samp:`{cmd}` is searched for on
:envvar:`PATH`.

args
----

Arguments are expanded into strings for :manpage:`execve(2)` fairly
simply.  Most :lname:`Idio` types do not have a sensible string
representation (think: functions, hash tables and arrays etc.) and it
is an error to try to use them as external command arguments.

Those that do:

* numbers (fixnums, bignums and C/ types) are printed

* strings in their UTF-8 form

* unicode in their UTF-8 form

* symbols are checked for the globbing meta-characters ``*``, ``?``
  and ``[``.

  If they do contain such characters then the results of
  :manpage:`glob(3)` are inserted into :samp:`{args}` otherwise the
  UTF-8 representation of the symbol is used.

* a pair is checked to see if it is a list of strings (nominally the
  result of calling :ref:`glob <ref:glob>`) and if so is inserted into
  :samp:`{args}` otherwise is an error

The glob-expansion (or list of strings) are the only elements that
will increase the number of arguments to the external command.

Variables
^^^^^^^^^

There is a downside for argument expansion where a casually passed
argument happens to be an :lname:`Idio` variable which could produce
an unexpected error.

If ``sort`` was a :program:`make` target then:

.. code-block:: idio

   make sort

will likely produce an ``^rt-command-argv-type-error`` because
``sort`` was evaluated as the :lname:`Idio` function :ref:`sort
<ref:sort>` and a closure can't be converted to a string.

Quoting any potential clashes avoids the evaluation:

.. code-block:: idio

   make 'sort

Return Value
------------

External commands will return ``#t`` or ``#f`` depending on whether
they exited with 0 or not.  Well, they will initiate a return of
``#f`` if they fail.

You can use this in logical expressions (:ref:`and <ref:and special
form>`, :ref:`or <ref:or special form>`, :ref:`not <ref:not special
form>`) or conditional expressions (:ref:`if <ref:if special form>`,
:ref:`cond <ref:cond special form>`) like in the shell.

.. code-block:: idio-console

   Idio> (true) or (false)
   #t
   Idio> (true) and (false)
   #f
   job  33041: (false): completed: (exit 1)
   Idio> if (true) (printf "Y\n") (printf "N\n")
   Y
   #<unspec>
   Idio> if (false) (printf "Y\n") (printf "N\n")
   N
   #<unspec>
   job  33043: (false): completed: (exit 1)

.. note::

   We get the *job notification* messages because this session is
   interactive.  A script would not report job notifications.

Errors
======

It's worth repeating that :lname:`Idio` is reluctant to let any
external command errors go unnoticed.  As part of the flow of the
script, if something failed, especially an external command,
:lname:`Idio` should stop.

.. note::

   :lname:`Idio` is far more lenient in *interactive* sessions as it
   defaults to suppressing *pipefail* and *exit-on-error* and allows
   expressions to be aborted back to the top-level.

By default, any external command error, including those in pipelines
will result in a fatal error.

An ``^rt-command-status-error`` will be raised:

* if any command in a pipeline is killed or exits non-zero and the
  dynamic variable ``suppress-pipefail!`` is `false`

  .. code-block:: idio

     false | true | true

  will be a fatal error

  .. code-block:: idio

     suppress-pipefail! = #t
     false | true | true

  will succeed

  .. note::

     The equivalent of ``suppress-pipefail! = #t`` is the default for
     most shells and an interactive :lname:`Idio` session.

     ``suppress-pipefail! = #f`` is the default for :lname:`Idio`
     scripts.

* if a standalone command or the rightmost command in a pipeline is
  killed or exits non-zero

If you handle ``^rt-command-status-error`` conditions then you make
the decisions.

However, if your code eventually invokes the ``default-rcse-handler``
then it consults the dynamic variable ``suppress-exit-on-error!``.  If
that is `false`, the default, :lname:`Idio` will exit or kill itself
to give the same status to its parent.  Hence, both:

.. code-block:: idio

   true | true | false
   (false)

will be fatal, whereas:

.. code-block:: idio

   suppress-exit-on-error! = #t
   true | true | false
   (false)

will both succeed.

.. note::

   ``suppress-exit-on-error! = #t`` is equivalent to the shell's ``set
   -e``.

Async Commands
--------------

There is a corner case for *asynchronous* commands, those that are
part of *Process Substitution*, where they are not part of the flow,
they are adjunct to it.

Here, the default is to be *told* that the command failed (but not
exit because of it).

An ``^rt-async-command-status-error`` will be raised under the same
broad conditions as for normal standalone commands or pipelines.

You can handle the condition yourself (if you want to force an exit)
or change the ``suppress-async-command-report!`` dynamic variable to a
non-`false` value if you don't want to be told about it.

Operators
=========

Much of the heavy lifting of the shell work is done with *infix*
operators:

* ``<``, ``>`` etc. for I/O redirection

* ``|`` for pipelines

In fact, these operators are written in :lname:`Idio` itself
demonstrating the ability to extend the language's functionality.

I/O Operators
-------------

The I/O operators, in particular, aren't as flexible as a normal shell
largely because, as a programming language, :lname:`Idio` can carry
file descriptors around, or :ref:`libc/dup2 <ref:libc/dup2>` them or
whatever, which a shell can't.  So you should be doing that, not
trampling over file descriptor 3.

The I/O operators are slightly less flexible in terms of positioning,
they can only be placed *after* the command to be run, partly because
they are *infix* operators:

.. code-block:: idio

   ls -l > "foo"
   cat < "foo"

as, essentially, the infix I/O operation, here, ``>`` and ``<``, is
taken to be separating the command from its source/target expression.

The I/O operators only handle redirection of *stdin*, *stdout* and
*stderr*.  To what, though, is a bit more interesting.

By and large, you would be using *handles* in :lname:`Idio` where
:lname:`Idio` supports the notion of current *input*, *output* and
*error* handles.

Those current handles aren't restricted to file or pipe handles, they
can be string handles too.

Which, by extension, means you can do I/O redirection of an external
command to or from string handles, for example:

.. code-block:: idio

   osh := (open-output-string)
   cat "foo" > osh
   str := get-output-string osh

Of course, this sort of indirection to get the contents of a file as a
string is *Command Substitution*, in :lname:`Bash` parlance, for which
there is:

.. code-block:: idio

   str := collect-output cat "foo"

which works just as well for a pipeline:

.. code-block:: idio

   str := collect-output zcat "foo.tgz" | tar tf \-

``collect-output`` is one of the :ref:`job meta-commands <job
meta-commands>` and, like the others, could do with a syntactic short
cut.

Syntax
^^^^^^

The general form is :samp:`... {op} {expr}` where the options for
:samp:`{expr}` vary by :samp:`{op}`:

* ``<``, ``>``, ``>>``, ``2>``, ``2>>``

  :samp:`{expr}` can be:

  * an (an appropriately directioned) FD handle (file or pipe)

  * an (an appropriately directioned) string handle

  * a string indicating a file name which will be opened in the
    appropriate direction

  * ``#n`` meaning :file:`/dev/null` will be opened in the appropriate
    direction

  with the ``>>`` variants meaning append

* ``<&``, ``>&``, ``2>&``

  :samp:`{expr}` can be:

  * an (an appropriately directioned) FD handle (file or pipe)

  * an (an appropriately directioned) string handle

  * a ``fixnum`` or ``C/int`` integer:

    * 0 indicating the current input handle

    * 1 indicating the current output handle

    * 2 indicating the current error handle

Pipelines
---------

Pipelines work as you would expect.  The pipeline operator, ``|``, has
a higher priority than the I/O redirection operators meaning that, in
effect, I/O redirection is more closely tied to the command:

.. code-block:: idio

   ls > #n | wc

will mean :program:`wc` will see no input.

.. _`job meta-commands`:

Meta-Commands
^^^^^^^^^^^^^

There are several :ref:`job meta-commands <ref:job-control/job
meta-commands>` which affect the overall pipeline rather than any
individual pipe within the pipeline.  They are the first word(s) in a
pipeline:

* ``collect-output`` which collects the output of the command and
  returns it as a string

  .. code-block:: idio

     hn := collect-output uname -n

  :var:`hn` will be a string from the output of ``uname -n``

  .. note::

     ``collect-output`` will :ref:`strip-string <ref:strip-string>`
     the output from the command of trailing newlines.

* ``fg-job`` is the default and runs the job in the foreground

* ``bg-job`` runs the job in the background

  .. code-block:: idio

     bg-job sleep 60

  ``bg-job`` returns ``#t`` (backgrounding a job is always successful)

* ``pipe-into`` establishes a pipe as the input for the job and
  returns an output pipe handle

  You can write into the pipe handle to generate input for the job.

  This is similar to :lname:`Perl`'s ``open ("| ...")``.

  .. code-block:: idio

     oph := pipe-into sed -e "s/^/boo! /"
     hprintf oph "%s\n" HOSTNAME
     close-handle oph

  for which something like the following will be printed

  .. parsed-literal::

     boo! *hostname*

* ``named-pipe-into`` established a named pipe as the input for the
  job and the pipe's name is returned.

  The caller is expected to open the returned pipe name for writing.

  On many systems the pipe's name will be :file:`/dev/fd/{n}` but on
  some systems it will be a FIFO in the file system.

  This is the equivalent of :lname:`Bash`'s *Process Substitution*
  form ``>(...)``.

* ``pipe-from`` establishes a pipe as the output for the job and
  returns an input pipe handle

  You can read from the pipe handle to get the job's output.

  This is similar to :lname:`Perl`'s ``open ("... |")``.

* ``named-pipe-from`` established a named pipe as the output for the
  job and the pipe's name is returned.

  The caller is expected to open the returned pipe name for reading.

  On many systems the pipe's name will be :file:`/dev/fd/{n}` but on
  some systems it will be a FIFO in the file system.

  This is the equivalent of :lname:`Bash`'s *Process Substitution*
  form ``<(...)``.

* ``time`` flags that a report on the accumulated resources of the job
  should be produced when the job completes

  .. code-block:: idio-console

     Idio> time sleep 2
     Real 2.011
     User 0.001
     Syst 0.001
     #t


Both of the ``named-pipe-*`` variants are asynchronous commands.

Not all of the meta-commands are compatible.

.. include:: ./commit.rst
