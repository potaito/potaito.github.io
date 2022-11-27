---
layout: post
title:  "pfBlockerNG DNSBL failing to load"
date:   2019-04-22 17:00:00 +0100
categories: pfsense pfblockerng
---

This post covers some trouble I had with pfBlockerNG not reloading the `unbound` DNS resolver properly. As it turns out, it was an issue with a config file, probably caused by an update.

## Setup
I wanted to setup pfBlockerNG on my pfSense router, which consists of two parts: IP blocking and DNS "black-holing". 
1. The IP-based blocking worked fine out of the box. Its concept is fairly straight forward as it adds a firewall rule that drops packets to and from "bad" IP addresses. These addresses are found online in curated lists. 
2. What I had trouble with though was the DNS Blocker `DNSBL`, the other half of pfBlockerNG. In particular when I forced a reload of pfBlockerNG, it would fail to reload `unbound`, which is the DNS Resolver service on pfSense. `DNSBL` relies on `unbound` in the sense that it adds itself to `unbound` and then handles specific DNS request that are black-listed. 

## Analysis
The following is an excerpt of the pfBlockerNG log during the reload process:

```
------------------------------------------------------------------------
Assembling DNSBL database... completed [ 04/22/19 17:35:28 ]
Reloading Unbound Resolver..
DNSBL enabled FAIL - restoring Unbound conf *** Fix error(s) and a Force Reload required! ***
[1555947403] unbound-control[76777:0] warning: control-enable is 'no' in the config file.
[1555947478] unbound-control[76777:0] error: connect: Operation timed out for 127.0.0.1 port 8953 Restore previous database Failed! ..... Not completed. [ 04/22/19 17:40:29 ]
```

So apparently pfBlockerNG tries to restart the `unbound` service, but fails doing so because of the error  "*control-enable is 'no' in the config file*". By the way: In older versions of pfBlockerNG it would only print that it failed to reload `unbound` without further explanation.

When searching for "*control-enable is 'no'*" online, I stumbled upon a [netgate forum entry](https://forum.netgate.com/topic/142446/should-unbound-control-work-by-default), where user **jimp** explains that apparently `unbound` has an issue with a 0-byte config file. I checked on my device and sure enough the file `remotecontrol.conf` existed but was empty:

```shell
[2.4.4-RELEASE][alessandro@pfSense.lan]/var/unbound: ls -lah
total 40
drwxr-xr-x   3 unbound  unbound   512B Apr 22 17:53 .
drwxr-xr-x  26 root     wheel     512B Dec 12 13:42 ..
-rw-r--r--   1 root     unbound   252B Apr 22 17:53 access_lists.conf
drwxr-xr-x   2 unbound  unbound   512B Dec 12 13:42 conf.d
-rw-r--r--   1 root     unbound     0B Apr 22 17:53 dhcpleases_entries.conf
-rw-r--r--   1 root     unbound   3.3K Apr 22 17:40 dnsbl_cert.pem
-rw-r--r--   1 root     unbound     0B Apr 22 17:53 domainoverrides.conf
-rw-r--r--   1 root     unbound   791B Apr 22 17:53 host_entries.conf
-rw-r--r--   1 root     unbound     0B Apr 22 17:37 pfb_dnsbl.conf
-rw-r--r--   1 root     unbound   1.5K Apr 22 17:40 pfb_dnsbl_lighty.conf
-rw-r--r--   1 root     unbound     0B Dec 10  2017 remotecontrol.conf
-rw-r--r--   1 unbound  unbound   759B Apr 22 17:53 root.key
-rw-r--r--   1 unbound  unbound   1.2K Dec 10  2017 root.key.8966-1
-rw-r--r--   1 root     unbound   2.1K Apr 22 17:53 unbound.conf
-rw-r-----   1 unbound  unbound     0B Dec 10  2017 unbound_control.key
-rw-r-----   1 unbound  unbound     0B Dec 10  2017 unbound_control.pem
-rw-r-----   1 unbound  unbound     0B Dec 10  2017 unbound_server.key
-rw-r-----   1 unbound  unbound     0B Dec 10  2017 unbound_server.pem
```

## Fixing the problem
It took the following steps to resolve the problem:
0. Stop the `unbound` service via GUI.
1. Delete the empty files. In my case the key files `unbound_control.key` etc. were also empty, so these needed to be removed as well. Otherwise `unbound` would not start:
    ```shell
    rm /var/unbound/remotecontrol.conf
    rm /var/unbound/unbound_*
    ```
2. Back in the GUI and in the "DNS Resolver" service settings: Simply hit "save" without making any modifications. This should regenerate the files that were deleted before. 
3. Restart the `unbound` service via GUI. Checking the `/var/unbound` directory afterwards shows that the deleted files were all restored and this time larger than 0 bytes:
    ```
    [2.4.4-RELEASE][root@pfSense.lan]/var/unbound: ls -la
    total 60
    drwxr-xr-x   3 unbound  unbound   512 Apr 22 18:19 .
    drwxr-xr-x  26 root     wheel     512 Dec 12 13:42 ..
    -rw-r--r--   1 root     unbound   252 Apr 22 18:19 access_lists.conf
    drwxr-xr-x   2 unbound  unbound   512 Dec 12 13:42 conf.d
    -rw-r--r--   1 root     unbound     0 Apr 22 18:19 dhcpleases_entries.conf
    -rw-r--r--   1 root     unbound  3355 Apr 22 17:40 dnsbl_cert.pem
    -rw-r--r--   1 root     unbound     0 Apr 22 18:19 domainoverrides.conf
    -rw-r--r--   1 root     unbound   791 Apr 22 18:19 host_entries.conf
    -rw-r--r--   1 root     unbound     0 Apr 22 17:57 pfb_dnsbl.conf
    -rw-r--r--   1 root     unbound  1498 Apr 22 17:40 pfb_dnsbl_lighty.conf
    -rw-r--r--   1 root     unbound   300 Apr 22 18:19 remotecontrol.conf
    -rw-r--r--   1 unbound  unbound   759 Apr 22 18:19 root.key
    -rw-r--r--   1 unbound  unbound  1252 Dec 10  2017 root.key.8966-1
    -rw-r--r--   1 root     unbound  2257 Apr 22 18:19 unbound.conf
    -rw-r-----   1 unbound  unbound  2459 Apr 22 18:19 unbound_control.key
    -rw-r-----   1 unbound  unbound  1330 Apr 22 18:19 unbound_control.pem
    -rw-r-----   1 unbound  unbound  2459 Apr 22 18:19 unbound_server.key
    -rw-r-----   1 unbound  unbound  1318 Apr 22 18:19 unbound_server.pem
    ```

4. Now that `unbound` is happy again, go to pfBlockerNG's update tab and force a reload. This time `unbound` started without any issues:
    ```
    ------------------------------------------------------------------------
    Assembling DNSBL database... completed [ 04/22/19 18:22:04 ]
    Reloading Unbound Resolver..... completed [ 04/22/19 18:22:16 ]
    DNSBL update [ 141195 | PASSED  ]... completed
    ------------------------------------------------------------------------
    ```

## Testing

By default pfBlockerNG has easylists that block some of the most common trackers and malicious URLs. One of the lists loaded is this one: https://easylist-downloads.adblockplus.org/easyprivacy.txt. As a final test, we can form a DNS query for a domain that appears in this list and should thus be black-holed. 

```shell
$ dig @192.168.1.1 google-analytics.com

; <<>> DiG 9.10.3-P4-Ubuntu <<>> @192.168.1.1 google-analytics.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22775
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;google-analytics.com.		IN	A

;; ANSWER SECTION:
google-analytics.com.	60	IN	A	10.10.10.1

;; Query time: 3 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Mon Apr 22 18:25:05 CEST 2019
;; MSG SIZE  rcvd: 65
```

As can be seen, `dig` returns the IP address `10.10.10.1` for this particular domain, which is not the correct address. Therefore any HTTP(S) request to this domain would would end up in the void, which is good. 

A note about `dig`: With `dig @<ip>` it is possible to query a specific DNS server with a name resolution request. In this case the device running pfSense is what we want to test. Without the extra `@` argument, the local machine's (Ubuntu or whatever you are on) DNS proxy like `dnsmasq` might return a cached answer for the domain. Likewise a test with the `ping` command as suggested in some tutorials can give wrong results for the same reasons.

*At time of writing I was running pfSense version 2.4.4 and pfBlockerNG v2.2.5_22.*