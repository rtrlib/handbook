BGP Routing Daemons with RPKR/RTR
=================================

RTRlib is currently used by several Routing Daemons such as `Quagga <http://www.nongnu.org/quagga/>`_ or `BIRD <http://bird.network.cz/>`_.

The BIRD Internet Routing Daemon
--------------------------------

.. role:: bash(code)
  :language: bash

To set up BIRD, first `download <http://bird.network.cz/?download>`_, unzip and :bash:`cd` into it. To build BIRD, run

.. code-block:: bash

  ./configure
  make
  make install

You may need to execute these and any following commands in this handbook as :bash:`sudo`. More information on the building process can be found in the README of BIRD.
  
Before any validations with BIRD can be done, it must be configured. First, a ROA table and the validation function must be added to :bash:`/usr/local/etc/bird.conf`.
At the top of this file write:

.. code-block:: bash

  roa table rtr_roa_table ;

  function test_ripe_beacons()
  {
    print "Testing ROA";
    print "Should be TRUE TRUE TRUE:",
      " ", roa_check(rtr_roa_table, 84.205.83.0/24, 12654) = ROA_UNKNOWN,
      " ", roa_check(rtr_roa_table, 93.175.146.0/24, 12654) = ROA_VALID,
      " ", roa_check(rtr_roa_table, 93.175.147.0/24, 12654) = ROA_INVALID;
  }

The first line automatically creates a ROA table when the BIRD daemon is started. The function itself checks for three entries in the ROA table
and prints the corresponding validity status.

To try out whether BIRD receives actual responses, there is an IPC that runs on the BIRD socket. Clone the `BIRD-RTRlib-CLI <https://github.com/rtrlib/bird-rtrlib-cli>`_
repository on GitHub and build it:

.. code-block:: bash

  cmake .
  make

In case that the RTRlib was not installed in the default directory, run

.. code-block:: bash

  cmake -DRTRLIB_INCLUDE=<rtrlib> -DRTRLIB_LIBRARY=</path/to/rtrlib.[a|so|dylib]> .
  make

If everything was build correctly, there now should be an executable called :bash:`bird-rpki-client`. To see all the options of this program run the help option
:bash:`./bird-rpki-client --help`

The BIRD socket must now be opened. In order to do that, head back to the BIRD directory and type the following command:

.. code-block:: bash

  ./bird -c /usr/local/etc/bird.conf -s /tmp/bird.ctl -d 

``/tmp/bird.ctl`` is the location and name of the socket that will be created. It is required by the ``bird-rpki-client``. With the option ``-d``
BIRD runs in the foreground. That's necessary to view the output of the ``test_ripe_beacons`` function.

Open a new Terminal. Now connect to the BIRD socket and receive the RPKI data with the following command. It can also be found in the README of the bird-rpki-client.

.. code-block:: bash

  ./bird-rpki-client -b /temp/bird.ctl -r rpki-validator.realmv6.org:8282 -t rtr_roa_table

The options do the following:

| :bash:`-b`: the location of the BIRD socket.
|
| :bash:`-r`: the address and port of the RPKI cache server. Change it if you want to use a different one.
|
| :bash:`-t`: the table in which the gathered rpki-data is filled into. We created this one earlier in the bird.conf
|

After executing this line, you will see that, after establishing a connection to the cache server, the ROA entries are piped into the BIRD ROA table.
Start the BIRD CLI with the following command:

.. code-block:: bash

   sudo ./birdc -s /tmp/bird.ctl

All the commands of the CLI can be viewed by typing ``?``. To list all the entries from the ROA table enter:

.. code-block:: bash

  bird> show roa
  194.3.206.0/24 max 24 as 24954
  03.4.119.0/24 max 24 as 38203
  200.7.212.0/24 max 24 as 27947
  200.7.212.0/24 max 24 as 19114
  103.10.79.0/24 max 24 as 45951
  ...

There will be a lot of similar output. The content of the ``bird-rpki-client`` was successfully written to the ROA table. Search, for example, for the prefix
93.175.146.0/24 and BIRD will return the entry with its corresponding ASN.

.. code-block:: bash

  bird> show roa 93.175.146.0/24
  93.175.146. sudo ./birdc -s /tmp/bird.ctl0/24 max 24 as 12654

To do the actual validation of the prefixes that were defined in ``test_ripe_beacons`` execute:

.. code-block:: bash

  bird> eval test_ripe_beacons()
  void()
  
To see the output of the function switch to the terminal that is running the BIRD daemon. The output will look like:

.. code-block:: bash

  bird: Testing ROA
  bird: Should be TRUE TRUE TRUE: TRUE TRUE TRUE

After seeing this line, the prefixes were successfully tested.

The Quagga Routing Software Suite
---------------------------------

A Routing Daemon such as Quagga implements TCP/IP routing via protocols such as OSPF, RIP and BGP. It acts as a router that fetches and shares routing information
with other routers. Regarding BGP, Quagga supports version 4.
An unofficial release implements support for the RPKI so BGP updates can be verified against a ROA. Doing so requires the support of the RTRlib so Quagga can
initialize a connection to a cache server using the RTR protocol.

To install Quagga, clone the Git repository from `here <https://github.com/rtrlib/quagga-rtrlib>`_ and switch the branch like this:

.. code-block:: bash

  git clone https://github.com/rtrlib/quagga-rtrlib.git
  cd quagga-rtrlib
  git checkout feature/rtrlib
  
This repository is a fork of the original and implements RPKI support. Before building it, make sure your system meets the perquisites:

* automake:	1.9.6
* autoconf:	2.59 
* libtool:	1.5.22
* texinfo:	4.7
* GNU AWK:	3.1.5

If all of these packages are installed, Quagga can be build. Some steps might require ``sudo`` privileges:

.. code-block:: bash

  ./bootstrap
  ./configure --enable-rpki
  make
  make install

