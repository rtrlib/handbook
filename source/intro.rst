.. _intro:

************
Introduction
************

The RTRlib implements the client-side of the RPKI-RTR protocol :cite:`RFC-6810`
and the BGP Prefix Origin Validation :cite:`RFC-6811`.
The latest release of the RTRlib also supports the Internet-Draft
:cite:`draft-ietf-sidr-rpki-rtr-rfc6810-bis` to enable
the maintenance of router keys which are required to deploy BGPSEC in the future.

Securing BGP using RPKI
=======================

The global deployment of a Resource Public Key Infrastructure
(RPKI :cite:`RFC-6480`) is a first step towards securing the Internet routing.

RPKI-enabled routers do not store ROAs itself but only the validated content of
these authorities.
To achieve high scalability as well as limit resource utilization on BGP
routers, the validation of ROAs is performed by trusted RPKI cache servers,
which are deployed at the network operator site.
The RPKI-RTR protocol defines a standard mechanism to maintain exchange of
the prefix origin AS relations between the cache server and routers.
In combination with a BGP prefix origin validation scheme a router is able to
verify received BGP updates without suffering from cryptographic complexity.

The RTRlib is a lightweight C library that implements the RPKI/RTR protocol for
the client end (i.e., routers) and the proposed prefix origin validation scheme.
The RTRlib provides functions to establish a connection to a single or multiple
trusted caches using TCP or SSH transport connections, and further allows to
determine the validation state of prefix to origin AS relations.

The figure below shows a typical RPKI deployment, where trusted cache server
collect ROAs from global RPKI repositories of RIRs, such as RIPE and APNIC.
Each local RPKI cache periodically updates and verifies the stored ROAs, and
pushes this preprocessed data to connected RPKI enabled BGP routers using
the RTR protocol.

.. image:: ../images/rpki-rtr-overview.jpg

.. only:: html

   .. rubric:: Further Reading

   Detailed insights on the RTRlib implementation, its performance :cite:`whss-roslr-13`,
   and the utilized standard protocols, can be found in the following references:

.. bibliography:: handbook.bib
    :style: unsrt
    :cited:
