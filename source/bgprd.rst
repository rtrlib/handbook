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

You may need to execute these and any following commands in this handbook as :bash:`sudo`. For more information on building it can be found in the README of BIRD.
Now start the daemon with 

.. code-block:: bash

  ./bird

You can check whether the daemon is running by typing

.. code-block:: bash

  ps -A | grep bird
  
To try out BIRD and receive actual responses, there is an IPC that runs on the BIRD socket. Clone this `repository on GitHub <https://github.com/rtrlib/bird-rtrlib-cli>`_
and build it with these commands, if the RTRlib was installed in the default directory:

.. code-block:: bash

  cmake .
  make

In case that the RTRlib was not installed in the default directory, run

.. code-block:: bash

  cmake -DRTRLIB_INCLUDE=<rtrlib> -DRTRLIB_LIBRARY=</path/to/rtrlib.[a|so|dylib]> .
  make

If everything was build correctly, there now should be an executable called :bash:`bird-rpki-client`. To see all the options of this program run the help command

.. code-block:: bash

  ./bird-rpki-client --help

Now connect to the BIRD socket and receive the RPKI data with the following command. It can also be found in the README of the bird-rpki-client.

.. code-block:: bash

  ./bird-rpki-client -b /usr/local/var/run/bird.ctl -r rpki-validator.realmv6.org:8282

The :bash:`-b` option is passed the location of the BIRD socket. Depending on the system you are running this on, you might need to change this path to :bash:`/var/run/bird.ctl`
The :bash:`-r` option takes the address and port of the RPKI cache server. Change it if you want to use a different one.

The Quagga Routing Software Suite
---------------------------------

A Routing Daemon such as Quagga implements TCP/IP routing via protocols such as OSPF, RIP and BGP. It acts as a router that fetches and shares routing information
with other routers. Regarding BGP, Quagga supports version 4.
An unofficial release implements support for the RPKI so BGP updates can be verified against a ROA. Doing so requires the support of the RTRlib so Quagga can
initialize a connection to a cache server using the RTR protocol.

