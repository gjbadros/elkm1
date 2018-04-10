# Python ElkM1 library

Library for interacting with ElkM1 alarm/automation panel.

https://github.com/gwww/elkm1

## Requirements

- Python 3.5 (or higher)

## Description

This package is created as a library to interact with an ElkM1 alarm/automation
pattern. The motivation to write this was to use with the Home Assistant
automation platform. The library can be used for writing other ElkM1 integration
applications. The IO with the panel is asynchronous over TCP or over the
serial port (serial port not implemented yet).

## Installation

```bash
    $ pip install elkm1
```

## Overview

Connect to the Elk panel:

```python
    from elkm1 import Elk

    elk = Elk({'url': 'elk://192.168.1.100'})
    elk.loop.run_until_complete(elk.connect())
    elk.run()
```

The above will connect to the Elk panel at IP address 192.168.1.100. the `elk://`
prefix specifies that the connect is plaintext. Alternatively, `elks://` will 
connect over TLS. In this case a userid and password must be specified
and the call to `Elk` would be:

```python
    elk = Elk({'url': 'elks://192.168.1.100',
                  'userid': 'testuser', 'password': 'testpass'})
```

The `Elk` object supports the concept of `Elements`. An `Element`
is the base class representation of `Zones`, `Lights`, etc. So, for
example there is a list of zones: `elk.zones` and each zone can be
accessed by `elk.zones[index]`. Each element has a `__str__`
representation so that it is easy to print its contents.

All `Elements` are referenced starting at 0. Even though the Elk panel
refers to, for example, zones 1-208, the library references them
as zones 0-207. All translation from base 0 to 1 and vice-versa is
handled internally in the `elkm1.message` module.

After creating the `Elk` object and connecting to the panel the 
library code will synchronize all the elements to the data from the Elk panel.

Many Elk messages are handled by the library, caching their contents. When a
message causes a change to an attribute of an `Element`,
callbacks are called so that user use of the library can be notified
of changing elements. The following user code shows registering a callback:

```python
    def call_me(attribute, value):
       print(attribute_that_changed, new_value)

    for zone_number in range(Max.ZONES.value):
      elk.zones[zone_number].add_callback(call_me)
```

The library encodes, decodes, and processes messages to/from the
Elk panel. All the encoding and decoding is done in `elkm1.message` module.

Messages received are handled with callbacks. The library 
internally registers callbacks so that decoded messages 
can be used to update an `Element`. The user of the
library may also register callbacks. Any particular message
may have multiple callbacks.

When the message is received it is decoded 
and some validation is done. The message handler is called
with the fields of from the decoded message. Each type of 
message has parameters that match the message type. All handler parameters
are named parameters.

Here is an example of a message handler being registered and how it is called:

```python
    def zone_status_change_handler(zone_number, zone_status):
      print(zone_number, zone_status)

    add_message_handler('ZC', zone_status_change_handler)
```

The above code registers a callback for 'ZC' (Elk zone status change)
messages. When a ZC message is received the handler functions are called
with the zone_number and zone_status.

## Utilities

The `bin` directory of the library has one utility program and
a couple of example uses of the library.

### `mkdoc`

The utility `mkdoc` creates a Markdown table of the list of Elk
messages with a check mark for those messages have encoders/decoders
and an X for those messages are not planned to be implemented.
There are no parameters to `mkdoc`. It outputs to stdout.
The data for the report comes from the ElkM1 library code mostly.
A couple of things are hard coded in the mkdoc script, notably
the "no plans to implement" list.

### `simple`

The `simple` Python script is a trivial use of the ElkM1 library.
It connects to the panel, syncs to internal memory, and
continues listening for any messages from the panel.

### `elk`

The `elk` Python script is a bit of a command interpretor. It can run in
two modes. Non-interactive mode is the default. Just run the `elk` command.
The non-interactive mode is similar to `simple` except there are a
couple of message handlers (`timeout` and `unknown` handlers).

The `elk` can also be run in interactive mode by invoking it by
`elk -i`. In this mode is uses curses (full screen use of the terminal)
that has a command line and an output window. `TAB` switches between
the command line and output windows. In the output window the arrow keys
and scrollwheel scroll the contents of the window.

In the command line there are a
number of commands. Start with `help`. Then `help <command>` for 
details on each command. In general there are commands to dump the internal
state of elements and to invoke any of the encoders to send a message 
to the Elk panel.

For example, `light <4, 8, 12-14` would invoke the `__str__` method
for the light element to print the cached info for lights 0-3, 8, and 12-14.

Another example would be `pf 3` which issues the pf (Turn light off)
command for light number 3 (light 4 on the panel -- remember 0
versus 1 base).

All of the commands that send messages to the panel are automatically
discovered and are all the XX_encode functions in the ``elkm1.message``
module. The docstring and the XX_encode's parameters are shown as part
of the help.