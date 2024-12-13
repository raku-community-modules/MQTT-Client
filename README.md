[![Actions Status](https://github.com/raku-community-modules/MQTT-Client/actions/workflows/linux.yml/badge.svg)](https://github.com/raku-community-modules/MQTT-Client/actions) [![Actions Status](https://github.com/raku-community-modules/MQTT-Client/actions/workflows/macos.yml/badge.svg)](https://github.com/raku-community-modules/MQTT-Client/actions) [![Actions Status](https://github.com/raku-community-modules/MQTT-Client/actions/workflows/windows.yml/badge.svg)](https://github.com/raku-community-modules/MQTT-Client/actions)

NAME
====

MQTT::Client - Minimal MQTT v3 client interface for Perl 6

SYNOPSIS
========

```raku
use MQTT::Client;

my $m = MQTT::Client.new('test.mosquitto.org');
await $m.connect;

$m.publish("hello-world", "$*PID is still here!");

react {
    whenever $m.subscribe("typing-speed-test.aoeu.eu") {
        say "Typing test completed at { .<message>.decode("utf8-c8") }";
    }

    whenever Supply.interval(10) {
        $m.publish("hello-world", "$*PID is still here!");
    }
}
```

METHODS
=======

new($server, $port)
-------------------

Returns a new `MQTT::Client` object. Note: it does not automatically connect. Before doing anything, `.connect` should be called!

connect
-------

Attempts to connect to the MQTT broker, and returns a `Promise` that will be kept after connection confirmation from the broker.

connection
----------

Returns a `Promise` that isn't kept until the connection ends.

publish(Str $topic, Buf|Str $message)
-------------------------------------

retain(Str $topic, Buf|Str $message)
------------------------------------

Publish a message to the MQTT broker. If the $message is a Str, it will be UTF-8 encoded. Topics are always UTF-8 encoded in MQTT.

subscribe($topic)
-----------------

Subscribes to the topic, returning a `Supply` of messages that match the topic. Note that messages that are matched by multiple subscriptions will be passed to all of the supplies that match.

Tap the supply to receive, for each message, a hash with the keys `topic`, `message`, and a boolean `retain`. Note that `message` is a `Buf` (binary buffer), which in most cases you will need to `.decode` before you can use it.

FUNCTIONS
=========

MQTT::Client::filter_as_regex(topic_filter)
-------------------------------------------

Given a valid MQTT topic filter, returns the corresponding regular expression.

NOT YET IMPLEMENTED
===================

The following features that are present in Net::MQTT::Simple for Perl, have not yet been implemented in `MQTT::Client` for Raku:

  * dropping the connection when a large message is received

  * unsubscribe

  * automatic reconnecting

  * SSL

  * connection timeouts

  * simple functional interface

NOT SUPPORTED
=============

  * QoS (Quality of Service)

Every message is published at QoS level 0, that is, "at most once", also known as "fire and forget".

  * DUP (Duplicate message)

Since QoS is not supported, no retransmissions are done, and no message will indicate that it has already been sent before.

  * Authentication

No username and password are sent to the server.

  * Last will

The server won't publish a "last will" message on behalf of us when our connection's gone.

  * Large data

Because everything is handled in memory and there's no way to indicate to the server that large messages are not desired, the connection is dropped as soon as the server announces a packet larger than 2 megabytes.

  * Validation of server-to-client communication

The MQTT spec prescribes mandatory validation of all incoming data, and disconnecting if anything (really, anything) is wrong with it. However, this minimal implementation silently ignores anything it doesn't specifically handle, which may result in weird behaviour if the server sends out bad data.

Most clients do not adhere to this part of the specifications.

AUTHOR
======

Juerd Waalboer

COPYRIGHT AND LICENSE
=====================

Copyright 2015 - 2019 Juerd Waalboer

Copyright 2024 Raku Community

This library is free software; you can redistribute it and/or modify it under the Artistic License 2.0.

SEE ALSO
========

[Net::MQTT::Simple](https://metacpan.org/pod/Net::MQTT::Simple) inr Perl

