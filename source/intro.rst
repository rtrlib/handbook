Introduction
============

The RTRlib implements the client-side of the RPKI-RTR protocol (`RFC 6810`_) and
BGP Prefix Origin Validation (`RFC 6811`_). The latest release of the RTRlib also
supports Internet-Draft `draft-ietf-sidr-rpki-rtr-rfc6810-bis`_, which enables
the maintenance of router keys.
Router keys are required to deploy BGPSEC.

Background
----------

RPKI-enabled routers do not store ROAs itself but only the validated content of
these authorities.
The validation of ROAs will be performed by trusted cache servers, which will be
deployed at the network operator site.
The RPKI-RTR protocol defines a standard mechanism to maintain the exchange of
the prefix origin AS mapping between the cache server and routers.
In combination with a BGP prefix origin validation scheme a router is able to
verify received BGP updates without suffering from cryptographic complexity.

The RTRlib is a lightweight C library that implements the RPKI/RTR protocol for
the client end (i.e., router) and the proposed prefix origin validation scheme.
The RTRlib provides functions to establish a connection to a single or multiple
trusted caches using TCP or SSH transport connections, and firther allows to
determine the validation state of a prefix to origin AS mapping.

.. image:: ../images/rpki-rtr-overview.jpg

Further Reading
---------------

- `RFC 6480`_ : An Infrastructure to Support Secure Internet Routing
- `RFC 6810`_ : the RPKI to Router Protocol (RTR)
- `RFC 6811`_ : on BGP Prefix Origin Validation

.. _RFC 6480: https://tools.ietf.org/html/rfc6480
.. _RFC 6810: https://tools.ietf.org/html/rfc6810
.. _RFC 6811: https://tools.ietf.org/html/rfc6811
.. _draft-ietf-sidr-rpki-rtr-rfc6810-bis: https://tools.ietf.org/html/draft-ietf-sidr-rpki-rtr-rfc6810-bis
