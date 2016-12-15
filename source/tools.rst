.. _tools:

*************************
Tools based on the RTRlib
*************************

.. _RIPE RIS Beacons: https://www.ripe.net/analyse/internet-measurements/routing-information-service-ris/current-ris-routing-beacons

In the following sections we give an overview on several software tools, which
utilize the RTRlib and its features.
These tools range from low level shell commands to easy-to-use browser plugins.
For all tools we provide small usage examples; where ever appropriate we will
use the `RIPE RIS Beacons`_ (see :numref:`beacons`) with well known RPKI
validation results.

.. _beacons:
.. table:: RIPE RIS beacons for RPKI tests

    ==================== ============== ============
     IP Prefix            Valid Origin   Result
    ==================== ============== ============
    93.175.146.0/24       AS12654        valid
    2001:7fb:fd02::/48    AS12654        valid
    93.175.147.0/24       AS196615       invalid AS
    2001:7fb:fd03::/48    AS196615       invalid AS
    84.205.83.0/24        None           not found
    2001:7fb:ff03::/48    None           not found
    ==================== ============== ============

*Note*: for all prefixes the RPKI validation results are based on
origin AS 12654 that is owned by RIPE.
Most examples require a connection to a RPKI cache server, for that we
provide a public cache with *hostname* ``rpki-validator.realmv6.org``
and *port* ``8282``.


.. _rtrclient:

RTRlib Client
=============

The RTRlib client (``rtrclient``) is a default part of the RTRlib package.
It emulates an RPKI enabled BGP router by using  the client side functionality
of the RTR protocol to connect to a trusted RPKI cache server and receive all
currently valid ROAs.
By following the instruction given in the previous section (:ref:`install`)
it will be installed automatically.

To establish a connection with a RPKI cache server the client can use *TCP* or
*SSH* transport sockets.
It then communicates with the cache server utilizing the RTR protocol provided
by the RTRlib to receive all cryptographically verified ROAs from the cache.
To run the programm you have to specify the transport protocol as well as the
hostname and port of a RPKI cache server; additionally you can set several
options.
To get a complete reference over all options for the command simply run
``rtrclient`` in a shell.

:numref:`lst-rtrclient` shows how to connect the ``rtrclient`` with a cache
server as well as 10 lines of the resulting output.
The listing shows ROAs for IPv4 and IPv6 prefixes with allowed range for prefix
lengths and the associated origin AS number.
Each line represents either a ROA that was added (``+``) or removed (``-``)
from the selected RPKI cache server.
The RTRlib client will receive and print such updates until the program is
terminated, i.e., by ``CTRL+C``.

.. code-block:: Bash
    :caption: Output of the rtrclient tool
    :name: lst-rtrclient

    rtrclient tcp -k -p rpki-validator.realmv6.org 8282
    Prefix                                     Prefix Length         ASN
    + 89.185.224.0                                19 -  19        24971
    + 180.234.81.0                                24 -  24        45951
    + 37.32.128.0                                 17 -  17       197121
    + 161.234.0.0                                 16 -  24         6306
    + 85.187.243.0                                24 -  24        29694
    + 2a02:5d8::                                  32 -  32         8596
    + 2a03:2260::                                 30 -  30       201701
    + 2001:13c7:6f08::                            48 -  48        27814
    + 2a07:7cc3::                                 32 -  32        61232
    + 2a05:b480:fc00::                            48 -  48        39126


.. _validator:

RTRlib Validator
================

The RTRlib command line validator (``cli-validator``) is also part of RTRlib
package and is already installed.
This tool provides a simple command line interface to validate IP prefix to
origin AS relations against ROAs received from a trusted RPKI cache server.

To execute the program you must provide parameters ``hostname`` and ``port`` of
a known RPKI cache server, afterwards you can validate IP prefixes by typing
``prefix``, ``prefix length``, and ``origin ASN`` separated by spaces. Press
``ENTER`` to run the validation.
The result will be shown instantly below the input.
*Note*: the ``cli-validator`` can validate IPv4 and IPv6 prefixes by default.

:numref:`lst-validator` shows the validation of all RIPE RIS beacons using
our RPKI cache server instance.
The output is structured into ``input query | ROAs | result``, separated by
pipe (``|``) symbols.
The validation results are ``0`` for *valid*, ``1`` for *not found*,
and ``2`` for *invalid*.
For *valid* and *invalid* the output shows the matching or conflicting
ROAs for the given prefix and AS number.
If multiple ROAs exist for a prefix, they are listed successively separated
by commas (``,``).

.. code-block:: Bash
    :caption: Output of the cli-validator tool
    :name: lst-validator

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



RPKI Validator Browser Plugin
=============================

The RPKI Validator plugin for web browsers allows to check the RPKI validation
of visited URLs, i.e., the associated IP prefix and origin AS of the URL.
A small icon indicates the validation state of the visited URL, which is
either valid (|valid|), invalid (|invalid|) or was not found (|not_found|).

The plugin is available as an add-on (or extension) for the web browsers
Firefox and Chrome .
While the `Firefox add-on`_ is available through the add-on store, Chrome users
have to download and install the extension themselves as follows:

#. download the `Chrome extension <https://github.com/rtrlib/chrome-extension>`_ from GitHub
#. open a new tab in Chrome and enter ``chrome://extensions``
#. activate `Developer Mode` via the checkbox in the top right
#. click the `Load unpacked extension` button and navigate to the source

The screenshots show the results of the RPKI Validator browser plugin for
Firefox (*valid* :numref:`fig-valid`, *invalid* :numref:`fig-invalid`,
and *not found* :numref:`fig-notfound`) for certain websites .

.. _fig-valid:
.. figure:: ../images/rbv_valid.png
    :width: 90 %
    :align: center

    Screenshot of RPKI Validator plugin in Firefox showing result *valid*.

.. _fig-invalid:
.. figure:: ../images/rbv_invalid.png
    :width: 90 %
    :align: center

    Screenshot of RPKI Validator plugin in Firefox showing result *invalid*.

.. _fig-notfound:
.. figure:: ../images/rbv_notfound.png
    :width: 90 %
    :align: center

    Screenshot of RPKI Validator plugin in Firefox showing result *not found*.


.. |valid| image:: ../images/valid.png
.. |invalid| image:: ../images/invalid.png
.. |not_found| image:: ../images/notFound.png

.. _Firefox add-on: https://addons.mozilla.org/en-US/firefox/addon/rpki-validator/
.. _Chrome: https://github.com/rtrlib/chrome-extension

RPKI READ
=========

The *RPKI Realtime Dashboard* (`RPKI READ`_) aims to provide a consistent
(and live) view on the RPKI validation state of currently announced IP prefixes.
That is, it verifies relation of an IP prefix and its BGP origin AS
(autonomous system) utilizing the RPKI.

The RPKI READ monitoring system has two parts:

#. the backend storing latest validation results in a database, and
#. the (web) frontend displaying these results as well as an overview of statistics derived from them.

The backend connects to a live BGP stream, e.g. of a BGPmon_ instance or via
BGPstream_.
It then parses  received BGP messages and extracts IP prefixes and origin AS
information.
These prefix to origin AS relations are validated using the RTRlib client
to query a trusted RPKI cache server.

The RPKI READ frontend presents a dashboard like interface showing a live
overview of the RPKI validation state of all currently advertised IP prefixes
observed by a certain BGP source (see :numref:`fig-read`).
Further, the frontend provides detailed statistics and also allows the user
to search for validation results of distinct prefixes or all prefixes originated
by a certain AS.

.. _fig-read:
.. figure:: ../images/rpki_read.png
   :alt: RPKI READ screenshot
   :width: 90 %
   :align: center

   Screenshot of the RPKI READ web frontend

.. _RPKI READ: https://rpki-read.realmv6.org/
.. _BGPmon: http://www.bgpmon.io/
.. _BGPstream: https://bgpstream.caida.org/

RPKI MIRO
=========

The RPKI *Monitoring and Inspection of RPKI Objects* (`RPKI MIRO`_)
aims for easy access to RPKI certificates, revocation lists, ROAs etc.
to give network operators more confidence in their data.
Though, RPKI is a powerful tool, its success depends on several aspects.
One crucial piece is the correctness of the RPKI data.
RPKI data is public but might be hard to inspect outside of shell-like
environments.

The main objective of RPKI MIRO is to provide an extensive but painless
insight into the published RPKI content.
RPKI MIRO is a monitoring application that consists of three parts:

#. standard functions to collect RPKI data from remote repositories,
#. a browser to visualize RPKI objects, and
#. statistical analysis of the collected objects.

.. _fig-miro:
.. figure:: ../images/rpki_miro.png
   :alt: RPKI MIRO screenshot
   :width: 90 %
   :align: center

   Screenshot of the RPKI MIRO web interface.

Using RPKI MIRO you can lookup any IP prefix and its associated ROA, e.g. the
RIPE RIS beacon ``93.175.147.0/24``.
Open a browser and goto URL http://rpki-browser.realmv6.org, in the menu switch
from ``AFRINIC`` to ``RIPE`` and set a filter for the prefix ``93.175.147.0/24``
with attribute ``resource``.
Expand the ROA tree view on the left side to get the corresponding ROA for the
beacon prefix, the resulting web view should look like the screenshot
in :numref:`fig-miro`.

.. _RPKI MIRO: http://rpki-miro.realmv6.org/

RPKI RBV
========

The RPKI *RESTful BGP Validator* (`RPKI RBV`_) is web application that provides
a RESTful API to validate IP prefix to origin AS relations.
The validation service can be accessed via a plain and simple
`web page <http://rpki-rbv.realmv6.org/html/validate.html>`_
(see also :numref:`fig-rbv`) or directly using its RESTful API.

.. _fig-rbv:
.. figure:: ../images/rpki_rbv.png
   :alt: RPKI RBV screenshot
   :width: 75 %
   :align: center

   Screenshot of the RPKI RBV web interface

RBV provides two distinct APIs to run RPKI validation queries, the APIs allow
RESTful GET queries with the following syntax and formatting of the URL path:

#. ``/api/v1/validity/<asn>/<prefix>/<masklen>``
#. ``/api/v2/validity/<host>``

*Note*: the AS number in ``<asn>`` has to be prepended with *AS*;
and ``<host>`` can either be an IP address or a DNS hostname.
To test the APIs type the following queries for the RIPE RIS beacon
``93.175.146.0/24`` into the address bar of your favorite web browser:

.. code-block:: bash

    rpki-rbv.realmv6.org/api/v1/validity/AS12654/93.175.146.0/24
    rpki-rbv.realmv6.org/api/v2/validity/93.175.146.1

The result will be a JSON object as shown in :numref:`lst-rbv-json`.

.. code-block:: JSON
    :caption: Sample JSON output of RPKI RBV
    :name: lst-rbv-json

    {
        "validated_route": {
            "info": {
                "origin_country": "EU",
                "origin_asname": "RIPE-NCC-RIS-AS Reseaux IP Europeens Network Coordination Centre (RIPE NCC), EU"
            },
            "route": {
                "prefix": "93.175.146.0/24",
                "origin_asn": "AS12654"
            },
            "validity": {
                "state": "Valid",
                "code": 0,
                "description": "At least one VRP Matches the Route Prefix",
                "VRPs": {
                    "unmatched_as": [],
                    "unmatched_length": [],
                    "matched": [{
                        "prefix": "93.175.146.0/24",
                        "max_length": "24",
                        "asn": "AS12654"
                    }]
                }
            }
        }
    }

For detailed instruction how to install and set up the API visit
the `RBV Repository <https://github.com/rtrlib/rbv>`_ on GitHub.

.. _RPKI RBV: https://rpki-rbv.realmv6.org/
.. _RBV Github: https://github.com/rtrlib/rbv

Other Third-Party Tools
=======================

`RIPE <https://www.ripe.net/manage-ips-and-asns/resource-management/certification/tools-and-resources/>`_
provides an (almost) complete overview on other tools related to RPKI and
BGP security, in general.
