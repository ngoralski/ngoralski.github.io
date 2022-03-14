# Tasmota

## Configuration
### Define Switches Model
#### SonOff Switches (T0 EU 2 Gang Switch (T0EU2C)) 
Other Parameters / Model
``` editorconfig
{"NAME":"Sonoff T0 2CH","GPIO":[17,255,255,255,0,22,18,0,21,56,0,0,0],"FLAG":0,"BASE":29}
{"NAME":"Generic","GPIO":[32,1,1,1,0,225,33,0,224,320,0,0,0,0],"FLAG":0,"BASE":29}
```

#### SonOff Switches (T0 EU 3 Gang Switch (T0EU3C)) 
Other Parameters / Model
``` editorconfig
{"NAME":"Sonoff T0 3CH","GPIO":[32,1,0,1,226,225,33,34,224,576,0,0,0,0],"FLAG":0,"BASE":30}
```

### Attach to MQTT for full discovery  
``` editorconfig
Backlog setoption19 1; MqttUser 0; MqttPassword 0; FullTopic homeassistant/%prefix%/%topic%/
```

### Define a shutter switch

Following [tasmota official documentation](https://tasmota.github.io/docs/Blinds-and-Shutters/#shutter-modes) 

First, close your blind and measure how much time it take to open it. 
Mine is 21s.

Then define the switch as a shutter
``` editorconfig
SetOption80 1
```

Define the default shutter mode
``` editorconfig
Shutter mode 1
```

Group buttons 1 and 2 together and activate it
``` editorconfig
Interlock 1,2
Interlock ON
```

If possible to avoid any injury on unexpected movement all RELAYS should start
in OFF mode when the device reboots:
``` editorconfig
PowerOnState 0
```

As i've inverted the wire.
``` editorconfig
ShutterButton 1 up
ShutterButton 2 down
```


Now we will configure the shutter with the measured time to close it.

We will use this [Google Spreadsheet](https://docs.google.com/spreadsheets/d/1-okVzGfdltbx8kMcObw4I0B22m9ZCQL3JgifgwjbzTc/edit#gid=0)

Step 1
```
shutteropenduration 21
shuttercloseduration 21
ShutterSetHalfway 50
shuttermotordelay 1
```

```
shutteropenduration 21
shuttercloseduration 21
ShutterSetHalfway 50
shuttermotordelay 0
```

Ensure that your blind is closed and execute the following command to define the 0 position.
Step 2
```
shuttersetclose
```

Measure in cm the position of the blind on each action and fill the spreadsheet
```
shutterposition 30
shutterposition 80
shutterposition 30
```

shuttercloseduration 21

shuttermotordelay 0.25
shuttercallibration 15.8 41.8 72.6 107.9

07:44:18.346 RSL: STATUS13 = {"StatusSHT":{"SHT0":{"Relay1":1,"Relay2":2,"Open":100,"Close":100,"50perc":50,"Delay":0,"Opt":"0000","Calib":[300,500,700,900,1000],"Mode":"0"}}}

Correction :
shuttermotordelay 0.5
shutteropenduration 23

``` shell
SetOption80 1
Shutter mode 1
Backlog Interlock 1,2; Interlock ON
Backlog PowerOnState 0; ShutterButton 1 up; ShutterButton 2 down; shutteropenduration 21; shuttercloseduration 23; ShutterSetHalfway 50; shuttermotordelay 0.5
```