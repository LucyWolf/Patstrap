# Patstrap

<p>
    <a href="#" alt="size"><img alt="Repository size" src="https://img.shields.io/github/repo-size/danielfvm/Patstrap?color=39d45f"></a>
    <a href="//github.com/danielfvm/Patstrap/commits/master" alt="commit"><img alt="Last Commit" src="https://img.shields.io/github/last-commit/danielfvm/Patstrap?color=39d45f"></a>
    <a href="//github.com/danielfvm/Patstrap/commits/master/" alt="Commits"><img alt="GitHub commit activity (branch)" src="https://img.shields.io/github/commit-activity/m/danielfvm/Patstrap/master"></a>
    <a href="//github.com/danielfvm/Patstrap?tab=GPL-3.0-1-ov-file#readme" alt="licence"><img alt="GitHub License" src="https://img.shields.io/github/license/danielfvm/Patstrap"></a>
    <a href="#" alt="stars"><img alt="GitHub Repo stars" src="https://img.shields.io/github/stars/danielfvm/Patstrap"></a>
</p>

An open hardware and software project which tries to implement haptic head pat feedback to the player in VR. This project focuses mainly on VRChat's OSC support but might in the future also support other games. The project consists of a hardware part the "Headpat-Strap" or just "Patstrap", a Server running on the PC and the required edits on a VRChat-Avatar to support the communication over OSC. Keep in mind that this is only a hobby project, but feel free to experiment, edit the code or tweak the hardware to your liking.
> [!WARNING]
> This project is in the development phase. Please note that components of the project are subject to frequent changes.



## Hardware
### Parts
To make this project you will need:
- [ESP8266 WEMOS D1 Mini V4.0.0](https://de.aliexpress.com/item/1005006246661815.html) If you use a different ESP, you may need to change the pins in the code.
- 2x [150KΩ Resistors](https://de.aliexpress.com/item/1005001627995396.html)
- 2x [Transistors](https://de.aliexpress.com/item/1005003450640801.html)  (I used BC547)
- 2x [Vibrating motors](https://aliexpress.com/item/1005001446097852.html)

And optionally if you are using a battery:
- 1x [180KΩ Resistors](https://de.aliexpress.com/item/1005001627995396.html)
- 2x [Diodes 1N5817](https://de.aliexpress.com/item/1005002620053985.html)
- [Battery](https://www.amazon.com/dp/B0B7N2T1TD?psc=1&ref=ppx_yo2ov_dt_b_product_details)
- [Battery charger](https://de.aliexpress.com/item/1005006274938832.html)
- [On/Off switch](https://aliexpress.com/item/1005003938856402.html)

### Electronics
An ESP8266 was used for this project, but can be switched with any other WLAN-capable IC.
For the haptic feedback two [Vibrating Mini Motor Disc](https://www.adafruit.com/product/1201) from Adafruit were used for a 3D spaced feedback on the head.
Both of them can be directly wired to the ESP but for higher performance it is recommended to switch them with two transistors like the BC547b.

![Untitled Sketch_Steckplatine](https://github.com/user-attachments/assets/1c01b1d1-f472-4f66-8379-30596556ac9b)


### Hausing
The 3D-Model and the `.scad` file of the hausing, which includes some space for the Motor Discs and the ESP8266, is available under `/model` and can be 3D-Printed. Alternatively you can use a normal headband and simply hot glue the Motor Discs and the ESP on a headband.
> [!WARNING]
> The models in `/model` are not up to date and might not fit anymore.

### Battery
For battery support please refer to the [SlimeVR Docs](https://docs.slimevr.dev/diy/tracker-schematics.html). SlimeVR uses a `TP4056` for charging and powering the device. If you want to measure the battery level you need to add the 180kOhm resistor at Pin A0 (enable `Battery sense` in slimevr docs for circuit diagram). 
### PCB
Optionally you can order a PCB for the Patstrap. The Gerber files required for ordering can be found [here](/pcb). Thanks to The-Prophet for making the PCB.
> [!WARNING]
> The PCB has not been tested yet and is still a Work In Progress!

## Software
### Firmware
To upload the firmware to the ESP we use [Visual Studio Code](https://code.visualstudio.com/download) and [PlatformIO](https://platformio.org/platformio-ide). Please refer to the [SlimeVR Docs](https://docs.slimevr.dev/firmware/setup-and-install.html) as a reference on how to install the IDE and the extension. After the installation you can download this repository and open the `/firmware` folder in Visual Studio Code.

Open the `platformio.ini` and change `-DWIFI_CREDS_SSID` and `-DWIFI_CREDS_PASSWD` to your local network's name and password. Keep in mind that it must be the same network your computer is running on in order for the device to communicate with the server program. If you are <ins>not using an ESP8266</ins> you will need to change `board = esp12e` to your board and most likely also the PIN-Layout in the `main.cpp` file. If you want to measure battery level don't forget to uncomment `-DUSE_BATTERY` (remove `;`). If you used a PNP transistor instead of the NPN like BC547b, uncomment `-DUSE_PNP`.

Your config file should look something like this:
```ini
[env]
monitor_speed = 9600
monitor_echo = yes
monitor_filters = colorize
framework = arduino

build_unflags = -Os
build_flags = -O2
; Set your wifi name and password here - Make sure it's within the same Network as the server software!
  -DWIFI_CREDS_SSID='"WIFI_NAME"'
  -DWIFI_CREDS_PASSWD='"WIFI_PASSWORD"'

; Uncomment below if you used a PNP transistor, if you followed the guide you proably want to leave it commented. => inverts the output of the haptic motors
;  -DUSE_PNP

; Uncomment below if you use a battery
;  -DUSE_BATTERY

; The port used to communicate to the patstrap server, if you change this you will also need to change the --esp-port in the server software
  -DPORT='8080'

[env:esp12e]
platform = espressif8266
board = esp12e
framework = arduino
upload_speed = 921600
```
After your edits you can plug-in your ESP, press build (✓) and flash (→) it. You can find the buttons in Visual Studio Code at the bottom on the left side.

![image](/img/flash.png)

### Server
Under releases you can find the binary files to run the server on your computer. The server is the middle man that allows communication between the device and VRChat. The server supports both Windows and Linux. The server opens up a window where the current connection status is displayed. If flashing the hardware worked and the device is running you should see the text `connected`. You can also verify the connection by looking at the ESP-LED.
* Fast blinking (~10 times per second): Not connected with WLAN.
* Slow blinking (~1 time per second): Connected to WLAN, not connected with server.
* Continues light: Connected.
* No light: Not turned on?

If connection was successful `Patstrap connection` should turn green. Furthermore you can now test the hardware by clicking `Pat left` and `Pat right`.

![image](/img/gui.png)

#### Port
You can change the OSC Port as well as the Port to the ESP by adding a launch argument. 
To change the OSC port start the server using the cmd and enter `patstrap.exe --osc-port 1234`. 
To change the ESP port enter `patstrap.exe --esp-port 1234` instead, but make sure that you also change the port in the `platformio.ini` file.
If you dont want to manually set the port everytime you start the server you can create a `.bat` file with the following content:
```
@echo off
patstrap.exe --osc-port 1234
```
> [!IMPORTANT]
> Make sure that the `.bat` file is in the same folder as the `.exe` file. Alternatively add the fullpath to the `.exe` in the `.bat` file.

If you are using [VRCOSC](https://github.com/VolcanicArts/VRCOSC) you can add the `.exe` to the startup and (if needed) can add the launch arguments there.

### VRChat
#### Avatar - Unity
For the Patstrap to work you will need to [enable OSC Support in VRChat](https://docs.slimevr.dev/server/osc-information.html) and edit your Avatar Model to include the required Colliders for detecting a head pat. For this you need to have a working avatar setup in unity. 

1. Create Empties

    First open up your Avatar in Unity, go to `armature -> Hips -> Spine -> Chest -> Neck -> Head` and add two `Empty` objects as a child of the head. It should look like the following image. Optionally you can rename them for better organization.
   
    ![image](/img/empties.png)

2. Add Contact Receivers 

    Open up the just added objects and click on `Add Component` and select `VRC Contact Receiver`. 
    
    ![image](/img/receivers.png)

3. Positioning 

    Now move the contact receivers to your left and right of your avatar's head. Change the size and form if required. The position and size should resemble the following.
    
    ![image](/img/positioning.png)

4. Configure Contact Receivers

    Under the section `Collision Tags` click on `Add` and select `Hand` and repeat this step for `Finger`. In the section `Receiver` change `Receiver Type` to `Proximity` and the `Parameter` to one of the following fitting names.
    * `pat_right` for the collider placed on the right side
    * `pat_left` for the collider placed on the left side

    The end result should look similar to the following image. Repeat this step for the other two contact receivers.
    
    ![image](/img/configure.png)

5. Upload

    Now you should be ready to test and upload your avatar.

### Testing & Debugging
After you uploaded your avatar and enabled osc support, the `VRChat connection` indicator should turn green as soon as the server software receives any avatar parameter including the head pat parameter. If this is not the case the following steps should help you finding the issue.
1. Check if the parameters `pat_right` and `pat_left` were added to your uploaded avatar. You can find the json file at `~\AppData\LocalLow\VRChat\VRChat\OSC\{userId}\Avatars\{avatarId}.json`. If that is not the case, make sure the Avatar setup step was done correctly. For more information you can also check [VRChat's docs](https://docs.vrchat.com/docs/osc-avatar-parameters).
2. Use [Protokol](https://hexler.net/protokol) for debugging and check if `pat_left` or `pat_right` appear in the log. Make sure you set the port to the port used by [VRChat (default 9001)](https://docs.vrchat.com/docs/osc-overview).
3. Start Patstrap Server in the CMD and look for error messages.
4. If nothing worked feel free to file an [issue](https://github.com/danielfvm/Patstrap/issues) or ask me on the [Discord Server](https://discord.gg/QsuHQXECw2) or write me directly at `DeanCode#3641`.



## Contribute
I am grateful for any help, so if you like this project and want to help make it better feel free to make a [pull request](https://github.com/danielfvm/Patstrap/pulls), share your opinions, ideas or problems in `issues`. If you want to help but have no idea what needs to be done, here some general details: 

### Goals
* Designing a hardware that is cheap and easy for others to replicate.
* Electronic components should idealy be available on stores like amazon.
* Documenting every step necessary to reproduce hardware / software.

### TODO's and Issues
* Design casing for ESP
    * Currently I am thinking of putting battery on one side and esp on the other, leaving a gap in between for VR headset's top strap.
    * Put vibration motor more to the side for a more clear distinction between left and right
    * Try isolating the motors from the strap to reduce noise - or - Find better alternative to the `Vibrating Mini Motor Disc`, maybe one that can vibrate on lower frequencies (less noise?)
* Add setting for changing between "Vibrate on collision" and "Vibrate on motion"
* Add options to add more vibration motors - (more then just left / right)
    * Add a list to the server gui to be able to add & remove new motors
    * Change communication between server and hardware from 1 byte (4 bits per motor) to a more flexible system
    * Update hardware software accordingly
* Video and update instructions in `README.md`

## Related projects
* [BS_Patstrap_support](https://github.com/tapafon/BS_Patstrap_support) by tapafon is a IPA mod that adds Patstrap support to Beatsaber 
* [SR_Patstrap_support](https://github.com/tapafon/SR_Patstrap_support) by tapafon is a Melon mod that adds Patstrap support to Synth Riders 
* [HapticPatPat](https://github.com/kikookraft/HapticPatPat) by kikookraft is a fork that adds Bluetooth support for lower latency

## Changelog 
#### v0.3
* Changed design of server app
* Hardware reconnecting to server should work now (also with different IP)
* Improved shutdown of server app 
* Added support for battery measurement
* Added support in the firmware for both PNP and NPN transistor
* Improved PWM signal output (uses now builtin function from arduino)
* Updated README to match the new setup.
#### v0.2.1
* Added `windows-debug.exe` built (opens cmd for additional logs)
#### v0.2
* Simplified background in server application
* Fixed test buttons
* Potentially fixed a div by zero crash
* Fixed VRChat status indicator not working



## Credits
This project uses and refers to many parts from the [SlimeVR](https://www.crowdsupply.com/slimevr/slimevr-full-body-tracker) project which is an open hardware, full body tracking solution and a great project that you definitely should checkout.
