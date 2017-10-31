.. _bgprd:

*********************************
BGP Routing Daemons with RPKI/RTR
*********************************

For several Routing Daemons such as `Quagga` [#quagga]_ and `BIRD` [#bird]_
exist RPKI enabled extensions that are based on the RTRlib.

The BIRD Internet Routing Daemon
================================

.. role:: bash(code)
  :language: bash

To set up BIRD, first download [#bird-download]_ the latest release, unzip and
change into the source directory. To build BIRD, run:

.. code-block:: bash

    ./configure
    make
    make install

You may need to execute these and any following commands in this handbook as :bash:`sudo`.
More information on the building process can be found in the README of BIRD.

Before any validations with BIRD can be done, it must be configured accordingly.
First, a ROA table and the validation function must be added to :bash:`/usr/local/etc/bird.conf`.
At the top of this file write:

.. code-block:: bash

    roa table rtr_roa_table;

    function test_ripe_beacons()
    {
        print "Testing ROA";
        print "Should be TRUE TRUE TRUE:",
        " ", roa_check(rtr_roa_table, 84.205.83.0/24, 12654) = ROA_UNKNOWN,
        " ", roa_check(rtr_roa_table, 93.175.146.0/24, 12654) = ROA_VALID,
        " ", roa_check(rtr_roa_table, 93.175.147.0/24, 12654) = ROA_INVALID;
    }

The first line automatically creates a ROA table when the BIRD daemon is started.
The function itself checks for three entries in the ROA table
and prints the corresponding validity status.
The BIRD socket must now be opened. In order to do that type the following command:

.. code-block:: bash

  ./bird -c /usr/local/etc/bird.conf -s /tmp/bird.ctl -d

With the option ``-d`` BIRD runs in the foreground.
That's necessary to view the output of the ``test_ripe_beacons`` function.
``/tmp/bird.ctl`` is the location and name of the socket that will be created.
It is required by the ``bird-rtrlib-cli`` which we will install next.

Open another new terminal. To try out whether BIRD receives actual responses,
there is an IPC that runs on the BIRD socket.
Clone the `BIRD-RTRlib-CLI` repository on GitHub and build it:

.. code-block:: bash

    git clone https://github.com/rtrlib/bird-rtrlib-cli
    cd bird-rtrlib-cli
    cmake .
    make

In case that the RTRlib was not installed into the default directory, run

.. code-block:: bash

    cmake -DRTRLIB_INCLUDE=<rtrlib> -DRTRLIB_LIBRARY=</path/to/rtrlib.[a|so|dylib]> .
    make

If everything was build correctly,
there now should be an executable called :bash:`bird-rtrlib-cli`.
To see all the options of this program run :bash:`./bird-rtrlib-cli --help`.
Now connect to the BIRD socket and receive the RPKI data with the following command:

.. code-block:: bash

    ./bird-rtrlib-cli -b /tmp/bird.ctl -r rpki-validator.realmv6.org:8282 -t rtr_roa_table

The options do the following:

| :bash:`-b`: the location of the BIRD socket.
|
| :bash:`-r`: the address and port of the RPKI cache server. Change it if you want to use a different one.
|
| :bash:`-t`: the table in which the gathered rpki-data is filled into. We created this one earlier in the bird.conf
|

After executing this line, you will see that, after establishing a connection
to the cache server, the ROA entries are piped into the BIRD ROA table.
Head back to the BRID directory and start the BIRD CLI with the following command:

.. code-block:: bash

    sudo ./birdc -s /tmp/bird.ctl

All the commands of the CLI can be viewed by typing ``?``.
To list all the entries from the ROA table enter:

.. code-block:: bash

    bird> show roa
    194.3.206.0/24 max 24 as 24954
    03.4.119.0/24 max 24 as 38203
    200.7.212.0/24 max 24 as 27947
    200.7.212.0/24 max 24 as 19114
    103.10.79.0/24 max 24 as 45951
    ...

Type ``q`` to exit. There will be a lot of similar output.
The content of the ``bird-rtrlib-cli`` was successfully written to the ROA table.
Search, for example, for the prefix ``93.175.146.0/24`` and BIRD will return
the entry with its corresponding ASN.

.. code-block:: bash

    bird> show roa 93.175.146.0/24
    93.175.146.0/24 max 24 as 12654

To do the actual validation of the prefixes that were defined in ``test_ripe_beacons`` execute:

.. code-block:: bash

    bird> eval test_ripe_beacons()
    (void)

To see the output of the function, switch to the terminal that is running the
BIRD daemon. The output will look like:

.. code-block:: bash

    bird: Testing ROA
    bird: Should be TRUE TRUE TRUE: TRUE TRUE TRUE

After seeing this line, the test function was executed and the prefixes were successfully tested.

The Quagga Routing Software Suite
=================================

A Routing Daemon such as Quagga implements TCP/IP routing via protocols such as
OSPF, RIP and BGP. It acts as a router that fetches and shares routing information
with other routers. Regarding BGP, Quagga supports version 4.
An unofficial release implements support for the RPKI so BGP updates can be
verified against a ROA. Doing so requires the support of the RTRlib so Quagga
can initialize a connection to a cache server using the RTR protocol.

To install Quagga, clone the Git repository and switch the branch like this:

.. code-block:: bash

    git clone https://github.com/rtrlib/quagga-rtrlib.git
    cd quagga-rtrlib
    git checkout feature/rtrlib

This repository is a fork of the original and implements RPKI support. Before
building it, make sure your system meets the perquisites:

* automake:	1.9.6
* autoconf:	2.59
* libtool:	1.5.22
* texinfo:	4.7
* GNU AWK:	3.1.5

If all of these packages are installed, Quagga can be build. Some steps might
require ``sudo`` privileges:

.. code-block:: bash

    ./bootstrap
    ./configure --enable-rpki
    make
    make install

The ``--enable-rpki`` option tells the configure script to include the RTRlib.

Now that Quagga is built, start the BGP and Zebra daemons. Zebra acts as a
process between the package stream of the kernel and daemons like BGP or OSPF.
Execute ``bgpd`` and ``zebra``:

.. code-block:: bash

    ./bgpd/bgpd
    ./zebra/zebra

To interact with BGPD, connect to it via ``vtysh``, a command line interface that gains access to such daemons.

.. rubric:: Footnotes

.. [#quagga]        Quagga -- http://www.nongnu.org/quagga/
.. [#bird]          BIRD -- http://bird.network.cz/
.. [#bird-download] BIRD download -- http://bird.network.cz/?download
