!!! note "This feature is included only in tasmota-knx build"     

To use in other builds you must [compile your own build](Compile-your-build). Add the following to `user_config_override.h`:
```
#ifndef USE_KNX
#define USE_KNX         // Enable KNX IP Protocol Support (+9.4k code, +3k7 mem)
#endif
```
----
## What is KNX?

[<img src="https://www.knx.org/wGlobal/wGlobal/layout/images/knx-logo.png" />](https://www.knx.org/knx-en/for-professionals/index.php)

The [KNX IP Protocol](https://en.wikipedia.org/wiki/KNX_(standard)) is an _international open standard_ for smart homes and smart buildings automation. It is a decentralized system. Each device can talk directly to each other without the need of a central controller or server. Any panel or server is just for telesupervision and for sending requests. KNX IP Protocol uses a UDP multicast on _224.0.23.12 : 3671_, so there is no need for a KNX Router unless you want to communicate to KNX Devices that are not in the WIFI Network (Twisted Pair, RF, Powerline).

Each device has a physical address (like a fixed IP) as **1 . 1 . 0** and that address is used for configuration purposes.

Each device can be configured with group addresses as **2 / 2 / 1** and that address can be used for sending/receiving commands.
So, for example, if 2 devices that are configured with the **2 / 2 / 1** for turning on/off their outputs, and other device send _Turn ON_ command to **2 / 2 / 1**, both devices will turn on their outputs.

## Integration

Several home automation systems have KNX support. For example, [Home Assistant](https://github.com/home-assistant/home-assistant) has a [XKNX Python Library](https://github.com/XKNX/xknx) to connect to KNX devices using a KNX Router. If you don't have a **KNX Router**, you can use a **Software KNX Router** like [KNXd](https://github.com/knxd/knxd) on the same Raspberry Pi than Home Assistant. KNXd is used by Home Assistant for reading this UDP Multicast, although KNXd has other cool features that need extra hardware like connect to KNX devices by Twister Pair, Power Line or RF.

If using the Home Assistant distribution called **Hassio**, everything for KNX is already included by default.

If you use the ETS (KNX Configurator Software) you can add any TasmotaTasmota KNX as a dummy device.

If the Tasmotadevice is connecting to a Wifi Repeater you might experience some issues receiving KNX Telegrams. This also applies to mDNS and Emulation features.

## Implemented Features 

The implemented features, up to now, in KNX for Tasmota are:

General:

* buttons (just push)
* relays (on/off/toggle)
* lights (led strips, etc. but just on/off)

Sensor lists that you can use in KNX is (only one sensor per type):

* Temperature
* Humidity
* Energy (v, i, power)

For using rules:

* send KNX command (on/off)
* receive KNX command (on/off)
* send values by KNX (any float type, temperature for example)
* receive a KNX read request
* send and receive SCENE commands

## Usage Examples ##

There are multiple possible configurations. Here are explained just a few as example. The options for selecting relays, buttons, sensors, etc. are only available if were configured on _Configure Module Menu_.

To configure KNX, enter on the Configuration Menu of Tasmota and select Configure KNX.

<img src="https://raw.githubusercontent.com/ascillato/Tasmota_KNX/KNX_development/.github/Config_Menu.jpg" />
<img src="https://raw.githubusercontent.com/ascillato/Tasmota_KNX/KNX_development/.github/KNX_menu.jpg" />

_Note on KNX communication enhancement option: As Wifi Multicast communication is not reliable in some wifi router due to IGMP problems or Snooping, an enhancement was implemented. This option increase the reliability by reducing the chances of losing telegrams, sending the same telegram 3 times. In practice it works really good and it is enough for normal home use. When this option is on, Tasmota will ignore toggle commands by KNX if those are sent more than 1 toggle per second. Just 1 toggle per second is working fine._


### 1) Setting Several Tasmota to be controlled as one by a Home Automation System: ###

We can set one of the group address to be the same in all the devices so as to turn them on or off at the same time.
In this case, so as to inform the status of all the relays to the Automation System, just one of the devices have to be configured as the responder. If you use the same Group Address for sending and receiving, you have to take into account not to make loops.

DEVICE 1

<img src="https://raw.githubusercontent.com/ascillato/Tasmota_KNX/KNX_development/.github/1.jpg" />

DEVICE 2

<img src="https://raw.githubusercontent.com/ascillato/Tasmota_KNX/KNX_development/.github/2.jpg" />

### 2) Setting 2 Tasmota to be linked as stair lights: ###

We can set one device to send the status of its output and another to read that and follow. And the second device can send the status of its button and the first device will toggle. With this configuration we can avoid to make a loop.

DEVICE 1

<img src="https://raw.githubusercontent.com/ascillato/Tasmota_KNX/KNX_development/.github/3.jpg" />

DEVICE 2

<img src="https://raw.githubusercontent.com/ascillato/Tasmota_KNX/KNX_development/.github/4.jpg" />

### 3) Setting a button as initiator of a scene:

Just setting one device to send the push of a button, and the rest just use that value to turn them on. In this case, there is no toggle. Every time the button is pushed, the turn on command is sent.

DEVICE 1

<img src="https://raw.githubusercontent.com/ascillato/Tasmota_KNX/KNX_development/.github/5.jpg" />

DEVICE 2

<img src="https://raw.githubusercontent.com/ascillato/Tasmota_KNX/KNX_development/.github/6.jpg" />

### 4) Setting a Temperature sensor:

We can configure to send the value of temperature or humidity every teleperiod. This teleperiod can be configured. See TasmotaTasmota [docs](Commands.md). It is recommended also to set the reply temperature address.

<img src="https://raw.githubusercontent.com/ascillato/Tasmota_KNX/KNX_development/.github/7.jpg" />

### 5) Using rules: ###

More functionality can be added to Tasmota using rules.

* In the KNX Menu, can be set a Group Address to send data or commands by rules, as **KNX TX1** to **KNX TX5**

In rules we can use the command ``KnxTx_Cmnd1 1`` to send an ON state command to the group address set in **KNX TX1** slot of the KNX menu.
Also, we can use the command ``KnxTx_Val1 15`` to send a 15 value to the group address set in **KNX TX1** slot of the KNX menu.

* In the KNX Menu can be set a Group Address to receive commands by rules as **KNX RX1** to **KNX RX5**

In rules we can use the events to catch the reception of COMMANDS from KNX to those RX Slots.

Example: ``rule on event#knxrx_cmnd1 do var1 %value% endon`` to store the command received in the variable VAR1

In rules we can use the events to catch the reception of VALUES from KNX to those RX Slots.

Example: ``rule on event#knxrx_val1 do var1 %value% endon`` to store the value received in the variable VAR1

Also, if a Read request is received from KNX Network, we can use that in a rule as for example: ``rule on event#knxrx_req1 do knxtx_val1 %var3% endon``

NOTE: KnxTX_valn command, KNXRX_Reqn trigger and sensors' telegrams, uses KNX DPT14 (32 bits float) since 9.1.0.2 . Old versions use DPT9 (16 bits float). Old and new versions can not send values between each other. Only commands. It is recommended to have all devices on the same version.

### 6) Rule to send KNX Telegram with BH1750 Sensor Data: ###

* If you want to send your sensor values by KNX **every teleperiod time** to the Group Address defined in KNX_TX1, you can use the following rule:

```
rule1 1
rule1 on tele-BH1750#Illuminance do knxtx_val1 %value% endon
```

* If you want to send your sensor values by KNX only **when it changes in a delta of 10 lx** to the Group Address defined in KNX_TX1, you can use the following rule:

```
rule1 1
rule1 on system#boot do backlog var1 0; var2 0 endon on BH1750#Illuminance>%var1% do backlog var1 %value%; knxtx_val1 %value%; var2 %value%; add1 5; sub2 5 endon on BH1750#Illuminance<%var2% do backlog var2 %value%; knxtx_val1 %value%; var1 %value%; add1 5; sub2 5 endon
```
### 7) Rules to Monitor Heartbeat of Main Tasmota controlling several Secondary Tasmota. ###
There is no Online/Offline on KNX protocol. Rules can be used to add this new behavior. KNX communications is used to regularly check for that MAIN device to be alive. If there is no answer in some threshold time, your SECONDARY devices can turn themselves off.
With this solution the MAIN device sends its relay state as a heartbeat message (like a watchdog timer reset) to your SECONDARY devices. This message should reset an internal ruletimer of your SECONDARY devices. If that message is not received, for example 30 seconds, their relays will turn off.

On the MAIN device set a GroupAddress to send this Heartbeat like KNXTX1 3.3.3 in the KNX webmenu.
 
RULES
```
Rule1 ON system#boot DO BACKLOG KnxTx_Cmnd2 1; var1 1; ruletimer1 10 ENDON
```
At system boot (120Vac power applied) the MAIN device turns on its relay. It also sends KNX commands to turn on SECONDARY devices, it stores the value 1 in var1 and starts the ruletimer1 10 seconds countdown(heartbeat).
```
Rule1 + ON power1#State DO var1 %value% ENDON
```
Whenever MAIN power state changes (by pushbutton, Web UI, timer or MQTT) the power1#State value is stored in var1 (If MAIN loses Ac power no heartbeat is sent).
```
Rule1 + ON rules#timer=1 DO BACKLOG knxtx_val1 %var1%;ruletimer1 10 ENDON
```
When the ruletimer1 countdown reaches 1 KNX transmits var1 and restart ruletimer1 at 10 seconds. The cycle repeats.
```
RULE1 1
```
OUTPUT 1 -> 4.4.4
Output1 of MAIN is sent to 4.4.4. When received by SECONDARY, the SECONDARY matches the state with MAIN.

* In the KNX Menu, set a Group Address to send data or commands by rules, as **KNX TX2**

KNXTX2 -> 4.4.4.
At boot time KnxTx_Cmnd2 1 is sent from MAIN to 4.4.4. When received by SECONDARY, the SECONDARY turns its relay on.   

MAIN

<img src="https://github.com/DelroyTaylor/pics/blob/main/Main.JPG" />

SECONDARY

<img src="https://github.com/DelroyTaylor/pics/blob/main/Secondary.JPG" />

On each of the SECONDARY devices set a Group Address to receive this Heartbeat like KNXRX1 3.3.3 in the KNX webmenu.

```
RULE1 ON power1#boot DO ruletimer1 20 ENDON
```
When SECONDARY boots, the relay remains off until commanded by MAIN. The SECONDARY starts the ruletimer1 countdown at 20 seconds.
```
RULE1 + ON rules#timer=1 DO BACKLOG power1 0; ruletimer1 20 ENDON
```
Whenever ruletimer1 reaches 1 second it turns the relay OFF and restarts the ruletimer1 countdown at 20 seconds.
```
RULE1 + ON event#knxrx_val1 DO BACKLOG power1 %value%; ruletimer1 20 ENDON
```
If the SECONDARY receives a heartbeat transmission from MAIN it stores the value as the state for the relay. These values have following effect:
* 0 / OFF = relay(s) OFF
* 1 / ON = turn relay(s) ON
* 2 / TOGGLE = toggle relay(s)

```
RULE1 1
```

4.4.4 -> OUTPUT 1 
Output1 of MAIN is sent to 4.4.4. When received by SECONDARY, the SECONDARY matches the state with MAIN.
