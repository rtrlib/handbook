.. _background:

**********
Background
**********

The global deployment of a *Resource Public Key Infrastructure*
(RPKI :cite:`RFC-6480`) is one step towards securing the Internet routing
through cryptography.
The RPKI allows the holder of a distinct IP prefix to authorize certain
autonomous systems (AS) to originate corresponding routes, which is
cryptographically verifiable through *Route Origination Authorizations* (ROAs)
that are stored in the RPKI.

RPKI enabled routers do not store such ROAs themselves, but only the validated
content of those.
To achieve high scalability as well as limit resource utilization on BGP
routers, the validation of ROAs is performed by trusted RPKI cache servers,
which are deployed at the network operator site.
The RPKI-RTR protocol defines a standard mechanism to maintain exchange of
the prefix origin AS relations between the cache server and routers.
In combination with a BGP prefix origin validation scheme a router is able to
verify received BGP updates without suffering from cryptographic complexity.

The RTRlib is a lightweight C library that implements the RPKI-RTR protocol for
the client end (i.e., routers) and the proposed prefix origin validation scheme.
The RTRlib provides functions to establish a connection to a single or multiple
trusted caches using TCP and SSH transport connections, and further allows to
determine the validation state of prefix to origin AS relations.

:numref:`overview` shows a typical RPKI deployment, where trusted cache servers
collect ROAs from global RPKI repositories of the RIRs, such as RIPE and APNIC.
Each local RPKI cache periodically updates and verifies the stored ROAs, and
pushes the preprocessed data to connected RPKI enabled BGP routers using
the RTR protocol.

.. _overview:
.. figure:: ../images/rpki-rtr-overview.jpg
    :width: 90 %
    :align: center

    Overview on a typical RPKI deployment, showing global RPKI repositories,
    trusted cache servers, and RPKI enabled BGP routers.

Further Reading
===============

Detailed insights on the implementation of the RTRlib  and its performance can
be found in :cite:`whss-roslr-13`.
Further information is available in the standard specifications and
protocols in RFCs 6810 :cite:`RFC-6810` and 6811 :cite:`RFC-6811`, to which
the RTRlib complies.
Even more background material on BGP security extensions can be found in
:cite:`RFC-7353`, :cite:`draft-ietf-sidr-bgpsec-overview`,
and :cite:`draft-ietf-sidr-bgpsec-protocol`

.. only:: html

    .. rubric:: References

.. bibliography:: handbook.bib
    :style: unsrt
    :cited:
