mqtt_ups_sched
=============
 ## Simple MQTT UPS Scheduler for FreeBSD.
 This simple and configurable script will add basic UPS like functionality to FreeBSD server featuring `mosquitto_sub` command to query data from an existing MQTT Broker server such as [SolarAssistant](https://solar-assistant.io/), also it meant to be executed locally and can be configured to shutdown your server during battery mode or even executing a custom command for definite purposes.
 
#### Tested devices
* EASun Power ISolar MLV 3KW-U
* SRNE HF2430U60-100

 Tested with FreeBSD 13.2 with mosquitto 2.0.15, and SolarAssistant Software version 2023-08-16.

#### Supported devices may include*
* SRNE
* EASun
* PowMR

 And similar rebranded AIO inverters supporting SolarAssistant or similar.
 
Host requirements
===============
[MOSQUITTO(8)](https://man.freebsd.org/cgi/man.cgi?query=mosquitto&sektion=8&manpath=FreeBSD+13.2-RELEASE+and+Ports) - an MQTT broker

Basic Usage
-----------
```shell
Usage: mqtt_ups_sched [option]
Options:
      start       Start mqtt_ups_sched script.
      stop        Stop mqtt_ups_sched sctipt.
      stats       Display raw device states.
      stats_json  Display device states in JSON format.
      version     Display mqtt_ups_sched version.
      help        Display this help message.
```

Inverter Output States Sample(From SRNE/EASun)
-------------------------------------------------------
```shell
mqtt_ups_sched stats
[SRNE V2] 2023-12-30 15:51:15 DEVICE STATUS:
solar_assistant/inverter_1/pv_voltage/state 0.0
solar_assistant/inverter_1/grid_frequency/state 60.04
solar_assistant/inverter_1/pv_power/state 0
solar_assistant/inverter_1/battery_voltage/state 27.6
solar_assistant/inverter_1/load_apparent_power/state 441
solar_assistant/inverter_1/temperature/state 50.5
solar_assistant/inverter_1/load_percentage/state 14
solar_assistant/inverter_1/battery_current/state 0.2
solar_assistant/inverter_1/grid_power/state 453
solar_assistant/inverter_1/device_mode/state Grid
solar_assistant/inverter_1/grid_voltage/state 119.1
solar_assistant/inverter_1/ac_output_frequency/state 60.04
solar_assistant/inverter_1/pv_current/state 0.0
solar_assistant/inverter_1/ac_output_voltage/state 119.5
solar_assistant/inverter_1/load_power/state 370
solar_assistant/total/battery_power/state 6
solar_assistant/total/battery_state_of_charge/state 100
solar_assistant/total/bus_voltage/state 333.5
```

Configuration Sample #1:
=================
```shell
# Run-time configuration file for mqtt_ups_sched.
# Auto-generated file from mqtt_ups_sched.

# Set the broker local IP address.
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
BROKER_TOPIC_STATS="# -R -C 18"

# Provide a device name here for logging and reference.
DEVICE_NAME="SRNE V2"

# Set the minimum battery voltage to be considered as "system battery low" mode.
DEVICE_BATT_LOW="23"

# Set the battery last changed date in "MMDDYYYY" for logging and reference.
DEVICE_BATT_CHANGE="10252023"

# Set the battery max age in years before replacement for logging and reference.
DEVICE_BATT_MAXAGE="5"

# Set the desired command to be executed when shutdown delay time is reached, eg: "shutdown -p now".
SYS_SHUTDOWN_CMD="/sbin/shutdown -p now"

# Set the desired shutdown delay time in seconds.
SYS_SHUTDOWN_DELAY="360"

# Set "YES" here if the system must shutdown immediately if system battery low mode/threshold is reached.
SYS_LOWBATT_FASTSHUTDOWN="YES"

# Set "YES" here to enable system beeps during some state events.
SYS_BEEP_ENABLE="YES"

# Set the system max retry count in seconds during commlost/errors before the script halts.
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
DEVICE_NAME="[SRNE_INV_1]"

~~~~~~~~~~~
```
#### The above configuration will yield the following select "state" output:
```shell
mqtt_ups_sched stats
[SRNE_INV_1] 2023-12-30 16:55:10 DEVICE STATUS:
solar_assistant/inverter_1/battery_voltage/state 27.6
solar_assistant/inverter_1/grid_voltage/state 118.3
solar_assistant/inverter_1/load_power/state 360
```

### Sample log output file:
```shell
cat /var/log/mqtt_ups_sched.log
[SRNE V2] 2023-12-30 10:16:18: mqtt_ups_sched script started successfully!
[SRNE V2] 2023-12-30 10:16:18: System running on battery power!
[SRNE V2] 2023-12-30 10:20:56: System running on utility power!
[SRNE V2] 2023-12-30 15:01:20: mqtt_ups_sched script stopped successfully!
[SRNE V2] 2023-12-30 15:04:33: mqtt_ups_sched script started successfully!
```

#### Example on how to quick find all "topics/states" of your AIO inverter using `mosquitto_sub`:
```shell
mosquitto_sub -h "192.168.1.120" -R -v -t "#"
solar_assistant/inverter_1/pv_voltage/state 0.0
solar_assistant/inverter_1/grid_frequency/state 59.95
solar_assistant/inverter_1/pv_power/state 0
solar_assistant/inverter_1/battery_voltage/state 27.6
solar_assistant/inverter_1/load_apparent_power/state 450
solar_assistant/inverter_1/temperature/state 49.7
solar_assistant/inverter_1/load_percentage/state 15
solar_assistant/inverter_1/battery_current/state 0.2
solar_assistant/inverter_1/grid_power/state 439
solar_assistant/inverter_1/device_mode/state Grid
solar_assistant/inverter_1/grid_voltage/state 118.7
solar_assistant/inverter_1/ac_output_frequency/state 59.95
solar_assistant/inverter_1/pv_current/state 0.0
solar_assistant/inverter_1/ac_output_voltage/state 119.2
solar_assistant/inverter_1/load_power/state 377
solar_assistant/total/battery_power/state 6
solar_assistant/total/battery_state_of_charge/state 100
solar_assistant/total/bus_voltage/state 333.5
```
##### In the above example, the SRNE inverter shows a total of 18 "topics/states", so to display all states with the `mqtt_ups_sched` command, the  below configuration can be used:
```shell
BROKER_TOPIC_STATS="# -R -C 18"
```
#####  Where the `#` wildcard is used to sub to all "topics/states", the `-R` will avoid printing stale messages and the `-C 18` will tell to exit after receiving the 'msg_count' messages/states.

## Installation
 Just download the script and place it under `/usr/local/sbin` set proper permissions and execute it, then you can edit the configuration file after.
 Note that for `mosquitto_sub` authentication, the user can edit the `mosquitto_sub` configuration file to do so, then set file permissions accordingly.

