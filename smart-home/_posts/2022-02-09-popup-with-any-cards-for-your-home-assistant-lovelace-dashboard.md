---
title:  "Popup with any cards for your Home Assistant Lovelace dashboard"
toc: false
header:
  teaser: /img/popup-with-any-cards-for-your-home-assistant-lovelace-dashboard/example.png
  og_image: /img/popup-with-any-cards-for-your-home-assistant-lovelace-dashboard/example.png
---
Sounds nice, right? That was my dream for some time and I finally found the way to implement it using a powerful [browser_mod](https://github.com/thomasloven/hass-browser_mod) addon for Home Assistant.

<!--more-->

<iframe src="https://giphy.com/embed/xLtYH7xdTYg2I4ExyI" width="452" height="480" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>

First of all, you need to install [browser_mod](https://github.com/thomasloven/hass-browser_mod) preferably through [HACS](https://hacs.xyz/). The main purpose of browser_mod is to control any browser/device where you open your Home Assistant dashboard. But today we will use only one from a huge amount of its features.

There are actually two ways to implement popups replacing more-info dialogs. The newest one is described [here](https://github.com/thomasloven/hass-browser_mod#replacing-more-info-dialogs) and can replace more-info dialog per entity.

The old one is calling a popup using [Lovelace actions](https://www.home-assistant.io/lovelace/actions/) and I prefer it because you can attach it to any card regardless of entity it displays. You need to use `fire-dom-event` action for this. So here is an example of tap action for any card that will show a popup with two cards in a `vertical-stack`: `markdown` and `alarm-panel`. The popup will have the title "Home state".

```yaml
tap_action:
  action: fire-dom-event
  browser_mod:
    command: popup
    title: Home state
    card:
      type: vertical-stack
      cards:
        - type: markdown
          content: "## Home operating mode: day"
        - entity: alarm_control_panel.home_alarm
          name: Home alarm
          states:
            - arm_away
          type: alarm-panel
```