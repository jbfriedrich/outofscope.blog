---
title: HE IPv6 tunnel with Ubnt EdgeMax Lite router
date: 2016-01-02T21:26:15
feature_image: https://images.unsplash.com/photo-1528845922818-cc5462be9a63?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ&s=070a3027c856334de5b5432d83226cc7
summary: "IPv6 gets more and more important. I think 2016 will be the year when it get’s adopted on a large scale and will finally start to be mandatory with every new Internet connection — at least in a Dual Stack configuration."
tags:
  - ipv6
  - networking
languages:
  - en
---

My web and DNS servers are available via [IPv6](https://en.wikipedia.org/wiki/IPv6) for a while now. There is not much demand (yet) for it, but it’s available for users who wan’t/can use it. Because IPv6 is currently not that widely spread, I hesitated for a long time to put up the effort to configure a proper IPv6 tunnel at home.

Well, I had some spare time on my hands and stumbled upon some [interesting](https://www.reddit.com/r/Ubiquiti/comments/33zkhu/useful_edgerouter_cli_commands_settings/) [sites](https://community.ubnt.com/t5/EdgeMAX/tunnelbroker-hurricane-electric-IPV6-tunnel-config-question/m-p/424813#M6390) on the Internet. So let’s go!

# Availability of IPv6

It looks like no company is providing native IPv6 access to the Internet — at least on the consumer level or for small business accounts. According to some online publications, Deutsche Telekom has already started their IPv6 rollout in their networking — including their cellular network in Germany. They have supposedly chosen to use [Dual Stack](https://en.wikipedia.org/wiki/IPv6#Dual_IP_stack_implementation) for the transition period. I say “supposedly” because I use a different provider at home and my T-Mobile cell phone internet access seems to be IPv4 only, so I cannot verify it.

# Dual stack where you can; tunnel where you must

There are several tunnel brokers available on the Internet. [Sixxs](https://www.sixxs.net/) disqualified itself years ago with an extraordinarily arrogant attitude that annoys the hell out of me, so I chose to use [Hurricane Electric](https://www.tunnelbroker.net/). This choice really was a no-brainer, as I also use their [DNS services](https://dns.he.net/) for quite some time now and never had any problems.

# Let’s dig a tunnel

Firs things first, [register with Hurricane Electric](https://www.tunnelbroker.net/register.php) and create a tunnel. The creation wizard will show you all required information — like your IPv4 tunnel endpoints and your IPv6 subnet(s). The [Ubiquiti EdgeMax Lite router](https://www.ubnt.com/edgemax/edgerouter-lite/) that I use is a quite versatile and powerful little thing. Log in via SSH and start creating the tunnel:

``` shell
configure
edit interfaces tunnel tun0 set encapsulation sit
set local-ip <Client IPv4 Address>
set remote-ip <Server IPv4 Address>
set address <Client IPv6 Address>
set description "HE.NET IPv6 Tunnel"
exit 
set protocols static interface-route6 ::/0 next-hop-interface tun0
commit
save
```

You now should be able to ping the server IPv6 address and your tunnel is established.

# Configure interfaces and network address

Now configure your internal network. HE will route a /64 prefix to you, so choose an IP address from your new network and assign it to your internal network (usually the ::1 address). Also configure the router to advertise it’s network on the internal interface so that clients can automatically get an IPv6 IP address:

``` shell
set interfaces ethernet eth1 address '<IPv6 router IP>'
set interfaces ethernet eth1 ipv6 dup-addr-detect-transmits 1
set interfaces ethernet eth1 ipv6 router-advert cur-hop-limit 64
set interfaces ethernet eth1 ipv6 router-advert link-mtu 0
set interfaces ethernet eth1 ipv6 router-advert managed-flag false
set interfaces ethernet eth1 ipv6 router-advert max-interval 300
set interfaces ethernet eth1 ipv6 router-advert other-config-flag false
set interfaces ethernet eth1 ipv6 router-advert prefix '2001:xx:xxxx:xxxx::/64' autonomous-flag true
set interfaces ethernet eth1 ipv6 router-advert prefix '2001:xx:xxxx:xxxx::/64' on-link-flag true
set interfaces ethernet eth1 ipv6 router-advert prefix '2001:xx:xxxx:xxxx::/64' valid-lifetime 2592000
set interfaces ethernet eth1 ipv6 router-advert reachable-time 0
set interfaces ethernet eth1 ipv6 router-advert retrans-timer 0
set interfaces ethernet eth1 ipv6 router-advert send-advert true
```

# Change Default Ports For HTTP GUI and SSH

For security reasons change the service ports for the router’s web UI and SSH access to something else, as your IPv6 IP **is reachable from the Internet now!** I think this will be the most important thing to think about during the transition to IPv6 — no “security” by IPv4 NAT anymore. Everything is accessible on the Internet — if not properly shielded by a firewall on the router or the asset itself!

``` shell
set service ssh port 8022
set service gui https-port 8443
```

# Common Useful Address Groups

Set up some groups to make handling of these special address ranges easier in the future:

``` shell
set firewall group address-group Private-RFC-Ranges description 'RFC 1918 Private Ranges'
set firewall group address-group Private-RFC-Ranges address 10.0.0.0/8
set firewall group address-group Private-RFC-Ranges address 172.16.0.0/12
set firewall group address-group Private-RFC-Ranges address 192.168.0.0/16
set firewall group ipv6-address-group IPv6-FE80 description 'fe80::/10 (aka Link-Local) Network'
set firewall group ipv6-address-group IPv6-FE80 ipv6-network 'fe80::/10'
```

# Enable Hardware Offloading

The EdgeMax Lite router offers hardware offloading, so let’s use it!

``` shell
set system offload ipsec enable
set system offload ipv4 forwarding enable
set system offload ipv4 vlan enable
set system offload ipv6 forwarding enable
set system offload ipv6 vlan enable
```

If you are using DSL, it might also be a good idea to enable this

``` shell
set system offload ipv4 pppoe enable
set system offload ipv6 pppoe disable
```

## Optional: Configure MSS Clamping on DSL connections

Some sites on the Internet have problems with [PMTU discovery](https://en.wikipedia.org/wiki/Path_MTU_Discovery), so it is best to clamp the MSS manually. For PPPoE users, this command will ‘fix’ connectivity to remote sites where ICMP is blocked, and PMTU is broken:

``` shell
set firewall options mss-clamp interface-type all
set firewall options mss-clamp mss 1452
set firewall options mss-clamp6 interface-type all
set firewall options mss-clamp6 mss 1412
```

# Configuring the firewall

Last but not least, define a set of firewall rules to make sure you are safe and sound:

``` shell
set firewall ipv6-name HE-To-LAN default-action drop
set firewall ipv6-name HE-to-LAN description 'HE to LAN'
set firewall ipv6-name HE-to-LAN rule 1 action accept
set firewall ipv6-name HE-to-LAN rule 1 description 'Drop non-related incoming IPv6'
set firewall ipv6-name HE-to-LAN rule 1 state established enable
set firewall ipv6-name HE-to-LAN rule 1 state related enable
set firewall ipv6-name HE-to-LAN rule 2 action drop
set firewall ipv6-name HE-to-LAN rule 2 state invalid enable

set firewall ipv6-name LAN-to-HE default-action accept
set firewall ipv6-name LAN-to-HE description 'LAN to HE'
set firewall ipv6-name LAN-to-HE rule 1 action accept
set firewall ipv6-name LAN-to-HE rule 1 state established enable
set firewall ipv6-name LAN-to-HE rule 1 state related enable
set firewall ipv6-name LAN-to-HE rule 2 action drop
set firewall ipv6-name LAN-to-HE rule 2 state invalid enable
```

The first rule-set will drop all incoming IPv6 traffic unless it is related, while the second rule-set will by default accept all traffic. Now we have to assign these rule-sets to the interfaces we have.

## Traffic to the router and your IPv6 network

Assign the HE-to-LAN ruleset for forwarded packets on the inbound tunnel interface **and** and for packets destined for the router itself:

``` shell
set interfaces tunnel tun0 firewall in ipv6-name HE-to-LAN
set interfaces tunnel tun0 firewall local ipv6-name HE-to-LAN
```

## Traffic from the LAN to the IPv6 internet

This rule-set will allow traffic from the inside to the Internet via IPv6:

``` shell
set interfaces ethernet eth1 firewall in ipv6-name LAN-to-HE
```

Congratulations. You are now able to use IPv6! To make sure everything went alright, [test your firewall](http://www.ipv6scanner.com/cgi-bin/main.py) and your general [IPv6 connectivity](http://ipv6-test.com/) and work towards your 20/20 rating!

{{< img caption="IPv6 Test" src="https://media.jason.re/16/1/CXe4EP6WYAEvSlN.jpg" >}}
