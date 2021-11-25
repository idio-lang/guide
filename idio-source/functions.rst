.. include:: ./global.rst

**************
Idio Functions
**************

This is not a comprehensive list but rather a flavour.

Some functions are closely tied to certain value types and others are
a bit more generic.

Values
======

Constants
---------

We can test for some kinds of constants with :ref:`boolean?
<ref:boolean?>`, :ref:`null? <ref:null?>`, :ref:`void? <ref:void?>`,
:ref:`eof-object? <ref:eof-object?>`.

Numbers
-------

Numbers are implemented as either :ref:`fixnums <ref:fixnum type>` or
:ref:`bignums <ref:bignum type>`.  Fixnums are fairly lightweight,
bignums less so.

Test generically with :ref:`number? <ref:number?>` and :ref:`integer?
<ref:integer?>` and more specifically with :ref:`fixnum?
<ref:fixnum?>` and :ref:`bignum? <ref:bignum?>`.

Use :ref:`read-number <ref:read-number>` to turn a string into a
number.

Basic arithmetic with :ref:`+ <ref:+>`, :ref:`- <ref:->`, :ref:`*
<ref:*>` and :ref:`/ <ref:/>`.

.. aside::

   These arithmetic functions correspond with, say, :lname:`Bash`'s
   ``[[`` compound command's condition expressions :samp:`-{op}`,
   eg. ``-eq``.

   That is, they are, effectively, the *name* of the equivalent
   arithmetic operation.  This avoids the clash between, say, the
   arithmetic operator, ``<`` and the I/O redirection operator, ``<``.

Arithmetic comparisons with :ref:`lt <ref:lt>`, :ref:`le <ref:le>`,
:ref:`eq <ref:eq>`, :ref:`ne <ref:ne>`, :ref:`ge <ref:ge>` and
:ref:`gt <ref:gt>`.

All of the above four arithmetic and five comparison functions have
binary infix operators such that :samp:`x {op} y` becomes
:samp:`binary-{op} x y` with corresponding functions :ref:`binary-+
<ref:binary-+>`, etc..

Further arithmetic with :ref:`abs <ref:abs>`, :ref:`quotient
<ref:quotient>`, :ref:`expt <ref:expt>` (`x` to the power `y`),
:ref:`sqrt <ref:sqrt>` and :ref:`exp <ref:exp>` (Euler's number to the
power `x`) with the complementary :ref:`log <ref:log>` (natural
logarithm).

Trig with :ref:`cos <ref:cos>`, :ref:`sin <ref:sin>`, :ref:`tan
<ref:tan>` and :ref:`atan <ref:atan>`, :ref:`acos <ref:acos>` and
:ref:`asin <ref:asin>`.

For bignums you can test for :ref:`real? <ref:real?>` (not an
integer!) and exactness with :ref:`exact? <ref:exact?>` and
:ref:`inexact? <ref:inexact?>`.  You can extract the :ref:`mantissa
<ref:mantissa>` and :ref:`exponent <ref:exponent>`.

Unicode
-------

Test with :ref:`unicode? <ref:unicode?>` and compare with
:ref:`unicode=? <ref:unicode=?>`.

If your module has imported the :ref:`unicode module <ref:unicode
module>` (the default ``Idio`` module does) then you'll have a few
more useful Unicode *Category* and *Property* predicates available:
:ref:`Alphabetic?  <ref:unicode/Alphabetic?>`, :ref:`Decimal_Number?
<ref:unicode/Decimal_Number?>`, :ref:`Lowercase?
<ref:unicode/Lowercase?>`, :ref:`Uppercase?
<ref:unicode/Uppercase?>`, :ref:`Titlecase_Letter?
<ref:unicode/Titlecase_Letter?>`, :ref:`Punctuation?
<ref:unicode/Punctuation?>`, :ref:`Symbol? <ref:unicode/Symbol?>`,
:ref:`White_Space? <ref:unicode/White_Space?>` and so on.

There are a couple of transformer functions: , :ref:`->Uppercase
<ref:unicode/->Uppercase>`, :ref:`->Lowercase
<ref:unicode/->Lowercase>`, :ref:`->Titlecase
<ref:unicode/->Titlecase>`.

Strings
-------

Test with :ref:`string?  <ref:string?>` and :ref:`pathname?
<ref:pathname?>`.

The number of code points in the string is given by
:ref:`string-length <ref:string-length>` and you can get individual
code points with :ref:`string-ref <ref:string-ref>`.

You can change a code point with :ref:`string-set! <ref:string-set!>`
or all of them with :ref:`string-fill! <ref:string-fill!>` but you
can't put a "wider" code point into a "narrower" string.

Use :ref:`append-string <ref:append-string>` (or
:ref:`concatenate-string <ref:concatenate-string>`) to join strings
(or a list of strings) together and :ref:`join-string
<ref:join-string>` to use a delimiter.

Get the index of a code point with :ref:`string-index
<ref:string-index>` or :ref:`string-rindex <ref:string-rindex>`.

You can explicitly get a substring of another with :ref:`substring
<ref:substring>` noting that the second index is up to but not
including -- for example, the string up to where ``string-index`` said
some code point was.

Split strings up with :ref:`fields <ref:fields>` (using :ref:`IFS
<ref:IFS>`) or :ref:`split-string <ref:split-string>` (using
:var:`IFS` by default) or :ref:`split-string-exactly
<ref:split-string-exactly>` which is more exacting than the
traditional shell "splitting on whitespace", say.

You can do some string comparisons though it is worth noting that, for
consistency, these rather inefficiently convert the string into UTF-8
and then call either :manpage:`strncmp(3)` or
:manpage:`strncasecmp(3)` (as appropriate).  See :ref:`string\<?
<ref:string\<?>`, :ref:`string\<=? <ref:string\<=?>`, :ref:`string=?
<ref:string=?>`, :ref:`string\>=? <ref:string\>=?>`, :ref:`string\>?
<ref:string\>?>` and their case-insensitive variants
:ref:`string-ci\<? <ref:string-ci\<?>`, etc..

Symbols and Keywords
--------------------

You can test with :ref:`symbol? <ref:symbol?>` and :ref:`keyword?
<ref:keyword?>`.

Handles
-------

Test a handle with :ref:`handle? <ref:handle?>`, :ref:`input-handle?
<ref:input-handle?>`, :ref:`output-handle? <ref:output-handle?>`,
:ref:`eof? <ref:eof?>` and :ref:`closed-handle? <ref:closed-handle?>`,
the result of :ref:`close-handle <ref:close-handle>`.

Handle attributes include :ref:`handle-line <ref:handle-line>`,
:ref:`handle-pos <ref:handle-pos>` and :ref:`handle-name
<ref:handle-name>`.

Access the contents with :ref:`read-line <ref:read-line>`,
:ref:`read-lines <ref:read-lines>` and :ref:`read-char
<ref:read-char>`.

Output can be broadly two flavours: the *printed* representation of an
object, something the reader might read back in; and a *display*
representation, something the user might want to experience.  This
broadly only affects strings and characters:

.. csv-table::
   :header: printed, displayed
   :widths: auto
   :align: left

   ``"hello\tworld"``,``hello	world``
   ``#U+0127``, ``ħ``

Writing to (output!) handles has some basic methods:

* for printing, :ref:`write <ref:write>` any value and :ref:`puts
  <ref:puts>` a string, in particular

* for displaying, :ref:`display <ref:display>` and :ref:`edisplay
  <ref:edisplay>` to the current error handle

For general formatted :ref:`printing`, :ref:`hprintf <ref:hprintf>`
(handle-printf) is the function underlying :ref:`printf <ref:printf>`,
:ref:`eprintf <ref:eprintf>` to the current error handle and
:ref:`sprintf <ref:sprintf>` to get a string.

You can change the position of a handle, like a file position, with
:ref:`seek-handle <ref:seek-handle>` and :ref:`rewind-handle
<ref:rewind-handle>`.

File Handles
^^^^^^^^^^^^

Test a file handle with :ref:`file-handle? <ref:file-handle?>`,
:ref:`input-file-handle?  <ref:input-file-handle?>` and
:ref:`output-file-handle? <ref:output-file-handle?>`.

Create a file handle with :ref:`open-input-file <ref:open-input-file>`
and :ref:`open-output-file <ref:open-output-file>` or the more general
:ref:`open-file <ref:open-file>` which accepts an
:manpage:`fopen(3)`-style mode string.

Get the underlying file descriptor with :ref:`file-handle-fd
<ref:file-handle-fd>`.

String Handles
^^^^^^^^^^^^^^

Test a string handle with :ref:`string-handle? <ref:string-handle?>`,
:ref:`input-string-handle?  <ref:input-string-handle?>` and
:ref:`output-string-handle? <ref:output-string-handle?>`.

Create a string handle with :ref:`open-input-string
<ref:open-input-string>` and :ref:`open-output-string
<ref:open-output-string>`.

Get the accumulated string from an output string handle with
:ref:`get-output-string <ref:get-output-string>`.

General Functions
=================

Conditional Expressions
-----------------------

The canonical conditional expression is :ref:`if <if special form>`
with :ref:`cond <cond special form>` being the
``if .. elif .. elif .. else ..`` variant.  ``cond`` has an extremely
powerful ``=>`` operator implementing an *anaphoric if*.

``if`` breaks a few conventions in that it doesn't use ``then`` or
``else``, it is simply:

    :samp:`if {condition} {consequent} {alternative}`.

If either of :samp:`{consequent}` or :samp:`{alternative}` are simple
values then, together with the one line per expression, it can look
awkward.

Here, the :samp:`{consequent}`, ``#n``, feels lost in front of the
opening brace of the :samp:`{alternative}`.

.. code-block:: idio

   if (pair? a) #n {
     eprintf "not a pair!\n"
   }

Not great.

:ref:`when <ref:when>` is syntactic sugar for ``if`` where there is no
alternate clause.  It might scan better for maintainers.

Use :ref:`not <ref:not special form>` to invert the boolean result
with :ref:`unless <ref:unless>` implicitly doing it for you.

Selecting between known cases (think: explicit values) is done with
:ref:`case <ref:case>`.  :ref:`regex-case <ref:regex-case>` and
:ref:`pattern-case <ref:pattern-case>`, in particular, offer something
closer to the shell's ``case`` statement.

Looping Expressions
-------------------

The :ref:`do <ref:do>` expression loops over a body of code,
initialising then incrementing some loop variables in a near identical
fashion for :lname:`C`'s :samp:`for ({init}; {test}; {step}) {body}`
statement.

The only meaningful difference is that in addition to the
:samp:`{test}` for end of loop you can specify some code to determine
the value to return from ``do``.  Remember, everything returns a
value.

Iteration Over Collections
--------------------------

You can either :ref:`map <ref:map>` over a list, array or keys of a
hash table or loop with :ref:`for-each <ref:for-each>` depending on
whether you want to collect the results of each iteration.

You can iterate over lists with :ref:`fold-left <ref:fold-left>` and
:ref:`fold-right <ref:fold-right>`.

Equality
--------

Things can be:

* :ref:`eq? <ref:eq?>` -- the same in memory (at least,
  indistinguishable)

* :ref:`eqv? <ref:eqv?>` -- the same value (numbers and strings)

* :ref:`equal? <ref:equal?>` -- the same collections of values (lists,
  arrays and hash tables)

(File) Predicates
-----------------

You can ask the usual file predicate questions with :ref:`f?
<ref:f?>`, :ref:`d?  <ref:d?>`, :ref:`l?  <ref:l?>` etc. with :ref:`r?
<ref:r?>`, :ref:`w? <ref:w?>` and :ref:`x? <ref:x?>` available.

.. _`printing`:

Printing
--------

Through :ref:`printf <ref:printf>` etc. you can control how
:ref:`values are printed <ref:printing values>`.  Primarily numbers
with the usual ``%d``/``%x`` variants.  Fixnums and some integer
:lname:`C` types can use a ``%b`` binary output format.

If you define a structure you can control how it is printed.

Most of the :ref:`libc <ref:libc module>` structures are similarly
controlled -- albeit, changing that format requires a recompile!

Sorting
-------

:ref:`Sorting <ref:sorting>` revolves around the :ref:`sort
<ref:sort>` function which takes an optional `accessor` argument which
allows indirect sorting.

Hence there are :ref:`sort-mtime <ref:sort-mtime>`, :ref:`sort-size
<ref:sort-size>` etc. functions which take a list of filenames and
sort them by some :manpage:`stat(2)` attribute.

Loading Code
------------

:ref:`loading <ref:loading>` is based on the :ref:`load <ref:load>`
function.

``load`` will always run through the file it loads.  You can effect a
load-once system with a combination of :ref:`require
<ref:require>`'ing a file which :ref:`provide <ref:provide>`'s some
feature.

You can :ref:`import <ref:import>` a module of code -- which uses the
``require``/``provide`` mechanism as an aside.

Matching
--------

Use :ref:`regex-case <ref:regex-case>` and the simplified
:ref:`pattern-case <ref:pattern-case>` for :ref:`POSIX regex
<ref:POSIX regex>` pattern matching.

The underlying standard library :ref:`regcomp <ref:regcomp>` and
:ref:`regexec <ref:regexec>` functions are available for more direct
use.

Path Manipulation
-----------------

It's not *quite* as easy to manipulate PATHs in :lname:`Idio` as in
the shell but there is a port of some :lname:`Bash` functions to
:ref:`manipulate PATHs <ref:path manipulation>` that might be quite
useful.

Only append :file:`/usr/local/bin` if it is a directory:

.. code-block:: idio

   path-append 'PATH "/usr/local/bin" :test d?

``d?``, here, is the file predicate :ref:`d? <ref:d?>` and is just a
regular function meaning you can write your own with more bespoke
considerations.

Globbing
--------

File name globbing *can't* work in :lname:`Idio` in the same way it
works in a shell because :lname:`Idio` allows (most of) the globbing
meta-characters (``*``, ``?``, ``[``) to be valid code points in
symbols/variable names.

You can workaround this if you quote the glob expression (or at least
ensure that it is a symbol -- periods in filenames, ``.``'s, are
inconvenient):

.. code-block:: idio

   ls -l '*.c

Otherwise, you'll need to call the :ref:`glob <ref:glob>` function
directly which will return a list of matches.

``glob`` can return no matches, the empty list, ``#n``, so you will
need to test before passing the list onto an external command (as the
constant ``#n`` has no sensible stringified representation for
external commands):

.. code-block:: idio

   files := glob "*.c"

   if (null? files) {
     eprintf "WARNING: no files matched *.c\n"
   } {
     ls -l files
   }

Here, :var:`files`, a list of file names, will have been expanded out
as one of the arguments to the external command :program:`ls`.

Indexing
--------

Causing plenty of annoyance is the ``.`` :ref:`value-index
<ref:value-index>` operator.

The goal is a `Templates::Toolkit` / `Jinja2`-style accessor method
which does something sensible for lists, arrays, hash tables, strings,
appropriately tagged :lname:`C` structures and so on.:

.. code-block:: idio

   arr := #[ 1 2 3 ]
   arr.0		; 1
   arr.2 = 99		; #[ 1 2 99 ]
   
   str := "hello"
   str.0		; #\h

   sb := libc/stat "."
   sb.st_ino		; 69256525
   
.. note::

   In this particular case trying to *set* an element of the string
   will fail as it is flagged as a `const` string (because it was
   constructed by the reader).  You would need to :ref:`copy-string
   <ref:copy-string>` first.

   .. code-block:: idio

      str := copy-string "hello"
      str.1 = #\E			; "hEllo"

   Not all :lname:`C` structures have a `setter`.

Without any form of `Type Inference` then this must query the value
type before continuing but it is considerably more readable.

The indexing element can be a *variable* so that :samp:`arr.{i}` does
the right thing.  Of course, if ``st_ino`` is a variable then the
previous expression ``sb.st_ino`` might have unintended consequences.
You can force a name/symbol with :ref:`quote <ref:quote special
form>`: ``sb.'st_ino``.

The indexing element can also be a *function* in which case the
function is applied to the value, that is :samp:`{v}.{f}` becomes an
invocation of :samp:`{f} {v}`.

This is useful for splitting a line into :ref:`fields <ref:fields>`:

.. code-block:: idio-console

   Idio> (read-line).fields
   hello        world
   #[ "hello        world" "hello" "world" ]

Here, ``fields`` has returned an array of the original string and the
:ref:`IFS <ref:IFS>`-separated words in the string.

.. note::

   The reason ``.`` is annoying is that we are all prone to writing
   shell-like statements such as

   .. code-block:: idio

      ls *.txt

   which ``value-index`` can't figure out.

.. include:: ./commit.rst
