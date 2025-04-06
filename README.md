# ESP32 IR blaster for the Unfolded Circle Remote Two.

This project implements the [Unfolded Circle Remote Two](https://www.unfoldedcircle.com/) dock API to add additional IR blasters
to your remote setup.

This allows the installation of an IR blaster in an independant location from where your
remote and charging dock are located. For example, in an equipment rack or cupboard, or for
multi-room setups.

⚠️ Since Unfolded Circle has released their dock firmware I have not yet updated the software in this repo for Hardware v0.3. Use koeblaster brach in [this repo](https://github.com/itcorner/ucd2-firmware/tree/koeblaster) in the meanwhile.


## Requirements

1. An [Unfolded Circle Remote Two](https://www.unfoldedcircle.com/).
1. An supported ESP32 hardware (see Section [Supported Electronics](#supported-electronics))
1. [PlatformIO](https://platformio.org/) to build and upload firmware
1. Depending on used hardware: Some basic soldering / breadboarding skills and basic understanding of electronic.
1. Depending on used hardware: Some basic electronic parts and IR led.


## Getting Started

1. Clone project from Github and add to PlatformIO.
2. Select the build environment matching your dock hardware.
    - Add your specific board to ``platformio.ini``. Feel free to contribute this back as a pull-request.
3. If required, modify pin mappings in ``lib/config/blaster_config.h``, or by setting `-D defines` in the ``build_flags`` section of your build environment in ``platformio.ini``, respectively (see section [Customizing Pinout/Configuration](#customizing-pinout-configuration)).
    -  For enabling the *learning of IR codes* make sure you have added ``-DBLASTER_ENABLE_IR_LEARN=true`` to the build_flags of your used PlatformIO environment (or in blaster_config.h) and check the assigned GPIO pin.
    -  **Note:** IR learning feature is still under development and tested only with some basic IR codes.
4. Build *firmware* and *filesystem image* via PlattformIO actions for your target hardware.
5. Upload *firmware* and *filesystem image* to your ESP32 hardware (**Hint:** make sure no serial monitor is active during filesystem upload, otherwise upload will fail!).
    -  Details about SPIFFS filesystem image can be found in the respective [Section](#spiffs-filesystem-image)
6. Start discovering process on your Remote Two. The dock should be detected.
7. Perform setup process for dock via Remote Two.
8. Check serial logging for debugging.

⚠️ Do not forget to select the dock as "IR output device" in the configuration of your remotes!


## Customizing Pinout/Configuration 

The following defines are available in ``lib/config/blaster_config.h`` and can be overwritten by setting ``build_flags`` in ``platformio.ini``.

| Define | Description | Options |
|:-------|:------------|:--------|
|`BLASTER_INDICATOR_MODE` | Defines the type of indicator led used for custom dock | `INDICATOR_OFF` no indicator led available<br/>`INDICATOR_LED` single color led (__default__)<br/>`INDICATOR_PIXEL` digital RGB neopixel led |
|`BLASTER_PIN_INDICATOR` | Defines GPIO pin where indicator led is connected to  | Default: `5`<br/>Has no effect if `BLASTER_INDICATOR_MODE=INDICATOR_OFF` |
|`BLASTER_ENABLE_IR_INTERNAL` | Enables/Disables internal IR channel of dock | `true` channel is enabled (__default__)<br/> `false` channel is disabled |
|`BLASTER_PIN_IR_INTERNAL` | Defines GPIO pin of internal IR channel | Default: `12`<br/>Has no effect if `BLASTER_ENABLE_IR_INTERNAL=false` |
|`BLASTER_ENABLE_IR_OUT_1` | Enables/Disables external IR channel 1 of dock | `true` channel is enabled (__default__)<br/> `false` channel is disabled |
|`BLASTER_PIN_IR_OUT_1` | Defines GPIO pin of external IR channel 1 | Default: `13`<br/>Has no effect if `BLASTER_ENABLE_IR_OUT_1=false` |
|`BLASTER_ENABLE_IR_OUT_2` | Enables/Disables external IR channel 2 of dock | `true` channel is enabled (__default__)<br/> `false` channel is disabled |
|`BLASTER_PIN_IR_OUT_2` | Defines GPIO pin of external IR channel 2 | Default: `14`<br/>Has no effect if `BLASTER_ENABLE_IR_OUT_2=false` |
|`BLASTER_ENABLE_IR_LEARN` | Enables/Disables learning of IR codes by an IR receiver | `true` learning is enabled<br/> `false` learning is disabled  (__default__)|
|`BLASTER_PIN_IR_LEARN` | Defines GPIO pin IR receiver is connected to | Default: `16`<br/>Has no effect if `BLASTER_ENABLE_IR_LEARN=false` |
|`BLASTER_ENABLE_ETH` | Enables/Disables an wired Ethernet interface<br/>Currently only Olimex Boards supported. | `true` Ethernet interface available. Requires definition of `OLIMEX_ESP`<br/> `false` Ethernet interface not available  (__default__)|
|`OLIMEX_ESP` | Defines ESP32 package used in Olimex POE Board<br/>Impacts the pins for ethernet interface | `WROOM` tested and confirmed working<br/>`WROVER` untested ⚠️, therefore breaks build on purpose. Adaption in `eth_service.cpp` required!<br/>Has no effect if `BLASTER_ENABLE_ETH=false` |
|`BLASTER_ENABLE_OTA` | Enables Arduino OTA flashing.<br/>**Note**: This OTA function has nothing to do with firmware updates via Remote Two! | `true` OTA flashing is enabled <br/>`false` OTA flashing is not enabled (__default__)| 


## SPIFFS Filesystem Image

For providing details about the dock via a web user interface, files stored in an SPIFF file system are used.
The actual content of the SPIFFS filesystem image is the `data` directory of the project.

`data/spiffs_manifest.json` holds an index about all available files and their expected MD5 sums.
During boot, the dock will check availability of all files listed in the index and validate their checksum.

⚠️ Currently, the `fs_service` will only allow access to files mentioned in the index and available in the filesystem image, irrespecive of the checksum validataion. This might change in future.

**Hint:** In `tools/update_spiffs_manifest.py` a Python script is available to update the contents and MD5-checksums of the filesystem image index. Make sure the script is executed from the project's main folder.



# Supported Electronics


## KoeBlaster Hardware

*KoeBlaster* is designed to be a drop-in replacement of the Logitech Harmony Hub PCB working with Remote Two.

Schematic is tested and working. Comments are of course appreciated!

PCB is manufactured and working. 

[Manufacturing Data](kicad/KoeBlaster/exports) is available in the repository.

⚠️ When manufacturing, make sure you are using a **PCB height of 1mm**, as otherwise the mounting inside the Harmony Hub are not fitting. ⚠️

### Change History

| Revision | Changes | Known-Issues | 
|:-------|:------------|:------------|
| 0.3 | Supporting SMD or THT leds, split top and side led control, optimized BOM for JLCPCB manufacturing | none (yet) |
| 0.2 | fixes of hw problems from v0.1 | none (yet) |
| 0.1 | initial design | value of current limitation resistors for IR leds too high.<br/>short-circuits on external blaster sockets.<br/>CH340 outputs voltages higher than 3.3V, therefore problems with hardware reset circuit. |

### Schematic

![Serving suggestion](https://github.com/itcorner/ESP32-IRBlaster-UCR2/blob/main/kicad/KoeBlaster/exports/v0.3/schematic_v0.3.pdf?raw=true)

### Board

#### Top

![Serving suggestion](https://github.com/itcorner/ESP32-IRBlaster-UCR2/blob/main/kicad/KoeBlaster/exports/v0.3/KoeBlaster-F_Fab.svg?raw=true)

![Serving suggestion](https://github.com/itcorner/ESP32-IRBlaster-UCR2/blob/main/kicad/KoeBlaster/exports/v0.3/KoeBlaster-F_Cu.svg?raw=true)

#### Bottom

![Serving suggestion](https://github.com/itcorner/ESP32-IRBlaster-UCR2/blob/main/kicad/KoeBlaster/exports/v0.3/KoeBlaster-B_Fab.svg?raw=true)

![Serving suggestion](https://github.com/itcorner/ESP32-IRBlaster-UCR2/blob/main/kicad/KoeBlaster/exports/v0.3/KoeBlaster-B_Cu.svg?raw=true)



## Olimex ESP32-PoE-ISO + MOD-IRDA

Olimex offers some nice modules for people wanting Plug&Play solutions. 
Using the PlattformIO Environment `env:olimex_poe_iso` the pins shall already be pre-configured correctly for use with the MOD-IRDA extension board.

⚠️ From Olimex two IRDA add-ons are available (MOD-IRDA and MOD-IRDA+). Only MOD-IRDA is supported by this project! Due to different interfacing MOD-IRDA+ add-on is not compatible! ⚠️


### Customizing the pinout 

If you need to change the pinout feel free to do so. Here are the pins configured for the Olimex Board per default:

| Setting | Description |
|:-------|:------------|
|`BLASTER_INDICATOR_MODE` = `INDICATOR_OFF` | Olimex Board does not offer a led for custom usage, therfore indicator is turned off | 
|`BLASTER_ENABLE_IR_INTERNAL` = `true` | Enables internal IR channel of dock used by MOD-IRDA add-on |
|`BLASTER_PIN_IR_INTERNAL` = `4` | GPIO 4 is wired to UEXT header and used by IR sender on MOD-IRDA add-on |
|`BLASTER_ENABLE_IR_OUT_1` = `false` | Disables external IR channel 1 of dock by default. <br/>If you plan to add external circuity set to `true` and configure `BLASTER_PIN_IR_OUT_1` accordingly (only `GPIO0-31` are supproted) |
|`BLASTER_ENABLE_IR_OUT_2` = `false` | Disables external IR channel 2 of dock by default. <br/>If you plan to add external circuity set to `true` and configure `BLASTER_PIN_IR_OUT_2` accordingly (only `GPIO0-31` are supproted) |
|`BLASTER_ENABLE_IR_LEARN` = `true`  | Enables learning of IR codes by an IR receiver | 
|`BLASTER_PIN_IR_LEARN` = `36` | GPIO 36 is wired to UEXT header and used by IR receiver on MOD-IRDA add-on |
|`BLASTER_ENABLE_ETH` = `true` | Enables the wired Ethernet interface of Olimex board | 
|`OLIMEX_ESP` = `WROOM` | Defines ESP32 package used in Olimex POE Board. Currently only `WROOM` is tested and confirmed working<br/>`WROVER` untested ⚠️, therefore breaks build on purpose. Adaption in `eth_service.cpp` required!<br/>Has no effect if `BLASTER_ENABLE_ETH=false` |


### Links to Olimex product pages

[Olimex PoE Iso Board](https://www.olimex.com/Products/IoT/ESP32/ESP32-POE-ISO/open-source-hardware)

![Serving suggestion](https://www.olimex.com/Products/IoT/ESP32/ESP32-POE-ISO/images/thumbs/310x230/ESP32-POE-ISO-1.jpg)

[Olimex MOD-IRDA](https://www.olimex.com/Products/Modules/Interface/MOD-IRDA/open-source-hardware)

![Serving suggestion](https://www.olimex.com/Products/Modules/Interface/MOD-IRDA/images/thumbs/310x230/MOD-IRDA-01.jpg)


## DIY Breadboard Example

At least one IR led + transistors and resistors is required for this to operate correctly.
IR learning was tested with an CHQ1838 Infrared receiver module.

Schematic (⚠️ needs update for IR learning!):

![Serving suggestion](https://github.com/itcorner/ESP32-IRBlaster-UCR2/blob/main/kicad/schematic.png?raw=true)



# Integrating with the remote

In the remote web interface:

1. Select *Integrations & docks*
1. Under *Docks* click *+*
1. Click on *Discover Docks*
1. Dock should be recognized either via Bluetooth or via WiFi/Ethernet
1. If recognized via Bluetooth, you have to mandatorily specify Wifi settings during adoption process.
1. Click *Next*
1. The blaster should be added as a new dock. If there were issues, then download the recent log files in
*Settings* / *Development* / *Logs*. There should be a reason for any failures logged by the remote.
1. For debugging, the blaster also provides extensive logging output via the serial interface.

# Caveats

The Unfolded Circle Remote Two API, while [documented](https://github.com/unfoldedcircle/core-api/blob/main/dock-api/README.md),
the API could change with a remote update before this library is updated. 

This project is not from Unfolded Circle, so do not ask for support there.

This is just a minimal implementation, and doesn't not support OTA updates, web config, etc at this stage.

The Unfolded Circle web interface will report an error when attempting to check for firmware updates. This is normal as
the remote does not know about this particular dock.
