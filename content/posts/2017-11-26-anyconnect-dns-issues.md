---
title: Cisco AnyConnect Client DNS issues
date: 2017-11-26T15:01:50
feature_image: https://images.unsplash.com/photo-1532339848923-c4beffa7abcb?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ&s=24a4e38bb8b9e913fcc7bc041dd2ed2b
summary: "Since I started using the Cisco AnyConnect VPN Client, I had DNS issues. I could not fully figure out why this happens on my system, but all signs lead to an issue with scoped DNS queries on macOS."
tags:
  - apple
  - networking
  - tech
languages:
  - en
---

Whenever the VPN is established, macOS uses the wrong order to resolve DNS entries. This leads to DNS query timeouts, but under certain circumstances to complete DNS failures, as the system insists on using the unreachable DNS server.

The problem exists for a long time now, at least since [macOS Mountain Lion](https://apple.stackexchange.com/questions/73076/mac-os-x-mountain-lion-dns-resolving-uses-wrong-order-on-vpn-via-dial-up-conne). For a long time I had to live with this annoyance, as the issue persisted with every new version of the AnyConnect client. But now I found a workaround, maybe even a long-term solution. It is possible to tell the macOS resolver to use certain DNS servers for certain domains. Setting this up is rather easy:

```shell
sudo mkdir /etc/resolver
echo 'nameserver 10.8.0.1' > /etc/resolver/domain.com
sudo killall -9 mDNSResponder && dscacheutil -flushcache
```

After restarting the mDNSResponder daemon and flushing the DNS cache, I was all set. I hope this will become a long-term solution for this issue, that plagued me for so long.
