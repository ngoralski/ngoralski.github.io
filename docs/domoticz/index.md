title: domoticz
date: 2022-01-16 07:32:04


## Overall
Following [kiwi](https://github.com/xbeaudouin)'s recommendation I will use [domoticz](https://www.domoticz.com/)
and [tasmota](https://tasmota.github.io/) systems.

I've tried to use some other systems like :

## Requirement
A machine that will host your systems like a NAS (with docker) or a raspberry pi. 

???+ danger
Using Debian 11 is not recommended as there is some issues with domoticz and python 3.9.
Except if you use a docker based system in order to have the required version.

## Hardware

I've ordered some sonoff switches that are compatible with tasmota via a quick flash of the esp.

- [2 buttons](https://www.amazon.fr/gp/product/B07T8Y8FWR/)
- [3 buttons](https://www.amazon.fr/gp/product/B07T8XT3VZ/)


???+ warning
some recent equipments are not tasmota compatible because of hardware update.
So avoid systems like the following :
[Etersky 3 buttons switches](https://www.amazon.fr/gp/product/B07TM9NMJN)
[LoraTap switches](https://www.amazon.fr/gp/product/B08BR4GM7P)

## Systems

### MQTT system
You can use a simple system like mosquitto that is available via Debian and Fedora repositories.

???+ warning
Don't forget to configure mosquitto to listen on lan interface with the following configuration option

``` editorconfig
bind_interface eth0
```

### Domoticz

#### Addons
(domoticz_mqtt_discovery)[https://github.com/emontnemery/domoticz_mqtt_discovery)

### Tasmota

#### On SonOff 
Other Parameters / Model
``` editorconfig
{"NAME":"TX T3EU3C","GPIO":[32,1,0,1,226,225,33,34,224,576,0,0,0,0],"FLAG":0,"BASE":30}
```

``` editorconfig
Backlog setoption19 1; MqttUser 0; MqttPassword 0; FullTopic homeassistant/%prefix%/%topic%/
```

As i will use the switch for a shutter
``` editorconfig
SetOption80 1
```
