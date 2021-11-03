.. include:: ./global.rst

###############
Idio User Guide
###############

:lname:`Idio` is a programming language for orchestrating commands
like a shell.  It is asking the question, can we write better shell
scripts by integrating shell-like constructions into a programming
language?

It is a programming language first and foremost so not all traditional
shell command syntax translates directly.

Also, as a programming language, it is intolerant of failure.  No more
shell scripts where errors slide casually by.

A few pointers:

* normal shell command syntax:

  .. code-block:: idio

     find /usr -name foo | wc -l

  that is, :samp:`{cmd} {args}`, with no extraneous parentheses or
  commas.  That applies to the programming language proper, not just
  the "shell" syntax

* (potential) variable names, ie. *symbols*, can be pretty flexible so
  ``/usr``, ``-name``, ``|`` and ``-l`` were all symbols in the above
  example

* which includes what would have been the shell meta-characters ``*``
  and ``?``

  * ``?`` is used for predicate names, for example, ``symbol?``

  * ``*`` is conventionally used to bookend global names

  Going further:

  * ``!`` is conventionally used to suffix mutation functions, for
    example, ``set!`` (whose *infix operator* is the rather more
    familiar ``=``)

  * ``/`` is used to join module and value names if you use the name
    "directly", for example, ``libc/exit``, to call the function
    ``exit`` defined in the module ``libc``

  * ``:`` is the prefix for keywords

  You get the picture, names can have punctuation characters in them
  to help communicate ideas and intent.

  There are limits on punctuation characters in symbols, mostly to
  prevent syntactic confusion: ``foo"bar "hello"`` and ``foo(bar (1 +
  2)`` would not help improve code comprehension.

* whitespace is important:

  .. code-block:: idio

     a:=10

  is requesting the value of a variable called ``a:=10`` and isn't
  defining anything (assuming you could discern the ``:=`` definition
  operator in the mix)

  This is the exact opposite of the shell which demands that there is
  no whitespace in an assignment.

* normal programming variables -- no sigils:

  .. code-block:: idio

     a := 10

     printf "a is %d\n" a

* *Wait!* How is the use of ``a`` in the ``printf`` expression
  different to the ``find``, ``foo`` and ``wc`` in the first?

  Answer: it isn't.  If a potential variable name is bound to a value,
  here, ``a`` was bound to ``10``, then you'll get the value,
  otherwise variable names resolve to "themselves", symbols, and carry
  on life as "values which happen to be symbols" until something wants
  to use them.

  In the case of the pipeline, the system eventually decides to invoke
  (the symbol) ``find`` -- with arguments ``/usr``, ``-name`` and
  ``foo`` each of which are (probably) symbols themselves.

  A symbol isn't invokable (it's not a function) so the system will
  break out of programming language mode and go into shell mode and
  search :envvar:`PATH` for an external command called ``find``.

  Symbols have a fairly obvious translation into a string as the
  arguments for the external command.

  .. tip::

     Try to use *strings* for filenames, so ``"/usr"`` and ``"foo"``
     in this case.

     That isn't because ``/usr`` and ``foo`` can't be translated into
     strings for an external command but because none of the
     redirection operators are expecting symbols as arguments so it's
     a good habit to be in.

* everything is :samp:`{cmd} {args}` *including* things like **infix**
  arithmetic operations:

  .. code-block:: idio

     a + 1

  where the *reader* spots the infix operator ``+`` and does a little
  rearranging to the "normal" :samp:`{cmd} {args}` form:

  .. code-block:: idio

     + a 1

  *before* the *evaluator* gets to work.

  Now the ``+`` is in "functional position", ie. :samp:`{cmd}`, and
  will be resolved to be the function that adds numbers together.

  The ``|`` in the first example was also identified as an infix
  operator and "a little rearrangement" took place.

* infix operators can be a bit annoying:

  .. code-block:: idio

     zcat file | tar xvf \-

  Without the ``\``, the ``-`` is being treated as an arithmetic infix
  operator and so the reader is waiting for the second argument.  Here
  we can *escape* the ``-`` from the clutches of the reader and get on
  with our day.

* the other annoying infix operator is ``.``, used to index into
  compound values (arrays, structures etc.)

  If you forget to use strings for filenames you can get caught out:

  .. code-block:: idio

     find /usr -name foo.txt | wc -l

  where the ``foo.txt`` part is seen by the reader as ``foo . txt``
  and is rearranged as the sub-expression ``(value-index foo txt)``
  where ``foo`` and ``txt`` are perfectly good (potential) variable
  names.

  They almost certainly aren't variable names, though, and
  ``value-index`` will complain that ``foo`` is not indexable.

  You could escape it like before:

  .. code-block:: idio

     find /usr -name foo\.txt | wc -l

* **one expression per line**

  You cannot put more than one expression on a line, there is no
  statement separator or, in :lname:`Bash`-speak, *control operator*
  like :lname:`Bash`'s ``|| & && ; ;; ;& ;;& ( ) | |& <newline>``.

* however, the reader will extend an expression across lines until it
  gets a matching brace, parenthesis or bracket or the matching string
  delimiter

  Strings can be multi-line.

  This leads to a normal form for defining functions:

  .. code-block:: idio
     :linenos:

     define (add a b) "
     add `a` to `b`
     ...
     " {
       ;; calculate a result in a local variable
       r := a + b

       ;; return the result
       r
     }

  Here the string-opening ``"`` on line 1 means the documentation
  string is not complete until the matching ``"`` on line 4 where the
  ``{`` before the end of line means the body is not complete until
  the matching ``}`` on line 10.

* single element expression problem

  The ``r`` on line 9, above, is clearly returning the result of the
  addition.  That's idiomatic of a programming language (well, many
  programming languages).

  With our shell-hats on, though, that means that:

  .. code-block:: idio

     ls

  **doesn't** run the external command :program:`ls` but instead
  returns the value of the variable ``ls`` which, in all probability,
  resolves to the symbol ``ls``.

  To force the execution of ``ls``, that is, instead of resolving a
  value we want to force the invocation of a single element expression
  we need to wrap it in parentheses:

  .. code-block:: idio

     (ls)

  Annoying.  It only affects single-word external commands (although
  :program:`make`, :program:`env`, :program:`sort` spring to mind).

  If you pass any arguments then it must be a function invocation and
  the system can figure out it needs to execute :program:`ls`.

* *everything* returns a value

  Everything returning a value is a thing and turns out to be useful.
  *Who knew?*

* there is deliberately no support for *interactive* sessions

  :lname:`Idio` is about better shell *scripting*.

  There are far better interactive shells out there than :lname:`Idio`
  might ever be including ancillary software like :program:`rlwrap`.

  So, use :program:`rlwrap` and realise it's not important.

  :lname:`Idio` has normal *Job Control* functionality in respect of
  controlling terminals, Process Group IDs etc. just no support
  whatsoever for interactivity beyond terminal cooked mode.

  Remember, :lname:`Idio` is intolerant of failure and the default
  behaviour is to quit when an external command fails.

  That *is* quite annoying in interactive sessions so those have this
  behaviour suppressed (albeit re-asserted in pipelines).

Before we go on and get lost in the details, here's a

.. warning::

   :lname:`Idio` is

   **experimental**
       some of the syntax and features fit in the "let's see how this
       goes" pattern

   **immature**
       yes, there are bugs and plenty of them

       there are longhand forms that need shorthand syntaxes

       it needs to be used in anger more

   **incomplete**
       as a programming language, :lname:`Idio` is far from complete
       and even as a shell there are several missing pieces

Now, that's not a great sales pitch but it really is just a warning
that :lname:`Idio` is new and hasn't seen enough of the world not to
be too fragile in the face of pesky users.

That said, :lname:`Idio` is quite usable and, in fact, is used to
generate it's own code during a build as well as much of the
boilerplate code of interfaces into libraries and the reStructuredText
of the `Reference Manual <https://idio-lang.org/docs/ref/>`_.

Let's dig into some detail.

.. toctree::
   :maxdepth: 1

   var-val
   functions
   errors
   shell
   running
   examples/index
   evaluation

.. include:: ./commit.rst
