Pelagicorized version of arkose-dbus-proxy
------------------------------------------
For ubuntu, you'll want the libdbus-glib-1-dev package installed

Running
=======
To run the proxy, make it, and then try:

    ./dbus-proxy /tmp/my_proxy_socket bus-type < example_conf.json

Where:

* `/tmp/my_proxy_socket` is the socket to create for communication.
* `bus-type` should be set to either `session` or `system`.
* `example_conf.json` is the configuration file to use.

You can then interact with the socket via, for instance, d-feet.

Configuration files
===================
The Pelagicorized version of the Arkose D-Bus proxy is configured using JSON
files (as opposed to the pure text files used in the original program. The
idea is that the Application Manager and the Pelagicorized D-Bus proxy should
be able to share configuration files.

The content of the JSON file can vary as long as the "dbus-gateway-config-<bustype>"
attribute holds a JSON array of JSON objects with certain name/value pairs,
example_conf.json shows an example of how the JSON configuration files should be
structured.

A note on 'direction' in the configuration:
The values used for direction is 'outgoing' and 'incoming'. One way to picture it
is to consider anything connecting to the bus specified when starting dbus-proxy
as being on the 'inside' and thus any interaction with the bus specified when
starting dbus-proxy is 'outgoing'.

A word on eavesdropping connections
===================================
In the D-Bus proxy, eavesdropping connections such as the dbus-monitor  will be
ignored. That is, if an eavesdropping connection receives a message, the proxy
will not consider the message to have been handled yet.

Allowing eavesdropping is considered a system configuration and is done in the
D-Bus configuration files, usually located in either /etc/dbus-1/session.conf
or in configuration include files in /etc/dbus-1/session.d/.