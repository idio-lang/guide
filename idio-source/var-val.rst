.. include:: ./global.rst

*****************************
Creating Variables and Values
*****************************

So, three basic rules:

#. whitespace is important

#. one expression per line -- remembering that the reader will keep
   going until it finds the matching brace, parenthesis, bracket or
   double-quote

#. everything returns a value

.. aside::

   Not pernickety enough, some might say.

Also, :lname:`Idio` is a bit pernickety about things not working.

The memory used by :lname:`Idio` is Garbage Collected so that, by and
large, you don't need to be concerned about pro-actively allocating or
freeing memory.

Of course, as with any programming language, if you *maintain*
references to scarce operating system resources, like file descriptors
through file or pipe handles, then you'll eventually run out.

Variables
=========

Variable names can contain lots of meaningful punctuation characters
hence why whitespace is important.

Declaration
-----------

Variables must be *declared* otherwise the system will think that you
don't know what you're doing.

In a very formal way you would :ref:`define <ref:define special form>`
variables:

.. code-block:: idio

   define a 10

but there's a more familiar infix operator, ``:=``:

.. code-block:: idio

   a := 10

That's created a "top-level" lexical scope variable in the current
:ref:`module <module>`.

If you were inside a block, ``{ ... }``, then the declaration is local
to the block:

.. code-block:: idio

   a := 10

   {
     a := 5	; local to block declaration,
		; occludes the top-level

     a		; 5
   }

   a		; 10

Environment Variables
^^^^^^^^^^^^^^^^^^^^^

As a variation on a theme, ``:*`` [#]_ creates an environment variable
or, more precisely, a dynamically scoped variable tagged as an
environment variable.

The environment part doesn't do much, yet.  Only when the system
decides to run an external command will it gather up all the existing
tagged-as-environment variables and create an environment for the
command.

All existing operating system environment variables
(:manpage:`environ(3P)`) at startup are created as these environment
variables.  Just like a regular shell.

Rather than remembering the old value, assigning a new and then
re-assigning the old value back, you might want to "re-create" an
environment variable for the transient period of a block:

.. code-block:: idio

   ls -l		; /usr/bin/ls (probably)

   {
     PATH :* "/danger/will/robinson"

     ls -l		; /danger/will/robinson/ls (possibly, or not found)
   }
   
   ls -l		; /usr/bin/ls (again)

Here, the "new" :envvar:`PATH` occludes the "main" one (or any other
similarly defined "new" versions) and lasts, like any dynamically
scoped variable until the end of the block it is defined in.  Next
time :envvar:`PATH` is used we'll be back to the original version.

.. note::

   :ref:`string interpolation`, say, to prepend something to
   :envvar:`PATH`, is a bit clumsy in :lname:`Idio`, see below

.. [#] As a mnemonic for environment, think "colon-star" for the stars
       in the firmament around us.

Dynamic Variables
^^^^^^^^^^^^^^^^^

Similarly, ``:~`` [#]_ creates a regular dynamically scoped variable.

For both dynamic and environment variables it is an error to try to
use the variable outside of the scope of its definition.

.. [#] As a mnemonic for dynamic, think "colon-wobbly-hand" for it
       coming and going depending on context.

Computed Variables
^^^^^^^^^^^^^^^^^^

Similarly, ``:$`` [#]_ creates a lexically scoped :ref:`computed
variable <ref:computed value>`.

When you create a computed variable you defined it with either or both
of a `getter` and `setter` function, the getter taking no arguments
and the setter one (the value you are setting it with).

Those functions can do whatever.

Subsequently, whenever the evaluator sees a reference to the variable
it will replace it with a call to the getter and where it is being
assigned to it will be replaced with a call to the setter.

If you don't define a getter or setter then it is an error to
reference or set the variable as appropriate.

.. code-block:: idio

   var-get := #n
   var-set := #n

   {
     local-v := 19

     var-get = function () {
       local-v + 10
     }

     var-set = function (v) {
       local-v = v
     }
   }

   var :$ var-get var-set

   var				; 29
   var = -9			; local-v is -9
   var				; 1

As a more practical example, :lname:`Idio` defines :ref:`SECONDS
<ref:SECONDS>` as a read-only value which returns the number of
seconds that :program:`idio` has been running for.  There is no
setter.

.. code-block:: idio

   printf "running for %ds\n" SECONDS

.. [#] As a mnemonic for computed, think "colon-dollar" for it
       referencing a function for getting or setting.

Nested Recursive Functions
^^^^^^^^^^^^^^^^^^^^^^^^^^

As mentioned before, ``:+`` [#]_ creates a lexically scoped
recursively aware variable inside a block.  Clearly this is only
useful for a function.

You cannot use this at the top-level, use regular ``define``.

.. [#] As a mnemonic think "colon-plus" as calling myself repeatedly.

Assignment
----------

Changing a variable or, rather, changing the value a variable
references could use the formal :ref:`set! <set special form>` or its
more familiar ``=`` infix operator:

.. code-block:: idio

   a := 10

   {
     a = 7	; change top-level

     a		; 7

     a := 5	; local to block declaration from now on,
		; occludes the top-level

     a		; 5
   }

   a		; 7

Assignment works for variables of all kinds and scopes.

Values
======

Variables *reference* values and only values haves types.

Constants
---------

``#t`` and ``#f`` are the usual booleans.

Only ``#f`` is `false` and **everything else** is `true`.  As a
consequence, many lookup functions, in particular, return a useful
value or ``#f`` so that you can test with the returned value and go.

``#n`` is a nil/null value.  It's most common use is marking the end
of a list.

A few other :ref:`constants <ref:constants>` appear, in errors or at
the prompt.  They are all unusable in source code as there's no
mechanism to create them.  They'll have the form :samp:`#<{name}>`:
``#<unspec>``, ``#<void>``, ``#<eof>`` etc..

Numbers
-------

There are two types of :ref:`numbers <ref:number types>`, "small"
integer :ref:`fixnums <ref:fixnum type>` or "arbitrary" precision
:ref:`bignums <ref:bignum type>`, which can be used interchangeably.

They look like regular numbers: ``-1``, ``1.23``, ``10e99`` etc..

There's a small caveat that to avoid one of the infix operators,
floating point numbers must have a digit before the decimal place.

There is no support for non-finite quantities such as *NaN* or
infinities.

Characters
----------

"Characters" is a loose term in Unicode_/ISO10646_ when you really
mean *code points*.  There's two ways of expressing a :ref:`unicode
<ref:unicode type>` code point:

* :samp:`#U+{HHHH}` where the number of hex digits, :samp:`{H}`, is
  "enough" and leading zeroes are not required:

  .. code-block:: idio

     c1 := #U+127
     c2 := #U+0127

  both are U+0127 (LATIN SMALL LETTER H WITH STROKE)

* :samp:`#\\x` where :samp:`{x}` is the UTF-8 representation of the
  code point

  .. code-block:: idio

     c1 := #\ħ

  is also U+0127 (LATIN SMALL LETTER H WITH STROKE)

  The usefulness of this form depends on editors and font support.

Strings
-------

:ref:`Strings <ref:string type>` are an efficient array of unicode
code points contained within double-quotes and using ``\`` as an
escape character.

Strings can be multi-line.

The usual escapes are:

* ANSI C: ``\t``, ``\n`` etc.

  .. code-block:: idio

     s1 := "hello\nworld"
     s2 := "hello
     world"

  are the same (assuming there are no other non-printing characters in
  the second example, just the string extending across two lines).

* Unicode escapes:

  * :samp:`\\u{HHHH}` where there are up to four :samp:`{H}`\ s

  * :samp:`\\U{HHHHHHHH}` where there are up to eight :samp:`{H}`\ s
    -- albeit only six should ever be required for Unicode code
    points.

* byte escapes: :samp:`\\x{HH}` with up to two hex digits

  This exists to create pathnames with non-UTF-8 byte sequences.

  :lname:`Idio` strings with ASCII NULs in them are fine,
  ``"hello\x0world"``, but operating system interfaces will not be so
  happy.  A "format" error will be raised if you forget and pass such
  a string to those functions that might try to pass it to an
  operating system interface.

Pathnames
^^^^^^^^^

:ref:`Pathnames <ref:pathnames>` are a specially tagged string.  They
exist because filesystems don't have an encoding associated with them
and are strictly a sequence of bytes (excluding 0x00) and so "strings"
returned from the filesystem need to be flagged as different.

It's a bit awkward but mostly correct.

Problems arise if you try to compare your UTF-8 encoded source code
string with the byte-sequence string from the filesystem.

Of course, you can create pathnames in source to do such comparisons
with the ``%P{...}`` expression:

.. code-block:: idio

   utf8-name := "© 2021"
   file-name := %P{\xA9 2021}

Here, :var:`utf8-name` is seven bytes long as the UTF-8 encoding of
U+00A9 (COPYRIGHT SIGN) is 0xC2 0xA9 whereas :var:`file-name` starts
with 0xA9 directly (and is invalid UTF-8).

.. hint::

   If an operating system interface is queried for a filename,
   :lname:`Idio` will always return the filename tagged as a pathname.

.. _`string interpolation`:

String Interpolation
^^^^^^^^^^^^^^^^^^^^

There's no "trivial" :ref:`string interpolation <ref:string
interpolation>` in :lname:`Idio`, we need to do a little more work
than the common shell-like embedding of variables,
``"/danger/will/robinson:${PATH}"``, although the expression to be
expanded can more useful:

.. code-block:: idio

   {
     PATH :* #S{/danger/will/robinson:${PATH}}

     ...
   }

where the ``#S{...}`` construct is looking for embedded
:samp:`$\{{expr}\}` expressions.  The :samp:`{expr}` is a proper
expression although it will commonly be a variable name.

Whatever the result of evaluating :samp:`{expr}`, it will converted to
a string (if not one already) and the various snippets of string
joined together.

It's very useful for code generation.

Symbols
-------

:ref:`Symbols <ref:symbol type>`, "names", perhaps, are a type in
their own right.  They are normally used as variable names, that is,
references to values, but are quite handy as flags.

In a small number of cases you might need to :ref:`quote <ref:quote
special form>` the symbol to prevent the evaluator resolving a
variable name to a value when you actually want the variable name:

.. code-block:: idio

   a := 10

   printf "%s is %d\n" 'a a

Keywords
--------

:ref:`Keywords <ref:keyword type>` are, essentially, symbols that
start with ``:`` and are used as semantic flags: indicating an
optional parameter to a function, say.

Handles
-------

:ref:`Handles <ref:handle type>` are used for I/O and come in a few
flavours: the obvious file handle, and less obvious pipe handle which
together are file descriptor (FD) handles.  There's also string
handles which act like file handles but use memory rather than the
file system.

You don't create a handle directly but, rather, create one of the
flavours then largely continue using generic handle functions.

File Handles
^^^^^^^^^^^^

:ref:`open-input-file <ref:open-input-file>`, :ref:`open-output-file
<ref:open-output-file>` etc..  The usual sorts of interfaces.

Pipe Handles
^^^^^^^^^^^^

Pipe handles tend to be created as input to or output from an external
command.  They are notionally identical to file handles except you
can't seek on them.

String Handles
^^^^^^^^^^^^^^

:ref:`open-input-string <ref:open-input-string>` returns an input
handle based on the supplied string.  You can read and seek about in
it like a file handle.

:ref:`open-output-string <ref:open-output-string>` returns an output
handle where the backing store is memory.  Write to it like a output
file handle.

To get the resultant string back you need :ref:`get-output-string
<ref:get-output-string>` which returns whatever you wrote.  Compare
with using, say, :program:`cat`, to get back the contents of the file
you just wrote.

C Data Types
------------

For interaction with :ref:`libc <ref:libc module>`, the standard
library, and :lname:`C` extensions, :lname:`Idio` supports the
fourteen :lname:`C` base types and module-oriented typedefs thereof
and :lname:`C` pointer types through the :ref:`C module <ref:C
module>`.

There is limited manipulation of :lname:`C` data types, they are
generally passed around opaquely.  You can do some comparisons and
some limited arithmetic of identical types -- there's no implicit
casting between types.  The general idea being that you ask the
:lname:`C` API for some value which you can pass on to another
:lname:`C` API.

The :lname:`C` pointer types can be tagged which, with appropriate
support, means you can access fields of :lname:`C` structs reasonably
easily:

.. code-block:: idio

   sb := libc/stat "."
   sb.st_ino		; 69256525
   
Here, :var:`sb` is a ``C/pointer`` type (and, specifically, a "CSI"
tagged as :ref:`libc/struct-stat <ref:libc/struct-stat>`) and the
``st_ino`` field is the portable ``ino_t`` type.

If you are interested, you can query what an ``ino_t`` really is on
your system in a couple of ways:

.. code-block:: idio-console

   Idio> type->string sb.st_ino
   "C/ulong"
   Idio> libc/ino_t
   ulong

* :ref:`type-\>string <ref:type-\>string>` will return a
  representation of the :lname:`C` base type

* there is a variable named after the module's typedef which has the
  (symbolic) value of the :lname:`C` base type which is useful for
  creating instances of that type with :ref:`C/integer-\>
  <ref:C/integer-\>>`:

  .. code-block:: idio

     C/integer-> 2 libc/ino_t

  for those people who feel confident about their inodes...

Functions
---------

Functions are first class values in their own right.  It is normal for
them to be returned from function calls or blocks like any other
value.

Declaration
^^^^^^^^^^^

The most common definition form is:

.. code-block:: idio

   define (add a b) "
   add `a` to `b`
   ...
   " {
     ;; calculate a result in a local variable
     r := a + b

     ;; return the result
     r
   }

``define`` is playing a little trick and is actually creating an
anonymous *function value* under the hood:

.. code-block:: idio

   define add (function (a b) "
   add `a` to `b`
   ...
   " {
     ;; calculate a result in a local variable
     r := a + b

     ;; return the result
     r
   })

The anonymous function, here, is wrapped in parentheses otherwise
``define`` would be being given (far) too many arguments.

Just to re-iterate: ``define`` is :samp:`define {var} {val}` where
:samp:`{val}` is a single expression like ``"hello"`` or :samp:`(2 *
3)` or, here, a function value definition, which itself is
:samp:`function {formals} {[docstr]} {body}` and which, because
``define`` wants a single expression, is wrapped in parentheses like
the multiplication.

We can play this anonymous function trick ourselves with (a variant
not bothering with local variables or documentation):

.. code-block:: idio

   add := function (a b) {
     a + b
   }

Here, everything to the right of the declaration, ``:=``, is given
over to the evaluator which recognises it as a function declaration:
:samp:`function {formals} {[docstr]} {body}`.

We can even go one step further and have the function returned at the
end of a block -- because everything returns a value:

.. code-block:: idio

   add := {
     n := 10

     function (a b) {
       a + b + n
     }
   }

This is a bit more interesting as the block also defines a local
variable, ``n``, which the function is using in its calculation.
**Nothing else** can see that variable, it is entirely *private* to
this function.

You can only return one thing at a time so there is a final trick if
you want two or more functions to share such a private variable.
Here, you recall, we can assign values to top-level variables inside
the block:

.. code-block:: idio

   add := #f
   sub := #f

   {
     n := 10

     add = function (a b) {
       a + b + n
     }

     sub = function (a b) {
       n - (a + b)
     }
   }

Here, whilst the top-level variables ``add`` and ``sub`` are
re-assigned inside the block, the block itself returns ``#<unspec>``,
a "there is no sensible value to return" value -- not that anyone
cares as no-one is using the value returned from the block, it is
dropped on the floor.

.. aside::

   This is an exercise for the reader...

At this point, you might think that with ``n`` as a fixed value things
are pretty limited.  But that block looks very much like the blocks
being used as the bodies of the functions.  Function take parameters,
so ``n`` could have been a parameter.  Call a function
``create-add-sub`` with some value, :samp:`{x}`, and the functions
``add`` and ``sub`` (technically, the function values the ``add`` and
``sub`` variables reference!) now use :samp:`{x}` in their
calculations.

Nested Functions
^^^^^^^^^^^^^^^^

Now we've got the idea that we can define functions inside other
functions, we have *nested* functions.

We don't have to save them out to the top-level we could just be using
them as helper functions within the body of the outer function.

There's nothing to stop the helper functions having helper functions
inside them.  You could go a bit wild, here, but try to think of the
person who has to maintain your code.

Nested Recursive Functions
""""""""""""""""""""""""""

*Corner case alert!*

Free variables are the problem here, these are variables which are
referenced inside a block of code which the block did not contain the
definition for:

.. code-block:: idio

   define (foo a) {
     a + n
   }

``n`` isn't defined anywhere (obvious) so we assume that it is defined
in our top-level or the top-level of one of the modules we import.
That seems reasonable, what else can we do?  When we come to use ``n``
we'll shuffle about looking for an ``n`` in our top-level or the
top-level of one of the modules we import.

What if we want to call ourselves?

.. code-block:: idio

   define (foo a*) {
     if (null? a*) #n {
       foo (pt a*)
     }
   }

(this function doesn't do anything useful other than walk along a list
to the end and returns ``#n``)

We know that is going to be rewritten to:

.. code-block:: idio

   define foo (function (a*) {
     if (null? a*) #n {
       foo (pt a*)
     }
   })

and, as the precursor to defining ``foo``, the evaluator will see the
function value creation:

.. code-block:: idio

   function (a*) {
     if (null? a*) #n {
       foo (pt a*)
     }
   }

which contains the free variable ``foo``.  Hmm, we haven't defined
``foo`` yet but we're trying to use it.

Well, we sort of get away with this as when we come to use the
function value we'll find ``foo`` in our own top-level and promptly
call ourselves, which is what we want.

Technically, though, you could subsequently redefine ``foo`` and we'll
get the new ``foo`` instead.  Which might not be what you want.

This is even worse inside a block as *names* disappear and when the
code is called there'll be no ``foo`` at the top-level.  Oh dear.

.. code-block:: idio

   {
     ...

     define (foo a*) {
       if (null? a*) #n {
	 foo (pt a*)
       }
     }

     foo '(1 2 3)
   }

There is an answer to all of this pernickety nonsense with what is
called ``letrec`` in :lname:`Lisp`\ y languages.  Inside a block, any
use of ``define`` is rewritten as a call to ``letrec``.

``letrec`` plays a little trick like the one we played above where we
defined ``add`` and ``sum`` as ``#f`` outside the block and
re-assigned to them inside the block.  That means that the evaluator
can see a use of the variable in its lexical scope and can therefore
reference that instance rather than guessing that the variable is
probably defined at the top-level somewhere.

Inside a block, ``letrec`` has a ``:+`` infix operator:

.. code-block:: idio

   {
     ...

     foo :+ function (a*) {
       if (null? a*) #n {
	 foo (pt a*)
       }
     }

     foo '(1 2 3)
   }

Pairs and Lists
---------------

A :ref:`pair <ref:pair type>` is, unsurprisingly, an object that
references two things, a *head* and a *tail*.  You'd create one with
:samp:`pair {head} {tail}`.

The head and the tail can reference any value but the most common
structure is for the head to reference something and the tail to
reference another pair.  That second pair's head is likely to
reference a similar kind of thing as the first pair's head and the
second pair's tail to reference another pair.  And so on.  The final
pair's tail will reference ``#n``.

That arrangement of pairs is called a *list* and occurs pretty
frequently.  So frequently that there's a couple of ways in:

.. code-block:: idio

   l1 := pair 1 (pair 2 (pair 3 #n))
   l2 := list 1 2 3

``l1`` and ``l2`` are identical constructions.  Most people would
prefer the latter construction if you have all the elements to hand
but if you're building a list piecemeal then you'll see a lot of
recursive loops using :ref:`pair <ref:pair>`.

The value the *head* references doesn't have to be as simple as an
integer, it could be a list itself, a hash table, a ....

In the case of the head being a list, it is commonly called an
association list:

.. code-block:: idio

   l3 := '((#\a "apple" 'fruit)
           (#\b "banana" 'fruit)
	   (#\c "carrot" 'vegetable))

.. aside::

   The ``'``, a synonym of :ref:`quote <ref:quote special form>`, is
   used to prevent the evaluator looking at the expression and trying
   to invoke the function calls ``(#\a "apple" 'fruit)`` etc..

where ``#\a`` is the key "associated" with the list ``(#\a "apple
'fruit)``.  Functions like :ref:`assq <ref:assq>` etc. look for a key,
the ``#\a``, and return the associated list if found.

Arrays
------

An :ref:`array <array type>` is an *integer* indexed collection of
references to values starting at 0 (zero).  You can create an array
with :ref:`make-array <ref:make-array>` or by using a simple static
initialiser:

.. code-block:: idio

   arr := #[ 1 2 3 ]

(It's the :ref:`reader <reader>` creating the initial array, here, and
it doesn't understand variables so simple numbers and strings only.)

or something a bit more flexible:

.. code-block:: idio

   arr := array this that the-other

(:ref:`array <ref:array>` isn't really a constructor *per se* but just
calls :ref:`list->array <ref:list->array>` on your behalf.)

They are dynamically allocated but you can only access those elements
that have been placed.  You can't expect to create an array of one
element then try to access the fourteenth.

You can grow them by pushing elements onto the end with
:ref:`array-push! <ref:array-push!>` or front with
:ref:`array-unshift! <ref:array-unshift!>`.  The array will grow by
one element.

:ref:`array-pop! <ref:array-pop!>` and :ref:`array-shift!
<ref:array-shift!>` will shrink the array by one element.

You can use negative indices which become :samp:`{array-length} +
{index}` such that ``-1`` gets the last index, ``-2`` the second last
index, etc..  Obviously, if :samp:`{array-length} + {index}` is
actually negative you'll get a range error.

You can iterate over the array with :ref:`array-for-each-set
<ref:array-for-each-set>` and :ref:`fold-array <ref:fold-array>`.

Hash Tables
-----------

A :ref:`hash table <ref:hash table type>` is an indexed collection of
references to values.  The index can be any value except ``#n``.

.. sidebox::

   Function values *are* used as indexes into hash tables inside
   the VM.

So, want to use function values as an index to something?  Go right
ahead.

Test for a hash table with :ref:`hash? <ref:hash?>`.

You can create a hash table with :ref:`make-hash
<ref:make-hash>` or by using a simple static initialiser:

.. code-block:: idio

   ht := #{ (#\a & "apple") (#\b & "banana") }

(It's the :ref:`reader <reader>` creating the initial hash table,
here, and it doesn't understand variables so simple numbers and
strings only and single values for the head and tail defining each
key-value pair.)

or something a bit more flexible (noting the reader form ultimately
calls :ref:`alist-\>hash <ref:alist-\>hash>`):

.. code-block:: idio

   ht := alist->hash '((#\a "apple")
		       (#\b "banana"))

(which doesn't quite create the same values as the previous example as
here the hash values are lists of one string)

``make-hash`` let's you specify your own equivalence and hashing
functions.

Use :ref:`hash-ref <ref:hash-ref>` and :ref:`hash-set!
<ref:hash-set!>` to access and create/override entries and
:ref:`hash-delete!  <ref:hash-delete!>` to delete entries.
:ref:`hash-exists?  <ref:hash-exists?>`.

Use :ref:`hash-keys <ref:hash-keys>` and :ref:`hash-values
<ref:hash-values>` to get the set of keys and values.

You can iterate over the hash with :ref:`hash-walk <ref:hash-walk>`
and :ref:`fold-hash <ref:fold-hash>`.

Structures
----------

A :ref:`struct <ref:struct type>` is a *named* indexed collections of
references to values.

Names as in symbols which means you might need to be careful and quote
the field name if someone has created a variable that uses your
cunningly named ``n`` field.

There's two parts to structs, a *type* which declares the field names
and *instances* of that struct type.

Formally, structs have a parent type leading to a graph of
relationships between structs as in the :ref:`condition types
hierarchy <ref:condition types hierarchy>`.

If you define a structure you can control how it is printed.

.. _`module`:

Modules
-------

A :ref:`module <ref:module type>` allows you to create namespaces.

The mechanism is simple and you can only use true names, not variables:

.. parsed-literal::

   module *name*

   export (func1 value2)
   export func2

   import *other-module*

   ...

   provide *name*

With the module's source code in :file:`{name}.idio`.

Any names other than the ones you :ref:`export <ref:export>` cannot be
accessed by any importing module.  They can't see your top-level ``n``
variable or ``func`` function unless you export it.

It's all-the-names-or-nothing on the :ref:`import <ref:import>` front,
at the moment.

Some modules are quite polluting, the standard library, :ref:`libc
<ref:libc module>`, defines names that clash with regular shell
commands, for example.

Instead, you can pick and choose "direct" names, say, :ref:`libc/mkdir
<ref:libc/mkdir>`, using the combined :samp:`{module}/{name}` scheme,
rather than get every ``libc`` name with a crude import.

.. include:: ./commit.rst
