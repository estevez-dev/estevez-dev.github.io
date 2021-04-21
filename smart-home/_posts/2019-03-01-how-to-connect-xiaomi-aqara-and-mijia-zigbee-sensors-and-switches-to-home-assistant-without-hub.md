---
title: '[Updated] How to connect Xiaomi Aqara and Mijia ZigBee sensors and switches to Home Assistant without hub'
layout: post
redirect_from:
  - /2019/03/01/how-to-connect-xiaomi-aqara-and-mijia-zigbee-sensors-and-switches-to-home-assistant-without-hub
  - /how-to-connect-xiaomi-aqara-and-mijia-zigbee-sensors-and-switches-to-home-assistant-without-hub
---

The last thing you want to see setting up your smart home with Home Assistant is definitely huge amount of hubs and separate accounts. But we have some good sensors and switches from Xiaomi in other hand that requires Aqara hub and app to set up and work. And there is several ways to connect those devices to Home Assistant without that hub and app. I want to show you one of those possible solutions based on hass.io and Raspberry Pi 3B+.

## ZigBee for Raspberry Pi

Xiaomi devices communicate with hub using ZigBee so first of all we need some extension for our Raspberry Pi to enable ZigBee. Previously I was using [ZigBee shield from Elelabs](https://elelabs.com/products/elelabs_zigbee_shield.html) for this. It is small and looks nice. And it works. But I moved to [ConBee II](https://www.phoscon.de/en/conbee2) because of a lot of issues with delays and disconnections. You can use any you want but it is better to stick to the list of already tested and confirmed modules in [official HA docs](https://www.home-assistant.io/components/zha/).

**If you are using a shield instead of USB device** you need to edit `config.txt` file placed in `/boot` of hassio. This is easy to do because `/boot` is a separated FAT32 partition and it will be visible and editable even on Windows PC. So just insert your SD card with HA into your PC or laptop and add this line to the end of `cofig.txt`:

```
dtoverlay=pi3-miniuart-bt
```

Now we can attach ZigBee shield to our server, bring back the SD card and boot Home Assistant. Then wee need to find our device address. Usually it is `/dev/ttyAMA0` for shields or `/dev/ttyUSB0` for USB sticks. But if you don’t want to guess just go to **Supervisor** -> **System** and click **Hardware** button under “Host system”.

Alternatively device address could be found using ssh connection to your Home Assistant. By default ssh is not enabled in hassio. To enable it you need to install [SSH server add-on](https://www.home-assistant.io/addons/ssh/).

Assuming we can connect to our HA via ssh, the next step is to execute command there:

```
hassio hardware info
```

After executing it we will get some output:

```
audio:
  "0":
    devices:
    - chan_id: "0"
      chan_type: digital audio playback
    - chan_id: "1"
      chan_type: digital audio playback
    name: bcm2835_alsa - bcm2835 ALSA
    type: ALSA
disk: []
gpio:
- gpiochip0
- gpiochip128
input: []
serial:
- /dev/ttyAMA0
- /dev/ttyS0
```

Here you can find two serial ports listed under `serial:`. One of them definitely used by ZigBee shield.

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

## Home Assistant configuration

There is an official component for ZigBee integration in Home Assistant: [zha](https://www.home-assistant.io/components/zha/). If you are using ConBee device there is also another component available: [deCONZ](https://www.home-assistant.io/integrations/deconz/).

You don’t need to update your `configuration.yaml`. All we need is to create new **Integration** in your Home Assistant. Go to **Configuration** -> **Integrations** and click “+” button to add new integration. Search for “ZHA”. and fill in “device path” field with your shield or USB stick path (in my case it was `/dev/ttyAMA0` for Elelabs ZigBee shield).

Restart Home Assistant server after adding an integration.

## Connecting a device

If previous steps was successful you’ll find new entry under **Configuration** named **ZHA**. This is very useful interface to manage all your ZigBee devices and add new. So to add your first device click “Add devices” link. You will then have 60 seconds to pair your switch, sensor of light following it’s original instructions.

!! Device you want to connect should be placed right near your Raspberry Pi. After successfully connected it can be moved wherever you want in a range of ZigBee operating.

To connect Xiaomi Aqara or Mijia sensor we need:

1. Place the button as close as possible to your Home Assistant server
2. Click “Add devices” under **Configuration** -> **Integrations** -> **ZHA**
3. Hold small button on Aqara switch during 3-5 seconds until blue light will flash several times
4. Release the button and click it every 4-5 seconds to prevent device going to “sleep”
5. In a normal conditions you’ll soon see a new device added with an option to change it’s display name and Area.

All Aqara and Mijia sensors can be connected the same way. After success connection new entities should appear. Usually there are several Home Assistant entities created for each sensor.

## Smart switch handling

There is several types of smart switches from Xiaomi. For example – Aqara smart button WXKG11LM. It can be connected the same way as sensors, but you don’t need to click a button on device every second in step 4. Just hold reset button for 3-5 seconds, release it after blue light flashes and wait for successful device initialization.

Smart switches will appear in Home Assistant as `binary_sensor` but you can’t handle clicks using its state. You need to listen events instead. To handle WXKG11LM smart button click, double click or long click we need to know some data from the event that will be fired. Go to **Developer Tools** -> **Events**. Find ” Listen to events ” section in the bottom and subscribe to `zha_event`. Now you will see all data available in event object when clicking buttons on your device.

Usually you will need `unique_id` to know which button on which device was clicked, and `command` to know how the button was clicked. For example to handle right button single click for a specific switch we can create an automation:

```yaml
- id: bedroom_button_handler
  alias: Bedroom button handler
  trigger:
  - event_data:
      command: right_single,
      unique_id: "00:15:8d:00:02:c9:39:62:2:0x0012"
    event_type: zha_event
    platform: event
  action:
  - data_template:
      entity_id: 'light.bedlight'
    service_template: 'light.toggle'
```

That is all for now. Thanks for reading.