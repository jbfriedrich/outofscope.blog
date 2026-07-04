---
title: Windows 8 on a MacBook Pro Mid 2009
date: 2013-07-12T01:57:00
feature_image: https://images.unsplash.com/photo-1530133532239-eda6f53fcf0f?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ&s=31bc92370cefbe48f597bce89aae061d
tags:
  - apple
  - macos
  - tech
  - windows
languages:
  - en
---

Unfortunately I need to work with Windows from time to time. I am depending on some tools that are only available for that OS and a Virtual Machine is not always handy. So when Microsoft made the Windows 8 Pro upgrade offer back in January, I decided to finally upgrade my Windows XP and use Boot Camp and bring Windows 8 to my mid-2009 Macbook Pro. It was not that easy…

First of all Apple decided that a 2009 Macbook Pro is not worthy to run Windows 8. Windows 7 x64 was the last supported os for this hardware. People who are used to Apple’s politics know that this is not a real hardware limitation, its purely software support or software maintenance related. So these are the steps I needed to take to install it on the machine:

## Requirements

* Windows 8 DVD
* Windows 7 DVD
* [Latest Apple Boot Camp Drivers for Windows 8 (5.0.533)](http://support.apple.com/kb/DL1638)

## Installation

It is possible that an USB install media is working for you, for me only the DVDs would work. The Boot Camp assistant will not accept a Windows 8 install media, so we need to fool it.

Put the Windows 7 DVD into your Superdrive, then run the Boot Camp Assistant. It will partition your hard drive and reboot the machine to start the Windows 7 installation. Once you are in the Windows 7 installer, quit the installer (the Macbook will reboot) and press ‘Option’ when you hear the Mac chime. Once you can choose the OS to boot, replace the Windows 7 DVD with your Windows 8 DVD. Wait a few moments and you should be able to boot the DVD and start the Windows 8 installation and install it like you normally would.

## Use an Upgrade License for a clean install

If you were as clever as me, and used your Windows 8 Upgrade install media to do a clean install, I would suggest not to use any updates for now and also not to connect the Windows install to the Internet yet. Instead of that, execute _regedit.exe_ , navigate to `HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Windows/CurrentVersion/SetupOOBE` and change the value **MediaBootInstall** from **1** to **0**. After that, open a **Command Line Window** by running _cmd.exe_ and run the command _slmgr /rearm_ and reboot. After reboot you can connect your MacBook to the Internet and activate Windows online or by phone.

## Driver installation

Apple will not let you install the drivers by running _setup.exe_ from the root directory of the driver DVD. Instead you need to navigate to `BootCampDriversApple` and run _BootCamp.msi_ with Administrator permissions. Then the drivers are cleanly installed and work like a charm (at least they do for me).

## Permission problems

If you are having trouble with executing things as Administrator, and all options greyed out, you need to deactivate the **Admin Approval Mode** for local Administrators. Run _gpedit.msc_ and go to

Computer Settings > Security Settings > Local Policies > Security Options > User Account Control

Set the option **Run all administrators in Admin Approval Mode** to **Disabled**. Reboot after this configuration change. This should solve all your permission problems. After that I installed all available Windows Updates and the tools I required. With the Apple Trackpad Windows 8 is still strange, and feels weird, but its better than with a mouse I supose.
