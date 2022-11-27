---
layout: post
title:  "How to bridge Wifi and LAN in pfSense"
date:   2020-06-01 16:00:00 +0100
categories: pfSense technical
excerpt: The pfSense router that I built contains a WiFi card, besides the conventional LAN ports. The workflow in pfSense requires setting up these network interfaces as separate networks. This means that the networks will be on different subnets. But being on separate subnets breaks some applications, for example sending content from a phone to a smart TV or chromecast. These devices are usually discovered with broadcasts, and as such all devices need to be in the same subnet. This is why I bridged the LAN and WiFi network interfaces in pfSense, making them appear as a single virtual network
---


![pfSense Logo](/images/pfsenes_logo.png "pfSense")

The pfSense router that I built contains a WiFi card, besides the conventional LAN ports. The workflow in pfSense requires setting up these network interfaces as separate networks. This means that the networks will be on different subnets. But being on separate subnets breaks some applications, for example sending content from a phone to a smart TV or chromecast. These devices are usually discovered with broadcasts, and as such all devices need to be in the same subnet. This is why I bridged the LAN and WiFi network interfaces in pfSense, making them appear as a single virtual network. Below is a quick step by step on how to achieve this: 

1. Create LAN with the network interface IP being 192.168.1.1, and a WiFi interface with IP 192.168.2.1 as usual. Skip this step if these interfaces already exist.
2. In the WiFi interface, allow Intra-BSS Communication such that WiFi devices can discover and talk to one another.
3. Create a bridge interface: Interfaces > Assign > Bridges > Create a Bridge
   - Select LAN and WiFi interfaces for bridging.
   - Assign a static IP address for the bridge. This will be the gateway for all devices that connect to the network later. For example: 192.168.0.1
4. Go to Firewall -> Rules and add a "Allow BRIDGE to ANY using all protocols" rule for the bridge, such that it can communicate outwards.
5. Disable DHCP servers on LAN and WiFi, enable only DHCP on Bridge interface.
6. Edit the DNS Server to only handle the bridge, not the LAN and WiFi interfaces.