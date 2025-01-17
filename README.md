
# MQTTLED

This package allows control of an OpenWRT device's LEDs over MQTT. Configured using UCI or YAML. No changes are made to the existing triggers. Uninstalling will revert to your normal LED triggers.

## Config

Normal configuration is done through UCI. The configuration file is:

```
/etc/config/mqttled
```

The default configuration looks like this:

```
config mqtt 'mqtt'
    #MQTT Broker Address
    option host '192.168.1.1'
    option port '1883'
    #May work without this, should bind to all interfaces. Needs to be an interface matching in /etc/config/network
    option interface 'lan'
    option username ''
    option password ''
    #Not tested
    option tls '0'
    option cert ''
    option discovery 'homeassistant'
    #Set your topic here: e.g. OpenWRTleds/CustomDevice/led1
    option basetopic 'OpenWRTleds'
    option subtopic 'CustomDevice'
    #Reported in the 'device' page in Home Assistant
    option model 'OpenWRT Device'

config leds 'leds'
    #Setting this to 1 will ignore any includes
    option includeall '1'
    #excluded LEDs will allways be supressed
    list exclude 'mt76-phy0'
    list exclude 'rt2800soc-phy1::assoc'
    list exclude 'rt2800soc-phy1::quality'
    list exclude 'rt2800soc-phy1::radio'
    #list include 'blue:internet'

config trigger 'triggers'
    #Only triggers listed here will be presented to HA as 'effects'
    list triggers 'none'
    list triggers 'default-on'
    list triggers 'heartbeat'
    list triggers 'timer'
```

Changes should be made either with UCI e.g.

```
uci set mqttled.mqtt.host='192.168.1.254'
uci commit
```

or by editing the config file directly.
You will then need to restart the service with

```
service mqttled restart
```

## Info

### Topics

This service will publish a retained message to the `discoverytopic/light` on start. This will point Homeassistant to the state and availability topics of your device set under `basetopic/subtopic`. The current state is reported on `basetopic/subtopic/ledname/state`. The daemon is subscribed to `basetopic/subtopic/ledname/set` for commands.

### Attributes

The mqttled service will expose the LEDs as dimmable using the value set under `/sys/class/leds/led#/brightness`.

### Local Changes

Changes made to LED state on the device are __not__ polled. *i.e.* any changes you make to the LED states by any other means will not be reflected unless the service is restarted.
