---
layout: post
title: NFC Sofa Hack
tags: nfc home-automation node-red hack
---

![Inserting the NFC tag inside sofa armrest]({{ '/assets/images/2022/05/22/nfc-sofa-tag.png' | resize: "800x800>" }})

When I started playing with home automation I purchased this pack of [50 NFC215 Round Cards](https://www.amazon.com/dp/B08DD24Z5K) this are tags that
can hold up to [540 bytes of data](https://www.shopnfc.com/en/content/6-nfc-tags-specs), which is enough for short messages like web links, personal
information sharing, wifi credentials sharing, etc. in this opportunity I wanted to used them as a way to toggle the living room lights.

![a](https://youtu.be/tqbdaos9qr0)

At first I wanted the NFC tag to be programmed to send a simple UDP message to a specific IP (my home automation server running on a Raspberry Pi 4),
but it turns out that the standard for NFC actions that can be programmed is very limited to just open web links, create a new contact on a phone,
or add new Wifi credentials. in order make this there are software that allows you to program tons of actions like sending UDP packets, turn on
bluetooth, etc. The only catch is that the reading phone needs to have installed the software that interprets the action, the good thing is that, the
the software
[(NFC Tools Pro)](https://play.google.com/store/apps/details?id=com.wakdev.nfctools.prohttps://play.google.com/store/apps/details?id=com.wakdev.nfctools.pro)
does this in a clever way, when adding a custom action it actually creates 2 actions in the NFC tag: one to visit the link of the software
google play page for download, and if the phone already have it, it will just execute the action, so I am ok with this then.

what I ended up doing is sending a POST request to my Raspberryi Pi with a unique id that describes the position of this NFC tag, therefore, the actual
action of turning on the lights is later decided by the central Raspberryi Pi, the NFC tag only sends the message with it's ID, this allows me to configure
and alter the final action easily without having to re-program the NFC tag.

The POST messgage is later captured by an instance of [Node-RED](https://nodered.org/), which finally toggles the living room lights group created on
[Home Assistant](https://www.home-assistant.io/), picture below:

![Node-Red screenshot](/assets/images/2022/05/22/nodered.png)
