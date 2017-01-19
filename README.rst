============================
pyFirmata (Y Soft IOTA fork)
============================

pyFirmata is a Python interface for the `Firmata`_ protocol. It is compatible with Firmata 2.3.
It runs on Python 2.7, 3.3 and 3.4.

.. _Firmata: http://firmata.org

Test & coverage status:

.. image:: https://travis-ci.org/ysoftiota/pyFirmata.png?branch=master
    :target: https://travis-ci.org/ysoftiota/pyFirmata

.. image:: https://coveralls.io/repos/github/ysoftiota/pyFirmata/badge.svg?branch=master
    :target: https://coveralls.io/github/ysoftiota/pyFirmata?branch=master

Installation
============

Current could be only installed from source with ``python setup.py install``. You will
need to have `setuptools`_ installed::

    git clone https://github.com/tino/pyFirmata
    cd pyFirmata
    python setup.py install

.. _pip: http://www.pip-installer.org/en/latest/
.. _setuptools: https://pypi.python.org/pypi/setuptools


Usage
=====

Board init
----------

Library has autodetection function, which scans all opened ports and tries
to detect the only one connected board:

    >>> import pyfirmata
    >>> board = pyfirmata.util.autoload_board()
   
Ports are filtered with port description provided from the device, default
filtering is ``ports_filter="Arduino"`` (which filters for ports with
description starting as ``Arduino``), but it can be modified or fully disabled
by passing ``None``:

    >>> board = pyfirmata.util.autoload_board(ports_filter="YSoft IOTA Play")
    >>> board = pyfirmata.util.autoload_board(ports_filter=None)
    
If you have more devices connected to the computer at the same time or if you
want to specify port by hand, you could directly call board constructor:

    >>> board = pyfirmata.board('/dev/ttyACM0')
    >>> board = pyfirmata.board('COM10')

Basic usage
----------

    >>> board.digital[13].write(1)
    
To use analog ports, it is probably handy to start an iterator thread.
Otherwise the board will keep sending data to your serial, until it overflows::

    >>> it = util.Iterator(board)
    >>> it.start()
    >>> board.analog[0].enable_reporting()
    >>> board.analog[0].read()
    0.661440304938

If you use a pin more often, it can be worth it to use the ``get_pin`` method
of the board. It let's you specify what pin you need by a string, composed of
'a' or 'd' (depending on wether you need an analog or digital pin), the pin
number, and the mode ('i' for input, 'o' for output, 'p' for pwm). All
seperated by ``:``. Eg. ``a:0:i`` for analog 0 as input or ``d:3:p`` for
digital pin 3 as pwm.::

    >>> analog_0 = board.get_pin('a:0:i')
    >>> analog_0.read()
    0.661440304938
    >>> pin3 = board.get_pin('d:3:p')
    >>> pin3.write(0.6)

Board layout
============

Board layout is loaded directly from the board, but if you want to specify
board layout by hand (and disable loading from the board), you could do it.

There are two shortcut classes ``pyfirmata.Arduino`` and ``pyfirmata.ArduinoMega``,
which instantiate the Board class with a given layout dictionary:

    >>> from pyfirmata import Arduino
    >>> board = Arduino('/dev/ttyACM0')

You could also  as the ``layout`` argument and specify your own layout (but
we think that the autoloading is the best alternative). This is the layout
dict for the Mega for example::

    >>> mega = {
    ...         'digital' : tuple(x for x in range(54)),
    ...         'analog' : tuple(x for x in range(16)),
    ...         'pwm' : tuple(x for x in range(2,14)),
    ...         'use_ports' : True,
    ...         'disabled' : (0, 1, 14, 15) # Rx, Tx, Crystal
    ...         }
