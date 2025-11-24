mqtt_ups_sched
=============
 ## Simple MQTT UPS Scheduler for FreeBSD.
 This simple and configurable script will add basic UPS like functionality to FreeBSD server featuring `mosquitto_sub` command to query data from an existing MQTT Broker server such as [Solar-Assistant](https://solar-assistant.io/), [Home-Assistant](https://www.home-assistant.io/) Etc, it meant to be executed locally and can be configured to shutdown your server during battery mode or executing a custom command for definite purposes.

#### Tested devices
* EASun Power ISolar MLV 3KW-U
* SRNE HF2430U60-100

 Tested with FreeBSD 13.2 with mosquitto 2.0.15, and SolarAssistant Software version 2023-08-16.

#### Supported devices may include*
* SRNE
* EASun
* PowMR
* EG4
* Growatt
* Etc.

 And similar rebranded AIO inverters, technically should work with any inverter supported by Solar-Assistant, Home-Assistant or similar/custom MQTT broker server.

Host requirements
===============
[MOSQUITTO(8)](https://man.freebsd.org/cgi/man.cgi?query=mosquitto&sektion=8&manpath=FreeBSD+13.2-RELEASE+and+Ports) - an MQTT broker

Basic Usage
-----------
```shell
Usage: mqtt_ups_sched [option] | [topic]
Options:
      start       Start mqtt_ups_sched script.
      stop        Stop mqtt_ups_sched sctipt.
      stat        Display raw device states.
      jstat       Display device states in JSON format.
      log         Display device logs.
      version     Display mqtt_ups_sched version.
      help        Display this help message.
```

Inverter Output States Sample(From SRNE/EASun)
-------------------------------------------------------
```shell
mqtt_ups_sched stat
[iSolar-MLV-3KW-U] Status:
solar_assistant/battery_1/state_of_charge/state 96
solar_assistant/battery_1/cell_voltage_-_highest/state 3.333
solar_assistant/battery_1/temperature/state 28.1
solar_assistant/battery_1/current/state -0.4
solar_assistant/battery_1/power/state -11
solar_assistant/battery_1/cell_voltage_-_average/state 3.331
solar_assistant/battery_1/temperature_1/state 28.1
solar_assistant/battery_1/temperature_2/state 28.1
solar_assistant/battery_1/cell_voltage_-_lowest/state 3.317
solar_assistant/battery_1/voltage/state 26.7
solar_assistant/battery_1/capacity/state 100
solar_assistant/inverter_1/pv_voltage/state 0.0
solar_assistant/inverter_1/grid_frequency/state 59.99
solar_assistant/inverter_1/pv_power/state 0
solar_assistant/inverter_1/battery_voltage/state 26.6
solar_assistant/inverter_1/load_apparent_power/state 307
solar_assistant/inverter_1/temperature/state 34.0
solar_assistant/inverter_1/load_percentage/state 10
solar_assistant/inverter_1/battery_current/state 0.0
solar_assistant/inverter_1/grid_power/state 308
solar_assistant/inverter_1/device_mode/state Grid
solar_assistant/inverter_1/grid_voltage/state 118.4
solar_assistant/inverter_1/ac_output_frequency/state 59.99
solar_assistant/inverter_1/pv_current/state 0.0
solar_assistant/inverter_1/ac_output_voltage/state 119.1
solar_assistant/inverter_1/load_power/state 247
solar_assistant/total/battery_power/state -11
solar_assistant/total/battery_state_of_charge/state 96
solar_assistant/total/battery_temperature/state 28.1
solar_assistant/total/bus_voltage/state 322.5
```
```shell
mqtt_ups_sched stat "solar_assistant/inverter_1/serial_number/state"
[iSolar-MLV-3KW-U] Status:
solar_assistant/inverter_1/serial_number/state SR-XXXXXXXXXX-XXXXXX
```

Configuration Sample #1:
=================
```shell
# Run-time configuration file for mqtt_ups_sched.
# Auto-generated file from mqtt_ups_sched.

# Set the MQTT broker server local IP address.
BROKER_IP="192.168.1.141"

# Set the broker port, default is 1883 and must be allowed by firewall.
BROKER_PORT="1883"

# Set the broker topic of the battery voltage state, eg: "solar_assistant/inverter_1/battery_voltage/state".
BROKER_TOPIC_BATTVOLT="solar_assistant/inverter_1/battery_voltage/state"

# Set the broker topic of the device mode state, eg: "solar_assistant/inverter_1/device_mode/state".
BROKER_TOPIC_DEVSTATE="solar_assistant/inverter_1/device_mode/state"

# Set the broker state output only when operating in grid mode, eg: "Grid"
BROKER_TOPIC_GRIDMODE="Grid"

# Set the broker state output only when operating in battery mode, eg: "Solar/Battery".
BROKER_TOPIC_BATTMODE="Solar/Battery"

# Select the desired topic/states to be displayed separated by space, eg: "sys/inv/grid/state sys/inv/batt/state sys/inv/etc/state".
# Optionally you can display all device states with the parameter: "# -R -C XX", where "XX" is the max topic/state items of the device.
BROKER_TOPIC_STATS="# -R -C 30"

# Provide a device name here for logging and reference.
DEVICE_NAME="SRNE_INVERTER"

# Set the minimum battery voltage to be considered as "system battery low" mode, please use an integer or float value here.
DEVICE_BATT_LOW="23"

# Set the battery inatallation date in "YYYYMMDD" for logging and reference.
DEVICE_BATT_INSTALL="20231025"

# Set the battery max age in "Y" years before replacement for logging and reference.
DEVICE_BATT_MAXAGE="5"

# Set the desired command to be executed when shutdown delay time is reached, eg: "shutdown -p now".
SYS_SHUTDOWN_CMD="/sbin/shutdown -p now"

# Set the desired shutdown delay time in seconds, please use an integer value here, set to 0 to disable.
# Time here may be relative due to the time it takes to query the topics and may vary per system.
SYS_SHUTDOWN_DELAY="300"

# Set "YES" here if the system must shutdown immediately if system battery low mode/threshold is reached.
# Note that this will be executed regardless of the "SYS_SHUTDOWN_DELAY" parameter configuration if enabled.
SYS_LOWBATT_FASTSHUTDOWN="YES"

# Set "YES" here to enable system beeps during some state events.
SYS_BEEP_ENABLE="NO"

# Set the system max retry count in seconds during commlost/errors before the script halts, please use an integer value here.
SYS_RETRY_COUNT="30"
```
#### The above configuration will yield all the device states.

Configuration Sample #2:
===================
```shell
# Run-time configuration file for mqtt_ups_sched.
# Auto-generated file from mqtt_ups_sched.

~~~~~~~~~~~

# Select the desired topic/states to be displayed separated by space, eg: "sys/inv/grid/state sys/inv/batt/state sys/inv/etc/state".
# Optionally you can display all device states with the parameter: "# -R -C XX", where "XX" is the max topic/states count of the device.
BROKER_TOPIC_STATS="solar_assistant/inverter_1/battery_voltage/state solar_assistant/inverter_1/grid_voltage/state solar_assistant/inverter_1/load_power/state"

# Provide a device name here for logging and reference.
DEVICE_NAME="[SRNE_INVERTER]"

~~~~~~~~~~~
```
#### The above configuration will yield the following select "state" output:
```shell
mqtt_ups_sched stat
[SRNE_INVERTER] Status:
solar_assistant/inverter_1/battery_voltage/state 27.6
solar_assistant/inverter_1/grid_voltage/state 118.3
solar_assistant/inverter_1/load_power/state 360
```

### Sample log output file:
```shell
mqtt_ups_sched log
Log file: /var/log/mqtt_ups_sched.log
2023-12-30 10:16:18: mqtt_ups_sched script started successfully!
2023-12-30 10:16:18: SRNE_INVERTER System running on battery power!
2023-12-30 10:20:56: SRNE_INVERTER System running on utility power!
2023-12-30 15:01:20: mqtt_ups_sched script stopped successfully!
2023-12-30 15:04:33: mqtt_ups_sched script started successfully!
```

#### Example on how to quick find all "topics/states" of your AIO inverter using `mosquitto_sub`:
```shell
mosquitto_sub -h "192.168.1.120" -R -v -t "#"
solar_assistant/inverter_1/pv_voltage/state 0.0
solar_assistant/inverter_1/grid_frequency/state 59.95
solar_assistant/inverter_1/pv_power/state 0
solar_assistant/inverter_1/battery_voltage/state 26.6
solar_assistant/inverter_1/load_apparent_power/state 293
solar_assistant/inverter_1/temperature/state 34.0
solar_assistant/inverter_1/load_percentage/state 9
solar_assistant/inverter_1/battery_current/state 0.0
solar_assistant/inverter_1/grid_power/state 294
solar_assistant/inverter_1/device_mode/state Grid
solar_assistant/inverter_1/grid_voltage/state 122.4
solar_assistant/inverter_1/ac_output_frequency/state 59.95
solar_assistant/inverter_1/pv_current/state 0.0
solar_assistant/inverter_1/ac_output_voltage/state 122.8
solar_assistant/inverter_1/load_power/state 232
solar_assistant/battery_1/state_of_charge/state 95
solar_assistant/battery_1/cell_voltage_-_highest/state 3.332
solar_assistant/battery_1/temperature/state 28.1
solar_assistant/battery_1/current/state -0.4
solar_assistant/battery_1/power/state -11
solar_assistant/battery_1/cell_voltage_-_average/state 3.33
solar_assistant/battery_1/temperature_1/state 28.1
solar_assistant/battery_1/temperature_2/state 28.1
solar_assistant/battery_1/cell_voltage_-_lowest/state 3.316
solar_assistant/battery_1/voltage/state 26.7
solar_assistant/battery_1/capacity/state 100
solar_assistant/total/battery_power/state -11
solar_assistant/total/battery_state_of_charge/state 95
solar_assistant/total/battery_temperature/state 28.1
solar_assistant/total/bus_voltage/state 322.5
```
##### In the above example, the SRNE inverter shows a total of 30 "topics/states", so to display all states with the `mqtt_ups_sched` command, the  below configuration can be used:
```shell
BROKER_TOPIC_STATS="# -R -C 30"
```
#####  Where the `#` wildcard is used to sub to all "topics/states", the `-R` will avoid printing stale messages and the `-C 30` will tell to exit after receiving the total topic/states count.

## Installation
 Just download the script and place it under `/usr/local/sbin` set proper permissions and execute it, then you can edit the configuration file after.
 Note that for `mosquitto_sub` authentication, the user can edit the `mosquitto_sub` configuration file to do so, then set file permissions accordingly.

