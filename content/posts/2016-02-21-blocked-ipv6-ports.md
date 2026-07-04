---
title: HE & DigitalOcean - Blocked IPv6 ports
date: 2016-02-21T00:44:24
feature_image: https://images.unsplash.com/photo-1517373116369-9bdb8cdc9f62?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ&s=1477902d871f75ce35d69ab0dc5631c0
tags:
  - ipv6
  - networking
  - tech
languages:
  - en
---

I recently created several Virtual Machines on my new [Hetzner](https://www.hetzner.de/us/hosting/produkte_rootserver/ex51) server as part of my migration efforts from [DigitalOcean](https://m.do.co/c/8469d730e168) to my self-hosted ESXi hypervisor. I don’t migrate my VMs and services away from [DigitalOcean](https://m.do.co/c/8469d730e168) because I am not satisfied with their service, quite the contrary in fact. I am doing it because my own self-hosted hypervisor allows more flexibility for my learning and testing efforts.

While double-checking the VMs for running services and open ports, I noticed that my Hurricane Electric IPv6 address has some filtered ports that I did not notice before:

{{< img caption="Nmap" src="https://media.jason.re/16/2/Screen-Shot-2016-02-20-at-22-24-25.png" >}}

I looked at the [Hurricane Electric FAQ](https://ipv6.he.net/certification/faq.php) and it explains why these ports are filtered and how to get these ports un-blocked:

>  **Why can I not connect to IRC?**  
>  Due to a high and persistent amount of abuse, we’ve had to filter IRC access by default. If you need IRC access, complete the Sage level of the free IPv6 certification and then please send an email to ipv6@he.net explaining your situation. Approvals will be handled on a case-by-case basis. **I can’t send email via IPv6. What’s wrong?**  
>  Due to a high and persistent amount of abuse, we had to filter SMTP (tcp/25) connections by default. If you’re not providing email service yourself, you should be able to use port 587 instead to your provider’s email server. If you are providing email services over your tunnel and need port 25 opened, please send an email to ipv6@he.net explaining your situation. NOTE: this filtering does not affect the SMTP-related tests on the IPv6 certification program.

Pretty straight-forward and well-explained. Knowing that these ports are blocked, I wanted to use one of my IPv6-enabled DigitalOcean droplets as this should not filter anything and give me an unaltered list of open ports:

{{< img caption="Nmap" src="https://media.jason.re/16/2/Screen-Shot-2016-02-20-at-22-27-26.png" >}}

Man, I was so wrong! It looks like DigitalOcean is blocking ports as well, though their blocking made much less sense to me. I could not find anything in the [help section on their homepage](https://www.digitalocean.com/help/), but was able to find an article in their ‘Support Center’ that is accessible once you log in:

>  **How to send emails with IPv6 enabled**  
>  We currently do not allow SMTP traffic over IPv6 as a side effect of how email black lists treat IPv6 addresses.You can give priority to IPv4 addresses over IPv6 so that you can continue to send out email without disabling IPv6. You would do that by editing the Droplet’s _/etc/gai.conf_ file and removing the comment (#) from the following line:
>  Default Configuration:
>    #precedence ::ffff:0:0/96 100  
>  Configuration with Priority to IPv4:
>    precedence ::ffff:0:0/96 100

As mentioned in a [comment](https://www.digitalocean.com/community/tutorials/how-to-enable-ipv6-for-digitalocean-droplets?comment=15503) on their “[How To Enable IPv6 for DigitalOcean Droplets](https://www.digitalocean.com/community/tutorials/how-to-enable-ipv6-for-digitalocean-droplets)” site, there seems to be no way of getting rid of these restrictions. Even when you [contact their support](https://warrenguy.me/blog/ipv6-digital-ocean-crippled) on this specific issue.

I can understand the blocking of [IRC ports](https://en.wikipedia.org/wiki/Internet_Relay_Chat) and [port 25](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol) for the reasons Hurricane Electric is giving in their FAQ. However I cannot understand the blocking of the [ports 587, 465](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol), [110, 995](https://en.wikipedia.org/wiki/Post_Office_Protocol) and [143](https://en.wikipedia.org/wiki/Internet_Message_Access_Protocol).

**Lesson learned:** Always use additional external tools — like [MXTools](http://mxtoolbox.com/NetworkTools.aspx) — to verify your VM’s open ports, if your source IPv6 address is from DigitalOcean or Hurricane Electric!
