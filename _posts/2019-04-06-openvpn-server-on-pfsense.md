---
layout: post
title:  "OpenVPN Server on pfSense"
date:   2019-04-06 09:00:00 +0100
categories: pfsense openvpn
---

The following settings can be used as a basic configuration for a OpenVPN server running on pfSense.

1. **CA:** Create Certificate Authority and a server certificate
2. **Dynamic DNS:** (Optional) In case of a non-static WAN IP address, get a dynamic DNS domain. I recommend [FreeDNS](https://freedns.afraid.org).
3. Install the `openvpn-client-export` package, which automatically generates configurations to be used by the clients.
4. Create OpenVPN server with following settings
   - Protocol: UDP on IPv4 only
   - Device node: tunnel
   - Local port: 1194
   - `IPv4 Tunnel Network`: Separate network than LAN. This is crucial. The alternative would be bridging clients directly into LAN. I used `192.168.3.0/24` as tunnel network, while my LAN is `192.168.1.0/24`.
   ![pfSense openVPN tunnel network]({{ site.url }}/images/pfsense_openvpn_tunnel_settings.png "pfSense OpenVPN tunnel network")

5. Assign and activate the OpenVPN network interface. **This is the step I forget every time!** pfSense does not add and enable interfaces on its own. `Interfaces -> Assignments -> Add ovpns1 ()` and then enable it. No need for any other setting.
   ![pfSense openVPN NIC]({{ site.url }}/images/pfsense_openvpn_interface.png "pfSense OpenVPN NIC")

6. Create users if there are none yet and then export client configurations.
