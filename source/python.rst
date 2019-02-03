*********************
RTRlib Python Binding
*********************

The RTRlib is also available for scripting in Python using the
`RTRlib Python binding` [#rtrlib-python]_.
This section gives a quick overview on the usage of the Python binding.
An even more detailed documentation on the API and further usage examples can
be found on the corresponding `readthedocs.io` [#readthedocs]_ page.

Installation
============

The RTRlib Python binding runs on Linux and Apple macOS like the C library. It
supports both Python 2 and Python 3, in any recent release, detailed requirements
and install instructions are described here.

Getting Started
---------------

The Python binding for the RTRlib has several dependencies. For compilation
it requires the following external packages to be installed:

- Python, version 2.7 or 3.x
- C Compiler
- RTRlib C library

To use the Python binding, the following Python packages have to be installed
as well:

- cffi>=1.4.0
- enum34
- six

If you are using *virtualenv*, these are installed automatically during the
install step, otherwise you have to use your platforms package management tool
or just run ``pip install -r requirements.txt``.

Building and Installation
-------------------------

The setup process of the RTRlib Python binding is straight forward and complies
to well-known Python standards.
First, download the source code from Github:

.. code-block:: bash

    git clone https://github.com/rtrlib/python-binding.git
    cd python-binding

And second, build and install the package using Python commands:

.. code-block:: bash

    python setup.py build
    python setup.py install

Step-by-Step Example
====================

The following code listings show how to implement a simple RPKI validator based
on the RTRlib Python binding. The functionality basically reflects the
``rpki-rov`` tool shipped with the RTRlib C library (see
:ref:`rtrlib_tools_rpki-rov`).

----

First, import required Python packages as shown in :numref:`lst-python-import`;
namely `rtrlib`, but also some future imports in case of Python 2.

.. code-block:: Python
    :linenos:
    :caption: Import RTRlib package
    :name: lst-python-import

    # uncomment future imports, required for Python 2
    #from __future__ import print_function
    from rtrlib import RTRManager, PfxvState

Afterwards, initialized and start an instance of the *RTRManager*,
see :numref:`lst-python-rtrmgr`, mandatory parameters are *host* and *port* of
a trusted RPKI cache server.

.. code-block:: Python
    :linenos:
    :caption: Setup and run RTRManager
    :name: lst-python-rtrmgr

    mgr = RTRManager('rpki-validator.realmv6.org', 8282)
    mgr.start()

As soon as the *RTRManager* is up and running, it can validate any prefix to
origin AS relation as shown in :numref:`lst-python-validate`. The return value
in result contains the corresponding validation state, i.e., *valid*, *invalid*,
or *not_found*; other return values indicate an error during validation.

.. code-block:: Python
    :linenos:
    :caption: Validate prefix to origin AS relation
    :name: lst-python-validate

    result = mgr.validate(12345, '10.10.0.0', 24)
    if result == PfxvState.valid:
        print('Prefix Valid')
    elif result == PfxvState.invalid:
        print('Prefix Invalid')
    elif result == PfxvState.not_found:
        print('Prefix not found')
    else:
        print('Invalid response')


.. [#rtrlib-python] RTRlib Python binding -- https://github.com/rtrlib/python-binding
.. [#readthedocs]   ReadTheDocs --  https://python-rtrlib.readthedocs.io
