## Table of Contents

- [1. Meshcore region guide](#1-meshcore-region-guide)
- [2. The core concept](#2-the-core-concept)
- [3. Important notes](#3-important-notes)
- [4. Setting a region on your repeater](#4-setting-a-region-on-your-repeater)
  - [4.1 What does it mean?](#41-what-does-it-mean)
  - [4.2 Set region via Command line](#42-set-region-via-command-line)
  - [4.2 Set region in App](#42-set-region-in-app)
- [5. Add regions that can be used for channels (client app)](#5-add-regions-that-can-be-used-for-channels-client-app)
- [6 Flow charts](#6-flow-charts)
- [7 Interesting links](#7-interesting-links)

# Do I need a new bootloader?
When you are using a nRF52, you'll probably need a new bootloader created by oltaco. The default booloader will have a high change of failing (50/50).https://github.com/oltaco/Adafruit_nRF52_Bootloader_OTAFIX/releases

## How do I install the bootloader
1. Download the bootloader file. The file should start with `update-` and end with `.uf2`. Search for the current release and you devive. So for a wismesh tag search for eaxample: `update-wismesh_tag_bootloader-0.9.2-OTAFIX2.1-BP1.2_nosd.uf2
`
2. **IMPORTANT:** If you want to keep the old keys and config, export your config(only for clients using the app) or just the private key (commandline get prv.key).
3. Store the key/config somewhere
4. Connect your device to a pc over usb
5. Put your device in DFU mode. For the wismesh tag you'll have to press the back button 2 times.
6. You are now able to open the device file folder from your pc.
7. Drag the file into the folder
8. The device will restart
9. Check if the bootloader was installed by checking the HTM file
10. Open the web flasher
11. Erase your device
12. Flash the correct software on your device
13. (if client)Open your device if client and upload the config
13. (if repeater) Config your device via the web interface and then open the command line and type `set prv.key + your private key`
14. reboot

# How to OTA update
1. On flasher.meshcore.co.uk, download the non-merged version of the firmware for your ESP32 device (e.g. Heltec_v3_repeater-v1.6.2-4449fd3.bin, no "merged" in the file name)
2. From the MeshCore app, login remotely to the repeater you want to update with admin privilege
3. Go to the Command Line tab, type start ota and hit enter.
4. you should see OK to confirm the repeater device is now in OTA mode
5. The command start ota on an ESP32-based device starts a wifi hotspot named MeshCore OTA
6. From your phone or computer connect to the 'MeshCore OTA' hotspot
7. From a browser, go to http://192.168.4.1/update and upload the non-merged bin from the flasher
8. It is finished when it shows 100%