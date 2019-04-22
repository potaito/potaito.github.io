---
layout: post
title:  "My pfSense logs are all empty"
date:   2019-04-22 16:00:00 +0100
categories: pfsense
---

*Running pfSense version 2.4.4.*

I don't know why, but my pfSense firewall simply stopped logging at some point. Possibly after an upgrade. All the system logs, service logs and firewall events that are usually stored in `/var/log` were not logged anymore. Even a factory reset did not clear things up. What did help though was to delete all the files:

```shell
rm -rf /var/log/*
```

Alternatively there is also a GUI button in the logging settings, that resets all logs. It should do the same thing.

After a reboot the logs were re-created by the corresponding services:
```shell
[2.4.4-RELEASE][root@pfSense.lan]: ls -la /var/log
total 83804
drwxr-xr-x   5 root  wheel      512 Apr 22 17:12 .
drwxr-xr-x  26 root  wheel      512 Dec 12 13:42 ..
-rw-r--r--   1 root  wheel  5000000 Apr 22 17:56 dhcpd.log
-rw-r--r--   1 root  wheel  5000000 Apr 22 17:58 filter.log
-rw-r--r--   1 root  wheel  5000000 Apr 22 15:10 gateways.log
-rw-r--r--   1 root  wheel  5000000 Apr 22 15:10 ipsec.log
-rw-r--r--   1 root  wheel  5000000 Apr 22 15:10 l2tps.log
drwx------   2 www   www        512 Oct  3  2018 lighttpd
-rw-r--r--   1 root  wheel  5000000 Apr 22 17:56 nginx.log
-rw-r--r--   1 root  wheel  5000000 Apr 22 17:40 ntpd.log
-rw-r--r--   1 root  wheel  5000000 Apr 22 15:10 openvpn.log
drwxr-xr-x   2 root  wheel      512 Apr 22 17:40 pfblockerng
-rw-r--r--   1 root  wheel  5000000 Apr 22 15:10 poes.log
-rw-r--r--   1 root  wheel  5000000 Apr 22 15:10 portalauth.log
-rw-r--r--   1 root  wheel  5000000 Apr 22 15:10 ppp.log
-rw-r--r--   1 root  wheel  5000000 Apr 22 15:10 relayd.log
-rw-r--r--   1 root  wheel  5000000 Apr 22 17:57 resolver.log
-rw-r--r--   1 root  wheel  5000000 Apr 22 15:10 routing.log
drwxr-xr-x   3 root  wheel      512 Apr 22 16:45 snort
-rw-r--r--   1 root  wheel  5000000 Apr 22 17:57 system.log
-rw-r--r--   1 root  wheel      394 Apr 22 17:55 utx.lastlogin
-rw-r--r--   1 root  wheel      319 Apr 22 17:55 utx.log
-rw-r--r--   1 root  wheel  5000000 Apr 22 15:10 vpn.log
-rw-r--r--   1 root  wheel  5000000 Apr 22 17:58 wireless.log
```

By the way: the log-size is static as the logs are stored in form of a ring-buffer. Hence why they all have the same file size. 

What was wrong? I have no idea. The file permissions are all the same as they were before I manually deleted the log files. I even tried switching to a RAM disk for the logs, but that also did not sort out the problem. In any event, now the logs are working again!