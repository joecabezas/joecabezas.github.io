---
layout: post
title: Hack NFC para sofa
tags: nfc home-automation node-red hack
image: 2022/05/22/nfc-sofa-tag.png
---
Cuando comencé a jugar con la domótica, compré este paquete de
[50 tarjetas redondas NFC215](https://www.amazon.com/dp/B08DD24Z5K). Estas son
etiquetas que pueden almacenar hasta 540 bytes de datos, lo cual es suficiente
para mensajes cortos como enlaces web, intercambio de información personal,
intercambio de credenciales *wifi*, etc. En esta oportunidad quería usarlos como una
forma de alternar las luces de la sala de estar.

![](https://youtu.be/tqbdaos9qr0)

Al principio quería que la etiqueta NFC se programara para enviar un simple
mensaje UDP simple a una IP específica (mi servidor de domótica se ejecuta en
una Raspberry Pi 4), pero resulta que el estándar para las acciones NFC que se
pueden programar es muy limitado. como abrir enlaces web, crear un
nuevo contacto en un teléfono o agregar nuevas credenciales *wifi*. para hacer
esto hay software que te permite programar muchas de acciones como enviar
paquetes UDP, activar bluetooth, encender la luz del telefono, etc. El único
problema es que el teléfono lector necesita tener instalado el software que
interpreta la acción, lo bueno es que, el software
[(NFC Tools Pro)](https://play.google.com/store/apps/details?id=com.wakdev.nfctools.prohttps://play.google.com/store/apps/details?id=com.wakdev.nfctools.pro)
hace esto de manera inteligente, cuando agrega una acción personalizada, en
realidad crea 2 acciones en la etiqueta NFC: una para visitar el enlace de la
página de Google Play del software para descargar, y si el teléfono ya tiene la
aplicacion, entonces solo ejecutará la acción, esto es aceptable.

{%
  include picture_with_note.html
    preset="opener"
    src="2022/05/22/nfc-sofa-tag.png"
    alt="Insertando la etiqueta NFC en el brazo del sofa"
    class="w-75"
%}

Lo que terminé haciendo fue enviar una solicitud POST a mi Raspberry Pi con una
identificación única que describe la posición de esta etiqueta NFC, por lo
tanto, la acción real de encender las luces la decide luego el Raspberryi Pi
central, la etiqueta NFC solo envía el mensaje con su ID, esto me permite
configurar y modificar la acción final fácilmente sin tener que volver a
programar la etiqueta NFC.

El mensaje POST es capturado posteriormente por una instancia de
[Node-RED](https://nodered.org/)

{%
  include picture_with_note.html
    src="2022/05/22/nodered.png"
    alt="Node-Red screenshot"
%}

El cual toma el mensaje y filtra la identificación (en el caso del sofá, la
identificación sería `living_room_nfc_tag_sofa_right_arm`) y asigna una acción,
por ahora todas las etiquetas NFC que tengo son para alternar el grupo de luces
de la sala creado en [Home Assistant](https://www.home-assistant.io/), pero se
ve cómo se puede usar esto para otras cosas, como reproducir música, encender
la cafetera, el aire acondicionado, depositar comida al plato de una mascota,
enviar un mensaje MQTT o incluso activar cualquier comando de Linux.
