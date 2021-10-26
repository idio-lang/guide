.. include:: ./global.rst

****************
Evaluation Model
****************

If you've used :lname:`Scheme` then this will feel familiar.

.. _`reader`:

Reader
======

In the first instance, the *reader* reads in the UTF-8 source code.

The reader will read **one** expression per line although the line can
be stretched across multiple lines if it hasn't read in a matching
brace, parenthesis, bracket or double-quote.

You cannot have multiple expressions on one line -- they will be
evaluated as though they were the elements of an expression that *was*
a complete line!

The Abstract Syntax Tree the reader is going to generate will be a
single element or a list of elements.  Simply having multiple *words*
in an expression will generate a list.  An actual list in the source
generates the same:

.. code-block:: idio

   ls		; single element
   ls -l	; list of two elements
   (ls -l)	; same list of two elements

Such a list is commonly called a *form*.

The "actual list" format is useful when the expression is naturally
multiple lines and it is easier to convince your editor to indent
lines properly.  So, if the text used here was more complex you could
write:

.. code-block:: idio

   (if thing
       this
       that)

which will keep Schemers happy or you could use classic
line-continuation characters:

.. code-block:: idio

   if thing \
       this \
       that

which won't keep anyone *happy* but at least works.

Value Constructors
------------------

Some words look like the usual numbers and strings and so on and the
reader will construct a value (number, string, etc.) and insert that
into the list in place of the nominal word.

At this point, if you asked the reader to print the value back out, in
many cases it will be indistinguishable from the text of the original
source code but we'll sometimes get a "normal" form (the following
isn't the reader printing the values out but this just exemplifies the
potential change in printed form even if nothing has happened):

.. code-block:: idio-console

   Idio> 37
   37
   Idio> 123e4
   1.23e+6
   Idio> "foo"
   "foo"
   Idio> "foo
   bar"
   "foo\nbar"

Other words the reader doesn't recognize as constructors will be left
as words.  The reader is not the evaluator.

Reader Operators
----------------

The reader has been primed with some operators, infix and postfix,
which are allowed to re-arrange a form *as they see fit* albeit that
most really just re-arrange the words.  A *syntax transformation*
(although not a Syntax Transformer *per se* to any Schemers reading).

The canonical example are the binary arithmetic operators which
transform the likes of :samp:`{a} + {b}` into :samp:`+ {a} {b}`.

Technically, they transform it into :samp:`binary-+ {a} {b}` where
there is a subtle difference between the function ``+`` (which takes
any number of arguments -- including zero!) and the function
``binary-+`` which expects exactly two arguments.

That difference isn't important right now, more that the arrangement
of words in the form changes before it is passed to the evaluator.

You can prevent the reader implementing reader operators by escaping
the operator with ``\``:

.. code-block:: idio-console

   Idio> apply \+ 1 2 3
   6

which would otherwise complain that ``apply`` is not a number.

User-extensible
^^^^^^^^^^^^^^^

You can write your own reader operators, of course.  Most of the
standard operators (arithmetic, Job Control etc.) are "user-defined".

Evaluator
=========

The *evaluator* takes the AST from the reader and tries to find some
meaning in the expression.

This can get a little involved so quickly skipping to the end where
the result of the evaluator is some *intermediate code* which is a
form of high level assembly code for the *code generator*.  The code
generator, in turn, looks to translate that intermediate code into the
stack-oriented *byte code* of the VM.

Neither the code generator nor VM are suitable topics here so let's go
back to how we get some intermediate code.

The evaluator is only going to get one of two things: a single element
or a list of elements.

Single Element
--------------

For a single element -- and remember this because it'll happen a lot
when we recurse on arguments in a bit -- we try to see if this is:

* already a value (constructed by the reader) in which case we pass it
  on

  Technically it is stashed in a table of constants and an index to
  the table is passed on but, hey.

* is a word, ie. a variable, in which case we try to resolve it as a
  local name or a name from the wider context, ie. a global

  There'll be a difference in how the value is referenced in the byte
  code based on whether it was local or global but the point here
  being that it was the name of a variable.

* can't be looked up in which case we return the word as itself, the
  symbol

  This, in particular, breaks a :lname:`Scheme`-like evaluation model
  but is essential for a shell-like model to be able to execute
  (essentially) random "names" from the :envvar:`PATH`.

A List of Elements
------------------

This is deemed to be a function call of the form :samp:`{cmd} {args}`
and so we can look at :samp:`{cmd}` to figure out how to move
forwards:

* it is a *special form* like ``if`` or ``set!``

  These are the fundamental behavioural concepts and work a little
  differently to regular function calls.

  For special forms, we don't evaluate the arguments but pass them
  verbatim for the behavioural code of the special form to figure out.
  The canonical example is:

  .. code-block:: idio

     if #t (do-it) (sleep-forever)

  Thinking ahead to the eventual byte code we want generated, we don't
  want to evaluate either of the expressions ``(do-it)`` or
  ``(sleep-forever)`` until we've figured out whether or not the
  *condition expression*, ``#t``, was `true` or `false`.

  .. aside::

     Maybe that's a lookup table these days?  Still, it'll involve
     *doing* something.

  Here, the condition expression is trivial but it could be
  determining if the billionth digit of `pi` is odd or not.

  Of course, with our ability to see through the lines of a cheap
  example, the *alternate expression*, ``(sleep-forever)`` should
  *never* get called because the condition expression is *always*
  ``#t``, aka. `true` -- plus it sounds like calling ``sleep-forever``
  will take some time to complete and we don't want to hang about for
  that.

  The set of special forms is fixed and cannot be extended.

* it is an *expander* which works similarly but not quite the same as
  a special form

  Here, someone flashing a badge saying "Advanced User" and with a
  middle name of "Danger" might create a *template* to "create code."

  Expanders/templates are great but not for the faint-hearted.

* otherwise it is a regular function call

  Here, the evaluator looks at each element of the form and evaluates
  it recursively which neatly embeds any sub-expressions into the
  intermediate code:

  .. code-block:: idio

     a + (b * c)

  becomes (broadly):

  .. code-block:: idio

     + a (* b c)

  which looks like, initally

  .. parsed-literal::

     + a *sub-expr*

  but with, thanks to the recursion spitting out the sub-expression
  first, the code for:

  .. parsed-literal::

     * b c

  The overall byte code will evaluate :samp:`* {b} {c}` and the value
  of that sub-expression will be placed into the next bit of byte
  code, :samp:`+ {a} {sub-expr}`.

Namespaces
==========

If we take a snippet of code:

.. code-block:: idio

   n := 10

   define (foo a) {
     n + a
   }

   foo 7		; 17

then the evaluation of ``foo 7`` should return the value ``17``.  Not
totally unreasonable.

Inside the function ``foo`` we are using two variables:

* ``n`` is not a parameter to ``foo`` (nor any of its wrapping
  functions -- as it doesn't have any) so it is described as a *free
  variable* and, indeed, appears to be defined at the "top-level" (of
  something)

  In fact, ``n`` will have been added to the table of known top-level
  variables in this *module* (whatever this module is called).

  Anything else in this module can also use or set ``n``.

* ``a`` is a parameter to the function ``foo``

Local (parameter) variable names are preferred to free variables.

Nested Functions
----------------

Functions can be nested which means that inner parameter names occlude
outer ones.

If we try to occlude *all-the-things*:

.. code-block:: idio

   n := 10

   define (foo n) {
     a := 5

     define (bar a) {
       n + a
     }

     bar 1
   }

   foo 7		; 8

We call the function ``foo`` whose parameter ``n`` occludes the
top-level ``n``.  Inside ``foo`` we have defined a variable called
``a`` and also a function ``bar`` whose parameter ``a`` occludes
``foo``'s locally defined ``a``.

Therefore, when we call ``foo 7`` meaning that inside ``foo`` ``n`` is
``7`` and at the end of ``foo`` we call ``bar 1`` meaning that inside
``bar`` ``a`` is ``1``.

The invocation of ``bar 1`` returns ``7 + 1`` and as that is the last
thing in ``foo`` that is what ``foo`` returns.

Modules
-------

Modules are a very simple mechanism to separate namespaces.  Each
module has its own "top-level" and so each module can have its own
``n`` (a very popular name...).

Of course, we want to be able to *export* names (of values or
functions) to our users and, in turn, they want to *import* those
names.

In this case, we might export ``foo`` (but not ``n``) and then a user
of us could call :samp:`foo {i}` and the right thing would happen.

.. code-block:: idio

   module Foo

   export (foo)

   n := 10

   define (foo a) {
     n + a
   }

   provide Foo

The module's name, ``Foo``, must be a symbol and you must
:samp:`provide {name}` at the bottom (as a verification that the
module was loaded correctly and to prevent unnecessary reloads).

The filename the module is stored in must be :file:`{name}.idio`.

Your user will likely:

.. code-block:: idio

   import Foo

   n := 99

   foo 7		; 17

The mechanism for that is that the order of lookups for free
variables, this time, ``foo``, is:

* the current module's top-level

* the exported names of the most recently imported module

* the exported names of the next most recently imported module

* etc.

Inside ``foo``, when it was evaluated, the system had discovered
``Foo``, the module's, top-level variable ``n`` and has no need to go
searching for (or care about) the user's decision to also use ``n``.

Default Imports
^^^^^^^^^^^^^^^

All user-declared modules default to having the ``Idio`` module in
their import list, which, as a peculiarity, implicitly exports all of
its top-level names, and so the user's module gets to see all of the
standard functions.

The set of imported names is currently all-or-nothing of a module's
exported names.  That needs a little finesse!

Direct Names
^^^^^^^^^^^^

Some modules, the standard library, ``libc``, being an egregious
example, export a lot of names that clash with what we might want to
use as external command names:

.. code-block:: idio

   import libc

   mkdir "foo"

will get an ugly complaint about primitive arity or worse.  That's
because ``mkdir`` is exported by ``libc`` (amongst several hundred
others) and can therefore be resolved to a value (a primitive function
expecting two arguments -- hence the complaint) and not left to be
evaluated as itself, a symbol.

There's a couple of solutions, here:

#. we could *quote* the ``mkdir`` word, forcing it to be a symbol:

   .. code-block:: idio

      import libc

      'mkdir "foo"

#. we could not ``import libc`` but just use it selectively with
   *direct references* to its exported names

   .. aside::

      No crafty access to the other guy's ``n``!

   You can't access a name that isn't exported but you can access
   anything that is exported by its full name:

   .. code-block:: idio

      libc/exit 3

.. include:: ./commit.rst
