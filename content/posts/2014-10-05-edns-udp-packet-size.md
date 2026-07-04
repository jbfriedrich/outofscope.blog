---
title: EDNS UDP packet size
date: 2014-10-05T13:52:03
feature_image: https://images.unsplash.com/photo-1532339848923-c4beffa7abcb?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ&s=24a4e38bb8b9e913fcc7bc041dd2ed2b
tags:
  - linux
  - networking
  - tech
languages:
  - en
---

A couple of weeks ago I set up a local BIND in a CentOS 6.5 VM to have an internal DNS server for my VMs to use. After creating several local zone files and successful initial tests, everything worked fine. Some domains had high query times, and SSH logins sometimes took a bit longer than expected, but I did not have enough time to investigate further.

Today I had some free time on my hands and decided to re-visit the issue. The DNS server works fine, but whenever the cache is empty, it takes up to 3098ms for a DNS query. Once the result is in cache, everything works as expected. To get a better overview I started with enabling debug logging in ´named.conf´:

```less
logging {
    channel default_debug {
        file "data/named.run";
        severity debug;
        print-time yes;
        print-severity yes;
        print-category yes;
    };
    category default {
        default_debug;
    };
};
```

After a restart of the DNS service I tested several domains and found this suspicious log entry:
  
> 05-Oct-2014 11:24:28.014 edns-disabled: debug 1: success resolving 'domain.com.dlv.isc.org/DLV' (in 'isc.org'?) after reducing the advertised EDNS UDP packet size to 512 octets

I then remembered something a friend of mine told me a couple of months ago. Hetzner is using several Firewall and anti-DDOS techniques to prevent attacks. One of these techniques is blocking UDP packets greater a specific size (which is not publicly revealed). I found a [knowledgebase](https://kb.isc.org/article/AA-00708/0/Why-does-BIND-log-messages-about-disabling-EDNS-or-reducing-the-advertised-packet-size.html) article and [a way to test](https://www.dns-oarc.net/oarc/services/replysizetest/) if this is a problem with my local DNS server, or the one remote. It looks like the [Hetzner](http://www.hetzner.de/) is blocking UDP packages which are greater than 1440 or sometimes 2200 bytes, sometimes I get even lower values. Pretty inconsistent results.

To be on the safe side, I edited the _named.conf_ and set the values for "edns-udp-size" and "max-udp-size" to 512 bytes:

```less
edns-udp-size           512;
max-udp-size            512;
```

This seems to work for me now. I also contacted [Hetzner](http://www.hetzner.de/) and applied for the UDP whitelist.
