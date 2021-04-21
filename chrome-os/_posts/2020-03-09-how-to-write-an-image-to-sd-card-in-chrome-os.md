---
title: 'How to write an image to SD Card in Chrome OS'
layout: post
redirect_from:
    - /2020/03/09/how-to-write-an-image-to-sd-card-in-chrome-os
    - /how-to-write-an-image-to-sd-card-in-chrome-os
---

![image](/img/how-to-write-an-image-to-sd-card-in-chrome-os/unnamed-e1583760740384.jpg)

The way I was doing this previously was a command line tool for Linux. But it turns out there is a simple way to this with a couple of clicks.

**Chromebook Recovery Utility** is an app we need. You can find it in [Chrome Web Store](https://chrome.google.com/webstore/detail/chromebook-recovery-utili/jndclpdbaamdhonoechobihbbiimdgai). And the main reason of it’s existence is creating recovery image of your system to be able to restore it in case of failure.

But! It can simply write any image file even if it is inside a zip archive to any USB storage device. You just need to launch this app, click gear icon on the top and choose _Use local image option_.