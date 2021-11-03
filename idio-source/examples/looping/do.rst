.. include:: ../../global.rst

##
do
##

The usage of :ref:`do <ref:do>` *looks* irrationally complex but in
fact it is almost identical to :lname:`C`'s ``for`` statement.

In :lname:`C` you might say:

.. code-block:: C

   for (i=1,j=20; i<10; i++,j*=2) {
     ...
   }

where you are describing:

* a number of initialisation statements, ``i=1`` and ``j=2``

* a conditional expression to continue the loop, ``i<10``

* instructions on how to update the loop variables, ``i++`` and
  ``j*=2``

* a loop body, ``...``

You don't need to initialise anything.  You don't need to update
anything.  You are only required to have a loop conditional expression
and a body (also known as ``while``).

We don't have :lname:`C`'s post-increment or self-multiply operators
in :lname:`Idio` so we'll have to express those "long-hand" but the
``do`` loop will look remarkably similar.

Let's break ``do``'s syntax down, first: :samp:`do {var-clauses}
{test-result} {body}`.

Here, each *var-clause* in the list :samp:`{var-clauses}` looks like:
:samp:`({var} {init} {step})` where we declare the name of the
variable, have an expression to initialize it and :samp:`{step}`, an
expression to generate a new value for the next iteration of the loop.

For our two variables from :lname:`C`, the two *var-clause*\ s look
like:

    ``(i 1  (i + 1))``

    ``(j 20 (j * 2))``

and so, together, in a list look like:

.. code-block:: idio

   ((i 1  (i + 1))
    (j 20 (j * 2)))

The :samp:`{test-result}` list is a bit more interesting than
:lname:`C` in that not only do we have a conditional test for
continuing the loop but we can also define an expression for the value
to return from ``do``.  :lname:`C` doesn't return a value from ``for``
so this may be something new.

If you don't want to return a value, you still need to put something
in that slot, say, ``#n``.

So the :samp:`{test-result}` for us should look like ``((i lt 10)
#n)`` although we could have returned whatever was appropriate,
potentially using the loop variables, say, ``(i - j)`` -- or perhaps
something less predictable.

.. code-block:: idio

   (do ((i 1  (i + 1))
        (j 20 (j * 2)))
       ((i lt 10)
        #n)
       {
         ...
       })

In summary, ``do`` looks messier (lists of lists) but is functionally
identical to :lname:`C`'s ``for`` statement.

.. note::

   This example puts parenthesis around the whole ``do`` expression.
   That **doesn't** change the way it is interpreted, any expression
   with multiple elements is treated as a list by the reader.

   Here, it probably helps your ``$EDITOR`` indent the text properly
   with the clauses on separate lines!

.. rst-class:: center

   \*

As with :lname:`C`, you don't need to use any loop variables although
you do need to pass an empty list, ``#n``, in place of the
:samp:`{var-clauses}` list.

That means the previous ``do`` loop degenerates to:

.. code-block:: idio

   (do #n
       ((i lt 10)
        #n)
       {
         ...
       })

Here, the variable :var:`i` being tested in the conditional expression
will now be searched for in an outer scope, just like in :lname:`C`.
If it doesn't exist you'll get an error.  If it's not the :var:`i` you
were looking for then the unexpected might happen.

.. include:: ../../commit.rst
