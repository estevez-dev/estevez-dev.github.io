---
title:  "Home Assistant Lovelace mobile dashboard example"
header:
  teaser: /img/home-assistant-mobole-lovelace-dashboard-example/teaser.jpeg
  og_image: /img/home-assistant-mobole-lovelace-dashboard-example/view.png
---
I hate to dig into thousands of lines of someone's YAML code and think I'm not the only one. That's why I decided to tear down my Lovelace dashboard here for everyone who wants to reuse its components or get some inspiration for their dashboards.

My Lovelace dashboard for mobile devices is a result of a year of searching, editing and improving. The main card that made it look so nice is a grate [Button Card](https://github.com/custom-cards/button-card) by [RomRider](https://github.com/RomRider) - a really powerful tool to make whatever you want for your entities.

![dashboard](/img/home-assistant-mobole-lovelace-dashboard-example/view.png){:width="500px"}

## Theme

That was a [Clear Theme](https://github.com/naofireblade/clear-theme) from [naofireblade](https://github.com/naofireblade) at the beginning. But then I noticed that the last commit to Clear Theme was made more than a year ago and I also wanted to change active entity colour so now I'm maintaining and using a [fork of Clear Theme](https://github.com/estevez-dev/novago-theme).

## Room state

![Room state](/img/home-assistant-mobole-lovelace-dashboard-example/room-header.png){:width="400px"}

![Room state](/img/home-assistant-mobole-lovelace-dashboard-example/room-header-2.png){:width="400px"}

Made using [Custom Button Card templates](https://github.com/custom-cards/button-card#configuration-templates). It contains room name, a text representation of window/door state, movement indicator, temperature and air humidity data. You can get a template and usage example in my [custom button card templates collection](/smart-home/custom-button-card-templates#room-state).

## Entity buttons

![buttons](/img/home-assistant-mobole-lovelace-dashboard-example/buttons.png){:width="300px"}

This is the main button on my dashboard. It is flexible and responsive and can display various icons in the corners. Everything can be configured through [`variables`](https://github.com/custom-cards/button-card#variables) of [custom button card](https://github.com/custom-cards/button-card). You can get a template and usage example in my [custom button card templates collection](/smart-home/custom-button-card-templates#badge).

## Light buttons

![lights](/img/home-assistant-mobole-lovelace-dashboard-example/lights.png){:width="300px"}

This card is almost the same as the entity button, but with some changes. You can get a template and usage example in my [custom button card templates collection](/smart-home/custom-button-card-templates#light).

{% include ads_in_post.html %}

## Climate entities

![Heater](/img/home-assistant-mobole-lovelace-dashboard-example/heater_example.png){:width="300px"}
![Humidifier](/img/home-assistant-mobole-lovelace-dashboard-example/humidifier.png){:width="300px"}

The way to display my heaters and humidifiers. It is also a Custom Button Card with some additional styles and JavaScript templates. You can find it in the [collection](/smart-home/custom-button-card-templates#climate-device).

## Operation mode switch

![Humidifier](/img/home-assistant-mobole-lovelace-dashboard-example/mode-switcher.png){:width="300px"}

I have an [`input_select`](https://www.home-assistant.io/integrations/input_select/) to control my home operating mode. It has five possible states: `day`, `evening`, `night`, `away` and `guest`. Every mode button is a Custom Button Card. The central one has additional vertical border styles. See [layout](#layout) for the information of how it is grouped in the single card, and [template collection](/smart-home/custom-button-card-templates#mode-card) for template.

## Layout

I'm using [Horizontal Stack Card](https://www.home-assistant.io/lovelace/horizontal-stack/) to group button cards in a row without background like here:

![buttons](/img/home-assistant-mobole-lovelace-dashboard-example/buttons.png){:width="300px"}

But to group several button cards in one card with a custom background I'm using a more complex solution

![buttons](/img/home-assistant-mobole-lovelace-dashboard-example/stack-in-card.png){:width="300px"}

This is a combination of [Horizontal Stack Card](https://www.home-assistant.io/lovelace/horizontal-stack/), [Stack In Card](https://github.com/custom-cards/stack-in-card) and [card-mode](https://github.com/thomasloven/lovelace-card-mod).

Please remove comments if you can't save this card in the Home Assistant UI editor.
{: .notice--info}

```yaml
type: custom:stack-in-card
keep:
  margin: true # To keep cards margin inside a stack-in-card
card_mod: # card-mode styles. Adds a semi-transparent background and a bottom padding
  style:
    .: |
      ha-card {
        background: #ffffff77;
        padding-bottom: 16px;
      }
cards:
  - # Room state card
  - type: horizontal-stack
    cards:
      - # Light entity card
      - # Light entity card
      - # Light entity card
  - # Climate entity card
```
