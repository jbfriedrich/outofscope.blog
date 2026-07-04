---
title: Connection lost
date: 2014-09-22T02:31:23
feature_image: https://images.unsplash.com/photo-1516044734145-07ca8eef8731?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ&s=54c1a1cf746bafde44894e637dca25b2
tags:
  - networking
  - tech
languages:
  - en
---

Four weeks ago my [Ubiquiti EdgeMax EdgeRouter Lite](http://www.ubnt.com/edgemax/edgerouter-lite/) decided it was a good idea to [not start PHP anymore](http://community.ubnt.com/t5/EdgeMAX/EdgeMax-500-Internal-Server-Error/m-p/980503). Some people would say ‘Good choice and good riddance’ — on a router appliance a web interface comes in handy once in a while though. A Ubnt community member was able to help me very quickly, though something was very fishy. I just recently upgraded the firmware on the router, so how could the file system got corrupted? Did something strange happen during the upgrade? — Not that I could remember.

One evening two weeks ago then I was debugging a VPN problem I encountered a massive packet loss. First I thought it was the remote server, or my Internet provider. Soon though I had to realize that the first hop, the little EdgeRouter, was losing 70% of all packages sent to it. Also 60% packet loss on the second hop. No web interface available, not SSH connection possible. I decided to reboot the device… it did not come back! My first thought was a [memory issue](https://community.ubnt.com/t5/EdgeMAX/EdgeRouter-bricked-hanging-at-boot-Is-it-dead/td-p/400574), but I needed more information what was happening on the device. The lights were no help, everything looked normal. I connected a Cisco Serial Console Cable and found some kernel “Oops” during boot:

<iframe src="https://pastebin.com/embed_iframe/JESTY42e" style="border: none;width: 100%; margin-bottom: 30px;"></iframe>

This really looked weird. I contacted [Ubnt support](http://www.ubnt.com/support/). I was told to press the famous ‘any key’ during boot, before the kernel was loaded, and then run some tests. First of all a memory test:

```shell
Octeon ubnt_e100# mtest
Testing 80100000 ... 80ffffff:
Iteration: 6040
```

It ran for a couple of hours, but showed no issue at all. Support then had the idea to use the [‘EdgeMax Rescue kit’](http://community.ubnt.com/t5/EdgeMAX/EdgeMax-rescue-kit-now-you-can-reinstall-EdgeOS-from-scratch/m-p/514857/highlight/true#M12098) to re-install the router from scratch.

This unfortunately did not bring a solution either. But it helped to identify the underlying problem. It looks like the filesystem corruption a few weeks back was just a herald of the problems that would I have to face soon:

It looks like the flash memory is totally fucked up. Since it was a little over a year old, and therefore still covered by the EU wide 2 year warranty, I contacted [my reseller](http://varia-store.com/) and filed for RMA and sent the router back to them. I really hope that I will get a new device soon. It is a quite powerful little device, so I really miss it.
