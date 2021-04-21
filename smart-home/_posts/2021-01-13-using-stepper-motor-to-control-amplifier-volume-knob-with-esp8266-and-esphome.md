---
title: 'Using stepper motor to control amplifier volume knob with ESP8266 and ESPHome'

redirect_from:
  - /using-stepper-motor-to-control-amplifier-volume-knob-with-esp8266-and-esphome
---

My living room multimedia setup consists of numerous devices with varying degrees of stupidity. I'm chasing to improve it adding additional DIY hardware and functionality not always because of hate of TV remotes but also because I'm in love with IoT and soldering. First, I've added [WiFi power control to my Cambridge Audio amplifier](/diy-smart-appliance-adding-wifi-to-common-sound-amplifier), then I've implemented [WiFi-to-IR remote control to the TV](/building-wifi-ir-remote-control-for-any-tv-with-esp8266-and-esphome). Then I decided to improve sound quality from TV and added a [DAC](https://wikipedia.org/wiki/Digital-to-analog_converter) between the TV and the amplifier. It is connected to the TV via [Toslink](https://wikipedia.org/wiki/TOSLINK) optical cable. And it broke my volume control because you can't change the volume on a digital output port on a TV, you forced to rotate the knob on amplifier instead.

I saw several solutions to this problem.
1. The easiest, but boring and expensive, is to purchase an improved model of my amplifier with IR remote control.
2. Сheap and the one that looks right is to add a digital potentiometer to the amplifier input and control it with Wemos D1 mini I already have in my amp.
3. Most inaccurate but fancy and the one you will use to impress your family and friends is to add a stepper motor that will rotate the knob for you.

I think you already guessed the way I choose, so let's see how I achieved this.

![image](/img/using-stepper-motor-to-control-amplifier-volume-knob-with-esp8266-and-esphome/gif.gif)

## Components
As I already have Wemos D1 mini and a relay in my amplifier, you could see more components than you expect on photos, but, for simplicity, I'll list only those components used to control the knob in this article.

1. WEMOS (LOLIN) D1 mini WiFi board ([wemos.cc](https://www.wemos.cc/en/latest/d1/d1_mini.html))
2. AC-DC 220V to 5V Step-Down Mini Power Supply ([amazon](https://www.amazon.com/HLK-PM01-supply-module-intelligent-household/dp/B07G5GL4B8))
3. ULN2003 stepper motor driver
4. 5V 28BYJ-48 stepper motor

### Connections

* 5V DC power from power supply, 5V pin of Wemos D1 mini and +5V input of motor driver are connected together.
* Ground of the power supply, GND pin of Wemos D1 mini and -5V (ground) of motor driver are also connected together.
* IN1 of motor driver -> D5 on Wemos
* IN2 of motor driver -> D6 on Wemos
* IN3 of motor driver -> D7 on Wemos
* IN4 of motor driver -> D8 on Wemos

! Note that ESPHome currently supports only two types of stepper motor drivers. See ESPHome [documentation](https://esphome.io/components/stepper/index.html) for more info.

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

## Building

The AC-DC power supply was connected to the AC power input of the amplifier. 

[![image](/img/using-stepper-motor-to-control-amplifier-volume-knob-with-esp8266-and-esphome/power.png)](/img/using-stepper-motor-to-control-amplifier-volume-knob-with-esp8266-and-esphome/power.png)

Now to the knob. The knob actually rotates an analogue [potentiometer](https://wikipedia.org/wiki/Potentiometer) - mechanical component with variable resistance. On my amplifier, all potentiometers have a hexagonal-shaped hole on the other side of the rotating element:

[![image](/img/using-stepper-motor-to-control-amplifier-volume-knob-with-esp8266-and-esphome/hole.jpg)](/img/using-stepper-motor-to-control-amplifier-volume-knob-with-esp8266-and-esphome/hole.jpg)

This made the connection of the motor as easy as a couple of rasp strokes. I mean the shaft of my motor was slightly bigger than the hole on the potentiometer so I took a rasp and made it smaller. Then I used a couple of plastic racks with nuts and a bunch of hot glue to anchor the motor on amp housing.

[![image](/img/using-stepper-motor-to-control-amplifier-volume-knob-with-esp8266-and-esphome/shaft.jpg?cropResize=600,600)](/img/using-stepper-motor-to-control-amplifier-volume-knob-with-esp8266-and-esphome/shaft.jpg)

[![image](/img/using-stepper-motor-to-control-amplifier-volume-knob-with-esp8266-and-esphome/motor_top.jpg?cropResize=600,600)](/img/using-stepper-motor-to-control-amplifier-volume-knob-with-esp8266-and-esphome/motor_top.jpg)

[![image](/img/using-stepper-motor-to-control-amplifier-volume-knob-with-esp8266-and-esphome/motor_side.jpg?cropResize=600,600)](/img/using-stepper-motor-to-control-amplifier-volume-knob-with-esp8266-and-esphome/motor_side.jpg)

I know it is not the best binding but it works. I don't think this solution is permanent because I relay want to use digital potentiometers instead. I made this more for fun, but currently, this is the only way to remotely control the volume in my living room. Just to give you an opportunity to change your mind before you start a similar project (or, who knows, maybe to improve it) here is a list of issues it currently have:

* It is impossible to determine the current motor position because its position resets on each boot.
* It is easy to brake reduction gears in motor and even the amp when rotating the potentiometer further than it could.
* It is hard to rotate the knob manually because you need to rotate the motor as well.

If you still here lats go to the software part.

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

## ESPHome

ESPHome could rotate your stepper motors with [Stepper Component](https://esphome.io/components/stepper/index.html). Here is my stepper config in ESPHome:

```yaml
stepper:
  - platform: uln2003
    id: volume_motor
    pin_a: D5
    pin_b: D6
    pin_c: D7
    pin_d: D8
    sleep_when_done: true
    max_speed: 250 steps/s
    acceleration: inf
    deceleration: inf
```

The idea is to change the volume on a given relative amount of motor steps. This will simplify the adjustment to find the right volume step for you. I declared a new service in `api` section of my `amplifier.yaml`:

```yaml
api:
  password: "*************"
  services:
    - service: set_volume
      variables:
        target: int
      then:
        - stepper.report_position:
            id: volume_motor
            position: 0
        - stepper.set_target:
            id: volume_motor
            target: !lambda 'return target;'
```

It would be exposed in Home Assistant as `esphome.amplifier_set_volume` service and will take the only parameter `target`. So in automation action increasing the volume in my case is looking like this:

![image](/img/using-stepper-motor-to-control-amplifier-volume-knob-with-esp8266-and-esphome/action.png)

Or in YAML:

```yaml
service: esphome.amplifier_set_volume
data:
  target: 50
```

Decreasing the volume action is the same, but with negative `target`:

```yaml
service: esphome.amplifier_set_volume
data:
  target: -50
```