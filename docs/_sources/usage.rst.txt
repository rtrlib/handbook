.. _usage:

*****************************
Usage of the RTRlib C Library
*****************************

The RTRlib is supported by most Linux distributions as well as Apple macOS.
Before you start using or developing with the RTRlib C library, you have to
prepare your OS environment by installing a recent release package.


.. _install:

Installation
============

To install or build the RTRlib from source follow the steps described here.

Debian Linux
------------

If you are running Debian (Jessie), you can install the RTRlib via the APT
package manager, as follows:

.. code-block:: bash

    sudo apt-get install librtr0

Apple macOS
-----------

For macOS we provide a *Homebrew tap* to easily install the RTRlib.
First, setup Homebrew [#homebrew]_ and afterwards install the RTRlib package:

.. code-block:: bash

    brew tap rtrlib/pils
    brew install rtrlib

.. rubric:: Footnotes

.. [#homebrew]  Homebrew -- http://brew.sh

From Source
-----------

On any other Linux OS you will have to build and install the RTRlib from source.
The following minimal requirements have to be met, before building the library:

- the build system is based on `cmake`, version >= 2.6
- `libssh`, to establish SSH transport connections, version >= 0.5.0

Optional requirements are:

- `cmocka`, a framework to run RTRlib unit tests
- `doxygen`, to build the RTRlib API documentation

If the requirements are fulfilled in the system, the library and tools can be build.
First, either download or clone the RTRlib source code as follows:

.. code-block:: bash

    wget https://github.com/rtrlib/rtrlib/archive/v0.3.6.tar.gz
    tar xzf v0.3.6.tar.gz
    cd rtrlib-0.3.6
    # or
    git clone https://github.com/rtrlib/rtrlib/
    cd rtrlib

The contents of the RTRlib source code has the following subdirectory structure:

- ``cmake/``      CMake modules
- ``doxygen/``    Example code and graphics used in the Doxygen documentation
- ``rtrlib/``     Header and source code files of the RTRlib
- ``tests/``      Function tests and unit tests
- ``tools/``      Contains ``rtrclient`` and ``cli-validator``

Afterwards, the library can be build (we recommend an `out-of-source` build)
using `cmake`:

.. code-block:: bash

    mkdir build && cd build
    cmake -DCMAKE_BUILD_TYPE=Release ../
    make
    sudo make install

If the build command fails with any error, please consult the RTRlib README [#readme]_
and Wiki [#wiki]_, you may also join our `mailing list` [#mailinglist]_ or open
an issue on Github [#issue]_.

To enable debug symbols and messages, change the `cmake` command to:

.. code-block:: bash

    cmake -D CMAKE_BUILD_TYPE=Debug ../


For developers we provide a pre-build Doxygen API reference online [#doxygen]_
for the latest release of the RTRlib. Alternatively, and if `Doxygen` is
available on your system, you can build the documentation locally as follows:

.. code-block:: bash

    make doc


Further, you can also run the build-in tests provided by the RTRlib package
via `make`:

.. code-block:: bash

    make test

.. rubric:: Footnotes

.. [#readme]        README -- https://github.com/rtrlib/rtrlib/blob/master/README
.. [#wiki]          Wiki -- https://github.com/rtrlib/rtrlib/wiki
.. [#mailinglist]   Mailing list -- https://groups.google.com/forum/#!forum/rtrlib
.. [#issue]         Issue tracker -- https://github.com/rtrlib/rtrlib/issues
.. [#doxygen]       API reference -- https://rtrlib.realmv6.org/doxygen/latest

.. _devel:

Development with RTRlib
=======================

The RTRlib shared library is installed to ``/usr/local/lib`` by default,
and its headers files to ``/usr/local/include``, respectively.
To write an application in C/C++ using the RTRlib, include the main header file
into the code:

.. code-block:: C

    #include "rtrlib/rtrlib.h"

The name of the corresponding shared library is `rtr`.
To link an application against the RTRlib, pass the following parameter to the
compiler:

.. code-block:: bash

    -lrtr

If the linker reports an error such as ``cannot find -lrtr``, probably the
RTRlib was not installed to a standard location.
In this case, pass its location as an absolute path to the compiler,
add parameter:

.. code-block:: bash

    -L</path/to/librtr/>

On Linux you can alternatively try to update the linker cache instead,
run:

.. code-block:: bash

    ldconfig
    # verify with
    ldconfig -p | grep rtr

.. _coding:

Step-by-Step Example
====================

The RTRlib package includes two command line tools, the ``rtrclient`` and
the ``cli-validator``, see also :ref:`tools`.
The former connects to a single RTR cache server via TCP or SSH and prints
validated prefix origin data to STDOUT. You can use this tool to get first
experiences with the RPKI-RTR protocol. With the latter you can validate
arbitrary prefix origin AS relations against records received from a connected
RPKI cache. Both tools are located in the ``tools/`` directory. Having a look
into the source code of these tools will help to understand and integrate the
RTRlib into applications.

----

Any application using the RTRlib will have to setup a RTR connection manager
that handles synchronization with one (or multiple) trusted RPKI cache server(s).
The following provides an overview on important code segments.

First, create a RTR transport socket, for instance using TCP as shown in
:numref:`lst-create-socket`.

.. code-block:: C
    :linenos:
    :caption: Create a RTR transport socket
    :name: lst-create-socket

    struct tr_socket tr_tcp;
    struct rtr_socket rtr_tcp;
    char tcp_host[] = "rpki-validator.realmv6.org";
    char tcp_port[] = "8282";

    struct tr_tcp_config tcp_config = {
        tcp_host,   // cache server host
        tcp_port,   // cache server port
        NULL        // source address, empty
    };

    tr_tcp_init(&tcp_config, &tr_tcp);
    rtr_tcp.tr_socket = &tr_tcp;


Afterwards, create a group of RTR cache servers with preference `1`.
In this example (see :numref:`lst-create-group`), it includes only a single
cache instance.

.. code-block:: C
    :linenos:
    :caption: Create a group of RTR caches
    :name: lst-create-group

    rtr_mgr_group groups[1];
    groups[0].sockets = malloc(sizeof(struct rtr_socket*));
    groups[0].sockets_len = 1;
    groups[0].sockets[0] = &rtr_tcp;
    groups[0].preference = 1;


Now initialize the RTR connection manager (:numref:`lst-init-rtrmgr`) providing
a pointer to a configuration object, the preconfigured group(s), number
of groups, a refresh interval, an expiration interval, and retry interval,
as well as distinct callback functions.
In this case, a refresh interval of 30 seconds, a 600s expiration timeout,
and a 600s retry interval will be defined.
Afterwards, start the RTR Connection Manager.

.. code-block:: C
    :linenos:
    :caption: Initialize the RTR connection manager.
    :name: lst-init-rtrmgr

    struct rtr_mgr_config *conf;
    int ret = rtr_mgr_init(&conf, groups, 1, 30, 600, 600,
                           pfx_update_fp, spki_update_fp, status_fp, NULL);

    rtr_mgr_start(conf);


As soon as an update has been received from the RTR-Server, the callback
function will be invoked. In this example, `update_cb` (see :numref:`lst-callback`)
is called which prints the prefix, its minimum, and maximum length, as well as
the corresponding origin AS.

.. code-block:: C
    :linenos:
    :caption: RTR connection manager update callback
    :name: lst-callback

    static void update_cb(struct pfx_table* p, const pfx_record rec, const bool added){
        char ip[INET6_ADDRSTRLEN];
        if(added)
            printf("+ ");
        else
            printf("- ");
        ip_addr_to_str(&(rec.prefix), ip, sizeof(ip));
        printf("%-18s %3u-%-3u %10u\n", ip, rec.min_len, rec.max_len, rec.asn);
    }

With a running RTR connection manager, you can also execute validation queries.
For instance, validate the relation of prefix `10.10.0.0/24` and its origin
AS 12345 as shown in :numref:`lst-validate`.

.. code-block:: C
    :linenos:
    :caption: Validate a prefix to origin AS relation
    :name: lst-validate

    struct lrtr_ip_addr pref;
    lrtr_ip_str_to_addr("10.10.0.0", &pref);
    enum pfxv_state result;
    const uint8_t mask = 24;
    rtr_mgr_validate(conf, 12345, &pref, mask, &result);

For a clean shutdown and exit of the application, first stop the RTR
Connection Manager, and secondly release any memory allocated
(see :numref:`lst-stop-rtrmgr`).

.. code-block:: C
    :linenos:
    :caption: RTR connection manager cleanup
    :name: lst-stop-rtrmgr

    rtr_mgr_stop(conf);
    rtr_mgr_free(conf);
    free(groups[0].sockets);


Complete RTRlib Example
=======================

The code in :numref:`lst-full-example` shows a fully functional RPKI validator
using the RTRlib. It includes all parts explained in the previous section, and
shows how to setup multiple RPKI cache server connections using either TCP or
SSH transport sockets. For the latter, the RTRlib has to be build and installed
with `libssh` support.

.. code-block:: C
    :linenos:
    :caption: A complete code example for the RTRlib
    :name: lst-full-example

    #include <stdio.h>
    #include <stdlib.h>
    #include "rtrlib/rtrlib.h"

    int main(){
        //create a SSH transport socket
        char ssh_host[]     = "123.231.123.221";
        char ssh_user[]     = "rpki_user";
        char ssh_hostkey[]  = "/etc/rpki-rtr/hostkey";
        char ssh_privkey[]  = "/etc/rpki-rtr/client.priv";
        struct tr_socket tr_ssh;
        struct tr_ssh_config config = {
            ssh_host,       //IP
            22,             //Port
            NULL,           //Source address
            ssh_user,
            ssh_hostkey,    //Server hostkey
            ssh_privkey,    //Private key
        };
        tr_ssh_init(&config, &tr_ssh);

        //create a TCP transport socket
        struct tr_socket tr_tcp;
        char tcp_host[] = "rpki-validator.realmv6.org";
        char tcp_port[] = "8282";

        struct tr_tcp_config tcp_config = {
            tcp_host, //IP
            tcp_port, //Port
            NULL      //Source address
        };
        tr_tcp_init(&tcp_config, &tr_tcp);

        //create 3 rtr_sockets and associate them with the transprort sockets
        struct rtr_socket rtr_ssh, rtr_tcp;
        rtr_ssh.tr_socket = &tr_ssh;
        rtr_tcp.tr_socket = &tr_tcp;

        //create a rtr_mgr_group array with 2 elements
        struct rtr_mgr_group groups[2];

        //The first group contains both TCP RTR sockets
        groups[0].sockets = malloc(sizeof(struct rtr_socket*));
        groups[0].sockets_len = 1;
        groups[0].sockets[0] = &rtr_tcp;
        groups[0].preference = 1;       //Preference value of this group

        //The seconds group contains only the SSH RTR socket
        groups[1].sockets = malloc(1 * sizeof(struct rtr_socket*));
        groups[1].sockets_len = 1;
        groups[1].sockets[0] = &rtr_ssh;
        groups[1].preference = 2;

        //create a rtr_mgr_config struct that stores the group
        struct rtr_mgr_config *conf;

        //initialize all rtr_sockets in the server pool with the same settings
        int ret = rtr_mgr_init(&conf, groups, 2, 30, 600, 600, NULL, NULL, NULL, NULL);

        //start the connection manager
        rtr_mgr_start(conf);

        //wait till at least one rtr_mgr_group is fully synchronized with the server
        while(!rtr_mgr_conf_in_sync(conf)) {
            sleep(1);
        }

        //validate the BGP-Route 10.10.0.0/24, origin ASN: 12345
        struct lrtr_ip_addr pref;
        lrtr_ip_str_to_addr("10.10.0.0", &pref);
        enum pfxv_state result;
        const uint8_t mask = 24;
        rtr_mgr_validate(conf, 12345, &pref, mask, &result);

        //output the result of the prefix validation above
        //to showcase the returned states.
        char buffer[INET_ADDRSTRLEN];
        lrtr_ip_addr_to_str(&pref, buffer, sizeof(buffer));

        printf("RESULT: The prefix %s/%i ", buffer, mask);
        switch(result) {
            case BGP_PFXV_STATE_VALID:
                printf("is valid.\n");
                break;
            case BGP_PFXV_STATE_INVALID:
                printf("is invalid.\n");
                break;
            case BGP_PFXV_STATE_NOT_FOUND:
                printf("was not found.\n");
                break;
            default:
                break;
        }

        // cleanup before exit
        rtr_mgr_stop(conf);
        rtr_mgr_free(conf);
        free(groups[0].sockets);
        free(groups[1].sockets);
    }
