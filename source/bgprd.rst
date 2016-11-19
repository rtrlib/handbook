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
Now start the daemon with 

.. code-block:: bash

  ./bird

You can check whether the daemon is running by typing

.. code-block:: bash

  ps -A | grep bird
  
BIRD is now up and running. It must be configured before any validations can be done with it. First, a ROA table must be added to :bash:`/usr/local/etc/bird.conf`
At the top of this file write:

.. code-block:: bash

  roa table rtr_roa_table ;

This line automatically creates a ROA table when the BIRD daemon is started. To check whether it was created successfully, run :bash:`./birdc`.
The BIRD CLI will be executed. Refresh the configuration file and list all symbols:

.. code-block:: bash

  bird> configure
  Reading configuration from /usr/local/etc/bird.conf
  Reconfigured
  bird> show symbols roa
  rtr_roa_table   ROA table


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

Now connect to the BIRD socket and receive the RPKI data with the following command. It can also be found in the README of the bird-rpki-client.

.. code-block:: bash

  ./bird-rpki-client -b /usr/local/var/run/bird.ctl -r rpki-validator.realmv6.org:8282 --bird-roa-table=rtr_roa_table

The options do the following:

| :bash:`-b`: the location of the BIRD socket. Depending on the system you are running this on, you might need to change this path to :bash:`/var/run/bird.ctl`
|
| :bash:`-r`: the address and port of the RPKI cache server. Change it if you want to use a different one.
|
| :bash:`--bird-roa-table`: the table in which the gathered rpki-data is filled into. We created this one earlier in the bird.conf
|

After executing this line, you will see that, after establishing a connection to the cache server, the ROA entries are piped into the BIRD ROA table.


The Quagga Routing Software Suite
---------------------------------

A Routing Daemon such as Quagga implements TCP/IP routing via protocols such as OSPF, RIP and BGP. It acts as a router that fetches and shares routing information
with other routers. Regarding BGP, Quagga supports version 4.
An unofficial release implements support for the RPKI so BGP updates can be verified against a ROA. Doing so requires the support of the RTRlib so Quagga can
initialize a connection to a cache server using the RTR protocol.

