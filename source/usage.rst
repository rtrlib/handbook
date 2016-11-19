Usage of the RTRlib
===================

Build and Install RTRlib
------------------------

Please note, that the following requirements must be installed in order to build the library.

| `cmake`   version >= 2.6
| cmake is used as the build system of the library.
|
| `libssh`   version >= 0.5.0
| This is required to establish an SSH connection between RTR-Client and RTR-Server.
|

Optional packages are:

| `cmocka`
| Used for unit tests.
|
| `Doxygen`
| Needed for building the HTML documentation.
|

If the requirements are met, the library can be build. There are two options.
Building RTRlib without debugging options:

.. code-block:: Bash

    cmake -D CMAKE_BUILD_TYPE=Release .

To enable debug symbols and debug messages:

.. code-block:: Bash

    cmake -D CMAKE_BUILD_TYPE=Debug .

If the libssh isn't installed within the systems include and library directories you can run
cmake with the following parameters:

.. code-block:: Bash

    -D LIBSSH_LIBRARY=<path-to-libssh.so>
    -D LIBSSH_INCLUDE=<include-directory>

To specify another directory where the RTRlib will be installed, you can pass the following
argument to cmake:

.. code-block:: Bash

    -D CMAKE_INSTALL_PREFIX=<path>

Now to build the library itself with all the tests and tools execute

.. code-block:: Bash

    make
    
The build process of the library is now complete. An additional option is to copy libraries
and headers to system directories. To do this, execute

.. code-block:: Bash

    make install


Linking to RTRlib
-----------------

The name of the shared library is rtr. To link programs to the RTRlib,
pass the following parameter to gcc:

.. code-block:: Bash

    -lrtr

In case an error such as

| -/usr/bin/ld: cannot find -lrtr
| -collect2: error: ld returned 1 exit status
|

occurs while compiling, the library cannot be found. Pass the location to the compiler:

.. code-block:: Bash

    -L<path_to_librtr.so>


e.g.,

.. code-block:: Bash

    -L/usr/local/lib64/

The RTRlib includes a HTML documentation of the API. To build them, doxygen must be installed. The documentation will be located in the
`docs/` directory after the execution of:

.. code-block:: Bash

    make doc

Now, everything is set up. Here is an overview of the contents the RTRlib directories provide.


- `cmake/`      CMake modules
- `doxygen/`    Example code and graphics used in the Doxygen documentation
- `rtrlib/`     Header and source code files of the RTRlib
- `tests/`      Unit tests
- `tools/`      Contains the ``rtrclient`` and the ``cli-validator``


Usage Examples
--------------

The library comes with two command line tools to check the functionality. Both can be found in the `/tools` directory.
The first is the ``rtrclient``. With it, the establishment of a TCP or SSH connection can be tested, as well es printing
information about received updates. The program takes various parameter, the help prints:

| ./rtrclient tcp [options] <host> <port>
| ./rtrclient ssh [options] <host> <port> <username> <private_key> [<host_key>]
|
| Options:
| -k  Print information about SPKI updates.
| -p  Print information about PFX updates.
|
| Examples:
| ./rtrclient tcp rpki-validator.realmv6.org 8282
| ./rtrclient tcp -k -p rpki-validator.realmv6.org 8282
| ./rtrclient ssh rpki-validator.realmv6.org 22 rtr-ssh ~/.ssh/id_rsa ~/.ssh/known_hosts
| ./rtrclient ssh -k -p rpki-validator.realmv6.org 22 rtr-ssh ~/.ssh/id_rsa ~/.ssh/known_hosts
|

If the options are set, the client will output a lot of lines looking like this:

.. code-block:: Bash

  Entry added (+)/removed (-)   Announced Prefix        Prefix Length   ASN
                +               109.121.148.0           24 -  24        48057
                +               93.113.129.0            24 -  24        40975
                +               58.71.109.0             24 -  24         9299
                +               186.188.0.0             17 -  24        21826
                +               210.4.96.0              24 -  24        17639


The second tool is the ``cli-validator``, a command line tool used for prefix validation. Like the ``rtrclient`` this program requires the host and port
of the target RPKI cache. Upon successful connection, the user can enter a prefix in the format `Prefix Mask ASN` (separated by spaces), e.g.

| 93.175.146.0 24 12654
|

Note that both IPv4 and IPv6 Prefixes are allowed.

| 2000:7fb:fd02:: 48 12654
|

is just as valid.

Run the ``cli-validator`` with following parameters:

.. code-block:: Bash

    cli-validator rpki-validator.realmv6.org 8282


If the RTRlib was build with the `Debug` option (see chapter Build and Install RTRlib), there will be a lot of output about the status of the connection.
In case it was build in `Release` mode, no output except the result of the query will appear.
The result of the query will be a single digit after a pipe (``|``) at the end of the line:

- 0 = prefix is valid
- 1 = prefix was not found
- 2 = prefix is invalid
