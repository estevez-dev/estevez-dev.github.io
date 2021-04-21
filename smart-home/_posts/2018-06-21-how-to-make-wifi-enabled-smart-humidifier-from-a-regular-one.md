---

title:  "DIY Smart Appliance: How to make WiFi-enabled smart humidifier from a regular one"
redirect_from:
  - /how-to-make-wifi-enabled-smart-humidifier-from-a-regular-one
---
So there was a simple air humidifier… Actually this post is a good example on how to make your regular boring home appliance little smarter. The main part of my humidifier will be the Wemos (or Lolin) D1 mini board based on ESP8266 WiFi chip.

My air humidifier has no automatic turn on and off, no timers, only a regulator that also works like switch. The idea is to make it a WiFi friendly and work with Home Assistant through MQTT.

Actually I was pretty lucky with this device because there is another models of it with built in sensor, screen and other stuff so the hardware was prepared to connect additional modules. But we don’t want to measure humidity inside humidifier right? So the sensor will be a separate part of system.

## Teardown
![image](/img/how-to-make-wifi-enabled-smart-humidifier-from-a-regular-one/001.jpg)

So it has an AC/DC power supply. Now we need to find 5v power line for our additions. All internal parts are powered with 12V DC so we need a step down module. In my case there was an empty 12V connector right on the factory board:

![image](/img/how-to-make-wifi-enabled-smart-humidifier-from-a-regular-one/IMG_20180809_225854719_LL-1024x768.jpg)

## Assembling
So we will connect there a regulated power down module and set it up to a 5V output:

![image](/img/how-to-make-wifi-enabled-smart-humidifier-from-a-regular-one/IMG_20180809_232228916-1024x768.jpg)

![image](/img/how-to-make-wifi-enabled-smart-humidifier-from-a-regular-one/IMG_20180809_232152411_LL-1024x768.jpg)

Now the main hardware. It is a Wemos D1 Mini ESP8266 based board (well almost because in my case it is a no-name copy of it, don’t buy it please) and a beautiful relay brick from Itead Studio:

![image](/img/how-to-make-wifi-enabled-smart-humidifier-from-a-regular-one/IMG_20180809_232324992-1024x768.jpg)

Connect it all together. Wemos D1 mini can be powered with stable 5V DC power connected to its 5V pin:

![image](/img/how-to-make-wifi-enabled-smart-humidifier-from-a-regular-one/IMG_20180809_232936237_LL-1024x905.jpg)

Now we need to unsolder the wires from a factory button or switch and connect it to our relay:

![image](/img/how-to-make-wifi-enabled-smart-humidifier-from-a-regular-one/IMG_20180809_234139616-1024x768.jpg)

The last step will be a gluing all new parts inside our device. Don’t repeat my mistake and glue WiFi board with microUSB port pointing outside of humidifier, not inside it:

![image](/img/how-to-make-wifi-enabled-smart-humidifier-from-a-regular-one/IMG_20180809_235904921-1024x768.jpg)

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

## Improvements

The final version of  my humidifier also checks the state of factory switch and report the state to [MQTT binary sensor](https://www.home-assistant.io/integrations/binary_sensor.mqtt/) in Home Assistant.  Here is a final diagram of this project:

[![image](/img/how-to-make-wifi-enabled-smart-humidifier-from-a-regular-one/humidifier_mqtt.png)](/img/how-to-make-wifi-enabled-smart-humidifier-from-a-regular-one/humidifier_mqtt.png)

The software was also impoved. There was [WiFiManager for ESP8266 boards](https://github.com/tzapu/WiFiManager) added to make it possible to configure WiFi and MQTT settings without flashing the board.

## The code

You can find the sketch for Arduino IDE [on my github](https://github.com/estevez-dev/edwin-home/tree/master/devices/humidifier_mqtt).

## Home Assistant

An example of MQTT platform switch configuration in Home Assistant for humidifier:

```yaml
switch:
  – platform: mqtt
    name: "Humidifier"
    state_topic: "edwin/humidifier/state"
    command_topic: "edwin/humidifier/power"
    state_on: "on"
    state_off: "off"
    payload_on: "1"
    payload_off: "0"
```

## ESPHome migration

And finaly here is a yaml configuration of my humidifier after it was migrated to [ESPHome](https://esphome.io/):

```yaml
esphome:
  name: humidifier
  platform: ESP8266
  board: d1_mini

wifi:
  ssid: "****"
  password: "************"

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Edwin Humidifier"
    password: "************"

captive_portal:

# Enable logging
logger:

# Enable Home Assistant API
api:
  password: "************"

ota:
  password: "************"

switch:
  - platform: gpio
    name: "Humidifier"
    pin: D1
    restore_mode: ALWAYS_OFF
    
binary_sensor:
  - platform: gpio
    name: "Humidifier state"
    pin:
      number: D5
      mode: INPUT
```