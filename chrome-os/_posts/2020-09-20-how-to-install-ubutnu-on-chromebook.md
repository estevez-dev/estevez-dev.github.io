---
title: 'How to install Ubutnu on Chromebook'
layout: post
redirect_from:
    - /2020/09/20/how-to-install-ubutnu-on-chromebook
    - /how-to-install-ubutnu-on-chromebook
---

Many Chrome OS users come to this question sooner or later. Yes there is still an option to use Linux apps inside Chrome OS, but this apps runs in a virtual environment without direct access to the hardware. There is better option like [crouton](https://github.com/dnschneid/crouton), but it is still not the same as having the real Ubuntu (or any other your favorite GNU/Linux distribution) installation. Do you know that you even could have a dual-boot to Chrome OS or Linux? Well, not on every Chromebook, but still. I’m not going to copy-paste the guides from another sources. The best source is an official documentation of the tools you will use. I just want to point you in the right direction and also make a note for myself.

First thing I need to mention is [GalliumOS](https://galliumos.org/). This is a GNU/Linux distribution based on Xubuntu and optimized for Chromebooks. This Linux distribution was started long time ago at the age when Chromebooks was capable to open only four Chrome tabs without serious impact on performance. There was no Android apps support or Linux apps in virtual environment. And since then GalliumOS is perfect for old Chromebooks, but it starts to losing ground on modern, powerful devices.

I have a good example running development environment on my Dell Inspiron Chromebook 14 2-in-1. I tried to use GalliumOS as well as latest Ubuntu LTS. Overall performance was much higher with Ubuntu unfortunately.

So the best place to start is an [installation guide for GalliumOS](https://wiki.galliumos.org/Installing). You can just follow the steps, but instead of having a GalliumOS USB stick you can create one with your favorite GNU/Linux distribution. The guide is also linked with [MrChromebox.tech](https://mrchromebox.tech/) website and tools where you can read everything about alternative OS installations and use their [Firmware Utility Script](https://mrchromebox.tech/#fwscript) for easy firmware manipulations including backup and restore options.

Sure all this operations are not safe and can damage your Chromebook.

Here is a summery of all steps I performed for my Dell Inspiron Chromebook 14 to replace Chrome OS with Ubuntu:

1. Create bootable USB stick or SD card with latest Ubuntu LTS
2. Use [Firmware Utility Script](https://mrchromebox.tech/#fwscript) to backup original Chromebook firmware
3. Use [Firmware Utility Script](https://mrchromebox.tech/#fwscript) to install/update custom firmware to enable boot menu
4. Boot from prepared USB stick or SD card
5. Install Ubuntu replacing Chrome OS