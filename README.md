# LMN-3-Build-Guide

## Rasbperry Pi Setup
The steps below assume you have a working Raspberry Pi 4 (running Bullseye) you can log into and use, and that the username for the account is pi. If your username is different, please make sure and make the correct subsitutions. It's also recommended you login to the pi via SSH, assuming you enabled it and filled in the network information when using the [Raspberry Pi Imager](https://www.raspberrypi.com/software/). Network connectivity is required to SSH into the Pi and download the LMN-3 DAW. Commands that look like this:

```
$ I am a command!
```

should be entered into a terminal command window. The `$` should not be included in the commands and just denotes the prompt. 

## Connecting via SSH
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

After downloading, unzip the archive and run the application to make sure it starts:

```
$ unzip LMN-3-arm-linux-gnueabihf.zip
$ ./LMN-3
```
Once it starts up, you can close it as we won't need it running for the next steps. 

### Hiding the panel

We want to hide the main menu panel when its not being used. To do that, follow these instructions: 

`right click the menu -> Panel settings -> advanced -> check minimise panel when not in use and set size when minimzed to 0`

If you hover over the menu location it should unhide, and hide again when you move your mouse away from it. 

### Removing window decorations

Since we don't want to see the window decorations (the title bar with maximize, minimize and close buttons) we will need to remove them from the application. The system application that controls how windows are decorated is called the window manager. In Bullseye, this is mutter. I could not figure out how to remove decorations (or really any information on configuring mutter at all), so we will revert back to using openbox as the window manager instead, which should already be installed and available. 

First, we need to enable openbox by editing `/etc/xdg/lxsession/LXDE-pi/desktop.conf`:
```
$ sudo nano /etc/xdg/lxsession/LXDE-pi/desktop.conf
```
Change the line that sets the window manager to use openbox instead:
```
window_manager=openbox
```
Press ctrl+x to exit, and make sure and save the changes when prompted. 

Now we need to add a directory for user-level openbox configuration:
```
$ mkdir ~/.config/openbox
```

Copy the system-level configuration file to this new directory:
```
$ cp /etc/xdg/openbox/lxde-pi-rc.xml /home/pi/.config/openbox/rc.xml
```
Note the destination filename is rc.xml. 

Next, edit the configuration file:
```
$ nano /home/pi/.config/openbox/rc.xml
```
Scroll all the way down to the bottom of the file and find the configuration tag for applications. We need to add a new  application tag as a child of this applications tag (there may already be a few children too, just add the new one at the end.) This new application tag will undecorate only the LMN-3 application. The tag you need to add looks like this:
```
<application name="LMN-3">
  <decor>no</decor>
</application>
```

The end of your file should end up looking like this when it's all said and done:
```
...
...
    		<application name="Some-other-application">
    		</application>
    
		<application name="LMN-3">
			<decor>no</decor>
		</application>
	</applications>
</openbox_config>
```
Save and close the file. 

That's all that is needed to remove the window decorations. 

### Starting the LMN-3 DAW application on startup
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
Exec=/home/pi/LMN-3
```

Press ctrl+x to exit, and make sure and save the changes when prompted. 

Now reboot the system:
```
reboot
```
Once the system restarts, you should see that the application automatically started up and that the window decorations are gone. 
