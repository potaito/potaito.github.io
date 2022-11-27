---
layout: post
title:  "OpenVPN Setup on pfSense"
date:   2019-04-06 09:00:00 +0100
categories: pfsense openvpn
excerpt: The following settings can be used as a basic configuration for a OpenVPN server running on pfSense
---

The following settings can be used as a basic configuration for a OpenVPN server running on pfSense.

## Setting up the OpenVPN Server

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


## Setting up the users

Create a user in pfSense's user manager: `System -> User Manager`

- Specify Username
- Specify a password
- Save

Then add a certificate. For that, edit a user that was previously created. Then, under `User Certificates` click `+ Add`. Fill out the following information

- Certificate Authority: Should be the same as used in the setup of the Server.
- Key length: Leave default (2048)
- Digest Algorithm: Leave default (sha256)
- Lifetime (days): Leave default (3650 days)
- Common Name: Can be anything
- Certificate Type: User Certificate

That's it for setting up the user.

## Client export
`VPN -> OpenVPN -> Client Export`

At the top specify the field `Host Name Resolution` and pick a static IP address or a Dynamic DNS entry that was configured before, for instance using [FreeDNS](https://freedns.afraid.org). At the bottom of the page there should be a list with all configured users and there client export options. I had the best sucess with **Bundled Configurations: Archive**, and then connecting to the OpenVPN this way (tested on Ubuntu 18.04):

```
sudo apt install openvpn
sudo openvpn --config your-config.ovpn
```