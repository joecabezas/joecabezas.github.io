---
layout: post
title: Recordatorio de llaves para Smart Home
tags: home-automation node-red hack sonoff mqtt
image: 2022/06/15/keys-placed.png
---
Hace poco me quedé fuera del apartamento, afortunadamente era horario normal y
pude pedirle al conserje que me abriera la puerta, si esto sucede después de
las 10:00 p. m., según las políticas recientes, me cobrarían 100 USD en multas.

Como estaba trabajando en mi *smart home*, pensé que sería buena idea  que la
casa me advirtiera cada vez que estoy a punto de salir sin mis llaves, y asi
comenzó una semana de investigación.

aqui un video de como funciona:

![](https://youtu.be/D_sBs_NYOe8)

La investigación comenzó en primer lugar fijando objetivos, el colgador de
llaves que informa al *smart home* si las llaves están presentes deberá:

* Funcionar con pilas
* La batería debe durar más de un año.
* Debe envíar un mensaje por Zigbee (preferido) o MQTT (vía wifi)
* En reposo, NO consumirá energía (preferido) o poca energía (<20
microamperios)

Como ya tengo experiencia en la construcción de proyectos con el
microprocesador ESP8266, opté por este microprocesador, la única diferencia
esta vez es que necesita consumir la menor cantidad de energía posible ya que
funcionará con una batería pequeña como una CR2032 que tiene una capacidad de
alrededor 220 mAh.

Esto significa que cuando no esté en uso, utilizará una cantidad muy pequeña de
energía o, preferiblemente, nada de energía para que la batería asi dure más de
un año.

Para lograr esto, mi primera suposición fue usar una función del ESP8266
llamada *"Deep-Sleep"* ("Sueño profundo") que, de acuerdo con la table de
consumo de energía en la hoja de datos, es solo 20 uA cuando está en
*Deep-Sleep*

{%
  include picture_with_note.html
    src="2022/06/15/power-consumption-table.png"
    alt="Consumo de energia en distintos modos"
%}

Esto es muy simple de hacer, el ESP8266 solo necesita conectarse a la red wifi,
envíar un mensaje a través de MQTT a mi servidor local (que se ejecuta en una
Raspberry Pi 4) y luego entra en *Deep-Sleep* hasta que se active nuevamente.

Pero, ¿cómo puedo activar un colgador de llaves? Mi primera suposición fue una
solución mecánica en la que se activa una especie de palanca cada vez que las
llaves se colocan como un interruptor de límite.

{%
  include picture_with_note.html
    src="2022/06/15/end-stop-switch.png"
    alt="Un interruptor de límite"
%}

Asi, cuando se colocan las llaves, activará el microprocesador y enviará el
mensaje.

Tengo una idea mejor, ¿y si el interruptor en lugar de dar una señal al
microprocesador le da energía al microprocesador?, de esta manera cuando las
llaves no están presentes el circuito no consumirá energía, genial.

Esto resuelve el 50% del problema de energía, no hay consumo de energía cuando
las llaves no están presentes y solo 20uA cuando las llaves están presentes,
¿hay alguna forma de consumir energía solo cuando las teclas están colocadas?
Esto es lo que se me ocurrió:

{%
  include picture_with_note.html
    src="2022/06/15/mechanical-pulse-drawing.png"
    alt="Diagrama mecanico para un boton de pulso"
    class="w-75"
%}

La idea es tener una forma de hacer rodar una rueda que presione un botón al
colocar o sacar las llaves, de esta manera solo consume energía al cambiar de
estado, ¡perfecto!

Pero hay un problema, la pulsación del botón será de corta duración y no le
dará suficiente tiempo (y energía) al ESP8266 para conectarse al wifi y enviar
el  mensaje, ya que solo conectarse al wifi puede demorar unos 5 segundos, de
vuelta al pizarron...

En este punto de investigación, vi este video del brillante ingeniero [Andreas
Spiess](https://www.youtube.com/c/AndreasSpiess) en el que habla sobre una
forma de presionar un botón y hacer que el microprocesador se enganche en el
circuito de alimentación y se desconecte automáticamente cuando termina la tarea.

De esta forma, la pulsación del botón (1) alimentará el ESP8266, el cual
activará el MOSFET (3) para que la energía pase de la batería (4) al ESP (5),
luego se podrá soltar el botón y el ESP8266 todavía tendrá energía para hacer
una tarea más larga como conectarse a wifi y enviar el mensaje (imagen tomada
del video de Andreas).

{%
  include picture_with_note.html
    src="2022/06/15/mosfet-diagram.png"
    alt="Circuito con el MOSFET"
    class="w-75"
%}

Aquí está el video de esta idea siendo explicada:

![](https://youtu.be/nbMfb0dIvYc)

¡Esta es una solución muy inteligente! Así que agarré un chip ESP01 que *no*
estaba haciendo nada y programé un programa rápido para enviar un mensaje MQTT
y pude construir un prototipo funcional, aquí hay un video de cómo funciona
:blush:

![](https://youtu.be/L1iz5gWGAso)

En el momento en que hice funcionar este prototipo, un montón de sensores
llegaron por correo de Aliexpress, sensores de movimiento, botones y sensores
de puerta, todos ellos usan el protocolo Zigbee para la comunicación.

{%
  include picture_with_note.html
    src="2022/06/15/sonoff-sensors.png"
    alt="Sensores Sonoff"
%}

Entonces tuve una idea: ¿por qué no agarrar un sensor de puerta y adaptarlo
para que sea mi colgador de llaves?

Solo necesito colocar un imán en el llavero, así que comencé a modelar una
carcasa impresa en 3D que sostiene el circuito con una pequeña tuerca M3 que es
lo suficientemente ferromagnética para atraer un imán adherido a mi llavero.

{% capture text %}
{%
  include picture_with_note.html
    preset="square"
    src="2022/06/15/sonoff-circuit.png"
    alt="Circuito Sonoff"
%}
<!--split-->
{%
  include picture_with_note.html
    preset="square"
    src="2022/06/15/printing-enclosure.png"
    alt="Imprimiendo la carcasa"
%}
<!--split-->
{%
  include picture_with_note.html
    preset="square"
    src=" 2022/06/15/enclosure-nut.png"
    alt="Una tuerca M3 es puesta para el iman de mi llavero"
%}
{% endcapture %}
{% include row_of_items.html text=text %}

{% capture text %}
{%
  include picture_with_note.html
    preset="square"
    src="2022/06/15/enclosure-with-circuit.png"
    alt="El circuito dentro de la carcasa"
%}
<!--split-->
{%
  include picture_with_note.html
    preset="square"
    src="2022/06/15/magnet-in-keychain.png"
    alt="Un iman puesto en el llavero"
%}
<!--split-->
{%
  include picture_with_note.html
    preset="square"
    src="2022/06/15/keys-placed.png"
    alt="Llaves puestas y detectadas"
%}
{% endcapture %}
{% include row_of_items.html text=text %}

¡Y listo!, ahora el sensor envía un mensaje a la Raspberry Pi cuando está
cerrada (llaves colocadas) y cuando está abierta (llaves tomadas) como si fuera
una puerta, enviando el evento a mi instancia de Red-Node.

Cuando se abre la puerta principal, verifica si las llaves están colocadas, si
es así, le dice a mi Google Home que diga `"¡No olvides tus llaves!"`, y si se
toman las llaves, dirá `"¡Bienvenido!"` (ya que podemos inferir que estoy afuera
si las llaves no estan puestas), también, como extra, estoy reproduciendo un
pequeño sonido de notificación cada vez que se colocan las llaves como una
forma de confirmarme, me encanta cómo suena (ver video al principio)

Así es como se ve el flujo de Node-Red:

{%
  include picture_with_note.html
    src="2022/06/15/node-red-flow.png"
    alt="Flujo de Node-Red para la automatizacion de recordatorio de las
    llaves"
%}

Y aquí está la exportación JSON para que puedas importarla en tu instancia de
Node-Red

```json
[{"id":"099487586f45f55c","type":"mqtt in","z":"fec4b49ba92fcb02","name":"","topic":"zigbee2mqtt/main_door_contact_sensor","qos":"2","datatype":"json","broker":"d038737d9ec4f893","nl":false,"rap":true,"rh":0,"inputs":0,"x":190,"y":1360,"wires":[["75fa75f9290d2321","80b2102ecaacff89"]]},{"id":"75fa75f9290d2321","type":"switch","z":"fec4b49ba92fcb02","name":"contact is false","property":"payload.contact","propertyType":"msg","rules":[{"t":"false"}],"checkall":"true","repair":false,"outputs":1,"x":500,"y":1400,"wires":[["96c415c7e6adc921"]]},{"id":"80b2102ecaacff89","type":"switch","z":"fec4b49ba92fcb02","name":"contact is true","property":"payload.contact","propertyType":"msg","rules":[{"t":"true"}],"checkall":"true","repair":false,"outputs":1,"x":500,"y":1360,"wires":[[]]},{"id":"96c415c7e6adc921","type":"api-current-state","z":"fec4b49ba92fcb02","name":"","server":"a7e32585d87233f4","version":3,"outputs":1,"halt_if":"","halt_if_type":"str","halt_if_compare":"is","entity_id":"binary_sensor.main_door_contact_sensor_key_holder_contact","state_type":"str","blockInputOverrides":true,"outputProperties":[{"property":"payload","propertyType":"msg","value":"","valueType":"entityState"},{"property":"data","propertyType":"msg","value":"","valueType":"entity"}],"for":"0","forType":"num","forUnits":"minutes","override_topic":false,"state_location":"payload","override_payload":"msg","entity_location":"data","override_data":"msg","x":870,"y":1400,"wires":[["7a88cbe45a8a0840","e1b158241ac56f2d"]]},{"id":"7a88cbe45a8a0840","type":"switch","z":"fec4b49ba92fcb02","name":"off","property":"payload","propertyType":"msg","rules":[{"t":"eq","v":"off","vt":"str"}],"checkall":"true","repair":false,"outputs":1,"x":1230,"y":1400,"wires":[["5ac241fbc71c819a"]]},{"id":"e1b158241ac56f2d","type":"switch","z":"fec4b49ba92fcb02","name":"on","property":"payload","propertyType":"msg","rules":[{"t":"eq","v":"on","vt":"str"}],"checkall":"true","repair":false,"outputs":1,"x":1230,"y":1460,"wires":[["43963f905f72ddd8"]]},{"id":"5ac241fbc71c819a","type":"api-call-service","z":"fec4b49ba92fcb02","name":"","server":"a7e32585d87233f4","version":5,"debugenabled":false,"domain":"media_player","service":"volume_set","areaId":[],"deviceId":[],"entityId":["media_player.living_room_speaker"],"data":"{\"volume_level\": 1.0}","dataType":"json","mergeContext":"","mustacheAltTags":false,"outputProperties":[],"queue":"none","x":1410,"y":1400,"wires":[["b2af8179aa7c7953"]]},{"id":"43963f905f72ddd8","type":"api-call-service","z":"fec4b49ba92fcb02","name":"","server":"a7e32585d87233f4","version":5,"debugenabled":false,"domain":"media_player","service":"volume_set","areaId":[],"deviceId":[],"entityId":["media_player.living_room_speaker"],"data":"{\"volume_level\": 1.0}","dataType":"json","mergeContext":"","mustacheAltTags":false,"outputProperties":[],"queue":"none","x":1410,"y":1460,"wires":[["58259eec96fc6553"]]},{"id":"b2af8179aa7c7953","type":"api-call-service","z":"fec4b49ba92fcb02","name":"Joe, Don't forget your keys!","server":"a7e32585d87233f4","version":5,"debugenabled":false,"domain":"tts","service":"google_translate_say","areaId":[],"deviceId":[],"entityId":["media_player.living_room_speaker"],"data":"{\t   \"message\":\"Joe, Don't forget your keys!\",\t   \"cache\": \"true\"\t}","dataType":"jsonata","mergeContext":"","mustacheAltTags":false,"outputProperties":[],"queue":"none","x":1700,"y":1400,"wires":[[]]},{"id":"58259eec96fc6553","type":"api-call-service","z":"fec4b49ba92fcb02","name":"Welcome!","server":"a7e32585d87233f4","version":5,"debugenabled":false,"domain":"tts","service":"google_translate_say","areaId":[],"deviceId":[],"entityId":["media_player.living_room_speaker"],"data":"{\t   \"message\":\"Welcome!\",\t   \"cache\": \"true\"\t}","dataType":"jsonata","mergeContext":"","mustacheAltTags":false,"outputProperties":[],"queue":"none","x":1640,"y":1460,"wires":[[]]},{"id":"376c8512cb3712ff","type":"mqtt in","z":"fec4b49ba92fcb02","name":"","topic":"zigbee2mqtt/main_door_contact_sensor_key_holder","qos":"2","datatype":"json","broker":"d038737d9ec4f893","nl":false,"rap":true,"rh":0,"inputs":0,"x":230,"y":1480,"wires":[["4486b9ce45a6f9c2"]]},{"id":"4486b9ce45a6f9c2","type":"switch","z":"fec4b49ba92fcb02","name":"contact is true","property":"payload.contact","propertyType":"msg","rules":[{"t":"true"}],"checkall":"true","repair":false,"outputs":1,"x":520,"y":1480,"wires":[["fc1a27b69e93bc3f"]]},{"id":"fc1a27b69e93bc3f","type":"reusable","z":"fec4b49ba92fcb02","name":"","target":"set media player idle","outputs":1,"x":720,"y":1480,"wires":[["6f8a259d3efb8e8a"]]},{"id":"6f8a259d3efb8e8a","type":"api-call-service","z":"fec4b49ba92fcb02","name":"notification_sound.wav","server":"a7e32585d87233f4","version":5,"debugenabled":false,"domain":"media_player","service":"play_media","areaId":[],"deviceId":[],"entityId":["media_player.living_room_speaker"],"data":"{\"media_content_id\":\"http://192.168.1.80/notification_sound.wav\",\"media_content_type\":\"audio/mp3\"}","dataType":"json","mergeContext":"","mustacheAltTags":false,"outputProperties":[],"queue":"none","x":940,"y":1480,"wires":[[]]},{"id":"d038737d9ec4f893","type":"mqtt-broker","name":"","broker":"mosquitto","port":"1883","clientid":"","autoConnect":true,"usetls":false,"protocolVersion":"4","keepalive":"60","cleansession":true,"birthTopic":"","birthQos":"0","birthPayload":"","birthMsg":{},"closeTopic":"","closeQos":"0","closePayload":"","closeMsg":{},"willTopic":"","willQos":"0","willPayload":"","willMsg":{},"sessionExpiry":""},{"id":"a7e32585d87233f4","type":"server","name":"Home Assistant","version":2,"addon":false,"rejectUnauthorizedCerts":true,"ha_boolean":"y|yes|true|on|home|open","connectionDelay":true,"cacheJson":true,"heartbeat":false,"heartbeatInterval":"30"}]
```
