.. _usage:

*******************
Usage of the RTRlib
*******************

.. _install:

Build and Install the RTRlib
============================

The RTRlib supports most Linux distributions as well as Apple macOS.

Debian Linux
------------

If you are running Debian (Jessie), you can install the library via the APT
package manager, as follows

.. code-block:: Bash

    sudo apt-get install librtr0

Apple macOS
-----------

For macOS we provide *Homebrew* tap_ to easily install the RTRlib.
First, install Homebrew_ and afterwards install RTRlib as follows:

.. code-block:: Bash

    brew tap rtrlib/pils
    brew install rtrlib

.. _Homebrew: http://brew.sh
.. _tap: https://github.com/rtrlib/homebrew-pils

From Source
-----------

On any other OS you will have to build and install the RTRlib from source.
The following minimal requirements have to be met, before building the library:

- the build system is based on `cmake`, version >= 2.6
- `libssh` to establish SSH transport connections, version >= 0.5.0

Optional requirements are:

- `cmocka` a framework to run RTRlib unit tests
- `doxygen` to build the RTRlib API documentation


If the requirements are met, the library can be build.
First, either download or clone the RTRlib source code as follows:

.. code-block:: Bash

    wget https://github.com/rtrlib/rtrlib/archive/v0.3.6.tar.gz
    tar xzf v0.3.6.tar.gz
    cd rtrlib-0.3.6
    # or
    git clone https://github.com/rtrlib/rtrlib/
    cd rtrlib

The contents of the RTRlib source code has the following subdirectories:

- ``cmake/``      CMake modules
- ``doxygen/``    Example code and graphics used in the Doxygen documentation
- ``rtrlib/``     Header and source code files of the RTRlib
- ``tests/``      Function tests and unit tests
- ``tools/``      Contains ``rtrclient`` and ``cli-validator``

Afterwards build the library as follows, we recommend an `out-of-source` build
using the `cmake` build system:

.. code-block:: Bash

    mkdir build && cd build
    cmake -DCMAKE_BUILD_TYPE=Release ../
    make
    sudo make install

To enable debug symbols and messages, change the `cmake` command to:

.. code-block:: Bash

    cmake -D CMAKE_BUILD_TYPE=Debug .

If `Doxygen` is available, you can build the documentation, by running:

.. code-block:: Bash

    make doc

If the build command fails with any error, please consult the RTRlib README_
and Wiki_, you may also join our `mailing list`_.

.. _README: https://github.com/rtrlib/rtrlib/
.. _Wiki: https://github.com/rtrlib/rtrlib/wiki
.. _mailing list: https://groups.google.com/forum/#!forum/rtrlib

Development with the RTRlib
===========================

The RTRLib shared library is installed to ``/usr/local/lib`` and its headers
files to ``/usr/local/include`` respectively by default.
The name of the shared library is `rtr`. To link programs to the RTRlib,
pass the following parameter to gcc:

.. code-block:: Bash

    -lrtr

In case an error such as

| -/usr/bin/ld: cannot find -lrtr
| -collect2: error: ld returned 1 exit status
|

occurs while compiling, the library cannot be found.
Pass its location as an absolute path to the compiler:

.. code-block:: Bash

    -L<path_to_librtr.so>
