---
layout: post
title:  "Testing and troubleshooting MQTT"
date:   2019-03-24 08:00:00 +0100
categories: mqtt development linux technical
excerpt: In home-automation, MQTT is a popular messaging system based on the principle of publishers and subscribers. Sometimes it is a bit annoying to debug though, especially when it's used for the first time with a new system. For example microcontrollers and a home automation hub based on OpenHAB as in my case.
---

![ESP8266 with DHT33 sensor]({{ site.url }}/images/eps8266_dht.jpg "ESP8266 DHT ")
*The ESP8266 microcontroller that would not connect to my MQTT broker*

## TL;DR
```shell
# Install
sudo apt-get install mosquitto mosquitto-client

# MQTT Pub
mosquitto_pub -t sometopic  -h <broker_IP> -m <message string>

# MQTT Sub
mosquitto_sub -t sometopic  -h <broker IP> -m <message string>
```

## Preparation
In home-automation, MQTT is a popular messaging system based on the principle of publishers and subscribers. Sometimes it is a bit annoying to debug though, especially when it's used for the first time with a new system. For example microcontrollers and a home automation hub based on [OpenHAB](https://www.openhab.org) as in my case.

As always, instead of jumping immediately to the final solution and trying to make it work, it is often easier to begin with incremental steps. In my case this meant first debugging the communication of the microcontorller, and then the central hub. For this purpose I used an Ubuntu machine and two packages, `mosquitto` and `mosquitto-clients`:

```
sudo apt-get install mosquitto mosquitto-clients
```

`mosquitto` installs a message broker which is the central entity. `mosquitt-client` provides CLI commands for connecting to an MQTT broker and then publishing and subscribing to topics.

## Testing the local broker
Let's first test the newly installed broker. One method of checking whether it's running is to lits the currently active network sockets with

```bash
sudo netstat -tulpn
```

which prints a list of services that listen on some network port. There should be at least one entry for mosquitto, two if *IPv6* is active:
```
tcp    0   0   0.0.0.0:1883   0.0.0.0:*   LISTEN   1126/mosquitto
tcp6   0   0   :::1883        :::*        LISTEN   1126/mosquitto
```

Note that the broker is by default listening on port `1883`. Now subscribe to a test topic:

```
mosquitto_sub -t sometopic
```

and then in a new terminal start publishing messages:

```
mosquitto_pub -t sometopic -m 'hello there'
```

The subscriber should now receive and print the message.

## Testing the production broker
The MQTT broker that will ultimately be used is probably some headless raspberrypi or server. In order to test it, the same procedure as before can be used. Only this time we specify an IP address for the host running the MQTT broker:

```shell
# Subscribe on the local machine
# In this example the broker is on 192.168.1.10
mosquitto_sub -t sometopic -h 192.168.1.10

# Publish on the local machine
mosquitto_pub -t sometopic  -h 192.168.1.10 -m 'hello there'
```

## Testing the Publisher
Configure the MQTT publisher to connect to the local machine's IP address where the broker is running and start publishing messages. Again, check with `mosquitto_sub` if the messages reach the broker. If not this could indicate a network problem such as broken routing or filtered ports. As a firs step in debugging MQTT it's helpful to have the publisher and subscriber in the same (WiFi-) network.