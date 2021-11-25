.. include:: ../../global.rst

###############
ELK Interaction
###############

We're System Administrators and we want to interrogate our local
:strike:`ELK` :strike:`ElasticSearch` ElasticStack instance.

:lname:`Idio` doesn't have any immediate HTTP capabilities so we'll
use `curl <https://curl.se/>`_.

This example will use non-TLS connections.

We're going to use using the JSON output so remember to put ``import
json5`` at the top of the script (or type it in at the console).

Elastic Basic Security
======================

A little digression.  Later releases of v7 have encouraged even the
free tier to use authentication to connect.  That's good but we don't
want to use a :samp:`http://{user}:{pass}@{host}:9200/{path}` format
because we can't guarantee that :program:`curl` can successfully
prevent other users from seeing :samp:`{user}` and :samp:`{pass}`.

:program:`curl` does support the use of both ``--netrc`` and, more
usefully, ``--netrc-file`` for which we can make use of
*Process Substitution*.

Our command line will therefore include:

.. code-block:: idio

   curl --netrc --netrc-file (named-pipe-into print-netrc) ...

where the function ``print-netrc`` can be set up along the lines of:

.. code-block:: idio

   ELK-HOST := "elk1.example.com"
   ELK-USER := "elk-username"
   ELK-PASS := "elk-password"

   define* (print-netrc (:host host ELK-HOST)
			(:user user ELK-USER)
			(:pass pass ELK-PASS)) {
     printf #S{
   machine ${host}
   login ${user}
   password ${pass}
     }
   }

which gives us a bit of wiggle-room to alter the ``host``, ``user``
and ``pass`` by passing optional parameters (see :ref:`define*
<ref:define*>`).

Calling curl
============

We can create a similar function for parameterising the call the curl:

.. code-block:: idio

   define* (CURL
	    (res "")
	    (:method method "GET")
	    (:host host ELK-HOST)
	    (:user user ELK-USER)
	    (:pass pass ELK-PASS)) {

     (collect-output curl
		     -sX method
		     --netrc
		     --netrc-file (named-pipe-from print-netrc
						   :host host
						   :user user
						   :pass pass)
		     #S{http://${ELK-HOST}:9200/${res}})

   }

ELK APIs
========

Looking at some simple APIs, we can check we successfully talk to ELK
at all by invoking ``CURL`` with no :var:`res` option:

.. code-block:: idio

   printf "%s\n" (CURL)

to get something like:

.. code-block:: text

   {
     "name" : "elk2.example.com",
     ...
     "version" : {
       "number" : "7.15.1",
       ...
     },
     "tagline" : "You Know, for Search"
   }

_cat/nodes
----------

.. aside::

   The "cat nodes" API is "compact and aligned text", deliberately
   designed for humans.  We should be using the `nodes API
   <https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-info.html>`_
   but that gives us thousands of lines of JSON which distracts from
   the idea of handling data interchange.

The `cat nodes API
<https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-nodes.html>`_
can give us some JSON output:

.. code-block:: idio

   ;; nodes is an array
   nodes := parse-string (CURL "_cat/nodes?format=json")

``parse-string``, here, is ``json5/parse-string`` from the ``import
json5`` line you added.

:var:`nodes` is an array of as many objects as you have ElasticSearch
nodes in your cluster.

For interest, we can get the member names of the first object:

.. code-block:: idio

   hash-keys (array-ref nodes 0)

which will result in a list, something like:

.. code-block:: text

   ("ip" "master" "load_15m" "load_5m" "heap.percent" "cpu" "node.role" "name" "load_1m" "ram.percent")

based on which, we should be able to iterate over the array printing
out the name, IP address and cpu usage:

.. code-block:: idio

   for-each (function (n) {
	       printf "%-30s %15s %s\n" n."name" n."ip" n."cpu"
   }) nodes

to get:

.. code-block:: text

   elk1.example.com                 192.0.2.101  8
   elk2.example.com                 192.0.2.102  17


.. include:: ../../commit.rst
