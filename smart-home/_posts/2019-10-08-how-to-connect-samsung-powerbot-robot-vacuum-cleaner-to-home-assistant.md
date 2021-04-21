---
title: 'How to connect Samsung PowerBot robot vacuum cleaner to Home Assistant'
redirect_from:
  - /2019/10/08/how-to-connect-samsung-powerbot-robot-vacuum-cleaner-to-home-assistant
  - /how-to-connect-samsung-powerbot-robot-vacuum-cleaner-to-home-assistant

---

While there is a [component in Home Assistant for Samsung SmartThings integration](https://www.home-assistant.io/integrations/smartthings/), the things with vacuum cleaners is not so obvious as it can be. While I’m not sure about an older versions of Samsung robot cleaners, I can say that you can’t control newer versions (like VR9300 in my case) using only Home Assistant integration. Need some magic here.

> I’m assuming that you already have your robot cleaner added to SmartThings app.

## Robot vacuum state

First we need to enable SmartThings integration in Home Assistant. [The instructions on home-assistant.io](https://www.home-assistant.io/integrations/smartthings/#basic-requirements) is quite detailed so I don’t see a reason to describe process here. Just follow it carefully.

After SmartThings integration created and some time passed after it (2-3 minutes in my case) you’ll be able to see new entities added inside integration. Go to **Configuration** -> **Integrations** in your Home Assistant web UI and find SmartThings there:

![image](/img/how-to-connect-samsung-powerbot-robot-vacuum-cleaner-to-home-assistant/Screenshot-2019-10-08-at-20.51.54.png)

Open it and you’ll find switch and several sensors:

![image](/img/how-to-connect-samsung-powerbot-robot-vacuum-cleaner-to-home-assistant/Screenshot-2019-10-08-at-20.52.53.png)

“Виталька” is a name of my vacuum cleaner. In my case `switch.[name]` do absolutely nothing but `sensor.[name]_robot_cleaner_movement` represents almost real time robot state. We will use it later as well as `sensor.[name]_battery`.

## Robot Vacuum controls

As switch from SmartThings integration do nothing and there is no other way to control our robot, we will use IFTTT for that. To enable IFTTT service in Home Assistant we need to configure this component in your `configuration.yaml`:

```yaml
ifttt:
  key: YOUR_API_KEY
```

replacing `YOUR_API_KEY` with an actual key from your IFTTT [Webhooks Settings](https://ifttt.com/maker_webhooks/settings). We need the last part of URL:

![image](/img/how-to-connect-samsung-powerbot-robot-vacuum-cleaner-to-home-assistant/Screenshot-2019-10-08-at-21.09.17.png)

Restart your Home Assistant once IFTTT configuration added and after that you will be able to fire events to IFTTT using `ifttt.trigger` service. While we still on IFTTT website lets create a couple of applets. One for starting cleaning, and the other for bringing our robot back to charger.

Click on your account picture in the upper right corner and choose **Create**.

![image](/img/how-to-connect-samsung-powerbot-robot-vacuum-cleaner-to-home-assistant/Screenshot-2019-10-08-at-21.14.41.png)

Click on **+This** on the next screen and search for “webhook”:

![image](/img/how-to-connect-samsung-powerbot-robot-vacuum-cleaner-to-home-assistant/Screenshot-2019-10-08-at-21.17.55.png)

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

Click on “Webhooks” card, choose “Receive a web request” on the next screen, come up with your event name (for example `start_powerbot`) and hit Create _trigger_:

![image](/img/how-to-connect-samsung-powerbot-robot-vacuum-cleaner-to-home-assistant/Screenshot-2019-10-08-at-21.22.18.png)

Now click on **+Thant**, search for “Samsung” and choose “Samsung Robot Vacuum”:

![image](/img/how-to-connect-samsung-powerbot-robot-vacuum-cleaner-to-home-assistant/Screenshot-2019-10-08-at-21.25.55.png)

IFTTT will ask you now to connect your Samsung SmartThings account and after successful login you will be able to choose an action to perform on your robot vacuum:

![image](/img/how-to-connect-samsung-powerbot-robot-vacuum-cleaner-to-home-assistant/Screenshot-2019-10-08-at-21.28.32.png)

As our trigger called `start_powerbot` let’s not create a mess and choose “Start vacuum cleaning”. Lastly IFTTT will ask you to choose a device from your SmartThings account to trigger an action on.

The same way we need to create an applet for stopping our robot. Make the same steps just changing event name for trigger to `stop_powerbot` and choosing “End vacuum cleaning” for action.

To make it simple for future use let’s move to Home Assistant and create a couple of scripts for starting and stopping our device though IFTTT events. Here is an example from my `scripts.yaml`:

```yaml
start_powerbot:
  alias: 'Powerbot: Start autoclean (IFTTT)'
  sequence:
  - data:
      event: start_powerbot
    service: ifttt.trigger
stop_powerbot:
  alias: 'Powerbot: Make it go home (IFTTT)'
  sequence:
  - data:
      event: stop_powerbot
    service: ifttt.trigger
```

## Putting things together

Now we have scripts and sensors enough to create vacuum entity using template platform. Here is my example. It converts SmartThings robot cleaner state to the state supported by template vacuum in Home Assistant. Also it sets battery level to “0” if the state of robot battery sensor is unknown as [template vacuum](https://www.home-assistant.io/integrations/vacuum.template/) platform will throw an error if battery state value will be not integer:
{% raw %}
```yaml
vacuum:
  - platform: template
    vacuums:
      vitalka:
        friendly_name: Виталька
        value_template: "{% if is_state('sensor.vitalka_robot_cleaner_movement', 'cleaning') %}cleaning{% elif is_state('sensor.vitalka_robot_cleaner_movement', 'charging') %}docked{% elif is_state('sensor.vitalka_robot_cleaner_movement', 'homing') %}returning{% elif is_state('sensor.vitalka_robot_cleaner_movement', 'idle') %}idle{% elif is_state('sensor.vitalka_robot_cleaner_movement', 'alarm') %}error{% elif is_state('sensor.vitalka_robot_cleaner_movement', 'powerOff') %}error{% else %}unknown{% endif %}"
        battery_level_template: "{% if is_state('sensor.vitalka_battery','unknown') %}0{% else %}{{states('sensor.vitalka_battery')}}{% endif %}"
        start:
          service: script.start_powerbot
        return_to_base:
          service: script.stop_powerbot
```
{% endraw %}
That’s it! Now we just need to restart Home Assistant to make our new `vacuum` entity to appear.