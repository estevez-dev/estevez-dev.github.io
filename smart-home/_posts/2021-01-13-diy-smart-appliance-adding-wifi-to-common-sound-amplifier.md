---
title: 'DIY Smart Appliance: Adding WiFi power control to an integrated amplifier'
layout: post
redirect_from:
  - /diy-smart-appliance-adding-wifi-to-common-sound-amplifier
---

![image](/img/diy-smart-appliance-adding-wifi-to-common-sound-amplifier/ext.jpg)

Previously I was using JBL Bar connected to a TV in my living room. It was controlled with WiFi IR bridge, so whenever my Chromecast or PS4 was in playing state, the Bar was turned on and ready.

Now I have passive speakers with a simple integrated amplifier, and it can't be turned on or off without physical interaction. You need to press and release the button on the front panel to toggle amplifier power. So today we will add WiFi power control to Cambridge Audion AXA25 integrated amplifier.

It is not a complex task to simulate button pressing. We just need a relay and a delay. But we also need to detect amplifier state because there are no differences in turning on and turning off actions. Fortunately, my amplifier has a USB port on the back to power some USB devices. Putting something with LED indicator in this port helped me to detect that the port is powered only when the amplifier is on. Digital pins of ESP8266 boards are good tools to detect current presence on a USB port. We will need a 5V power line and a ground line from the port. Here is a pinout of USB port:

![image](/img/diy-smart-appliance-adding-wifi-to-common-sound-amplifier/usb_pinout.png)

Let's drow!

[![image](/img/diy-smart-appliance-adding-wifi-to-common-sound-amplifier/amp_diagram.png)](/img/diy-smart-appliance-adding-wifi-to-common-sound-amplifier/amp_diagram.png)

### Components
1. LOLIN (Wemos) D1 mini ([wemos.cc](https://www.wemos.cc/en/latest/d1/d1_mini.html))
2. 5V Relay Brick by Itead ([itead.com](https://www.itead.cc/electronic-brick-5v-relay.html))
3. AC-DC 220V to 5V Step-Down Mini Power Supply ([amazon](https://www.amazon.com/HLK-PM01-supply-module-intelligent-household/dp/B07G5GL4B8))

Wemos has a [relay shield](https://www.wemos.cc/en/latest/d1_mini_shield/relay.html) as well but it can commutate up to 10A current so the electromagnet consumes more power. It is better to use some low-current relay for low-current circuits.

As you can see I'm taking 220V AC power from amplifier and converting it to 5V DC because we need to power the relay even when the amplifier is off. There is an always powered low-current circuit exists in the amplifier, but we can't use it because of.... well, low current. Connecting a WiFi module and a relay to it will definitely destroy our amplifier.

[![image](/img/diy-smart-appliance-adding-wifi-to-common-sound-amplifier/int.jpg)](/img/diy-smart-appliance-adding-wifi-to-common-sound-amplifier/int.jpg)

<div class="adsblock">
<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"
     style="display:block; text-align:center;"
     data-ad-layout="in-article"
     data-ad-format="fluid"
     data-ad-client="ca-pub-6530242109614004"
     data-ad-slot="2178866199"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>
</div>

### Firmware

#### MQTT

If you want to use Arduino IDE to develop and flash the firmware to your Wemos D1 Mini board you'll need to add ESP8266 board manager and tools for Arduino. You can do this by performing [several simple steps from the official ESP8266 repository](https://github.com/esp8266/Arduino#installing-with-boards-manager).
Then you can see my [MQTT-based firmware sources](https://github.com/estevez-dev/edwin-home/tree/master/devices/amplifier_mqtt) I was using before migrating to ESPHome. It could be an example, or you can just use it all.

#### ESPHome

[ESPHome](https://esphome.io/) actually makes such projects much easier to implement, improve and support. It also has a very reliable [Home Assistant plugin](https://github.com/esphome/hassio) and integration. That is why I moved all my DIY projects to ESPHome and here is a configuration file of my amplifier:

```yaml
esphome:
  name: amplifier
  platform: ESP8266
  board: d1_mini

wifi:
  ssid: "****"
  password: "**************"

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Cambridge AXA25"
    password: "*******"

captive_portal:

# Enable logging
logger:

# Enable Home Assistant API
api:
  password: "******"

ota:
  password: "******"

binary_sensor:
  - platform: gpio
    id: amplifier_power
    internal: true
    pin:
      number: D2
      mode: INPUT_PULLDOWN_16

switch:
  - platform: gpio
    id: relay
    pin: D1
    restore_mode: ALWAYS_OFF
  - platform: template
    name: "Amplifier"
    id: amplifier
    lambda: |-
      if (id(amplifier_power).state) {
        return true;
      } else {
        return false;
      }
    turn_on_action:
      - if:
          condition:
            binary_sensor.is_off: amplifier_power
          then:
          - switch.turn_on: relay
          - delay: 300ms
          - switch.turn_off: relay
    turn_off_action:
      - if:
          condition:
            binary_sensor.is_on: amplifier_power
          then:
          - switch.turn_on: relay
          - delay: 300ms
          - switch.turn_off: relay
```

That's it for today. Thanks for reading.