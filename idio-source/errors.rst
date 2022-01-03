.. include:: ./global.rst

**************
Error Handling
**************

Error handling in :lname:`Idio` is a derivative of *condition*
handling.  Conditions are from :lname:`Scheme` and offer a far richer
mechanism for handling various states that might be reached within the
program.

The fundamental difference between condition handling in :lname:`Idio`
and exception handling common to many programming languages is that
the program state is not unwound when a condition is raised meaning
the handler has an opportunity to perform some appropriate action and
*return on behalf of* the condition-raising code.

There are a few problems here:

#. a handler returning a value to a function that has raised an
   *error* is rarely the best solution

#. *restarts* might be a better solution

#. the restarts mechanism hasn't been written (yet)

Trapping Errors
===============

Most people are familiar with the ``try``/``except`` mechanism from
:lname:`Python` and in :lname:`Idio` we can constrain ourselves to
that style.

The expression for handling an error is:

    :samp:`trap {condition*} {handler} {body}`

Here, :samp:`{condition*}` is a named :ref:`condition type
<ref:condition types hierarchy>` or a list of condition types.

:samp:`{handler}` is a unary function which will be called with the
condition that was raised as an argument.  Given how bespoke the
required behaviour is, it will usually be an anonymous function.

:samp:`{body}` is the expression (scope!) that the handler is valid
for.

The constraint is that the handler should call :ref:`trap-return
<ref:trap-return>` when it is done.

Using the canonical "divide by zero" error, here, the *condition
type*, ``^rt-divide-by-zero-error``, we can try the following:

.. code-block:: idio
   :caption: :file:`./x.idio`

   define (reciprocate n) {
     printf "1 / %s\n" n
     1 / n
   }

   trap ^rt-divide-by-zero-error (function (c) {
     eprintf "caught /0\n"

     trap-return c
   }) {
     reciprocate 1
     reciprocate 0
     reciprocate 2
   }

which, when we run it, gives:

.. code-block:: console

   $ IDIOLIB=. idio x
   1 / 1
   1 / 0
   caught /0

which is, more or less, what you would expect.

There are two things of interest:

#. ``trap`` returned a value, :var:`c`, the condition it was passed,
   although, as no-one is using the value returned from ``trap``, it
   is lost

#. if we hadn't have called ``reciprocate 0`` then we would have run
   ``reciprocate 2`` which would have returned ``0.5`` and as that was
   the last expression in its body then that is what ``trap`` would
   have returned

   Albeit, no-one is using the value returned from ``trap``...

.. warning::

   Returning a value to a function that has raised an *error* is most
   likely going to compound the problem.

   There is an example in :ref:`condition handlers <ref:condition
   handlers>`.



.. include:: ./commit.rst
