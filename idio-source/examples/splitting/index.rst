.. include:: ../../global.rst

#########
Splitting
#########

You can split strings.

Here we might write a function to get the first letters of each word:

.. code-block:: idio

   str := "hello world"

   define (foo s) {
     map (function (w) {
	    w.0
     }) s.split-string
   }

   printf "%s\n" str.foo		; (#\h #\w)
   printf "%s\n" str.foo.2		; w

Here, there is a cascade of implied calls:

* ``str.foo`` is ``str`` being "indexed" by ``foo`` which is a
  function and therefore is actually ``foo str``

* ``foo`` calls :ref:`map <ref:map>` with:

  * ``s.split-string`` which is ``s`` being "indexed" by
    :ref:`split-string <ref:split-string>` and is therefore
    ``split-string s``

    * ``split-string`` uses :ref:`IFS <ref:IFS>` which defaults to ``" \t\n"``

      that splits the original string into words

  * the anonymous function given to ``map`` returns ``w.0`` which is
    ``w`` indexed by ``0``

    Here, ``w`` is a string, each word from ``str.split-string``, and
    we are returning the first code point from that word

.. note::

   The ``.2`` in the ``str.foo.2`` is accessing the second element of
   a list.  Strings and array are indexed from 0 (zero) but lists from
   their first element.



.. include:: ../../commit.rst
