---
layout: post
title: Smart Home Key Reminder
tags: home-automation node-red hack sonoff mqtt
image: 2022/06/15/keys-placed.png
---

I recently locked myself out of the apartment, luckily it was business hours
and I could ask consierge to open the door for me, if this happens after 10pm,
as per recent policies, I would be charged 100 usd in penalties.

Since I was working in my smart home, I thought it would be neat that the home
warn you whenever I am about to exit without my keys, and a week on
investigation started.

Here is a video on how it works

![](https://youtu.be/D_sBs_NYOe8)

Investigation started by setting objectives first, the key holder that informs
the smart home if they keys are present shall be:

* Battery operated
* Battery shall last more than a year
* It sends a message over Zigbee (preferred) or MQTT (via wifi)
* On idle, it shall draw NO energy (preferred) or little energy (<20 micro
Ampere)

since I already have experience building ESP8266 projects, I opted for this
microprocessor, the only difference this time, it needs to draw the least
ammount of energy as possible since it shall be operated by a small battery
like a CR2032 which has a capacity of around 220mAh.

This means when not in use it shall use a very small ammount of energy or
preferable no energy at all in order to make the battery to last more than a
year.

In order to achieve this my first guess was to use a feature of the ESP8266
called "Deep-Sleep" which according to the power consumption by power modes
table in the datasheet is only 20uA when on Deep-Sleep.

{%
  include picture_with_note.html
    src="2022/06/15/power-consumption-table.png"
    alt="Power Consumption by Power Modes"
%}

Then this is very simple to do, the ESP8266 it only needs to connect to the
wifi network, sends a message over MQTT to my local broker (running in a
Raspberry Pi 4), and then it enters into Deep-Sleep until triggered again.

But, how can I trigger a key holder?, my first guess was a mechanical solution
in which activates a sort of a lever when the keys are hold like a limit switch

{%
  include picture_with_note.html
    src="2022/06/15/end-stop-switch.png"
    alt="A limit switch"
%}

Then the keys when hold it will trigger the microprocessor and sends
the message, or even a better idea, what if when the switch is closed it also
closes the circuit giving it power, this way, when the keys are not present the
circuit will not consume power, and then present it will give it power but it
will be mostly on Deep-Sleep mode.

This solves 50% of the power issue, no energy consumption when not triggered
and only 20uA when the keys are present, is there any way to only consume power
when they keys are placed?, this is what I came up with:

{%
  include picture_with_note.html
    src="2022/06/15/mechanical-pulse-drawing.png"
    alt="Mechanical Diagram for pulse button"
%}

The idea is to have a way to hold the keys and roll a wheel that presses a
button when the keys are being placed and again when taking them out, this way
it only consumes energy when changing state, perfect!

But there is an issue, the button press will be short lived and it wont give
enough power to the ESP8266 to connect to the wifi and send a message,
specially since just connecting to the wifi it can take about 5 seconds, back
to the drawing board again...

At this point investigating, I saw this video from brilliant engineer [Andreas
Spiess](https://www.youtube.com/c/AndreasSpiess) in which talks about a way to
press a button and making the microprocessor to latch itself into the power
circuit and auto disconnect when the task is done, this way, the button press
(1) will power the ESP8266, then immediatelly the ESP8266 will will activate
the MOSFET (3) so energy can go from the battery (4) to the ESP (5), after
that, the button can be relased and the ESP8266 will still have energy to do a
longer task like connecting to wifi and send the message (image taken from
Andreas video).

{%
  include picture_with_note.html
    src="2022/06/15/mosfet-diagram.png"
    alt="Mechanical diagram for pulse button"
%}

Here is the video of this idea being explained:

![](https://youtu.be/nbMfb0dIvYc)

This is a very clever solution!, so I grabbed a ESP01 chip that I had doing
nothing, and programmed a quick program to send a MQTT message and built a
working prototype, here is a video of it working :blush:

![](https://youtu.be/L1iz5gWGAso)

At the time I got this prototype working a bunch of sensors came by mail from
Aliexpress, motion sensors, buttons, and door sensors all of them uses Zigbee
protocol for communication.

{%
  include picture_with_note.html
    src="2022/06/15/sonoff-sensors.png"
    alt="Sonoff sensors"
%}

Then it me, why not just grab a door sensor and adapt it to be my key holder
and put the magnet in the keychain?, so I started modeling a 3d printed
enclosure that hold the circuit and a small M3 nut which is ferromagnetic
enough to attract a magnet attached to my key chain

{% capture text %}
{%
  include picture_with_note.html
    preset="square"
    src="2022/06/15/sonoff-circuit.png"
    alt="Sonoff Circuit"
%}
<!--split-->
{%
  include picture_with_note.html
    preset="square"
    src="2022/06/15/printing-enclosure.png"
    alt="Printing the enclosure"
%}
<!--split-->
{%
  include picture_with_note.html
    preset="square"
    src=" 2022/06/15/enclosure-nut.png"
    alt="An M3 nut is placed for the keychain magnet"
%}
{% endcapture %}
{% include row_of_items.html text=text %}

{% capture text %}
{%
  include picture_with_note.html
    preset="square"
    src="2022/06/15/enclosure-with-circuit.png"
    alt="Circuit inside the enclosure"
%}
<!--split-->
{%
  include picture_with_note.html
    preset="square"
    src="2022/06/15/magnet-in-keychain.png"
    alt="Magnet placed in keychain"
%}
<!--split-->
{%
  include picture_with_note.html
    preset="square"
    src="2022/06/15/keys-placed.png"
    alt="Keys placed and detected"
%}
{% endcapture %}
{% include row_of_items.html text=text %}

And done!, now the sensor sends a message to the Raspberry Pi when closed
(keys placed) and when open (keys taken) as if it was a door, then Red-Node
grabs this information which is being used for automations

When the main door is opened, it checks if the keys are placed, if so, it tells
my Google Home to say `"Don't forget your keys!"`, and if the keys are taken,
meaning that I am outside it will say `"Welcome!"`, also, as an extra, I am
playing a little notification sound whenever the keys are placed as a way to
give me confitmation, I love how this sounds (check video at the beginning)

Here is how the Node-Red flow looks like:

{%
  include picture_with_note.html
    src="2022/06/15/node-red-flow.png"
    alt="Node-Red flow for key reminder automation"
%}

And here is the JSON export so you can import it in your instance of Node-Red

```json
[{"id":"099487586f45f55c","type":"mqtt in","z":"fec4b49ba92fcb02","name":"","topic":"zigbee2mqtt/main_door_contact_sensor","qos":"2","datatype":"json","broker":"d038737d9ec4f893","nl":false,"rap":true,"rh":0,"inputs":0,"x":190,"y":1360,"wires":[["75fa75f9290d2321","80b2102ecaacff89"]]},{"id":"75fa75f9290d2321","type":"switch","z":"fec4b49ba92fcb02","name":"contact is false","property":"payload.contact","propertyType":"msg","rules":[{"t":"false"}],"checkall":"true","repair":false,"outputs":1,"x":500,"y":1400,"wires":[["96c415c7e6adc921"]]},{"id":"80b2102ecaacff89","type":"switch","z":"fec4b49ba92fcb02","name":"contact is true","property":"payload.contact","propertyType":"msg","rules":[{"t":"true"}],"checkall":"true","repair":false,"outputs":1,"x":500,"y":1360,"wires":[[]]},{"id":"96c415c7e6adc921","type":"api-current-state","z":"fec4b49ba92fcb02","name":"","server":"a7e32585d87233f4","version":3,"outputs":1,"halt_if":"","halt_if_type":"str","halt_if_compare":"is","entity_id":"binary_sensor.main_door_contact_sensor_key_holder_contact","state_type":"str","blockInputOverrides":true,"outputProperties":[{"property":"payload","propertyType":"msg","value":"","valueType":"entityState"},{"property":"data","propertyType":"msg","value":"","valueType":"entity"}],"for":"0","forType":"num","forUnits":"minutes","override_topic":false,"state_location":"payload","override_payload":"msg","entity_location":"data","override_data":"msg","x":870,"y":1400,"wires":[["7a88cbe45a8a0840","e1b158241ac56f2d"]]},{"id":"7a88cbe45a8a0840","type":"switch","z":"fec4b49ba92fcb02","name":"off","property":"payload","propertyType":"msg","rules":[{"t":"eq","v":"off","vt":"str"}],"checkall":"true","repair":false,"outputs":1,"x":1230,"y":1400,"wires":[["5ac241fbc71c819a"]]},{"id":"e1b158241ac56f2d","type":"switch","z":"fec4b49ba92fcb02","name":"on","property":"payload","propertyType":"msg","rules":[{"t":"eq","v":"on","vt":"str"}],"checkall":"true","repair":false,"outputs":1,"x":1230,"y":1460,"wires":[["43963f905f72ddd8"]]},{"id":"5ac241fbc71c819a","type":"api-call-service","z":"fec4b49ba92fcb02","name":"","server":"a7e32585d87233f4","version":5,"debugenabled":false,"domain":"media_player","service":"volume_set","areaId":[],"deviceId":[],"entityId":["media_player.living_room_speaker"],"data":"{\"volume_level\": 1.0}","dataType":"json","mergeContext":"","mustacheAltTags":false,"outputProperties":[],"queue":"none","x":1410,"y":1400,"wires":[["b2af8179aa7c7953"]]},{"id":"43963f905f72ddd8","type":"api-call-service","z":"fec4b49ba92fcb02","name":"","server":"a7e32585d87233f4","version":5,"debugenabled":false,"domain":"media_player","service":"volume_set","areaId":[],"deviceId":[],"entityId":["media_player.living_room_speaker"],"data":"{\"volume_level\": 1.0}","dataType":"json","mergeContext":"","mustacheAltTags":false,"outputProperties":[],"queue":"none","x":1410,"y":1460,"wires":[["58259eec96fc6553"]]},{"id":"b2af8179aa7c7953","type":"api-call-service","z":"fec4b49ba92fcb02","name":"Joe, Don't forget your keys!","server":"a7e32585d87233f4","version":5,"debugenabled":false,"domain":"tts","service":"google_translate_say","areaId":[],"deviceId":[],"entityId":["media_player.living_room_speaker"],"data":"{\t   \"message\":\"Joe, Don't forget your keys!\",\t   \"cache\": \"true\"\t}","dataType":"jsonata","mergeContext":"","mustacheAltTags":false,"outputProperties":[],"queue":"none","x":1700,"y":1400,"wires":[[]]},{"id":"58259eec96fc6553","type":"api-call-service","z":"fec4b49ba92fcb02","name":"Welcome!","server":"a7e32585d87233f4","version":5,"debugenabled":false,"domain":"tts","service":"google_translate_say","areaId":[],"deviceId":[],"entityId":["media_player.living_room_speaker"],"data":"{\t   \"message\":\"Welcome!\",\t   \"cache\": \"true\"\t}","dataType":"jsonata","mergeContext":"","mustacheAltTags":false,"outputProperties":[],"queue":"none","x":1640,"y":1460,"wires":[[]]},{"id":"376c8512cb3712ff","type":"mqtt in","z":"fec4b49ba92fcb02","name":"","topic":"zigbee2mqtt/main_door_contact_sensor_key_holder","qos":"2","datatype":"json","broker":"d038737d9ec4f893","nl":false,"rap":true,"rh":0,"inputs":0,"x":230,"y":1480,"wires":[["4486b9ce45a6f9c2"]]},{"id":"4486b9ce45a6f9c2","type":"switch","z":"fec4b49ba92fcb02","name":"contact is true","property":"payload.contact","propertyType":"msg","rules":[{"t":"true"}],"checkall":"true","repair":false,"outputs":1,"x":520,"y":1480,"wires":[["fc1a27b69e93bc3f"]]},{"id":"fc1a27b69e93bc3f","type":"reusable","z":"fec4b49ba92fcb02","name":"","target":"set media player idle","outputs":1,"x":720,"y":1480,"wires":[["6f8a259d3efb8e8a"]]},{"id":"6f8a259d3efb8e8a","type":"api-call-service","z":"fec4b49ba92fcb02","name":"notification_sound.wav","server":"a7e32585d87233f4","version":5,"debugenabled":false,"domain":"media_player","service":"play_media","areaId":[],"deviceId":[],"entityId":["media_player.living_room_speaker"],"data":"{\"media_content_id\":\"http://192.168.1.80/notification_sound.wav\",\"media_content_type\":\"audio/mp3\"}","dataType":"json","mergeContext":"","mustacheAltTags":false,"outputProperties":[],"queue":"none","x":940,"y":1480,"wires":[[]]},{"id":"d038737d9ec4f893","type":"mqtt-broker","name":"","broker":"mosquitto","port":"1883","clientid":"","autoConnect":true,"usetls":false,"protocolVersion":"4","keepalive":"60","cleansession":true,"birthTopic":"","birthQos":"0","birthPayload":"","birthMsg":{},"closeTopic":"","closeQos":"0","closePayload":"","closeMsg":{},"willTopic":"","willQos":"0","willPayload":"","willMsg":{},"sessionExpiry":""},{"id":"a7e32585d87233f4","type":"server","name":"Home Assistant","version":2,"addon":false,"rejectUnauthorizedCerts":true,"ha_boolean":"y|yes|true|on|home|open","connectionDelay":true,"cacheJson":true,"heartbeat":false,"heartbeatInterval":"30"}]
```
