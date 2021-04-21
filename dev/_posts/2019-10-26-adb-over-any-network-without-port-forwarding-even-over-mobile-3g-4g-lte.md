---
title: 'ADB over any network without port forwarding. Even over mobile 3G/4G/LTE'
layout: post
redirect_from:
    - /2019/10/26/adb-over-any-network-without-port-forwarding-even-over-mobile-3g-4g-lte
    - /adb-over-any-network-without-port-forwarding-even-over-mobile-3g-4g-lte
---

I think the first question raised here can be “Why do you need this?”. I’m developing a [Flutter application in Gitpod environment](/developing-flutter-application-in-gitpod-and-running-it-on-real-device) and need to run it on a real dive. There is no way to connect to my USB port from cloud-based IDE, but adb can connect over the network as well. While I’m at home there is no reason for me to use the method described in this guide because I have static IP and forwarded port on my router. I’m using this method only when I’m a [road warrior](https://en.wikipedia.org/wiki/Road_warrior_(computing)). But you can use it at home as well or in any situation where you can’t or don’t want to configure router or get public IP address.

The set of devices and tools I’m using as an example:

* Chromebook with Linux enabled (crostini)
* Android phone
* ngrok service and tool
* abd
* Gitpod cloud environment

We will review the cases when you are on mobile network connection and need to connect adb to your Android device from somewhere on internet. For example to run Flutter app from Gitpod environment. But again it is applicable also in case your laptop and phone are connected to the same WiFi point.

## 1. ngrok tool
Ngrok is a service that provides public url’s for your local devices, servers and other stuff. There is a free account available so go to [ngrok.com](https://ngrok.com/) and create one. Then go to [downloads](https://ngrok.com/download) page and get the client. There is also a quick start guide available on that page to authenticate `ngrok client` with ngrok authtoken command.

## 2. adb on device
Now we need to switch adb client on our Android device to use network connection instead of USB. We need to connect our device with USB cable to any computer with adb installed and run:

```
adb tcpip 5555
```

Your device is ready for debugging over the network. Adb on device will switch back to USB mode only after device restart. To make it manually:

```
adb usb
```

## 3. Set up connections

I’m assuming that if you are using mobile network, it is tethered to your laptop over WiFi. If you are using Chrome OS instant tethering, it would be harder to know your phone IP address, so I’m recommending to enable WiFi hotspot on your Android device manually and connect your laptop WiFi to that hotspot.

Now we need to know our gateway IP address because our Android device is a gateway. First of all try to ping default IP address. It should be the same for most Android devices:

```
ping 192.168.43.125
```

If ping was not successful here is how to find your gateway IP address on Chrome OS:

1. Go to Chrome OS Settings -> Network -> WiFi
2. Click on WiFi network you are connected to
3. Expand “Network” section. You will find Gateway IP address there

So `192.168.43.125` was my case. This is our Android device.

In case when you don’t use mobile network and your laptop and Android device both connected to the same WiFi, you just need to find your phone’s IP address from Settings -> WiFi -> Advanced on it.

Next we need to start ngrok on laptop to forward adb connection to our Android device:

```
ngrok tcp 192.168.43.125:5555
```

In a session info opened you can find external address and port in “Forwarding” line. For example: _tcp://0.tcp.ngrok.io:13840_

## 4. Connecting abd

There is the simplest step left. Connect adb from your environment (Gitpod in my case) to the address and port provided by ngrok:

```
adb connect 0.tcp.ngrok.io:13840
```

Done!