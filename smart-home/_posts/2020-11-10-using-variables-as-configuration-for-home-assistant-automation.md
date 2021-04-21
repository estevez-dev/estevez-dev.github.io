---
title: 'Using variables as configuration for Home Assistant automation'

redirect_from:
  - /using-variables-as-configuration-for-home-assistant-automation
---

Home Assistant 0.115 got an ability to use variables in automation and scripts. You can declate variables even using templates and use them across the script or automation. Here is an example from release notes:
{% raw %}
```yaml
automation:
  trigger:
    platform: sun
    event: sunset
    offset: -00:30
  variables:
    notification_service: notify.paulus_iphone
  action:
    - service: "{{ notification_service }}"
      data:
        message: Beautiful sunset!
```
{% endraw %}
It is very useful and fun feature but at first I didn't realize how powerful it is. Today I want to show you an example how you can use variables as some sort of configuration for automation to avoid creating a lot of `if`s and automations to handle identical tasks.

I have a lot of ZigBee wall switches. Previously I had a separate automation to handle each of them. Every automation was handling:
* First button press - toggle the lights with max brightness
* Second button press - toggle the lights with low brightness
* First button long press - increase the brightnress
* Second button long press - decrease the brightness

After implementing `variables` for automation I decided to store the mapping of device id, event (represents the button pressed and the type of press: long or short), corresponding brightness and light `entity_id` in `variables` like this:
```yaml
  variables:
    device_map:
      00:15:8d:00:02:cb:1a:9b:
        light_entity_id: light.kitchen
        brightness_buttons:
          1001: -30
          1002: 50
          2001: 30
          2002: 100
      00:15:8d:00:02:c9:39:62:
        light_entity_id: light.hallway
        brightness_buttons:
          1001: -30
          1002: 55
          2001: 30
          2002: 100
      00:15:8d:00:02:e9:fe:9e:
        light_entity_id: light.living_room
        brightness_buttons:
          1001: 30
          1002: 100
          2001: -30
          2002: 60
      00:15:8d:00:02:cb:64:b4:
        light_entity_id: light.hallway
        brightness_buttons:
          1001: 30
          1002: 100
          2001: -30
          2002: 60
      00:15:8d:00:02:c9:37:7d:
        light_entity_id: light.bedroom
        brightness_buttons:
          1001: 30
          1002: 100
          2001: -30
          2002: 24
      00:15:8d:00:02:58:61:74:
        light_entity_id: light.bathroom
        brightness_buttons:
          1001: 30
          1002: 100
          2001: -30
          2002: 40
```

So now I can take the brightness value from my `device_map` by device id and event. Also I can take `entity_id` of the light each wall switch should control.

Also when I'm toggling the light I need to use `brightness_pct` service data to set the brightness, but when changing the brightness I need to use `brightness_step_pct`. So I added another variable to set brightness service data name:
{% raw %}
```yaml
variables:
  brightness_attr: '{% if (trigger.event.data[''event''] == 2001) or (trigger.event.data[''event'']== 1001) %}brightness_step_pct{% else %}brightness_pct{% endif %}'
```
{% endraw %}
So when the button holded the event is `2001` or `1001` (first and second button) and I need `brightness_step_pct`. I'm getting `brightness_pct` in another case.

Now we can use all this variables in `action`:
{% raw %}
```yaml
action:
  - data:
      '{{brightness_attr}}': '{{ device_map[trigger.event.data[''unique_id'']].brightness_buttons[trigger.event.data[''event'']]}}'
      entity_id: '{{ device_map[trigger.event.data[''unique_id'']].light_entity_id}}'
    service: light.{% if (trigger.event.data['event'] == 2001) or (trigger.event.data['event']== 1001) %}turn_on{% else %}toggle{% endif %}
```
{% endraw %}

Here we are using the most of variables - we are gatting the right brightness value by `unique_id` of our device and by `event` that is representing the button and the type of press:
{% raw %}
```
{{ device_map[trigger.event.data[''unique_id'']].brightness_buttons[trigger.event.data[''event'']]}}
```

Also we are getting light entity_id by `unique_id` of the switch:

```
{{ device_map[trigger.event.data[''unique_id'']].light_entity_id}}
```

And the full automation:

```yaml
- id: wall_switch_handler
  alias: Wall switch handler
  trigger:
  - event_data:
      event: 1001
    event_type: deconz_event
    platform: event
  - event_data:
      event: 1002
    event_type: deconz_event
    platform: event
  - event_data:
      event: 2001
    event_type: deconz_event
    platform: event
  - event_data:
      event: 2002
    event_type: deconz_event
    platform: event
  condition:
  - condition: template
    value_template: '{{''wall_switch'' in trigger.event.data[''id'']}}'
  action:
  - data:
      '{{brightness_attr}}': '{{ device_map[trigger.event.data[''unique_id'']].brightness_buttons[trigger.event.data[''event'']]}}'
      entity_id: '{{ device_map[trigger.event.data[''unique_id'']].light_entity_id}}'
    service: light.{% if (trigger.event.data['event'] == 2001) or (trigger.event.data['event']
      == 1001) %}turn_on{% else %}toggle{% endif %}
  variables:
    brightness_attr: '{% if (trigger.event.data[''event''] == 2001) or (trigger.event.data[''event'']
      == 1001) %}brightness_step_pct{% else %}brightness_pct{% endif %}'
    device_map:
      00:15:8d:00:02:cb:1a:9b:
        light_entity_id: light.kitchen
        brightness_buttons:
          1001: -30
          1002: 50
          2001: 30
          2002: 100
      00:15:8d:00:02:c9:39:62:
        light_entity_id: light.hallway
        brightness_buttons:
          1001: -30
          1002: 55
          2001: 30
          2002: 100
      00:15:8d:00:02:e9:fe:9e:
        light_entity_id: light.living_room
        brightness_buttons:
          1001: 30
          1002: 100
          2001: -30
          2002: 60
      00:15:8d:00:02:cb:64:b4:
        light_entity_id: light.hallway
        brightness_buttons:
          1001: 30
          1002: 100
          2001: -30
          2002: 60
      00:15:8d:00:02:c9:37:7d:
        light_entity_id: light.bedroom
        brightness_buttons:
          1001: 30
          1002: 100
          2001: -30
          2002: 24
      00:15:8d:00:02:58:61:74:
        light_entity_id: light.bathroom
        brightness_buttons:
          1001: 30
          1002: 100
          2001: -30
          2002: 40
  initial_state: true
  mode: parallel
  max: 10
```

I have not only wall switches so I'm using template condition here to make sure the switch is the wall switch:

```yaml
  condition:
  - condition: template
    value_template: '{{''wall_switch'' in trigger.event.data[''id'']}}'
```

Also this automation should be able to run in parallel to make it possible to hanlde seweral switch clicks at the same time in different rooms by different persons:

```yaml
  mode: parallel
  max: 10
```
{% endraw %}