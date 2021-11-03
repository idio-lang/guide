.. include:: ../../global.rst

####
Case
####

The :ref:`case <ref:case>` expression is similar to :lname:`C`'s
``switch`` statement except it allows a wider variety of values to
test against.  They key point being that ``case`` uses specific value
comparisons (rather than generic conditional tests like :ref:`cond
<ref:cond special form>`).

In :lname:`C` you can test against, essentially, an integer, although
commonly ``#define``'d as a macro.

In :lname:`Idio` you can test against anything for which :ref:`eqv?
<ref:eqv?>` can return `true` or `false`.  This extends the testable
values from simple integers to numbers, unicode values, strings,
symbols.

``case`` takes a key and a number of clauses where each clause takes
the form :samp:`(({v} ...) {expr} ...)` except the last clause which
can take the form :samp:`(else {expr} ...)`.

The key is evaluated and then compared with :ref:`memv <ref:memv>`
against the list in the head of each clause.

If the comparison is `true` then the result of ``case`` is the result
of evaluating the expressions in the body of the clause.  No other
clauses are considered.

If no comparison is `true` and there is an ``else`` clause then it is
run otherwise the result is ``#<void>``.

.. warning::

   There is no evaluation of the :samp:`({v} ...)` so the individual
   :samp:`{v}` must be values that the *reader* can construct.  Hence,
   numbers, constants, strings, symbols.

.. note::

   These examples put parenthesis around the whole ``case``
   expression.  That **doesn't** change the way it is interpreted, any
   expression with multiple elements is treated as a list by the
   reader.

   Here, it probably helps your ``$EDITOR`` indent the text properly
   with clauses on separate lines!

Numeric Example
===============

Let's try making some critical distinctions between integers:

.. code-block:: idio

   v := ...

   (case v
         ((1 3 5) {
	   printf "a small odd integer\n"
	 })
         ((6 8 10) {
	   printf "a large even integer\n"
	 })
         (else {
	   printf "uncategorized!\n"
	 }))

which might be better re-written as:

.. code-block:: idio

   printf "%s\n" (case v
		  ((1 3 5) {
		    "a small odd integer"
		  })
		  ((6 8 10) {
		    "a large even integer"
		  })
		  (else {
		    "uncategorized!"
		  }))

demonstrating that ``case`` is returning a (string) value which
:ref:`printf <ref:printf>` is happily printing.

Symbolic Example
================

Let's try to distinguish between various foodstuffs:

.. code-block:: idio

   v := ...

   (case v
         ((apple banana) {
	   'fruit
	 })
         ((carrot) {
	   'vegetable
	 })
         ((tomato) {
	   error/type ^rt-botanic-or-culinary-error 'my-func "who is asking?" v
	 })
         (else {
	   'uncategorized
	 }))

Here we can return (from ``case``) a symbol based on our comprehensive
test.  It appears to be thrown away as no-one is capturing the result
of ``case`` but never mind.

We can also throw our hands up in horror when faced with the tricky
decision for a tomato as to whether we're in a botanic or culinary
context.

.. hint::

   ``^rt-botanic-or-culinary-error`` isn't a real error in
   :lname:`Idio`.

.. include:: ../../commit.rst
