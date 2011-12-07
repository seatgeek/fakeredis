fakeredis: A fake version of a redis-py
=======================================

fakeredis is a pure python implementation of the redis-py python client
that simulates talking to a redis server.  This was created for a single
purpose: **to write unittests**.  Setting up redis is not hard, but
many times you want to write unittests that do not talk to an external server
(such as redis).  This module now allows tests to simply use this
module as a reasonable substitute for redis.


How to Use
==========

The intent is for fakeredis to act as though you're talking to a real
redis server.  It does this by storing state in the redis client itself.
For example::

  >>> import fakeredis
  >>> r = fakeredis.FakeRedis()
  >>> r.set('foo', 'bar')
  True
  >>> r.get('foo')
  'bar'
  >>> r.lpush('bar', 1)
  1
  >>> r.lpush('bar', 2)
  2
  >>> r.lrange('bar', 0, -1)
  [2, 1]


Supported Commands
==================

All of the hashes, lists, sets, and sorted sets commands are implemented
in fakeredis.  There's also support for the ``keys, set, get`` commands.

Adding New Commands
===================

Adding support for more redis commands is easy:

* Add unittests for the new command.
* Implement new command.

To ensure parity with the real redis, there are a set of integration tests
that mirror the unittests.  For every unittest that is written, the same
test is run against a real redis instance using a real redis-py client
instance.  In order to run these tests you must have a redis server running
on localhost, port 6379 (the default settings).  The integration tests use
db=10 in order to minimize collisions with an existing redis instance.

Running the Tests
=================

To run all the tests, install the requirements file::

    pip install -r requirements.txt

If you just want to run the unittests::

    nosetests test_fakeredis.py:TestFakeRedis

Because this module is attempting to provide the same interface as the python
bindings to redis, a reasonable way to test this to to take each unittest and
run it against a real redis server.  fakeredis and the real redis server should
give the same result.  This ensures parity between the two.
You can run these "integration" tests like this::

    nosetests test_fakeredis.py:TestRealRedis

In terms of implementation, ``TestRealRedis`` is a subclass of
``TestFakeRedis`` that overrides a factory method to create
an instance of ``redis.Redis`` instead of ``fakeredis.FakeRedis``,
which is an actual python client for redis.

To run both the unittests and the "integration" tests, run::

    nosetests