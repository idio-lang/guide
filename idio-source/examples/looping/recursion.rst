.. include:: ../../global.rst

#########
Recursion
#########

There's a subtle distinction between top-level and in-block function
definitions (due to the way references to the values are made).

Top-Level
=========

Pretty much what you would expect:

.. code-block:: idio

   define (fib n) {
     if (n le 2) 1 {
       (fib (n - 1)) + (fib (n - 2))
     }
   }

In-block
========

Actually, in the first instance, you can just use ``define`` again:

.. code-block:: idio

   {
     define (fib n) {
       if (n le 2) 1 {
         (fib (n - 1)) + (fib (n - 2))
       }
     }

     fib 10
   }

but you may see the ``:+`` operator which is indicating that the
expression to be assigned is going to be recursive:

.. code-block:: idio

   {
     fib :+ function (n) {
       if (n le 2) 1 {
         (fib (n - 1)) + (fib (n - 2))
       }
     }

     fib 10
   }

``:+`` can only be used inside a block.

*Inside* a block, ``define`` and ``:+`` are identical.  Adjacent
``define`` and ``:+`` expressions inside a block are combined into a
single "letrec" expression to allow for mutually recursive functions.

Tail-Call Optimisation
======================

You can make your code more efficient if the recursion is the *last*
expression in a block thus enabling tail-call optimisation.

The Fibonacci expressions are *not* tail call optimised because the
calls to ``fib`` are sub-expressions of the final expression, ``+``.

Here, the use of infix operators is slightly clouding the flow of
control.

A common recursive loop involving the simultaneous construction of
results might look like:

.. code-block:: idio

   define (f l n s) {
     if (null? l) (list (reverse n) (reverse s)) {
       (cond
	((number? (ph l)) {
	  f (pt l) (pair (ph l) n) s
	})
	((string? (ph l)) {
	  f (pt l) n (pair (ph l) s)
	}))
     }
   }

   f '(1 "a" 2 "b") #n #n		; ((1 2) ("a" "b"))

Here, the function ``f`` wants to iterate over the list of items
pulling numbers and strings into separate lists.

Notice, though, that accepts both the list and the putative result
lists for numbers and strings, both of which are initialised to
``#n``.

As it runs over the list, the ``cond`` tests the head of the list as
either a number or a string and then recurses into ``f`` with the tail
of the original list and extending either the list of numbers or the
list of string by the head of the original list.

If you follow this by hand, as you walk over the original list
left-to-right, the "result" lists will be being created in reverse
order, ``(2 1)`` and ``("b" "a")`` as they get pushed onto the result
so far.

If, on entering the loop, the list was ``#n``, then we'll construct a
result (of a list of lists) where each sub-list is the ``reverse`` of
the list accumulated so far.

The calls to ``f`` are the last expression in the ``cond`` clauses and
as the ``cond`` is the only expression in the body of ``f`` then it
too is the last expression and the whole thing becomes a tail-call
optimised loop.

If you wanted to use lots of holding variables to make the expression
clearer then it all still works so long as the call the ``f`` is the
last expression in each ``cond`` clause:

.. code-block:: idio

   define (f l n s) {
     if (null? l) {
       rn := reverse n
       rs := reverse s
       list rn rs
     } {
       hl := ph l
       tl := pt l
       (cond
	((number? hl) {
	  nn := pair hl n
	  f tl nn s
	})
	((string? hl) {
	  ns := pair hl s
	  f tl n ns
	}))
     }
   }

.. include:: ../../commit.rst
