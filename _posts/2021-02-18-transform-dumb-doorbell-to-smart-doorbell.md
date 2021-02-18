---
title: Transform dumb doorbell to smart doorbell and connected to HA
tags: [home-assistant, HA, Debian, Supervised, Panasonic, SonOff, SonOff Mini]
style: border
color: primary
description: Making a smart doorbell using the dumb doorbell and Sonoff Mini then integrated to Home-Assistant.
---
Source: [KRDesigns.com blogs](https://www.krdesigns.com), [Home-Assistant](https://home-assistant.io), [Sonoff Mini](https://sonoff.tech/product/wifi-diy-smart-switches/sonoff-minir2), [AlexxIT SonoffLAN](https://github.com/AlexxIT/SonoffLAN)

## THE DEVICE
- Panasonic Electric Bell (EBG888)
- Panasonic Bell Button (EGG331)
- SonOff Mini

## SOFTWARE
- [Home Assistant](https://www.home-assistant.io)
- [AlexIT SonoffLAN add-on](https://github.com/AlexxIT/SonoffLAN)

## Why sonoff mini?
The reason why I used sonoff mini because its cheap, it support `pulse mode` which allow the switch to be call once and set it back to off using `inching settings`. Sonoff mini combine with AlexIT SonoffLAN add-on gave me an easy access to the the switch without doing much.

## How do I do it:
Combine all the hardware using this schematics   
![pic 1](https://raw.githubusercontent.com/krdesigns-com/krdesigns-com.github.io/master/img/pic1.png "my schema")

## The Prototype
This is the prototype before installing the entire project
![pic 2](https://raw.githubusercontent.com/krdesigns-com/krdesigns-com.github.io/master/img/doorbell.png "the prototype")

## Home-Assistant Code
```yaml
- alias: Notification Doorbell activated
  id: xxxx-xxxx-xxxx
  description: Notification doorbell being pushed
  trigger:
    - platform: state
      entity_id: switch.sonoff_xxxxxxxxxxxxx
      to: 'on'
  action:
    - service: tts.google_translate_say
      <etc>
    - service: camera.snapshot
      <etc>
    - service: notify.telegram
      <etc>
```

## Closing
I hope you like this project and please feel free to let me know how you do yours. Furthermore special thanks for those people who contribute at Home-Assistant. Great work and Great projects. 
