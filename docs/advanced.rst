==============
Advanced usage
==============

Custom Sessions
===============

Custom sessions can be used to modify the default HTTP behavior. Below are a few examples, most of them from `here <https://hodovi.ch/blog/advanced-usage-python-requests-timeouts-retries-hooks/>`_.

Headers
*******

To set a custom header on all requests. These headers are automatically merged with headers pynetbox sets itself.

:Example:

>>> import pynetbox
>>> import requests
>>> session = requests.Session()
>>> session.headers = {"mycustomheader": "test"}
>>> nb = pynetbox.api(
...     'http://localhost:8000',
...     private_key_file='/path/to/private-key.pem',
...     token='d6f4e314a5b5fefd164995169f28ae32d987704f'
... )
>>> nb.http_session = session


SSL Verification
****************

To disable SSL verification. See `the docs <https://requests.readthedocs.io/en/stable/user/advanced/#ssl-cert-verification>`_.

:Example:

>>> import pynetbox
>>> import requests
>>> session = requests.Session()
>>> session.verify = False
>>> nb = pynetbox.api(
...     'http://localhost:8000',
...     private_key_file='/path/to/private-key.pem',
...     token='d6f4e314a5b5fefd164995169f28ae32d987704f'
... )
>>> nb.http_session = session


Timeouts
********

Setting timeouts requires the use of Adapters.

:Example:

.. code-block:: python

    from requests.adapters import HTTPAdapter

    class TimeoutHTTPAdapter(HTTPAdapter):
        def __init__(self, *args, **kwargs):
            self.timeout = kwargs.get("timeout", 5)
            super().__init__(*args, **kwargs)

        def send(self, request, **kwargs):
            kwargs['timeout'] = self.timeout
            return super().send(request, **kwargs)

    adapter = TimeoutHTTPAdapter()
    session = requests.Session()
    session.mount("http://", adapter)
    session.mount("https://", adapter)

    nb = pynetbox.api(
        'http://localhost:8000',
        private_key_file='/path/to/private-key.pem',
        token='d6f4e314a5b5fefd164995169f28ae32d987704f'
    )
    nb.http_session = session


Threading
=========

pynetbox supports multithreaded calls (in Python 3 only) for
:py:meth:`.Endpoint.filter()` and :py:meth:`..all()` queries. It is **highly
recommended** you have `MAX_PAGE_SIZE` in your Netbox install set to anything
*except* `0` or `None`. The default value of `1000` is usually a good value to
use. To enable threading, add `threading=True` parameter to the
:py:class:`.Api`:

.. code-block:: python

   nb = pynetbox.api('http://localhost:8000', threading=True)


.. _registering-models:

Registering models
==================

When working with plugins, it can be useful to create custom models for data
returned by the plugin’s API (for instance, to add a :py:class:`DetailEndpoint
<.Endpoint.DetailEndpoint>` attribute).

To do so, you need to create a module containing classes named after the plugin
endpoints.

Here is an example using `netbox-dns
<https://github.com/auroraresearchlab/netbox-dns>`_:

.. code-block:: python

   # models.py
   from pynetbox.core.response import Record
   from pynetbox.core.endpoint import RODetailEndpoint


   class Zones(Record):
       @property
       def records(self):
           return RODetailEndpoint(self, "records", Record)

   # cli.py
   import pynetbox

   from . import models

   pynetbox.register_models("plugins/netbox-dns", models)

   nb = pynetbox.api(...)

   for zone in nb.plugins.netbox_dns.zones.all():
       for rr in zone.records.list():
           print(rr)
