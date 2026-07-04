---
title: Let's Encrypt - Beta Impressions
date: 2015-11-06T17:19:05
feature_image: https://images.unsplash.com/photo-1485761954900-f9a29f318567?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ&s=aaab19590fcc9502b5195bc6006249c4
tags:
  - linux
  - security
languages:
  - en
---

The [Let’s Encrypt Beta](https://letsencrypt.org) has finally started. I registered a couple of weeks ago and the domains I use regularly got white-listed. Just a few impressions so far:

* The client ist still pretty basic, it comes with a little wrapper that builds a virtual environment for all the required python modules (which is very nice and comfortable)
* The plugin to automatically configure Apache is still in alpha
* The plugin to automatically configure Nginx is buggy and what seems pre-alpha (I think it is not delivered/used currently at all)
* Don’t manually mess with `/etc/letsencrypt` as in **never ever!**

{{< img caption="Let's Encrypt Certificate" src="https://media.jason.re/15/11/Screen-Shot-2015-11-06-at-16-01-50.png" >}}

It is already comfortable to use — if you compare it to the manual process you had to undergo before. Once it is finished and all the bugs are ironed out, this thing will kick ass.

The certificates are already deployed on all my major sites, now I just have some maintenance work to do (remove unsafe ciphers etc). I started with [my blog](https://pixelschatten.net) and the [SSL Labs test](https://www.ssllabs.com/ssltest/analyze.html) looks pretty good.

{{< img caption="SSL Labs Rating" src="https://media.jason.re/15/11/Screen-Shot-2015-11-06-at-16-05-03.png" >}}

I will try to do more with it in the upcoming days and weeks, but between work and university I currently don’t have that much time for personal projects.

If you are not part of the beta program but want to support the Let’s Encrypt initiative and go bug hunting, or simply want to try how it works, just grab [the client from GitHub](https://github.com/letsencrypt/letsencrypt) and use the testing infrastructure they provide (the testing CA is called “Happy Hacker CA”). News and announcements about the beta can be found [here](https://community.letsencrypt.org/t/beta-program-announcements/1631), there are also configuration examples for [Nginx](https://community.letsencrypt.org/t/nginx-configuration-sample/2173) and Apache.

Last but not least, [Kenn White](https://twitter.com/kennwhite/status/662520849979674624) published a little [script suite on Github](https://github.com/kennwhite/install-letsencrypt) that downloads the official client and runs it to generate a certificate. It helps a lot to run the client on older Linux distributions or AWS instances — but on newer distributions a bit redundant in my opinion. Everything is in the early stages, and as the Let’s Encrypt initiative matures, I am sure the scripts will grow and be a great resource in the future! Kenn mentions other available clients on the page, make sure to check them out:

* A [Ruby Gem](https://github.com/unixcharles/acme-client)
* A single cross-plattform [Go application](https://github.com/xenolf/lego)
* A single [Python script](https://github.com/diafygi/letsencrypt-nosudo) (without sudo requirement)
