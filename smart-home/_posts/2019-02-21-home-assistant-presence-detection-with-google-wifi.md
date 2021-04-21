---
layout: post
title: '[Updated] Home Assistant presence detection with Google WiFi'
redirect_from:
  - /2019/02/21/home-assistant-presence-detection-with-google-wifi
  - /home-assistant-presence-detection-with-google-wifi
---

Previously my presence detection in Home Assistant works through Asus router and `asuswrt` component. And it was bad. Very bad. Recently I switched to Google WiFi and realized that the only way to implement presence detection with this router is IFTTT (Update: No). I thought it would be the slowest presence detection in the world but I was wrong.

! While you still can do this through IFTTT you need to know that there is much easier way not depending on your router model and any 3rd party services – [Nmap Tracker](https://www.home-assistant.io/integrations/nmap_tracker/). You just enable this component and configure your devices IP addresses.
!
! I configured static IP addresses for devices I wanted to track using [DHCP IP Reservation](https://support.google.com/wifi/answer/6274660?hl=en) on Google WiFi router.
!
! If you still want to use IFTTT, you are welcome to continue reading.

## Home Assistant IFTTT webhook
First of all we need to create a webhook for IFTTT in our Home Assistant instance through Integrations. Go to “Configuration” -> “Integrations” in Home Assistant web UI. Find “IFTTT” in “Set up a new integration” section and click “CONFIGURE”. You will be provided with webhook url to use in IFTTT applets. It should look similar to this:

```
https://ha.mydomain.com:8123/api/webhook/e7cd74856399e8934b8f5beffeeeaee4c351cdc8373647585ec040c7b69c2b999
```
Save this url somewhere because you will not be able to see it again in your Home Assistant UI.

## Home Assistant entities
For presence detection I’m using `binary_sensor`s with `device_class: presence`. Also the state of those sensors depends on corresponding `input_boolean`. It is made to be able to switch someone presence on and off manually by adding `input_boolean` to UI and switching its state. Don’t set the initial state for `input_boolean`. It will make it possible to save and restore its state on HA reboots. Also I set this sensors to have different pictures depends on presence.
{% raw %}
```yaml
input_boolean:
    is_yegor_home:
      name: “Yegor’s presence”

binary_sensor:
  - platform: template
    sensors:
      presence_yegor:
        friendly_name: "Yegor"
        value_template: "{{ is_state('input_boolean.is_yegor_home', 'on') }}"
        device_class: presence
        entity_picture_template: "/local/yegor_picture_bw.jpg"
```
{% endraw %}
## IFTTT applets
Next. There is an official Google WiFi service in IFTTT and it can create IF’s like “If some device connected/disconnected”. That’s exactly what we need.

![](/img/2019-02-21/ifttt_google_wifi.png)

Open [IFTTT website](https://ifttt.com/) and login with your account. Go to “My Applets” and click “New Applet” in upper right corner.

![](/img/2019-02-21/screenshot-ifttt.com-2019.02.21-11-10-25.png)

Then hit “+this”, search for “Google WiFi” and click it. You will be asked to authorize IFTTT in your google account. After that you will be able to select one of the triggers Google WiFi can send to IFTTT:

![](/img/2019-02-21/screenshot-ifttt.com-2019.02.21-11-20-06.png)

Let’s start with “Device connects”. Click it and select device name you want to track on the next step.

Next you will be asked to create an action by clicking “+that”. For action you need to find “Webhooks” service and choose the only action it provides – “Make a web request”.

Now we need to fill all the fields for web request:

URL: `The url of our Home Assistant webhook we created recently`
Method: `POST`
Content Type: `application/json`
Body: `{ "action": "call_service", "service": "input_boolean.turn_on", "entity_id": "input_boolean.is_yegor_home"}`

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

## Handling IFTTT requests

The last thing we need is to create a handler in Home Assistant for web requests from IFTTT. It will be a new automation:
{% raw %}
```yaml
- id: 'ifttt_webhook'
  alias: IFTTT Webhook
  trigger:
    platform: event
    event_type: ifttt_webhook_received
    event_data:
      action: call_service
  action:
    service_template: '{{ trigger.event.data.service }}'
    data_template:
      entity_id: '{{ trigger.event.data.entity_id }}'
```
{% endraw %}

This automation will handle any requests from IFTTT that will have `action: call_service` in their body with `service` and `entity_id` in it.

For disconnecting from Google WiFi we need to create another applet in IFTTT. It will be the same except of two things:

1. We should select “Device disconnects” from Google WiFi services when creating “+this” for applet.
2. We need to replace input_boolean.turn_on with input_boolean.turn_off in web request’s “Body” field when creating “+that” to make it looks like this:

```
{ "action": "call_service", "service": "input_boolean.turn_off", "entity_id": "input_boolean.is_yegor_home"}
```

## Automations based on presence
Assuming we’ve set up presence detection for everyone living in our home, we now need one variable to know if there is nobody home now. It would be helpful when creating automations that should be triggered when there is nobody home or when someone back home. So we will add all our presence sensors in a group in `groups.yaml`:

```yaml
presence:
    entities:
        - binary_sensor.presence_someone
        - binary_sensor.presence_yegor
        - binary_sensor.presence_someone_else
```

Now we can rely on that group’s state to check if there is nobody home. I’ve added a delay in 10 minutes to make sure this automation will not be triggered when I’m simply reboot my device or there was short connection lost to a router.

```yaml
- id: 'nobody_home'
  alias: 'Nobody home'
  trigger:
  - entity_id: group.presence
    for:
      minutes: 10
    from: 'on'
    platform: state
    to: 'off'
  condition: []
  action:
  - data:
      message: “Looks like there is nobody home now”
    service: notify.push
```

That is all for now. Thanks for reading.