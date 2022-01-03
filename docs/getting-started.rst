===============
Getting Started
===============

Installation
============

Using `pip`:

.. code-block:: text

   $ pip install pynetbox


Alternatively, you can clone the repo and run:

.. code-block:: text

   $ python setup.py install

Usage
=====

The pynetbox API is setup so that NetBox's apps are attributes of the
:py:class:`.Api` object, and in turn those apps have attribute representing
each endpoint (see :py:class:`.Endpoint`). Each endpoint has a handful of
methods available to carry out actions on the endpoint (e.g.
:py:meth:`.Endpoint.get`, :py:meth:`.Endpoint.all`,
:py:meth:`.Endpoint.filter`).

For example, in order to query all the objects in the `devices` endpoint you
would do the following:

.. code-block:: python

   import pynetbox
   nb = pynetbox.api(
       'http://localhost:8000',
       private_key_file='/path/to/private-key.pem',
       token='d6f4e314a5b5fefd164995169f28ae32d987704f'
   )

   devices = nb.dcim.devices.all()
   for device in devices:
       print(device.name)
