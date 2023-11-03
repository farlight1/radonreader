# RadonReader 2022 RD200 v1/v2

This project provides a tool which allows users collect current radon data from FTLab Radon Eye RD200 v1 and V2 (2022+) (Bluetooth only versions). 

Farlight v0.5 - 03/11/2023
- Forked from [EtoTen repo][EtoTen_repo]
- New known serial device numbers added
- Command line argument to select HCI interface when using more than one Bluetooth dongle
- Added readings for day/month averages and pulse count (now - for current C pulses and last - for last 10 min C pulses)
- Added new MQTT Json attribute readings

EtoTen v0.4 - 07/05/2022
- Forked Project from [ceandre repo][ceandre_repo]
- Changed compatability to Python3 
- Added support for new RD200 models made in 2022
- Added auto-scan ability 
- Change the read function to call the handler directly, instead of interacting with the UUIDs

Note: if specifying an (-a) MAC address, you now also have to specify a device type (-t) (either 0 for original RD200 or 1 for RD200 v2)

# Pre-req install steps:

<pre><code>sudo apt install libglib2.0-dev
pip3 install bluepy
pip3 install paho-mqtt

#Raspberry pi users must setcap
sudo setcap cap_net_raw+e /home/pi/.local/lib/python3.7/site-packages/bluepy/bluepy-helper
sudo setcap cap_net_admin+eip /home/pi/.local/lib/python3.7/site-packages/bluepy/bluepy-helper
</pre></code>

# MQTT

- Install mosquitto MQTT (apt, Docker, add-on in HA... and configure it with a local user and password
- On the host machine run: <pre><code>python3 radon_reader.py -v -ms localhost -mu radonuser -mw radon123  -ma -m</pre></code> and listen to topic:
 "environment/RADONEYE/#" and make sure the data appears (windows users can use [MQTT explorer][mqtt_explorer]
- An example of MQTT tree:
![MQTT tree][mqtt_tree]

# Home assistant integration via MQTT:

- Install mosquitto MQTT integration in HA
- On the host machine run: <pre><code>python3 radon_reader.py -v -ms localhost -mu radonuser -mw radon123  -ma -m</pre></code>  and listen to:
 "environment/RADONEYE/#" in the MQTT integration in HA to verify messages are being sent
- Add this to configuration.yaml and replace "your_device_MAC_id" with yours:
<pre><code>
mqtt:
  sensor:
    - name: "Radon Level"
      state_topic: "enviroment/RadonEye/"your_device_mac_id"/radon_value"
      unit_of_measurement: "Bq/m^3"
      value_template: "{{ value_json.value }}"
      json_attributes_topic: "enviroment/RadonEye/"your_device_MAC_id"/attributes"
      json_attributes_template: >
        { "unit": "{{value_json.unit}}",
          "day_average": "{{value_json.day_average}}",
          "month_average": "{{value_json.month_average}}",
          "pulse_count_now": {{value_json.pulse_count_now}},
          "pulse_count_last": {{value_json.pulse_count_last}} }
</pre></code>

- Now make the app send automatic updates to HA every 6 minutes
 <pre><code>crontab -e</pre></code> 
 <pre><code>*/6 * * * * /usr/bin/python3 /home/pi/radonreader/radon_reader.py -v -a 94:3c:c6:dd:42:ce -t 1 -ms localhost -mu radonuser -mw radon123  -mw radon123  -ma -m #update radon reading via MQTT every 3 minutes</pre></code> 

- Now you can add a sensor card to your HA view, this is an example using lovelace-multiple-entity-row in HACS
![HA Card][ha_card]

There is a HACS integration for HA that can be used to retrieve data from the RD200, have a look at the [jdeath/rd200v2][jdeath_integration] integration.

# Hardware Requirements
- FTLabs RadonEye RD200 v1 or v2
- Host machine w/Bluetooth LE (Low Energy) support (RPi 3B/4/etc...)

# Software Requirements
- Python 3.7
- bluepy Python library
- paho-mqtt Python library

# History
- 0.5 - Forked from [EtoTen][EtoTen_repo] and upgraded
- 0.4 - Forked and modified extensively 
- 0.3 - Added MQTT support

# Usage
<pre><code>usage: radon_reader.py [-h] [-a ADDRESS] [-t TYPE] [-i IFACE] [-b] [-v] [-s] [-m] [-ms MQTT_SRV] [-mp MQTT_PORT] [-mu MQTT_USER] [-mw MQTT_PW] [-ma]

RadonEye RD200 (Bluetooth/BLE) Reader

options:
  -h, --help       show this help message and exit
  -a ADDRESS       Bluetooth Address (AA:BB:CC:DD:EE:FF format)
  -t TYPE          type 0 for rd200 <2022 or 1 for =>2022 models
  -i IFACE         if several bluetooth devices are present, type 0, 1... for hci0, hci1... respectively
  -b, --becquerel  Display radon value in Becquerel (Bq/m^3) unit
  -v, --verbose    Verbose mode
  -s, --silent     Output only radon value (without unit and timestamp)
  -m, --mqtt       Also send radon value to a MQTT server
  -ms MQTT_SRV     MQTT server URL or IP address
  -mp MQTT_PORT    MQTT server service port (Default: 1883)
  -mu MQTT_USER    MQTT server username
  -mw MQTT_PW      MQTT server password
  -ma              Switch to Home Assistant MQTT output (Default: EmonCMS)
</code></pre>

# Example usage:
<pre><code>python3 radon_reader.py -a 94:3c:c6:dd:42:ce -t 1 -v #verbose output/ specific device MAC
python3 radon_reader.py -v #verbose output, auto scan
python3 radon_reader.py -v -a 94:3c:c6:dd:42:ce -t 1 -i 1 -ms localhost -mu radonuser -mw radon123 -ma -m #verbose output, specific device MAC, use hci1 device, mqtt to home assistant
</pre></code>

[//]: # (LINKS)
[mqtt_tree]: https://github.com/farlight1/radonreader/blob/master/assets/MQTT_json_Topic.jpg
[ha_card]: https://github.com/farlight1/radonreader/blob/master/assets/Tarjeta_radon.jpg
[license]: LICENSE
[ceandre_repo]: https://github.com/ceandre/radonreader
[EtoTen_repo]: https://github.com/EtoTen/radonreader
[mqtt_explorer]: https://mqtt-explorer.com/
[jdeath_integration]: https://github.com/jdeath/rd200v2
