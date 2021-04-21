---

title:  "Smart doorbell with Xiaomi Aqara button and Google Home"
redirect_from:
  - /2019/02/15/smart-doorbell-with-xiaomi-aqara-button-and-google-home
  - /smart-doorbell-with-xiaomi-aqara-button-and-google-home
---
So I have several Xiaomi Aqara buttons. Today I want to explain how to use this button as a smart doorbell that will play sound to your Google Home device through Home Assistant.

All my Aqara ZigBee devices connected to Home Assistant via [ConBee II](https://phoscon.de/en/conbee2) and [deCONZ](https://www.home-assistant.io/integrations/deconz/).

First of all let’s create a script that will be executed when button pressed. In your `scripts.yaml`:

```yaml
doorbell:
  alias: Doorbell
  sequence:
  - data:
      entity_id: media_player.googlehome4615 #Your Google Home device
    service: media_player.turn_on
  - delay: 00:00:03 #To make sure we are connected to Google Home
  - data:
      entity_id: media_player.googlehome4615 #Your Google Home device
      media_content_id: https://your.homeassistant.domain:8123/local/doorbell.mp3
      media_content_type: music
    service: media_player.play_media
  - delay: 00:00:05  #The same as doorbell.mp3 file ledgth
  - data:
      entity_id: media_player.googlehome4615
    service: media_player.turn_off
```

To make `doorbell.mp3` available by that url you need to place it to `/config/www` in your Home Assistant.

Now we need to create a handler of Aqara button in `automations.yaml`:

```yaml
- id: doorbell_button_handler
  alias: Doorbell button handler
  trigger:
  - event_data:
      entity_id: binary_sensor.lumi_lumiremoteb1acn01_01ef8594_1_on_off
    event_type: click
    platform: event
  action:
  - data:
      entity_id: script.doorbell
    service: script.turn_on
  initial_state: true
```

This automation will call our doorbell script on any type of click: short, long or double. That’s basically it for initial setup. All other things depends on your imagination. For example we can add a notification automation:

```yaml
- id: doorbell_notify
  alias: Doorbell notify
  trigger:
  - entity_id: script.doorbell
    from: 'off'
    platform: state
    to: 'on'
  condition: []
  action:
  - data:
      message: "Someone near your front door"
    service: notify.push
```