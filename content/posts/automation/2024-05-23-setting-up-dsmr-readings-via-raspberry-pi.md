---
layout: post
title: Setting up DSMR Meter Readings via a Raspberry Pi
excerpt: "Writing down how I pull readings from my DSMR meter and send them to my Home Assistant server."
modified: 2024-05-23T00:00:00-00:00
image: https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/367da639-c5d0-4ec0-a7cf-3fda8eec61cd
categories: [automation]
tags: [dsmr, home-assistant, electricity, raspberry-pi]
comments: true
share: true
---

The internet is probably full of tutorials on how to do this right, but I had to struggle a bit to figure it out for myself. My Pi randomly crashed, the other day, and I'm having to do some of the setup again and having done this for the first time months ago (and having now mostly forgotten what I did), I decided I should blog about it as I retread my steps and recreate my setup.

> **Aside**: The [internet](https://en.wikipedia.org/wiki/Unix_shell) tells me that the C shell was the first to introduce the `history` command. It would not be hyperbolic to say that without `history` my task of recreating my setup would be significantly more difficult. I used it extensively to figure out what I ran when I needed to do this all those months ago. I should probably containerise or chefise or ansibilise or something.

I live in a country that uses [DSMR](https://www.domoticz.com/wiki/Dutch_DSMR_smart_meter_with_P1_port) meters and have one in my home. My girlfriend wanted a closer look at our electricity consumption, so I decided to hook up a raspberry Pi to the meter's [P1](https://www.fluvius.be/sites/fluvius/files/2020-03/1901-fluvius-technical-specification-user-ports-digital-meter.pdf) port and send those readings to my home lab server using the Pi as a relay.

I bought a RJ12 --> USB cable off of Amazon and tried to read from my Pi's USB port by using first [`cu`](https://linux.die.net/man/1/cu) and then [pyserial](https://pythonhosted.org/pyserial/index.html) (which is a much friendlier interface).

```shell
# Using cu
cu -l /dev/ttyUSB0 -s 115200 --parity=none

# Using pyserial
python3 -m serial.tools.miniterm /dev/ttyUSB0 115200 --xonxoff
```

Initially neither tool gave me any readings and I nearly tore my luscious hair out before finally realising that it was a bad cable I was using. I returned it and bought a more respectable cable from [robbshop.nl](https://www.robbshop.nl/slimme-meter-kabel-usb-p1-1-meter).

And that gave me results!

![image](https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/bdc7ee3d-9f61-480a-b3ec-22e04f5e558e)

I had a few different ways of turning the meter readings into useful insights:
1. **Run HomeAssistant on my Pi with direct access to the port**: Not an option since my HA instance was already running on my home lab server. The Pi was meant to be a relay only.
2. **DSMR to MQTT**: I could run something that would read the DSMR telegrams and send them to an MQTT broker to which I could subscribe my HA integration. Definitely an option.
3. **Serial to Network Proxy**: The option I ended up going with since it felt the simplest to me. I would run a program that would expose the DSMR telegrams over a TCP connection.

The program I landed on for doing this is [ser2net](https://ser2net.sourceforge.net/) With a little help from ChatGPT I figured out this [ser2net config](https://github.com/chargebyte/ser2net/blob/master/ser2net.conf). Comments describe what the various fields do.

```yaml
# defines an alias confver, I'm not entirely sure if this does anything
define: &confver 1.0

# begins a new connection definition and assigns it an alias `con00`.
connection: &con00
  # specifies how incoming network connections will be accepted, in this case the Telnet protocol with RFC 2217 support (I no clue what that means), and the connection will be over TCP and listen on port 2013c (this part I understand)
  accepter: telnet(rfc2217),tcp,2013
  # specifies how the serial device will be connected, includes path to the serial device file and port settings, and it is to be configured in local mode (not sure what local mode means)
  connector: serialdev,/dev/ttyUSB0,115200n81,local
```

I paired this with HA's [DSMR Slimme Meter](https://www.home-assistant.io/integrations/dsmr) integration with it set to listen on the network, pointed at my Pi on port 2013.

And we have lift-off!

![image](https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/f7a28b8c-afc3-4119-bbca-8f468227f24f)

> **Note**: Solar readings are coming in via a different integrations and configured together with the DSMR readings using HomeAssistant's Energy Dashboard.
