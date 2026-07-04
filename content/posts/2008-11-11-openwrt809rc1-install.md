---
title: OpenWRT 8.09 RC1 install
date: 2008-11-11T03:01:17
feature_image: https://images.unsplash.com/photo-1537313796346-65fb768b1ebd?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ&s=927a06f2740d7379a61c67701c9ab812
tags:
  - networking
  - tech
languages:
  - de
---

Schwierig ist die Installation der aktuellen Version von “[OpenWRT](http://www.openwrt.org)” nicht. Da ich große Probleme mit der alten [7.09 Version](http://downloads.openwrt.org/kamikaze/7.09/) im Zusammenhang mit DHCP (die mir zugewiesene IP Adresse des Internet Providers wird manchmal nicht korrekt erneuert) hatte, habe ich mich entschlossen den “[Kamikaze](http://downloads.openwrt.org/kamikaze/8.09_RC1/)“-Akt zu versuchen.

Da ich bereits eine OpenWRT Installation auf dem Gerät habe, habe ich mich für die [Installationsvariante](http://wiki.openwrt.org/OpenWrtDocs/Installation#head-331299313683efd96a8ed1495425b4422fe8aa02) mittels “mtd” entschieden. Ich nenne noch ein WRT54G in Version 2.2 mein eigen, und installiere via MTD, daher [dieses Image](http://downloads.openwrt.org/kamikaze/8.09_RC1/brcm47xx/openwrt-brcm47xx-squashfs.trx) ausgewählt. Das Flashen ging gut von der Hand, wie ich es eigentlich gewohnt bin bei OpenWRT. Nach dem Reboot des Appliance und dem erneuern meines DHCP leases auf dem Mac, konnte ich gleich die erste Neuerung genauer in Augenschein nehmen. Es gibt wieder ein Webinterface, [LuCI](http://luci.freifunk-halle.net/)! Die Einrichtung ging damit sehr flott, innerhalb von 10 Minuten hatte ich meine Konfiguration “zusammengeklickt”. Ich denke ich hätte eher fertig sein können, habe mir aber ein wenig Zeit genommen um das Interface und seine Funktionen genauer unter die Lupe zu nehmen.

Alle Freunde der Console (zu denen ich mich auch zähle), können beruhigt aufatmen, “opkg” ist vorhanden und funktioniert genauso wie sein Vorgänger “ipkg”. Auch das Editieren der Konfigurationsdateien unter “/etc” funktioniert wie gewohnt. Man kann sich also in Zukunft aussuchen, ob man lieber eben schnell klickt oder sich auf die Console begibt und dort die “Advanced”-Einstellungen vornimmt.

Besonders die Konfiguration der Firewall gefällt mir sehr gut, man kann schnell Regeln für einzelne Zonen einrichten, oder erweiterte Regeln anlegen. Generell sind die Mache auf Sicherheit bedacht. Die Firewall hat eine brauchbare Grundkonfiguration (Pakete von extern werden “rejected”, Syn-Flood Schutz ist aktiv, Pakete von intern nach extern werden weiter geroutet), und auch das WLAN ist per Defaulteinstellung deaktiviert. Dies liegt unter anderem daran, dass man erst die sichere WPA2 Verschlüsselung auswählen kann, nachdem man das Paket “wpa_supplicant” nachinstalliert hat. Es scheint noch ein Bug in der RC1 vorhanden zu sein: Wenn man die Einstellung vornimmt, dass z.B. die Zone “WAN” nicht in die Zone “LAN” darf (DROP oder REJECT), werden die separat eingestellten RULES **nicht** mehr abgearbeitet! Dies ist leider total unsinnig, da keine Ausnahmen von der Regeln definiert werden können (z.B. um bestimmte Dienste von einer bestimmten IP zuzulassen) und wird hoffentlich in der finalen Version behoben.

Ein Tipp zum Schluss, nicht wundern wenn die Meldung

> An error ocurred, return value: 716563936

angezeigt wird, wenn man versucht ein Paket mittels “opkg” zu installieren. Dies ist der Rückgabewert, wenn ein Paket nicht gefunden wurde. Völlig normal, weiter machen! Ist halt noch ein RC Release. Subjektiv kommt mir die Latenz der Internetverbindung etwas niedriger vor, was allerdings täuschen kann. Es wird wohl eine Weile dauern, bis ich mein endgültiges Fazit abgeben kann aber alles in allem bin ich zufrieden.
