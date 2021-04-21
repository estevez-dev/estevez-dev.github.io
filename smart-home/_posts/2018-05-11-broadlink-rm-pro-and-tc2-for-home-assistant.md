---
layout: post
title:  "BroadLink RM Pro and TC2 for Home Assistant"
redirect_from:
  - /broadlink-rm-pro-and-tc2-for-home-assistant
  - /2018/05/11/broadlink-rm-pro-and-tc2-for-home-assistant
---
BroadLink RM Pro is a WiFi to RF/IR gateway for smart home. First of all you need it to control BroadLink TC2 smart switches. Also it is very useful if you want to control IR devices from Home Assistant.

There is a [documentation on home-assistant.io on broadlink platform](https://www.home-assistant.io/components/switch.broadlink/). Here is just a quick summery with notes.

1. Add new device to BroadLink E-Control app
2. In the app go to Menu -> Share -> Share to other phones in WLAN -> Cancel
3. Copy `jsonSubIr`, `jsonButton` and `jsonIrCode` files from phone’s `/broadlink/newremote/SharedData/folder` to PC
4. Install python 2.7 on PC
5. Run `pip install simplejson`
6. Copy `getBroadlinkSharedData.py` from [NightRang3r’s repository](https://github.com/NightRang3r/Broadlink-e-control-db-dump) to a folder near files from p.3
7. Run `python getBroadlinkSharedData.py`
8. Select a device to get codes
9. Find results in a newly created file in the same directory.
10. Remove all new line symbols from code prior of using it in Home Assistant

Home Assistant TC2 configuration example and how to make a light component based on switch component:

```yaml
switch:
  – platform: broadlink
    host: 192.168.2.93
    mac: "78:0F:77:00:C8:95"
    timeout: 15
    switches:
      living_room_lights_power:
        friendly_name: "Living room lights power"
        command_on: ‘6RRGAN8JCRYJFhYJCRYJFgkWCRYWCRYJFgkJFgkWCRYJFhYJCRYJFgkWFgkWCRYJFgkWCQkWFgkWCQkWCRYJFgkWFgkJFgkAAUg=’
        command_off: ‘6RRGAN8JFgkWCQkWFgkWCRYJFgkWCRYJCRYJFgkWCRYWCQkWCRYWCRYJCRYWCQkWCRYJFgkWFgkWCRYJCRYJFhYJCRYJFgkAAUg=’
      hallway_lights_power:
        friendly_name: "Hallway lights power"
        command_on: ‘6RRGAN8JFgkJFgkWCRYJFhYJCRYWCQkWCRYJFhYJFgkJFhYJFgkJFhYJCRYJFhYJFgkJFhYJFgkWCQkWFgkJFhYJCRYJFgkAAUg=’
        command_off: ‘6RRGAN8JFgkJFgkWFgkJFhYJCRYJFgkWFgkJFhYJCRYWCRYJCRYWCQkWFgkWCRYJCRYWCQkWCRYJFgkWCRYWCRYJFgkWCQkAAUg=’

light:
  – platform: template
    lights:
      living_room:
        friendly_name: "Living room lights"
        value_template: ""
        turn_on:
          service: switch.turn_on
          data:
            entity_id: switch.living_room_lights_power
        turn_off:
          service: switch.turn_off
          data:
            entity_id: switch.living_room_lights_power
      hallway:
        friendly_name: "Hallway lights"
        value_template: ""
        turn_on:
          service: switch.turn_on
          data:
            entity_id: switch.hallway_lights_power
        turn_off:
          service: switch.turn_off
          data:
            entity_id: switch.hallway_lights_power
```