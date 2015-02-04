---
layout: default6
category: links
title: Server mode
---

The Joshua decoder can be run as a TCP/IP server instead of a POSIX-style command-line tool. Clients can concurrently connect to a socket and receive a set of newline-separated outputs for a set of newline-separated inputs.

Threading takes place both within and across requests.  Threads from the decoder pool are assigned in round-robin manner across requests, preventing starvation.


# Invoking the server

A running server is configured at invokation time. To start in server mode, run `joshua-decoder` with the option `-server-port [PORT]`. Additionally, the server can be configured in the same ways as when using the command-line-functionality.

E.g.,

    $JOSHUA/bin/joshua-decoder -server-port 10101 -mark-oovs false -output-format "%s" -threads 10

## Using the server

To test that the server is working, a set of inputs can be sent to the server from the command line. 

The server, as configured in the example above, will then respond to requests on port 10101.  You can test it out with the `nc` utility:

    wget -qO - http://cs.jhu.edu/~post/files/pg1023.txt | head -132 | tail -11 | nc localhost 10101

Since no model was loaded, this will just return the text to you as sent to the server.

The `-server-port` option can also be used when creating a [bundled configuration](bundle.html) that will be run in server mode.
