.. include:: ../../global.rst

################
Pattern Matching
################

Pattern matching brings together the POSIX :manpage:`regex(3)` with a
form of :ref:`cond <ref:cond special form>` clauses.

Broadly, an expression is evaluated and expected to return a string.
That string is then used to match against with a series of
pattern/regular expressions.

If the match is successful then the result of the pattern matching
expression is the result of evaluating the :samp:`{expr}` part of the
clause as though it were an implicit ``=>`` clause with ``r`` as the
supplied parameter.  Something like:

.. code-block:: idio

   function (r) {
     expr
   }

:var:`r` will be the result of :ref:`regexec <ref:regexec>` and
therefore an array of at least one element which is the entire matched
string.  If the pattern match contained any parenthesised
sub-expressions they will be subsequent elements of the array.

.. note::

   The pattern matching expressions stash any compiled literal string
   regular expressions in a global table.  This means that in loops
   the regular expression doesn't need to be recompiled.  It also
   means the compiled regular expressions are not reaped until
   :lname:`Idio` exits.

.. note::

   These examples put parenthesis around the whole pattern matching
   expression.  That **doesn't** change the way it is interpreted, any
   expression with multiple elements is treated as a list by the
   reader.

   Here, it probably helps your ``$EDITOR`` indent the text properly
   with clauses on separate lines!

regex-case
==========

The general form is :samp:`regex-case e {[clauses]}` with each clause
being :samp:`("{regex}" {expr})`.

Suppose we want to match a common :samp:`{var}={value}` assignment:

.. code-block:: idio

   (regex-case (read-line)
     ("^([:alpha:][[:alnum:]_]*)=(.*)" {
       printf "%s is '%s'\n" r.1 r.2
     }))

Here we use :ref:`read-line <ref:read-line>` to read a line of input
from the current input handle.  If that string matched the POSIX regex
``"^([:alpha:][[:alnum:]_]*)=(.*)"`` then the :ref:`regexec
<ref:regexec>` result, an array, will be passed as the parameter ``r``
to the given expression.

That expression prints out the two regex sub-expressions, the
:samp:`{var}` and :samp:`{value}`.

pattern-case
============

``pattern-case`` is a derivative of `regex-case`_ where the "pattern
matches" have ``*`` and ``?`` replaced with ``.*`` and ``.`` and the
code continues like ``regex-case``.

Suppose we want an unreliable method to determine if this is a
BSD-style operating system:

.. code-block:: idio

   (pattern-case (collect-output uname -s)
     ("*BSD" {
       printf "%s is a BSD\n" r.0
     }))

Here we collect the output from the command ``uname -s`` and compare
it to the pattern ``"*BSD"``.  Operating systems like FreeBSD and
OpenBSD will match.

.. include:: ../../commit.rst
