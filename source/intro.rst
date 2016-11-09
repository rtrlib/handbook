Introduction
============

Background
----------

RPKI-enabled routers do not store ROAs itself but only the validated content of these authorities.
The validation of ROAs will be performed by trusted cache servers, which will be deployed at the network operator site.
The RPKI/RTR protocol defines a standard mechanism to maintain the exchange of the prefix/origin AS mapping between the cache server and routers.

In combination with a ​[BGP prefix origin validation scheme](http://tools.ietf.org/html/rfc6811) a router is able to verify received BGP updates without suffering from cryptographic complexity.

The RTRlib is a lightweight C library.
It implements the RPKI/RTR protocol for the router end and the proposed prefix origin validation scheme.
The RTRlib provides functions to establish a connection to a single or multiple trusted caches and to determine the validation state of a prefix/origin AS mapping.
![](http://www.realmv6.org/rpki-rtr-overview.jpg)

Other tools
* There is a nice overview at ​[RIPE](https://www.ripe.net/manage-ips-and-asns/resource-management/certification/tools-and-resources).
* Note, [​rtr-origin](http://subvert-rpki.hactrn.net/trunk/rtr-origin/) includes also a client. It is written in Python.
