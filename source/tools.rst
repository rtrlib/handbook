.. _tools:

Tools based on the RTRlib
=========================

.. _RIPE RIS Beacons: https://www.ripe.net/analyse/internet-measurements/routing-information-service-ris/current-ris-routing-beacons

In the following sections we give an overview on several software tools, which
utilize the RTRlib and its features.
These tools range from low lever shell commands to easy-to-use browser plugins.

For all tools we provide small usage examples; where ever appropriate we will
use the following `RIPE RIS Beacons`_ with well known RPKI validation results:

================== ============ ==========
IP Prefix          Valid Origin Result
================== ============ ==========
93.175.146.0/24    AS12654      valid
2001:7fb:fd02::/48 AS12654      valid
93.175.147.0/24    AS196615     invalid AS
2001:7fb:fd03::/48 AS196615     invalid AS
84.205.83.0/24     None         not found
2001:7fb:ff03::/48 None         not found
================== ============ ==========

*Note* all prefixes are validated against origin AS12654, owned by RIPE.

RTRlib Client
-------------

The RTRlib client (``rtrclient``) is part of the RTRlib package,
By following the installation instruction given in the previous section
(:ref:`install`) it will be installed automatically.

RTRlib Validator
-----------------------

The RTRlib command line validator (``cli-validator``) is also part of RTRlib
package and is already installed.
This tool provides a simple command line interface to validate IP prefix to
origin AS relations against ROAs in an RPKI cache.
To execute the program you must provide parameters ``hostname`` and ``port`` of
a known RPKI cache server, afterwards you can validate  IP prefixes by typing
``prefix``, ``prefix length``, and ``origin ASN`` separated by spaces. Press
``ENTER`` to run the validation, the result will be shown instantly below the
input.
The following listing shows the validation of all known RIPE RIS beacons using
our cache server:

.. code-block:: Bash

    cli-validator rpki-validator.realmv6.org 8282
    93.175.146.0 24 12654
    93.175.146.0 24 12654|12654 93.175.146.0 24 24|0
    2001:7fb:fd02:: 48 12654
    2001:7fb:fd02:: 48 12654|12654 2001:7fb:fd02:: 48 48|0
    93.175.147.0 24 12654
    93.175.147.0 24 12654|196615 93.175.147.0 24 24|2
    2001:7fb:fd03:: 48 12654
    2001:7fb:fd03:: 48 12654|196615 2001:7fb:fd03:: 48 48|2
    84.205.83.0 24 12654
    84.205.83.0 24 12654||1
    2001:7fb:ff03:: 48 12654
    2001:7fb:ff03:: 48 12654||1

The output is structured into ``input query | ROAs | result``, separated by pipe (``|``).
The validation results are ``0`` for *valid*, ``1`` for *not found*,
and ``2`` for *invalid*.
For *valid* and *invalid* the output also shows the matching or conflicting ROAs
for the given prefix and AS number; if multiple ROAs for a prefix exist they
are all listed separated by commas (``,``).

Firefox Browser Plugin
----------------------

.. |valid| image:: ../images/valid.png

.. |invalid| image:: ../images/invalid.png

.. |not_found| image:: ../images/notFound.png


To check prefixes while browsing the internet there is a validation tool for Firefox and Chrome. The Firefox add-on
`RPKI-Validator <https://addons.mozilla.org/en-US/firefox/addon/rpki-validator/>`_
shows the user whether the currently visited website has a valid prefix. A small icon indicates the status, which is either
valid (|valid|), invalid (|invalid|) or was not found (|not_found|).

Chrome user can profit from the extension as well, but they will have to install it by hand.
First download the `Chrome extension <https://github.com/rtrlib/chrome-extension>`_ from GitHub. Now open a new tab in
the browser and enter ``chrome://extensions``. Activate the checkbox that says `Developer Mode` in the top right corner.
Now click the `Load unpacked extension` button and navigate to the source directory of the downloaded files.
The package is now installed and can be used just like the Firefox add-on.

RPKI READ
---------

The *RPKI Realtime Dashboard* (READ)

RPKI MIRO
---------

.. _RPKI MIRO: http://rpki-miro.realmv6.org/

The RPKI *Monitoring and Inspection of RPKI Objects* (`RPKI MIRO`_)
aims for easy access to RPKI certificates, revocation lists, ROAs etc.
to finally give Internet operators more confidence in their data.
Though, RPKI is a powerful tool, its success depends on several aspects.
One crucial piece is the correctness of the RPKI data.
RPKI data is public but might be hard to inspect outside of shell-like environments.

The main objective of RPKI MIRO is to provide an extensive but painless insight
into the published RPKI content.
RPKI MIRO is a monitoring application that consists of three parts:

#. standard functions to collect RPKI data from remote repositories,
#. a browser to visualize RPKI objects, and
#. statistical analysis of the collected objects.

.. image:: ../images/rpki_miro.png

Using RPKI MIRO you can lookup any IP prefix and its associated ROA, e.g. the
RIPE RIS beacon ``93.175.147.0/24``.
Open a browser and goto URL http://rpki-browser.realmv6.org, in the menu switch
from ``AFRINIC`` to ``RIPE`` and set a filter for the prefix ``93.175.147.0/24``
with attribute ``resource``.
Expand the ROA tree view on the left side to get the corresponding ROA for the
beacon prefix, the resulting web view should look like the screen shot above.

RPKI RBV
--------

The RPKI *RESTful BGP Validator* (RBV) is web application that provides a RESTful
API to validate the BGP origin AS of a given IP prefix.
The validation service can be accessed via `web page <http://rpki-validator.realmv6.org/html/validate.html>`_.
or directly using the RESTful API.

.. image:: ../images/rpki_rbv.png

RBV has two distinct APIs
offers the ability to host BGP validation on your own.
An example of how it could look like can be found `here <http://rpki-validator.realmv6.org/html/validate.html>`_.
If a query like the one in the picture is sent, the result will be a JSON Object like this one

.. code-block:: JSON

  { "asn": "12654", "cache_server": "rpki-validator.realmv6.org:8282", "code": 1, "message": "Valid", "prefix": "93.175.146.0/24" }

To speak to the API via GET, send a request in the format

| your.webserver.net/api/v1/validity/<asn>/<prefix>/<masklen>[?params]
|

Note that the <asn> must be prepended by `AS`, e.g.,

| rpki-rbv.realmv6.org/api/v1/validity/AS12654/93.175.146.0/24
|

This time the resulting JSON will be a bit more detailed

.. code-block:: JSON

  {"validated_route": {"info": {"origin_country": "EU", "origin_asname": "RIPE-NCC-RIS-AS Reseaux IP Europeens Network Coordination Centre (RIPE NCC), EU"}, "route": {"prefix": "93.175.146.0/24", "origin_asn": "AS12654"}, "validity": {"state": "Valid", "code": 0, "description": "At least one VRP Matches the Route Prefix", "VRPs": {"unmatched_as": [], "unmatched_length": [], "matched": [{"prefix": "93.175.146.0/24", "max_length": "24", "asn": "AS12654"}]}}}}

For a detailed instruction how to install and set up the API visit the `RBV Repository <https://github.com/rtrlib/rbv>`_ on GitHub.

RTRlib Python Binding
---------------------

Other Third-Party Tools
-----------------------

- There is a nice overview at `RIPE <https://www.ripe.net/manage-ips-and-asns/resource-management/certification/tools-and-resources/>`_.
- Note, `rtr-origin <http://subvert-rpki.hactrn.net/trunk/rtr-origin/>`_ includes also a client. It is written in Python.
