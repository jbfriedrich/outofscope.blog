---
title: Disable IPv6 privacy extension on macOS
date: 2024-01-15T04:40:53+01:00
tags: [macos, networking, ipv6]
languages: [en]
---

To disable the IPv6 privacy extension in macOS, you are supposed to change the following setting via the sysctl command: 

```shell
% sudo sysctl -w net.inet6.ip6.prefer_tempaddr=0
Password:
net.inet6.ip6.prefer_tempaddr: 1 -> 0
```

After a reboot, the system should not use the privacy mode anymore. Unfortunately it seems that Apple has gotten rid of the `/etc/sysctl.conf` file. It is gone from my installations of macOS Sonoma and even after re-creating it, it does not seem to be used at startup.

After googling, I decided to create a launch daemon to apply the settings at startup. Creating the file `/Library/LaunchDaemons/com.startup.sysctl.plist` with the following content: 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>Label</key>
        <string>com.startup.sysctl</string>
        <key>LaunchOnlyOnce</key>
        <true/>
        <key>StandardErrorPath</key>
            <string>/private/tmp/sysctl.err</string>
        <key>StandardOutPath</key>
            <string>/private/tmp/sysctl.out</string>
        <key>ProgramArguments</key>
        <array>
            <string>/usr/sbin/sysctl</string>
            <string>-w</string>
            <string>net.inet6.ip6.prefer_tempaddr=0</string>
            <string>net.inet6.ip6.use_tempaddr=0</string>
            <string>net.inet6.ip6.ula_use_tempaddr=0</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
    </dict>
</plist>
```

Now applying the right permissions via `sudo chown root:wheel /Library/LaunchDaemons/com.startup.sysctl.plist` and verifying the key-balue pairs via:

```shell
% plutil /Library/LaunchDaemons/com.startup.sysctl.plist
/Library/LaunchDaemons/com.startup.sysctl.plist: OK
``` 

Then apply the configuration via `sudo launchctl bootstrap system /Library/LaunchDaemons/com.startup.sysctl.plist`. The settings will be applied immediately and you should be able to check it with:

```shell
% sysctl -a | grep tempaddr
net.inet6.ip6.use_tempaddr: 0
net.inet6.ip6.prefer_tempaddr: 0
net.inet6.ip6.ula_use_tempaddr: 0
```

After a reboot, you will be able to check `/private/tmp/sysctl.out` to see if the settings were applied even after the reboot. This should permanently disable the IPv6 privacy extensions.
