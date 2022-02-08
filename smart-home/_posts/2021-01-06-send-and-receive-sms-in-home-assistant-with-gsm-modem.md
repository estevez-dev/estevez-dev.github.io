---
title: 'Send and receive SMS in Home Assistant with GSM modem'
header:
  og_image: /img/teaser_500x300.jpg
redirect_from:
  - /send-and-receive-sms-in-home-assistant-with-gsm-modem
---

Yes, it is possible and you don't need to build anything from sources. Sending SMS from your home could be useful, for example, to send emergency alerts. But what about receiving and parsing SMS messages? Well, I used it to integrate my car security system with my Home Assistant. Now my Home Assistant could start the engine of my car automatically to warm it up before driving to work.

First of all, we need to set up notification service via GSM modem following the official [documentation](https://www.home-assistant.io/integrations/sms/). This will allow us to send SMS messages by calling `notify` service, for example in automation action:

```yaml
action:
  - service: notify.sms_yegor
    data:
      message: Hi!
```

![image](/img/send-and-receive-sms-in-home-assistant-with-gsm-modem/send_action.png)

But also this integration allows reading SMS messages sent to the modem phone number by listening to `sms.incoming_sms` event.

```yaml
trigger:
  - platform: event
    event_type: sms.incoming_sms
```
![image](/img/send-and-receive-sms-in-home-assistant-with-gsm-modem/income_trigger.png)

In automation `action`, we can now parse the message by searching keywords in it:
{% raw %}
```yaml
action:
  - choose:
      - conditions:
          - condition: template
            value_template: >-
              {{'engine is off' in trigger.event.data['text']}}
        sequence:
          -  ...
```
{% endraw %}
![image](/img/send-and-receive-sms-in-home-assistant-with-gsm-modem/parse_action.png)

Here we are checking for `engine is off` text in incoming SMS to perform some action. For example, to set the value of some `input_boolean`.