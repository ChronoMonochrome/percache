===============================================================================
percache
===============================================================================

*percache* is a Python module to persistently cache results of functions (or
callables in general) using decorators.

It is somehow similar to the `Memoize Example`_ from the Python Decorator
Library but with the advantage that results are stored *persistently* in a
cache. *percache* provides memoization across multiple invocations of the
Python interpreter.

Install with ``pip install percache``. *percache* works with Python 2.6, 2.7,
and 3.3 and has no dependencies outside the standard library.

.. _Memoize Example: http://wiki.python.org/moin/PythonDecoratorLibrary#Memoize

.. image:: https://travis-ci.org/obensonne/percache.png?branch=master
   :target: https://travis-ci.org/obensonne/percache

-------------------------------------------------------------------------------
Example
-------------------------------------------------------------------------------

::

    >>> import percache
    >>> @percache.Cache()
    ... def longtask(a, b):
    ...     print("running a long task")
    ...     return a + b
    ...
    >>> longtask(1, 2)
    running a long task
    3
    >>>
    >>> longtask(1, 2)
    3

As you can see at the missing output after the second invocation, ``longtask``
has been called once only. The second time the result is retrieved from the
cache.  *The key feature of this module is that this works across multiple
invocations of the Python interpreter.*

A requirement on the results to cache is that they are `pickable`_.

.. _pickable: http://docs.python.org/library/pickle.html#what-can-be-pickled-and-unpickled

Each cache file can be used for any number of differently named callables.

Alternative back-ends and live synchronization
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default *percache* uses a `shelve`_ as its cache back-end. Alternative
back-ends may be used if they are given as dictionary-like objects with a
``close()`` and ``sync()`` method::

    >>> class FooCache(dict):
    ...     def sync(self):
    ...         ...
    ...     def close(self):
    ...         ...
    >>> fc = FooCache()
    >>> cache = percache.Cache(fc, livesync=True)

In this example a cache is created in live-sync mode, i.e. results
*immediately* are stored permanently. Normally this happens not until a cache's
``close()`` method has been called or until it gets `finalized`_. Note that the
live-sync mode may slow down your percache-decorated functions (though it
reduces the risk of "loosing" results).

.. _finalized: http://docs.python.org/reference/datamodel.html#object.__del__

-------------------------------------------------------------------------------
Caching details (you should know)
-------------------------------------------------------------------------------

When caching the result of a callable, a SHA1 hash based on the callable's name
and arguments is used as a name of the cache file. The content of this file is
produced by using function ``dumps()`` of the python marshal module on the
callable's attribute ``__code__`` and hashing the result using SHA1.

-------------------------------------------------------------------------------
Housekeeping
-------------------------------------------------------------------------------

- To prevent the cache from getting larger and larger you can call the
  ``clear()`` method of a ``Cache`` instance. By default it clears *all*
  results from the cache. The keyword ``maxage`` my be used to specify a
  maximum number of seconds passed since a cached result has been *used* the
  last time. Any result not used (written or accessed) for ``maxage`` seconds
  gets removed from the cache.

-------------------------------------------------------------------------------
Changes
-------------------------------------------------------------------------------

Version 0.3.0
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Support Python 3.3 (next to 2.6 and 2.7)

Version 0.2.1
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Add missing README to PyPi package.

Version 0.2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Automatically close (i.e. sync) the cache on finalization.
- Optionally sync the cache on each change.
- Support for alternative back-ends (others than `shelve`_).
- Cache object are callable now, which makes the explicit ``check()`` method
  obsolete (though the old interface is still supported).

.. _shelve: http://docs.python.org/library/shelve.html

Version 0.1.1
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Fix wrong usage age output of command line interface.
- Meet half way with pylint.

Version 0.1
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Initial release

