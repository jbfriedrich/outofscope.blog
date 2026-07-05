---
title: Brick wall
date: 2015-05-13T19:55:00
tags:
  - firewall
  - windows
languages:
  - en
---

One way to permanently get rid of the Windows Firewall is to disable the service in the Services Manager, or set the startup type to “Manual”. Today I learned that it is not enough to disable the “Windows Firewall” service when you are on a Windows 7 or Windows 2008 R2 Server. You also have to disable the “Base Filtering Engine” if you do not want to hit a big red brick wall (also known as “Block mode”).

More on the topic in the [Technet blog](http://blogs.technet.com/b/networking/archive/2009/03/24/stopping-the-windows-authenticating-firewall-service-and-the-boot-time-policy.aspx) and this [Technet article](https://technet.microsoft.com/en-us/library/cc766337\(WS.10\).aspx).
