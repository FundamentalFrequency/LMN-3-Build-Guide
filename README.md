# LMN-3-Build-Guide
This repository holds the build guide for the LMN-3. It is a part of the larger LMN-3 project, composed of the following
repositories:
- [LMN-3-Build-Guide](https://github.com/stonepreston/LMN-3-Build-Guide)
- [LMN-3-DAW](https://github.com/stonepreston/LMN-3-DAW)
- [LMN-3-MCAD](https://github.com/stonepreston/LMN-3-MCAD)
- [LMN-3-ECAD](https://github.com/stonepreston/LMN-3-ECAD)
- [LMN-3-Firmware](https://github.com/stonepreston/LMN-3-Firmware)
- [LMN-3-Emulator](https://github.com/stonepreston/LMN-3-Emulator)
- [LMN-3-Keycaps](https://github.com/stonepreston/LMN-3-Keycaps)

## Bill of Materials
| Description                | Model                            | Quantity | URL                                                                                            |
|----------------------------|----------------------------------|----------|------------------------------------------------------------------------------------------------|
| Diode                      | 1N4148                           | 49       | https://www.amazon.com/McIgIcM-1n4148-switching-Standard-Through/dp/B06XB1R2NK                 |
| Rotary Encoder             | EC11                             | 4        | https://www.amazon.com/WayinTop-Encoder-Potentiometer-Electronics-Projects/dp/B08728K3YB?psc=1 |
| Switch                     | 5 pin mechanical keyboard switch | 45       |                                                                                                |
| Joystick                   | COM-09032                        | 1        | https://www.sparkfun.com/products/9032                                                         |
| Keycaps                    | Any                              | 45       |                                                                                                |
| Microcontroller            | Teensy 4.1                       | 1        | https://www.pjrc.com/store/teensy41.html                                                       |
| Raspberry Pi               | Raspberry Pi 4                   | 1        |                                                                                                |
| Screen                     | Hyperpixel 4                     | 1        | https://www.sparkfun.com/products/15357                                                        |
| USB to Mini USB Cable      | USB2AUB2RA1M                    | 1        | https://www.amazon.com/StarTech-com-Fiber-Optic-Cable-Multimode/dp/B012S11KV6                        |
| Raspberry Pi Power Adapter | 15W USB-C                        | 1        | https://www.raspberrypi.com/products/type-c-power-supply/ 
| Arduino Header Strip                    |                            | 2      | https://www.amazon.com/gp/product/B07PKKY8BX?psc=1                 |
| LMN-3 PCB            |                            | 1      | https://github.com/stonepreston/LMN-3-ECAD                |
| LMN-3 Case            |                            | 1      | https://github.com/stonepreston/LMN-3-MCAD                |

Note the links provided are simply the parts I used in my own build. If you can find the part somewhere else feel free to use that. The LMN-3-DAW has been tested on the 4 GB Raspberry Pi 4 only. Additionally, the Hyperpixel 4 screen also comes in a non touch version. The LMN-3-DAW does not rely on any touch capability, but the design of the touch-based Hyperpixel 4 looks nicer than the non-touch version. The non-touch version is a little bit cheaper though if you would like to keep costs down. 

## Rasbperry Pi Setup
The steps below assume you have a working Raspberry Pi 4 (running Bullseye) you can log into and use, and that the username for the account is pi. If your username is different, please make sure and make the correct subsitutions. It's also recommended you login to the pi via SSH, assuming you enabled it and filled in the network information when using the [Raspberry Pi Imager](https://www.raspberrypi.com/software/). Network connectivity is required to SSH into the Pi and download the LMN-3 DAW. Commands that look like this:

```
$ I am a command!
```

should be entered into a terminal command window. The `$` should not be included in the commands and just denotes the prompt. 

### Connecting via SSH
On your host machine, find the IP of your pi: 

```
$ sudo arp-scan --interface=wlp3s0 --localnet
```
Find the item in the list that looks like this:
```
10.0.0.146	dc:a6:32:6d:a2:29	Raspberry Pi Trading Ltd
```
Then SSH into the pi:
```
$ ssh pi@10.0.0.146
```

Replace `10.0.0.146` with whatever the IP was for your device. Now you should be remotely connected to the Pi, and can execute commands inside on it in your terminal session.

### Downloading the LMN-3 DAW
```
$ cd ~
$ wget https://github.com/stonepreston/LMN-3-DAW/releases/download/v0.1.0/LMN-3-arm-linux-gnueabihf.zip
```

After downloading, unzip the archive.

```
$ unzip LMN-3-arm-linux-gnueabihf.zip
```

You can run the app to make sure it runs. This will also create the config directory that is needed in the next step:
```
$ ./LMN-3
```
You should see some information printed to the console. Press `ctrl+c` to quit. 

### Hiding the application title bar
We need to hide the application title bar so that the application title and the buttons for minimise, maximise, and close dont appear. This is accomplished via configuration using the LMN-3 config file. The LMN-3 config directory is created when the application is first run, but we must create the config file ourselves:
```
$ nano ~/.config/LMN-3/config.yaml
```
Add the following text to the file to disable the title bar:
```
config:
  show-title-bar: false
```
Close the file and save using `ctrl+x`. 

### Starting the LMN-3 DAW application on startup
We need to run the application on startup. We need to add a bit of delay though so things can initialise and will use a bash script to accomplish this. 
```
$ nano ~/LMN_start.sh
```
Add the following text to the script:
```
#!/bin/bash
sleep 10
/home/pi/LMN-3
```
Make it executable:
```
$ chmod +x ~/LMN_start.sh
```

Create an autostart directory:
```
$ mkdir ~/.config/autostart
```
Then you can use nano to create and edit a desktop file:
```
nano ~/.config/autostart/LMN-3.desktop
```
Add the following text:
```
[Desktop Entry]
Type=Application
Name=LMN-3
Exec=/home/pi/LMN_start.sh
```

Press ctrl+x to exit, and make sure and save the changes when prompted. 

### Configuring the screen

Open `/boot/config.txt`:
```
$ sudo nano /boot/config.txt
```

Add the following lines to the very end of the file:
```
dtoverlay=vc4-kms-dpi-hyperpixel4
dtparam=rotate=90,touchscreen-swapped-x-y,touchscreen-inverted-x
```
See the following [issue](https://github.com/pimoroni/hyperpixel4/issues/177) for more information on configuring the Hyperpixel screen. 

Now reboot the system:
```
sudo reboot
```
Once the system restarts, you should see that the application automatically started up and that the window decorations are gone. If the screen is rotated incorrectly, add a rotate line with the correct angle needed to orient it correctly. The top of the screen should be near the side of the Pi with the power and HDMI ports. 

### Hiding the panel

We want to hide the main menu panel when its not being used. There is probably a way to configure this via the command line, but its easy enough to do with a mouse once you have the screen working. To do that, follow these instructions: 

`right click the menu -> Panel settings -> advanced -> check minimise panel when not in use and set size when minimzed to 0`

If you hover over the menu location it should unhide, and hide again when you move your mouse away from it. 
