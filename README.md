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

Example of a device YAML file:
```
substitutions:
  device_name: "my-wiz-dimmer"
  device_friendly_name: "Dimmer"
  api_key: "supersecretapikey"
  ota_password: "otapsswd"
  hotspot_name: "WiZ-dimmer-AP"
  hotspot_password: !secret fallback_hotspot_password
  log_level: "INFO"

packages:
  remote_package_files:
    url: https://github.com/bennydiamond/SLWZ-DIM-01_dimmer_esphome_template
    files: [.base.SLWZ-DIM-01_template.yaml]  # optional; if not specified, all files will be included
    ref: master  # optional
    refresh: 1d  # optional
```

## Customization

Refer to ESPHome's documentation on [packages](https://esphome.io/components/packages) to customize your device.

### Example of customization. 
To remove the Fallback Wifi Hotspot component from your device, you would just need to add the following at the bottom of your device's yaml file.

```
wifi:
  ap: !remove
```

To set static IP.
```
wifi:
  manual_ip:
    static_ip: 192.168.1.10
    gateway: 192.168.1.1
    subnet: 255.255.255.0
```
