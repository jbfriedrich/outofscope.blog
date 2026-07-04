---
title: VPN with Wireguard on macOS & iOS
date: 2019-02-28T03:04:34
feature_image: https://images.unsplash.com/photo-1535191042502-e6a9a3d407e7?ixlib=rb-1.2.1&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ
tags:
  - networking
  - vpn
  - linux
  - macos
  - ios
languages:
  - en
---

I read and hear a lot about [Wireguard](https://www.wireguard.com) the last couple of weeks. Even Linus Torvalds [seems to like it](http://lkml.iu.edu/hypermail/linux/kernel/1808.0/02472.html) – and praise like that is not easy to achieve. When people say so many good things about it, I should give it a try. I was looking for an OpenVPN alternative anyway.

## Installation

I expected some kind of "Piped Bash Script" installer, but Wireguard has prepared packages for almost every [Linux distribution](https://www.wireguard.com/install/). There is even a [macOS app](https://itunes.apple.com/us/app/wireguard/id1451685025?ls=1&mt=12) and an [iOS app](https://itunes.apple.com/us/app/wireguard/id1441195209?ls=1&mt=8) available in the app stores. This was the first, but not the only time I was impressed with Wireguard.

As underlying OS for the VPN server, I used the latest version of [Ubuntu LTS](https://www.ubuntu.com/download/server). The well-known simple command triplet starts the installation of the required components:

```shell
# Adding the reposistory from PPA
$ sudo add-apt-repository ppa:wireguard/wireguard

# Installing required packages
$ sudo apt-get update
$ sudo apt-get install wireguard linux-headers-4.15.0-45-generic iptables

# Generating and loading the wireguard kernel module
$ dkms autoinstall
$ modprobe wireguard

# Enabling package forwarding for the VPN clients
$ echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
$ sysctl -p /etc/sysctl.conf
```

For iOS I just downloaded the app from the app store. For macOS I did not use the app, but decided to install all required components via [brew](https://brew.sh), so that could start and stop the VPN from my Fish console:

```shell
$ brew install wireguard-tools qrencode
```

## Configuration

This was the second surprise. After a very painless and easy installation, the configuration was also straightforward. Most of the steps are identical, no matter if it is the client or the server. So lets start with generating all the required keys:

```shell
for device in server client ios
do
    wg genkey > ${device}.privkey &&
    wg pubkey < ${device}.privkey > ${device}.pubkey &&
    wg genpsk > preshared.key && 
    chmod 0600 ${device}.privkey ${device}.pubkey preshared.key
done
```

Now that we have generated the required private and public key files for all parties involved, we can create the config files. The VPN server first:

```shell
# /etc/wireguard/wg0.conf

# The internal IP address of the VPN server that is used over the VPN
Address = 10.0.0.1/24

# Commands to tbe run after the VPN interface is started or stopped.
# This is required to enable the server to act as a NAT gateway.
# Please replace 'eth0' with your actual network device name!
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

# Port that the server listens on
ListenPort = 51820
# Private key
PrivateKey = <insert value from server.privkey here>

[Peer]
# macOS client
PublicKey = <insert value from client.pubkey here>
PresharedKey = <insert value from preshared.key here>
AllowedIPs = 10.0.0.2/32

[Peer]
# iOS client
PublicKey = <insert value from ios.pubkey here>
PresharedKey = <insert value from preshared.key here>
AllowedIPs = 10.0.0.3/32, fd86:ea04:1115::2/128
```

All that is left is to enable and start the wireguard service via systemctl:

```shell
$ systemctl enable wg-quick@wg0.service
$ systemctl start wg-quick@wg0.service
```

If everything went according to plan, the service will be started and a new network interface `wg0` should be visible via `ip link show`. For the macOS client, the configuration file needs to look like this:

```shell
# ~/.config/wireguard/client.conf

[Interface]
Address = 10.0.0.2/24
PrivateKey = <insert value from client.privkey here>
ListenPort = 51820

[Peer]
PublicKey = <insert value from server.pubkey here>
PresharedKey = <insert value from preshared.key here>
# We route everything over the VPN, once the tunnel is established.
# If you dont want that, specify the IP addresses and networks you
# want to route here.
AllowedIPs = AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = <insert VPN server public IP here>:51820
# This is for if you’re behind a NAT and want the connection to be kept alive.
PersistentKeepalive = 25
```

After the configuration is created, we bring up the tunnel via `sudo wg-quick up ~/.config/wireguard/client.conf`.

To make our lives easier, we are preparing the iOS configuration on the Mac and create a QR code that we can use to configure the Wireguard app on the iPhone. But first the configuration file:

```shell
# ~/.config/wireguard/ios.conf

[Interface]
Address = 10.0.0.3/24, fd86:ea04:1115::2/128
PrivateKey = <insert value from ios.privkey here>
ListenPort = 51820
# The iOS client seems to need a DNS server value here,
# otherwise I was not able to resolve anything while
# connected to the VPN. Choose your favorite DNS resolvers here
DNS = 9.9.9.9, 8.8.8.8

[Peer]
PublicKey = <insert value from server.pubkey here>
PresharedKey = <insert value from preshared.key here>
# We route everything over the VPN, once the tunnel is established.
# If you dont want that, specify the IP addresses and networks you
# want to route here.
AllowedIPs = AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = <insert VPN server public IP here>:51820
# This is for if you’re behind a NAT and want the connection to be kept alive.
PersistentKeepalive = 25
```

Now that we are finished with the iOS configuration, we will generate the QR code via `qrencode -t ansiutf8 ~/.config/wireguard/ios.conf`. The code can now be used in the iOS app to add a new tunnel configuration.

## Final thoughts

Even though Wireguard is marked as "alpha" everywhere, it seems to be very stable and quite usable. There is even a package for the [Ubiquiti Edge Router](https://community.ubnt.com/t5/EdgeRouter/Release-WireGuard-for-EdgeRouter/td-p/1904764) available. Everyone who knows their way around a Cisco or Juniper console will find themselves right at home.

My impression so far: It is stable, performant (enough), very easy to install, easy to configure and, last but not least, very low maintenance. The iPhone app is great and integrates very well into iOS. After I had such a great experience so far, I will test the native macOS Wireguard app in the next couple of days.

I think I'll keep it!
