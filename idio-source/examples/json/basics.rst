.. include:: ../../global.rst

##########
Basic JSON
##########

JSON(5) data can be a few constants, number, string, array or objects.
Commonly data is transferred as an object, a natural counterpart to an
:lname:`Idio` :ref:`hash table <ref:hash table type>`.

The :ref:`JSON5 module <ref:json5 module>` is not loaded by default so
you'll need to :ref:`import <ref:import>` it.

Let's suppose we have a nominal sliver of JSON_ in a file:

.. code-block:: text
   :caption: :file:`foo.json`

   {
     "key": "value",
     "array": [
       "of",
       4,
       true,
       "things"
     ]
   }

Reading
=======

The :ref:`parse-file <ref:json5/parse-file>` function is what we need,
here:

.. code-block:: idio

   import json5

   ht := parse-file "foo.json"

   hash-keys ht			; ("key" "array")

   ht."key"			; "value"

   ht."array"			; #[ "of" 4 #t "things" ]

You'll notice from the ``"array"`` element that JSON's ``true`` has
become :lname:`Idio`'s ``#t``.

If we already has a string, then we can call :ref:`parse-string
<ref:json5/parse-string>`.  Here, we'll cheat by collecting the output
from :program:`cat`'ing the file:

.. code-block:: idio

   import json5

   str := (collect-output cat "foo.json")
   
   ht := parse-string str

   hash-keys ht			; ("key" "array")

If you look at :var:`str`, you'll see it has all of the original
newlines embedded.  It's just a string, complicated by all the
necessary escaping of double-quotes:

   ``"{\n  \"key\": \"value\",\n  \"array\": [\n    \"of\",\n    4,\n    true,\n    \"things\"\n  ]\n}"``

Generating
==========

Generating JSON(5) is easy enough noting that the :ref:`generate
<ref:json5/generate>` function (and :ref:`generate-json
<ref:json5/generate-json>` function) create a string, you need to send
it somewhere.

If we continue the previous example and change a couple of things:

.. code-block:: idio

   ht."key" = "new value"
   ht."array".1 = 3
   ht."array".2 = #f

   printf "%s\n" (generate ht)

should print out:

.. code-block:: text

   {
     "array": [
	 "of",
	 3,
	 false,
	 "things"
       ],
     "key": "new value"
   }

Note the sequence, here.  There is no implied ordering of members
within an object.

.. include:: ../../commit.rst
