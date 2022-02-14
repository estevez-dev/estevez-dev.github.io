---
title: 'Smart home example: Smart home in a rent apartment'
header:
  teaser: /img/smart-home-in-rent-apartment/modem.jpg
  og_image: /img/smart-home-in-rent-apartment/modem.jpg
redirect_from:
    - /smart-home-in-rent-apartment
---

At the beginning there was nothing. I mean nothing smart in my rental apartment. The first device in my setup was _Sonoff Basic_ WiFi relay. Unfortunately, I don't remember what I was controlling with it, but I remember that it was fun. And I started to think about the possibility to control the lights remotely too.

![image](/img/smart-home-in-rent-apartment/IMG_20180727_202555196.jpg)

My first light control solution was based on _Broadlink TC2_ wall switch and _Broadlink RM Pro_ hub for it. That was also fun at first, but when you start to dive into this world you want to go dipper and dipper.

![image](/img/smart-home-in-rent-apartment/broadlink.jpg)

It was a long path and now I can say I have a stable system I'm enjoying to use, built with minimal intrusion in wiring, walls and other parts of my rented apartment. I hope my example could help someone to stop thinking that you can't build a smart home if you don't have your housing.

## The core

The brain of the system is located on _Raspberry Pi 3B+_. It is a [Home Assistant](https://www.home-assistant.io/) instance - an open-source home automation server. Here are two main reasons I have it:
1. You can easily connect IoT devices and home appliances in one place without tons of standalone applications on your phone.
2. You do not depend on the internet connection, only on your local network that is much stable.

![image](/img/smart-home-in-rent-apartment/server.jpg)

## Lights

The first step was replacing all light bulbs with [MiLight](https://www.milight.com/) aka _Easybulb_ smart LED bulbs. Fortunately, I had only two types of bases across the apartment: E27 and E14. MiLight is relatively cheap and controlled with WiFi gateway through a 2.4 GHz radio interface. I like this solution because the bulbs are ready to accept commands right after powering on and you don't need to worry about 25 additional WiFi devices connected to your poor, overloaded router. Also, all bulbs at my apartment works two years without any failure.

![image](/img/smart-home-in-rent-apartment/milight.jpg)

MiLight works with Home Assistant using [LimitlessLED integration](https://www.home-assistant.io/integrations/limitlessled/), but also you can improve the gateway to make it work through MQTT. By improving I mean building your DIY hub on NodeMCU with [open-source firmware available on GitHub](https://github.com/sidoh/esp8266_milight_hub). The main advantages of the DIY hub are:
* You can connect up to 262 144 light groups to one hub, instead of 4
* Your system will work only inside your local network without cloud services connection

Regular wall switches were removed and the wires were connected to power light bulbs constantly. The holes for light switches were covered with wireless ZigBee wall switches by Aqara (Xiaomi).

![image](/img/smart-home-in-rent-apartment/wall_switch_01.jpg)

{% include ads_media.html %}

![image](/img/smart-home-in-rent-apartment/wall_switch_02.jpg)

I'm not sure is it possible to force Aqara wall switch to control MiLight smart bulbs under normal conditions. I think not. But it is a simple task if you are using Home Assistant server or any other local home automation solution.

![image](/img/smart-home-in-rent-apartment/wall_switch_03.jpg)

All ZigBee devices are connected directly to my home automation server through [ConBee II](https://phoscon.de/en/conbee2) USB stick and _deCONZ_ software to avoid using _Aqara Hub_ and Xiaomi cloud services.

I also have some ambient lights and desk lamps. It is _Yeelight_ smart WiFi bulb, two _Yeelight_ WiFi LED light strips and _Yeelight desk lamp_. _Yeelight_ WiFi devices could be controlled locally from Home Assistant with [Yeelight integration](https://www.home-assistant.io/integrations/yeelight/).

For some rooms, there is movement detection set up with _Aqara ZigBee motion sensors_.

## Climate

I have two simple convector electric heaters with just an on/off button and analogue thermostat, one in each bedroom. It is connected to _Sonoff TH10_ WiFi relay with digital temperature and humidity sensor also attached to _TH10_. Both relays are flashed with custom firmware called [Tasmota](https://tasmota.github.io/) to make it work through MQTT with Home Assistant server without any external connection. The analogue thermostat of the heater is set to maximum, but the power is controlled by template thermostat integration in Home Assistant. It links together the relay and the temperature sensor data and provides a simple interface to control heating.

![image](/img/smart-home-in-rent-apartment/heater.png)

The temperature for both bedrooms is taken from the same temperature sensors used to control heating. For other rooms, I'm using _Aqara ZigBee temperature and humidity sensors_.

I also have a DIY smart air humidifier I made from a regular one. It also works locally through MQTT and I have a [separate article about it](/how-to-make-wifi-enabled-smart-humidifier-from-a-regular-one). It was my first experience soldering something that works and is useful.

## Security

In my opinion, you can't call it "Security" until it can call the police or security agency for you automatically. My system can't do this yet. I also don't have security cameras, but I really want it in the future. Primarily for storing movement videos in the cloud, because I don't see a reason to store it at home where it can be stolen with the hardware. Also, I'm planning to organize smoke and leak detection.

So the only things to be described in this section are windows and doors sensors. I'm using it not for intrusion detection, but for some simple yet useful automation:
* Turning on the lights in the pantry when the door was opened, and turning it off by closing the door.
* Turning off the heaters in bedrooms when windows are opened, to prevent heating the street.
* Prevent air humidifier from turning on automatically when the window in the room is open.

The sensors are _Aqara ZigBee door and window sensors_.

## Multimedia

There is not much do describe here. I'm using _Google Chromecast_ for both TVs to make them smarter because they are really old. I also made a DIY MQTT-to-IR controller based on _Wemos D1 mini_ WiFi board. It allows me to turn kitchen TV on and off automatically whenever I need it.

I think there should be a separate article to describe every automation rule I have set up. In terms of hardware, this is pretty much all I can list except the control points:
* Every family member can control everything with their phones using [Home Assistant Android application](https://play.google.com/store/apps/details?id=io.homeassistant.companion.android).
* Most used lights and appliances can be controlled from the wall-mounted tabled. This is an old good Nexus 7 with LineageOS 14 and Fully Kiosk Browser app.
* Voice control with Google Assistant is possible through [Nabu Casa Claud](https://www.nabucasa.com/). 

![image](/img/smart-home-in-rent-apartment/tablet.jpg)