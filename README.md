# SLWZ-DIM-01_dimmer_esphome_template
ESPHome device package for Spex WiZ SLWZ-DIM-01 Wall Dimmer devices.
![Untitled](https://github.com/user-attachments/assets/076068ad-8de5-4176-ac34-f9872d5ede69)


These devices use a WiZ-branded ESP8266 module and thus can be reflashed with a custom firmware.
It is essentially a ESP-12S module but with only 1MB of flash space. Please refer to the datasheet PDF included in this repo for pinout information.
![PXL_20241126_163854048](https://github.com/user-attachments/assets/9eee0577-07c9-47f0-88dd-6be83182b9b2)


There is currently no known software exploit. Dissassembly of the device and soldering is required. Intermediate soldering skills required.


These units work in tandem with essentially what is a TuyaMCU, even though the stock WiZ firmware connects to its own backend.
WiZ devices are not Tuya devices but the protocol between the WiZ module and secondary MCU is identical to TuyaMCU.

All features exposed by the TuyaMCU are mapped in this configuration.

Extra feature added:

- Added a number input slider to set Dimmer's brightness without actually turning the light ON

# Initial device programming
Dissassembly is somewhat easy. There are 4 retention clips keeping the back box and faceplate togheter. 2 clips on each sides of the device. By gently prying apart the back box and the faceplate, you can insert a spudger in the seem and work the whole side to unclip it. The plastic is quite strong, you can apply a moderate amount of elbow grease.
![PXL_20241129_034032369](https://github.com/user-attachments/assets/3163991e-1a1f-4a32-ab79-03ea3e11cbd0)

Be cautious though, there are 2 cables connecting the base board to the control board. A 2-pin JST cable and a small flat cable.
You will need to disconnect both cables.

At this point, if you have a steady hand, you can proceed with soldering work without further dissassembly.

If you want to go the extra mile to avoid melting some plastics, you can unscrew the switch board and the control board from the front switch assembly.

As the TuyaMCU is also on the control board, you will need to desolder a small resistor to keep it from powering ON during flashing procedure.
It's the small 0 ohm resistor next to the flat cable that connects to the baseboard
![resistor](https://github.com/user-attachments/assets/a2ab25c0-e021-46cb-9bf0-0cef3cc51b9f)

With this resistor temporarily removed, you will successfully interface the ESP8266 bootloader via Uart1.
After flashing is complete, you can solder back this resistor or simply bridge the 2 pads together.

## Serial Flashing

Flashing via serial adapter is currently the only known method to flash a custom firmware onto WiZ devices.

![PXL_20241126_163854048](https://github.com/user-attachments/assets/b5dd633d-bf02-4f72-b734-abf77efc90ef)


If you wish to make a backup of your stock firmware, be aware that stored Wifi credentials are stored in clear into the flash area of the module.
Do not distribute your stock firmware binaries without sanitizing them prior if you ever used the device on your network with the stock firmware.

Use your favorite method to flash your ESPHome binary onto your device.

Once flashed, confirm your device boots by unstrapping GPIO0 from GND.
Once confirmed, you can proceed to solder back the previously removed resistor.


# How to use
Include this package and populate the necessary variables.
Syslog requires a time component. Use your own and define the substitution variable `syslog_time_esphome_id` to reference the instance ID.

Example of a device YAML file:
```yaml
substitutions:
  device_name: "my-wiz-dimmer"
  device_friendly_name: "Dimmer"
  api_key: "supersecretapikey"
  ota_password: "otapsswd"
  hotspot_name: "WiZ-dimmer-AP"
  hotspot_password: !secret fallback_hotspot_password
  default_log_level: "INFO"
  toggle_log_level_timeout_hours: "24"
  syslog_server_ip: "192.168.0.1"
  syslog_server_port: "514"
  syslog_log_level: "INFO"
  syslog_time_esphome_id: "id_time_esphome_component"

packages:
  remote_package_files:
    url: https://github.com/bennydiamond/SLWZ-DIM-01_dimmer_esphome_template
    files: [.base.SLWZ-DIM-01_template.yaml]  # optional; if not specified, all files will be included
    ref: master  # optional
    refresh: 1d  # optional
	
time:
  - platform: sntp
    id: ${syslog_time_esphome_id}
    timezone: America/Toronto
    servers:
     - 0.pool.ntp.org
     - 1.pool.ntp.org
     - 2.pool.ntp.org
```

## Customization

Refer to ESPHome's documentation on [packages](https://esphome.io/components/packages) to customize your device.

### Example of customization. 
To remove the Fallback Wifi Hotspot component from your device, you would just need to add the following at the bottom of your device's yaml file.

```yaml
wifi:
  ap: !remove
```

To set static IP.
```yaml
wifi:
  manual_ip:
    static_ip: 192.168.1.10
    gateway: 192.168.1.1
    subnet: 255.255.255.0
```

# Optional hardware mod

You can wire the main switch button signal that goes to the TuyaMCU directly on a GPIO of the WiZ module.
As the WiZ dimmer does not react to a long press of the button (the lights won't even toggle), we can intercept this signal and use it in event conditions, such as scenes.
Soldering a wire between a solder pad and one of the input pin of the WiZ module is the recommended way.
As illustrated below, switch signal is wired to GPIO13 on the WiZ module.
![PXL_20241211_162857101](https://github.com/user-attachments/assets/c23c222d-439f-4919-806e-ee7ef45104f2)
![PXL_20241211_162848897](https://github.com/user-attachments/assets/3341f3d2-e4a3-470d-9cd4-0af806e6267e)

The following sample code can then be added to your top level yaml file. It will merge with the template of this GitHub repo when compiling

```yaml
binary_sensor:
  - platform: gpio
    pin: 
      number: GPIO13
      mode:
        input: True
        pullup: False
      inverted: True
    name: "Switch"
    internal: True
    on_multi_click: 
      - timing: 
          - ON for at least 1500ms
        then:
          - event.trigger:
              id: button1
              event_type: longpress

    on_double_click: 
      then:
        - event.trigger:
            id: button1
            event_type: doubleclick


event:
  - platform: template
    id: button1
    device_class: button
    name: Button events
    event_types:
      - doubleclick
      - longpress
```

This will expose a "Button events" entity that will record events named "longpress" and "doubleclick"
"doubleclick" events are not recommended to be used as it will toggle twice the lights on that dimmer since the TuyaMCU will interpret each individual switch presses.
