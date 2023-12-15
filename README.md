# DEPRECATED: With a new firmware the local RPC call is only working with a certificate and a password and this script does not work anymore. 

# PowerOpti Local for Home Assistant

This repo contains a reverse engineered configuration for PowerOpti readout in a Home Assistant local setup. No need to use the limited API to readout the data when it is in your local network anyway. When the PowerOpti is setup by the mobile app, the PowerOpti opens a wifi to be joined and the traffic that is being made from the mobile app to the meter device is not encrypted. The mobile app is capable of reading out the meter directly. Its simple RPC call with no barrier.

Here is the example configuration yaml with two meter devices in the local network that produces valid sensor entities out of the rest call that can be used in a Home Assistant dashboard. Please adjust IP and sensorname accordingly.

[HomeAssistant](http://home-assistant.io)

[PowerOpti](https://www.powerfox.energy)

```yaml

# Loads default set of integrations. Do not remove.
default_config:

# Text to speech
tts:
  - platform: google_translate

automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml

rest:
# ------------------------------------------------------------------------------
# Sensor at 192.168.178.120
# Source : Household
# Name: poweropti_consumer

# Sensor at 192.168.178.121
# Source: Heating
# Name: poweropti_heating
# ------------------------------------------------------------------------------
  - resource: http://192.168.178.120/rpc
    method: POST
    scan_interval: 1
    headers:
      Content-Type: application/json
    payload: >
      {
        "id": "1",
        "jsonrpc": "2.0",
        "method": "getConfig",
        "params": {
          "key": "latest_data"
        }
      }
    
    sensor:
      - name: poweropti_consumer      
        json_attributes:
          - id
          - jsonrpc
          - result
        value_template: >
          {% set json = value_json.result | base64_decode %}
          {% set epochtime = (json | from_json())[0].t %}
          {% set meter_id = (json | from_json())[0].m %}
          {% set A_Plus = (json | from_json())[0].d[0].v %}
          {% set A_Plus_HT = (json | from_json())[0].d[1].v %}
          {% set A_Plus_NT = (json | from_json())[0].d[2].v %}
          {% set A_Minus = (json | from_json())[0].d[3].v %}
          {% set Watt = (json | from_json())[1].d[0].v %}
          {{ '{"Watt":' + (Watt | string)  + ',"Timestamp":' + (epochtime | string)  + ',"A_Plus":' + (((A_Plus | float) / 1000) | string) + ',"A_Minus":' + (((A_Minus | float) / 1000) | string) + '}' }}

  - resource: http://192.168.178.121/rpc
    method: POST
    scan_interval: 1
    headers:
      Content-Type: application/json
    payload: >
      {
        "id": "1",
        "jsonrpc": "2.0",
        "method": "getConfig",
        "params": {
          "key": "latest_data"
        }
      }
    
    sensor:
      - name: poweropti_heating
        json_attributes:
          - id
          - jsonrpc
          - result
        value_template: >
          {% set json = value_json.result | base64_decode %}
          {% set epochtime = (json | from_json())[0].t %}
          {% set meter_id = (json | from_json())[0].m %}
          {% set A_Plus = (json | from_json())[0].d[0].v %}
          {% set A_Plus_HT = (json | from_json())[0].d[1].v %}
          {% set A_Plus_NT = (json | from_json())[0].d[2].v %}
          {% set A_Minus = (json | from_json())[0].d[3].v %}
          {% set Watt = (json | from_json())[1].d[0].v %}
          {{ '{"Watt":' + (Watt | string)  + ',"Timestamp":' + (epochtime | string)  + ',"A_Plus":' + (((A_Plus | float) / 1000) | string) + ',"A_Minus":' + (((A_Minus | float) / 1000) | string) + '}' }}


template:
  - sensor:
      - name: "Verbraucher Strom aktuell"
        unit_of_measurement: "W"
        unique_id: "Verbraucher ID - 1"
        device_class: "power"
        state_class: "measurement"
        state: >
            {{ (states('sensor.poweropti_consumer')|from_json).Watt }}
      - name: "Verbraucher Strom Bezug"
        unit_of_measurement: "kWh"
        unique_id: "Verbraucher ID - 2"
        device_class: "energy"
        state_class: "total_increasing"
        state: >
            {{ (states('sensor.poweropti_consumer')|from_json).A_Plus }}
      - name: "Verbraucher Strom Einspeisung"
        unit_of_measurement: "kWh"
        unique_id: "Verbraucher ID - 3"
        device_class: "energy"
        state_class: "total_increasing"
        state: >
            {{ (states('sensor.poweropti_consumer')|from_json).A_Minus }}
      - name: "Heizung Strom aktuell"
        unit_of_measurement: "W"
        unique_id: "Heizung ID - 1"
        device_class: "power"
        state_class: "measurement"
        state: >
            {{ (states('sensor.poweropti_heating')|from_json).Watt }}
      - name: "Heizung Strom Bezug"
        unit_of_measurement: "kWh"
        unique_id: "Heizung ID - 2"
        device_class: "energy"
        state_class: "total_increasing"
        state: >
            {{ (states('sensor.poweropti_heating')|from_json).A_Plus }}

```
