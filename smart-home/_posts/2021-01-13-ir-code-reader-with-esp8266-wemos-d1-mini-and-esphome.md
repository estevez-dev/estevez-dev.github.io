---
title: 'IR code reader with ESP8266 Wemos D1 mini and ESPHome'
header:
  teaser: /img/ir-code-reader-with-esp8266-wemos-d1-mini-and-esphome/ready.jpg
  og_image: /img/ir-code-reader-with-esp8266-wemos-d1-mini-and-esphome/ready.jpg
redirect_from:
  - /ir-code-reader-with-esp8266-wemos-d1-mini-and-esphome
---

Quick step by step guide on assembling a IR receiver for reading IR codes from your remotes based on Wemos D1 mini, Wemos IR shield and using ESPHome.

## Components used
1. The housing from Broadling RM Pro+
2. [Wemos D1 mini](https://www.wemos.cc/en/latest/d1/d1_mini.html) development board
3. [Wemos IR shield](https://www.wemos.cc/en/latest/d1_mini_shield/ir.html)
4. USB to micro-USB cambe

## Assembling
First of all we need to make our shield work with all transmitting LEDs and the pins we need. In this example we will use `D3` for sending and `D4` for receiving.

![image](/img/ir-code-reader-with-esp8266-wemos-d1-mini-and-esphome/d4_ir_recv.png)
![image](/img/ir-code-reader-with-esp8266-wemos-d1-mini-and-esphome/shield_contacts_2.jpg)
![image](/img/ir-code-reader-with-esp8266-wemos-d1-mini-and-esphome/shield_contacts_1.jpg)

Now lets put the board into the housing with cable connected and glue it all:

![image](/img/ir-code-reader-with-esp8266-wemos-d1-mini-and-esphome/d1_mini.jpg)

Then put the shield on top:

![image](/img/ir-code-reader-with-esp8266-wemos-d1-mini-and-esphome/shield.jpg)

Finally, close the housing:

![image](/img/ir-code-reader-with-esp8266-wemos-d1-mini-and-esphome/ready.jpg)

{% include ads_media.html %}

## Using
In ESPHome let's create a config:

```yaml
esphome:
  name: ir_receiver
  platform: ESP8266
  board: d1_mini

wifi:
  ssid: "WiFi"
  password: "************"

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "IR Receiver"
    password: "************"

captive_portal:

# Enable logging
logger:
  level: DEBUG

# Enable Home Assistant API
api:
  password: "************"

ota:
  password: "************"

remote_receiver:
  pin:
    number: D4
    inverted: true
  dump:
    - samsung
```

As you can see it is set up to dump only Samsung-specific codes. You can read more about reading other codes on [esphome.io](https://esphome.io/components/remote_receiver.html). Also pay attention on `logger` section. It is set to `DEBUG` level.

Compile, flash, open logs and start to firing into our device with your IR remote. You'll see something like this:
```
[17:34:48][D][remote.samsung:055]: Received Samsung: data=0xE0E007F8
[17:34:51][D][remote.samsung:055]: Received Samsung: data=0xE0E020DF
```

You can now use this data with ESPHome [Remote Transmitter](https://esphome.io/components/remote_transmitter.html) component to build, for example, something like [this](/building-wifi-ir-remote-control-for-any-tv-with-esp8266-and-esphome).