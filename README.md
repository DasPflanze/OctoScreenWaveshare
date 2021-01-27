# OctoScreenWaveshare

How to install OctoScreen on OctoPi with an WaveShare 4inch RPi LCD (A) Touchscreen



## Requirements

- Raspberry Pi (in my case 3B)
- already running Octoprint (in my Case OctoPi 0.17.0/OctoPrint 1.3.12)
- WaveShare 4inch RPi LCD (A)



### 1. Connect the screen

At first we need to connect the screen to the Raspberry Pi. When plugged in, the screen will turn white.

### 2. Driver

Next we have to install the necessary drivers for the screen. For my screen I found the required steps in the official [WaveShare Wiki](https://www.waveshare.com/wiki/4inch_RPi_LCD_(A)).

```bash
cd ~
git clone https://github.com/waveshare/LCD-show.git
cd LCD-show/
chmod +x LCD4-show
./LCD4-show
```

Now we have to enter our password. If you are still using the standard **pi** user the password will be **raspberry**.

Next we need to confirm the installation of the needed packages by pressing `Y` followed by `Enter`. 

During the install the Raspberry is going to restart by itself so our SSH session will end. Also now the screen should begin to show the startup sequence of OctoPi.

#### Rotation

Should your screen be upside down don't worry, just SSH back into the Raspberry.

```bash
cd ~/LCD-show/
./LCD4-Show 180
```

This will rotate the screen by 180° other values would be `0`, `90` or `270`

### 3. Preparations for OctoScreen

Since the screen unfortunately has a low resolution, we have to upscale the resolution.

Therfore we need to change `/boot/config.txt`.

``` bash
nano /boot/config.txt
```

Use the arrow keys to get to the lower part of the file. There is a line

```
hdmi_cvt 480 320 60 6 0 0 0
```

We need to change it to

```
hdmi_cvt 800 533 60 6 0 0 0
```

Press `Strg` + `X` followed by `Y` end `Enter` to save the change.



Next we have to install some packages.

```bash
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install libgtk-3-0 xserver-xorg xinit x11-xserver-utils
sudo apt-get install git build-essential xorg-dev xutils-dev x11proto-dri2-dev
sudo apt-get install libltdl-dev libtool automake libdrm-dev
```

After each command we have to confirm the installation by pressing `Y` and `Enter`



To be able to use OctoScreen we also have to install a [video driver](https://github.com/ssvb/xf86-video-fbturbo). So you need to execute following commands line by line.

```bash
cd ~
git clone https://github.com/ssvb/xf86-video-fbturbo.git
cd xf86-video-fbturbo
autoreconf -vi
./configure --prefix=/usr
make
sudo make install
sudo cp xorg.conf /etc/X11/xorg.conf
```

Now is a good time to reboot your Raspberry.

```bash
sudo reboot
```

### 4. Installing OctoScreen

At this state we are going to install [OctoScreen](https://github.com/Z-Bolt/OctoScreen). While writing this HowTo Version 2.6.1 is the newest version of OctoScreen. To make sure you will have the newest version of OctoScreen please follow these [instructions](https://github.com/Z-Bolt/OctoScreen#install-from-a-deb-package). The following commands may be outdated when OctoScreen receives a update.

```
cd ~
wget https://github.com/Z-Bolt/OctoScreen/releases/download/2.6.1/octoscreen_2.6.1_armhf.deb
sudo dpkg -i octoscreen_2.6.1_armhf.deb
```

After executing these commands the Raspberry Display will show the OctoScreen Interface. But there will be a black bar at the bottom and the touchscreen may not work properly.



### 5. Finetuning

#### Resolution

To get rid of the black bar at the bottom we need to chance the resolution of OctoScreen.

```bash
sudo nano /etc/octoscreen/config
```

Here we have to change the line

```bash
OCTOSCREEN_RESOLUTION=800x480
```

to

```bash
OCTOSCREEN_RESOLUTION=800x533
```

To load the new settings we need to reload OctoScreen.

````bash
sudo service octoscreen restart
````



#### Touchscreen calibration

Now we start the calibration. If possible use a pen or something else to hit the red cross-hairs as accurate as possible.

```bash
DISPLAY=:0.0 xinput_calibrator
```

After finishing the calibration the tool will output something like this:

````sh
pi@octopi:~ $ DISPLAY=:0.0 xinput_calibrator
Calibrating standard Xorg driver "ADS7846 Touchscreen"
        current calibration values: min_x=0, max_x=65535 and min_y=0, max_y=65535
        If these values are estimated wrong, either supply it manually with the --precalib option, or run the 'get_precalib.sh' script to automatically get it (through HAL).
        --> Making the calibration permanent <--
  copy the snippet below into '/etc/X11/xorg.conf.d/99-calibration.conf' (/usr/share/X11/xorg.conf.d/ in some distro's)
Section "InputClass"
        Identifier      "calibration"
        MatchProduct    "ADS7846 Touchscreen"
        Option  "MinX"  "3352"
        Option  "MaxX"  "63699"
        Option  "MinY"  "2162"
        Option  "MaxY"  "63557"
        Option  "SwapXY"        "0" # unless it was already set to 1
        Option  "InvertX"       "0"  # unless it was already set
        Option  "InvertY"       "0"  # unless it was already set
EndSection
````

We need to copy the part from `Section "InputClass"` to `EndSection`

```bash
Section "InputClass"
        Identifier      "calibration"
        MatchProduct    "ADS7846 Touchscreen"
        Option  "MinX"  "3352"
        Option  "MaxX"  "63699"
        Option  "MinY"  "2162"
        Option  "MaxY"  "63557"
        Option  "SwapXY"        "0" # unless it was already set to 1
        Option  "InvertX"       "0"  # unless it was already set
        Option  "InvertY"       "0"  # unless it was already set
EndSection
```

and enter the previously copied part with following command.

````bash
sudo nano /etc/X11/xorg.conf.d/99-calibration.conf
````

In my case I had a wierd issue where if I touched in the upper right corner it would actually click in the lower left corner. I got it fixed by adding a Transformation Matrix to the file.

````
Section "InputClass"
        Identifier      "calibration"
        MatchProduct    "ADS7846 Touchscreen"
        Option  "MinX"  "3352"
        Option  "MaxX"  "63699"
        Option  "MinY"  "2162"
        Option  "MaxY"  "63557"
        Option  "SwapXY"        "0" # unless it was already set to 1
        Option  "InvertX"       "0"  # unless it was already set
        Option  "InvertY"       "0"  # unless it was already set
        Option  "TransformationMatrix" "0 1 0 1 0 0 0 0 1"
EndSection
````



Press `Strg` + `X` followed by `Y` end `Enter` to save the file.

After rebooting the calibration should be active and the touchscreen should work as desired.

```bash
sudo reboot
```



Now you should have a completely working OctoScreen setup! 