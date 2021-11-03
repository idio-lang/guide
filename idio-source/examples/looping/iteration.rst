.. include:: ../../global.rst

#########
Iteration
#########

There's a few (but not many) ways to iterate over some kinds of
objects.

There is the familiar `map`_ and its sibling `for-each`_ which works
identically but doesn't collect the per-item result and return it as a
list.

You can call `Folding Functions`_.

Basic Iteration
===============

map
---

:ref:`map <ref:map>` takes a function and a number of lists and will
return a list of values of applying the function to the iterated
elements of the lists.  The function should take as many arguments as
there are lists and the lists should all be the same length.

Single List
^^^^^^^^^^^

Commonly, you iterate over a single list meaning the function should
take a single argument.

The function is usually an anonymous function as its behaviour is
bespoke.  Here, multiply the elements by 2:

.. code-block:: idio

   map (function (v) {
     v * 2
   }) '(1 2 3)			; '(2 4 6)

although a named function is perfectly fine:

.. code-block:: idio

   define (*2 v) (* 2 v)

   map *2 '(1 2 3)		; '(2 4 6)

Multiple Lists
^^^^^^^^^^^^^^

As many arguments as there are lists!

What is the product of the list elements:

.. code-block:: idio

   map (function (v1 v2) {
     v1 * v2
   }) '(1 2 3) '(4 5 6)		; '(4 10 18)

for-each
--------

:ref:`for-each <ref:for-each>` is identical to :ref:`map <ref:map>`
but doesn't collect the function's results.  You would use it for its
side-effects:

.. code-block:: idio-console

   Idio> for-each (function (v) {
     printf "v=%s\n" v
   }) '(1 2 3)
   v=1
   v=2
   v=3
   #<void>

Noting that ``for-each`` returns ``#<void>``.

Folding Functions
=================

You can also call :ref:`fold-left <ref:fold-left>` and
:ref:`fold-right <ref:fold-right>` which accumulate a value rather
than collect a value per item.  They differ by whether the list is
iterated over left-to-right or right-to-left.

You call the *folding function* (``fold-left`` or ``fold-right``)
with:

* a function which should accept the accumulated value (as the first
  argument) plus as many parameters as there are lists

  The value returned by the function is the accumulated value for the
  next iteration.

* an initial value for the accumulated value

* the lists to be iterated over

The accumulated value is likely to be a counter or a (to be
constructed) list.

Single List
-----------

Counter
^^^^^^^

We could count the number of odd numbers in a list:

.. code-block:: idio

   fold-left (function (c v) {
     if (odd? v) (c + 1) c
   }) 0 '(1 2 3 4 5)			; 3

Here, if the list's value, :var:`v`, is odd then return the current
accumulated value plus 1 otherwise return the accumulated value.

List Generator
^^^^^^^^^^^^^^

We could improve our odd counter by accumulating the set of odd
numbers -- which we can call :ref:`length <ref:length>` on later:

.. code-block:: idio

   fold-left (function (r v) {
     if (odd? v) (pair v r) r
   }) #n '(1 2 3 4 5)			; '(5 3 1)

Notice the result list is in reverse order.  If you care, you can call
:ref:`reverse <ref:reverse>` on it, of course.

Multiple Lists
--------------

Counter
^^^^^^^

Let's count up the number of instances where the product of the two
lists is greater than 5:

.. code-block:: idio

   fold-left (function (c v1 v2) {
     if ((v1 * v2) gt 5) (c + 1) c
   }) 0 '(1 2 3) '(4 5 6)		; 2

(I'm sure that's a useful example to someone!)

.. include:: ../../commit.rst
