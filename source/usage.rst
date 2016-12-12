.. _usage:

*******************
Usage of the RTRlib
*******************


.. _install:

Build and Install the RTRlib
============================

The RTRlib is supported by most Linux distributions as well as Apple macOS.

Debian Linux
------------

If you are running Debian (Jessie), you can install the library via the APT
package manager, as follows

.. code-block:: Bash

    sudo apt-get install librtr0

Apple macOS
-----------

For macOS we provide a *Homebrew* tap_ to easily install the RTRlib.
First, install Homebrew_ and afterwards install RTRlib as follows:

.. code-block:: Bash

    brew tap rtrlib/pils
    brew install rtrlib

.. _Homebrew: http://brew.sh
.. _tap: https://github.com/rtrlib/homebrew-pils

From Source
-----------

On any other Linux OS you will have to build and install the RTRlib from source.
The following minimal requirements have to be met, before building the library:

- the build system is based on `cmake`, version >= 2.6
- `libssh` to establish SSH transport connections, version >= 0.5.0

Optional requirements are:

- `cmocka` a framework to run RTRlib unit tests
- `doxygen` to build the RTRlib API documentation

If the requirements are installed, the library can be build.
First, either download or clone the RTRlib source code as follows:

.. code-block:: Bash

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

Afterwards, build the library as follows (we recommend an `out-of-source` build)
using `cmake`:

.. code-block:: Bash

    mkdir build && cd build
    cmake -DCMAKE_BUILD_TYPE=Release ../
    make
    sudo make install

If the build command fails with any error, please consult the RTRlib README_
and Wiki_, you may also join our `mailing list`_.

To enable debug symbols and messages, change the `cmake` command to:

.. code-block:: Bash

    cmake -D CMAKE_BUILD_TYPE=Debug ../

If `Doxygen` is available, you can build the documentation, by running:

.. code-block:: Bash

    make doc

Further, you can also run the build-in tests provided by the RTRlib package
via `make`:

.. code-block:: Bash

    make test

.. _README: https://github.com/rtrlib/rtrlib/
.. _Wiki: https://github.com/rtrlib/rtrlib/wiki
.. _mailing list: https://groups.google.com/forum/#!forum/rtrlib

.. _devel:

Development with the RTRlib
===========================

The RTRlib shared library is installed to ``/usr/local/lib`` by default,
and its headers files to ``/usr/local/include``, respectively.
Writing an application in C/C++ using the RTRlib just include the main header
file into the code:

.. code-block:: C

    #include "rtrlib/rtrlib.h"

The name of the shared library is `rtr`.
To link an application against the RTRlib, pass the following parameter to gcc:

.. code-block:: Bash

    -lrtr

If the linker reports an error such as ``cannot find -lrtr`` the shared library
cannot be found.
This typically happens when the RTRlib was not installed to a standard location.
Thus, pass its location as an absolute path to the compiler:

.. code-block:: Bash

    -L</path/to/librtr/>

On Linux you can also try to update the linker cache by running:

.. code-block:: Bash

    ldconfig
    # verify with
    ldconfig -p | grep rtr

.. _coding:

Step-by-Step Coding Example
===========================

The RTRlib package includes two command line tools, the ``rtrclient`` and
the ``cli-validator``, see also :ref:`tools`.
The former connects to a single RTR cache server via TCP or SSH and prints
validated prefix origin data to STDOUT. You can use this tool to get first
experiences with the RPKI-RTR protocol. With the latter you can validate
arbitrary prefix to origin relations. Both tools are located in the ``tools/``
directory.

Having a look into the source code of these tools may help to understand and
integrate the RTRlib into your own applications. The following provides an
overview on important code segments.

----

Create a TCP Transport socket

.. code-block:: C

    struct tr_socket tr_tcp;
    char tcp_host[]	= ;
    char tcp_port[]	= "8282";

    struct tr_tcp_config tcp_config = {
        "rpki-validator.realmv6.org",   // TCP host
        "8282",                         // TCP port
        NULL                // empty source address
    };
    tr_tcp_init(&tcp_config, &tr_tcp);

    struct rtr_socket rtr_tcp;
    rtr_tcp.tr_socket = &tr_tcp;


Create a group of RTR-Servers with preference `1`. In this case, it includes
only one RTR-Server.

.. code-block:: C

    rtr_mgr_group groups[1];
    groups[0].sockets = malloc(sizeof(struct rtr_socket*));
    groups[0].sockets_len = 1;
    groups[0].sockets[0] = &rtr_tcp;
    groups[0].preference = 1;


#. Initialize the RTR Connection Manager with a configuration object, the group(s),
number of groups, a refresh interval, an expiration interval, and retry interval,
as well as callback functions. In this case, a refresh interval of 30 seconds,
a 600s expiration timeout, and a 600s retry interval will be defined.

.. code-block:: C

    struct rtr_mgr_config *conf;
    int ret = rtr_mgr_init(&conf, groups, 1, 30, 600, 600,
                           pfx_update_fp, spki_update_fp, status_fp, NULL);


Start the RTR Connection Manager

.. code-block:: C

    rtr_mgr_start(conf);


As soon as an update has been received from the RTR-Server, the callback
function will be invoked. In this example, `update_cb` will be invoked and
prints the prefix, the min and max length, as well as the origin AS.

.. code-block:: C

    static void update_cb(struct pfx_table* p, const pfx_record rec, const bool added){
        char ip[INET6_ADDRSTRLEN];
        if(added)
            printf("+ ");
        else
            printf("- ");
        ip_addr_to_str(&(rec.prefix), ip, sizeof(ip));
        printf("%-18s %3u-%-3u %10u\n", ip, rec.min_len, rec.max_len, rec.asn);
    }


Validate the relation of prefix `10.10.0.0/24` and its origin AS 12345 as follows.

.. code-block:: C

    struct lrtr_ip_addr pref;
    lrtr_ip_str_to_addr("10.10.0.0", &pref);
    enum pfxv_state result;
    const uint8_t mask = 24;
    rtr_mgr_validate(conf, 12345, &pref, mask, &result);


Finally, stop the RTR Connection Manager and release memory.

.. code-block:: C

    rtr_mgr_stop(conf);
    rtr_mgr_free(conf);
    free(groups[0].sockets);
