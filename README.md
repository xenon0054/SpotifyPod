# sPot

This code is meant to accompany [this project](https://hackaday.io/project/177034-spot-spotify-in-a-4th-gen-ipod-2004) in which a Spotify client is built into an iPod "Classic" from 2004. Everything is meant to run on a Raspberry Pi Zero W.

Since we are using the lite version of raspbian, some extra packages need to be installed:

# Instructions

# Installing SpotifyPod (The code is modified for [Ricardo's Alternate Build](http://rsflightronics.com/spotifypod))
## What you need to do before installing SpotifyPod 
### You need raspberry pi lite os [legacy version](https://www.raspberrypi.com/software/operating-systems/#:~:text=Raspberry%20Pi%20OS%20Lite%20(Legacy)) We will also be using raspotify version 0.14 which we will be installing later

## *IMPORTANT* This code works [2inch LCD Module ST7789V controller 240 x 320 Pixels](https://www.amazon.com/dp/B081NBBRWS/ref=pe_3044161_185740101_TE_item) For more hardware instructions please follow [Ricardo's Alternate Build](http://rsflightronics.com/spotifypod))

## Installing updates
```
sudo apt-get update -y
sudo apt-get upgrade -y
```
## Install Required Packages
```
sudo apt install python-setuptools python3-setuptools -y

sudo apt install python3-pip -y

sudo wget https://github.com/dtcooper/raspotify/releases/download/0.14.0/raspotify_0.14.0.librespot.20200130T014147Z.3672214_armhf.deb

sudo apt-get install python3-tk -y

sudo apt-get install redis-server -y

sudo apt-get install openbox -y

sudo apt install xorg -y

sudo apt-get install lightdm -y

sudo apt-get install x11-xserver-utils -y

sudo apt-get install git -y
```
## Change directory to home
```
cd ~
```
## Clone this repo and Install Dependencies
```
sudo dpkg -i raspotify_0.14.0.librespot.20200130T014147Z.3672214_armhf.deb

git clone https://github.com/dupontgu/retro-ipod-spotify-client

cd retro-ipod-spotify-client/frontend

pip3 install -r requirements.txt
```
## Installing Pi-btaudio
```
git clone https://github.com/bablokb/pi-btaudio.git

cd pi-btaudio

sudo tools/install
```
## Install PigPio
```
wget https://github.com/joan2937/pigpio/archive/master.zip

unzip master.zip

cd pigpio-master

make

sudo make install
```
## Setup spotify API
### Create a Spotify Developer Account, then create an app where you will get your Client ID, Client Secret, and create a Redirect URI 
https://developer.spotify.com/dashboard/applications/
![Create_app](https://user-images.githubusercontent.com/76131600/206828247-0f1085e5-aade-4dce-8936-f5f54766b830.jpg)
### Add an app name and description can be anything
![Add_app_name](https://user-images.githubusercontent.com/76131600/206828252-c608c53d-1d47-4a14-aec7-8c7df368ac42.jpg)
![Add_host_uri](https://user-images.githubusercontent.com/76131600/206828258-36a8b5f4-b7b2-46fa-ae81-7758ea84eef4.jpg)
![Add_uri_2](https://user-images.githubusercontent.com/76131600/206828266-05447afb-37e0-490f-b4a0-0e2e8103f696.jpg)
<img width="604" alt="add_uri_3" src="https://user-images.githubusercontent.com/76131600/206828283-340e65e5-b049-4387-b6fc-4129e22b5141.png">
![add_rui_4](https://user-images.githubusercontent.com/76131600/206828288-3aee4087-279a-43e5-8e04-525637d11357.jpg)
### Save Client id and Save Secret id
![Secret_and_client](https://user-images.githubusercontent.com/76131600/206828315-cc4aa85a-25a0-45af-b82e-b6ad4f7aae82.jpg)

## Disable screen blanking
```
sudo raspi-config
Display Option > Screen Blanking > OFF / Disable
```
## Disable Cursor and Screen Savers
### Add or Create and Add this:
```
sudo nano ~/.bash_profile
```
### Then:
```
'#!/bin/bash'

[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && startx -- -nocursor

# Disable any form of screen saver / screen blanking / power management

xset s off

xset s noblank

export SPOTIPY_CLIENT_ID='your_SPOTIPY_CLIENT_ID'

export SPOTIPY_CLIENT_SECRET='your_SPOTIPY_CLIENT_SECRET'

export SPOTIPY_REDIRECT_URI='http://localhost:8080'

export DISPLAY=:0.0
```
## Configure Xintric
```
sudo nano /etc/X11/xinit/xinitrc
```
###  
```
#!/bin/sh

# /etc/X11/xinit/xinitrc
# global xinitrc file, used by all X sessions started by xinit (startx)
# invoke global X session script 
#. /etc/X11/Xsession' 

exec openbox-session #-> This is the one that launches Openbox ;)`
```
## Run "spotifypod.py" with Autostart
```
sudo nano /etc/xdg/openbox/autostart
```
### Then paste this before exit
```
#cd /home/pi/retro-ipod-spotify-client/frontend/
#sudo -H -u pi --preserve-env=SPOTIPY_REDIRECT_URI,SPOTIPY_CLIENT_ID,SPOTIPY_CLIENT_SECRET python3 spotifypod.py &
#sudo /home/pi/retro-ipod-spotify-client/clickwheel/click &
```
### Please comment this out until I remind you to uncomment it, save the file and exit, then we are now going to make the the program that makes the clickwheel work an executable
```
cd /home/pi/retro-ipod-spotify-client/clickwheel
nano click.c
```
change #define DATA_PIN 25 to #define DATA_PIN 5
```
gcc -Wall -pthread -o click click.c -lpigpio -lrt
```
### when were running "gcc -Wall -pthread -o click click.c -lpigpio -lrt" just leave it alone till you see a blinking text box
```
sudo chmod +x click
```
## Spotify credentials
```
sudo nano /etc/xdg/openbox/environment

export SPOTIPY_CLIENT_ID='your_SPOTIPY_CLIENT_ID'

export SPOTIPY_CLIENT_SECRET='your_SPOTIPY_CLIENT_SECRET'

export SPOTIPY_REDIRECT_URI='http://localhost:8080'
```
## Synchronize Spotify Data
### Synchronizing Spotify data! Last but not least, if you want to make sure all your playlists artists, etc are synchronized every time you turn on your Spotypod, you can simply modify the script view_model.py
```
sudo nano view_model.py
```
### go to line 16
```
#spotify_manager.refresh_devices()

spotify_manager.refresh_data()
```
## Configure Raspotify
```
sudo nano /etc/default/raspotify
```
### Uncomment and fill the following line:
```
OPTIONS="--username XXXXXXXX --password XXXXXXXXX"
```
### Enter your spotify login details, In the first XXX add your email address, and in the second XXX add your password
### Ex:
```
OPTIONS="--username email_address@hotmail.com --password thisismypassword"
```

### Paste the following
```
# The displayed device type in Spotify clients.
# Can be "unknown", "computer", "tablet", "smartphone", "speaker", "tv",
# "avr" (Audio/Video Receiver), "stb" (Set-Top Box), and "audiodongle".
DEVICE_TYPE="smartphone"
```
## Entering Device id
To obtain your device id you first have to go to https://developer.spotify.com/console/get-users-available-devices/ 
when you go the the website your ipod may not appear connect to it using the spotify app on the device list then retry the website and it should show up
```
cd /home/pi/retro-ipod-spotify-client/frontend/

nano spotify_manager.py
```
Go to line 173 using ctrl+shift+_ and type 173 You can comment out the entire refresh_device def and replace it with
```
def refresh_devices():
    device = UserDevice('xxxxxxxxxxxxxxxxxxxxxxxxxxxxx','raspotify', True)
    DATASTORE.setUserDevice(device)
```
Replace the XXXXXXX with your device id 
Ex: 
def refresh_devices():
    device = UserDevice(‘123456789','raspotify', True)
    DATASTORE.setUserDevice(device)
After that you want to go save and exit the file then
```
cd ~/.local/lib/python3.7/site-packages/spotipy/

nano client.py
```
Go to line 1734 using ctrl+shift+_
Paste this in:
```
data = {"device_ids": ["xxxxxxxxxxxxxxxxxxxxxxxxxx"], "play": force_play}
    return self._put("me/player", payload=data)
```
Replace the xxxxxx with your device id

# Post Install
## Generate .cache files
```
sudo apt install midori

cd ~

git clone https://github.com/perelin/spotipy_oauth_demo

cd ~/spotipy_oauth_demo

sudo apt-get install python-pip python-dev

sudo apt-get install python3-pip python-dev

pip install -r requirements.txt
```
Before running it modify the client id and secret also your scopes  in with:
```
nano spotipy_oauth_demo.py
```
### Scope:
```
SCOPE = "user-follow-read," \
        "user-library-read," \
        "user-library-modify," \
        "user-modify-playback-state," \
        "user-read-playback-state," \
        "user-read-currently-playing," \
        "app-remote-control," \
        "playlist-modify," \
        "playlist-read-private," \
        "playlist-read-collaborative," \
        "playlist-modify-public," \
        "playlist-modify-private," \
        "streaming," \
        "user-follow-modify," \
        "user-follow-read"
```
Now you can save and exit the file
### before reboot do the following:
```
sudo raspi-config

system options --> boot / auto login --> console autologin
```
### Turn on vnc or connect a mouse to the pi, then reboot

### Then you can run this program:
```
python spotipy_oauth_demo.py
```
Recommend using a Pi 4 or anything faster than a zero w, but if this is the only pi you have you should be fine but it will take a while. Your pi should be displaying a black screen then right click and click on browser then go to internet/ browser (midori). Once you did that go to http://localhost:8080/ There should be a spotify sign in page, sign in, once signed in it should redirect you to a page with gibberish code and spotify credits. Once you're on that screen exit the page and in your terminal stop running the program.

### Now move your cache file to the one in spotifypod
```
cp ~/spotipy_oauth_demo/.spotipyoauthcache ~/retro-ipod-spotify-client/frontend/.cache

chmod 777 ~/retro-ipod-spotify-client/frontend/.cache
```
Once your done you can turn off vnc
If you get ERNO98 already in use go to [issue 30](https://github.com/dupontgu/retro-ipod-spotify-client/issues/30)

## Install Display code
ssh into the pi and edit the config file
```
sudo nano /boot/config.txt
```
Add the code below:
# DIsplay code
hdmi_group=2
hdmi_mode=87
hdmi_cvt=320 240 60 1 0 0 0
hdmi_force_hotplug=1
### Also you have to comment out this code:
```
dtoverlay=vc4-kms-v3d
```
Then save it, after that download the display code:
```
sudo apt install cmake git
cd ~
git clone https://github.com/juj/fbcp-ili9341.git
cd fbcp-ili9341
```
Then we want to change the native resolution to take advantage of the whole display.
```
nano st7735r.h
```
Go to line 18 which is ST7789
Change DISPLAY_NATIVE_HEIGHT from 240 to 320
After that save it and then nano into st7735r.cpp
```
nano st7735r.cpp
```
Go to line 95 with ctrl+shift+_
Comment out the line beginning with SPI_TRANSFER with // then below it add this:
```
SPI_TRANSFER(0x37, 0, 0);
```
Then exit the file then run this command:
```
mkdir build

cd build

cmake -DST7789=ON -DGPIO_TFT_DATA_CONTROL=24 -DGPIO_TFT_RESET_PIN=25 -DSPI_BUS_CLOCK_DIVISOR=30 -DSTATISTICS=0 -DDISPLAY_BREAK_ASPECT_RATIO_WHEN_SCALING=ON -DUSE_DMA_TRANSFERS=OFF ..
```
Once finished run:
```
make -j
```
After that is finished we want to make it run on boot:
```
sudo nano /etc/rc.local
```
Enter this before exit:
```
sudo /home/pi/fbcp-ili9341/build/fbcp-ili9341 &
```
# Uncomment the Autostart things:
```
sudo nano /etc/xdg/openbox/autostart

cd /home/pi/retro-ipod-spotify-client/frontend/

sudo -H -u pi --preserve-env=SPOTIPY_REDIRECT_URI,SPOTIPY_CLIENT_ID,SPOTIPY_CLIENT_SECRET python3 spotifypod.py &

sudo /home/pi/retro-ipod-spotify-client/clickwheel/click &
```
## Bluetooth
For bluetooth I these instructions:
To install blue-alsa [link](https://sigmdel.ca/michel/ha/rpi/bluetooth_n_buster_01_en.html#:~:text=Install%20the%20blueALSA%20proxy.)
To set up auto connect [link](https://github.com/dtcooper/raspotify/wiki/Play-via-Bluetooth-Speaker#:~:text=Raspberry%20Pi%20Desktop-,via%20asound.conf,-You%20need%20to)
### Use this to pair bluetooth device:
![pairing](https://user-images.githubusercontent.com/76131600/206840406-e13c1af3-674a-44b8-a52b-62cb6c34f66f.png)

Please only follow the configuration instructions you already downloaded it in the previous link. [link](https://github.com/bablokb/pi-btaudio#:~:text=to%20the%20device-,Configuration,-After%20installation%2C%20edit)

## When Done
### Reboot and whenever you need to use the pod connect to it from your phone/ computer on the device list as if it were a speaker. It might take a while but it should connect eventually then you're free to exit the app and play songs from the pod.

## Issues
Trying to recreate the Bluetooth settings (If you want to try and figure it out, I suggest going to #66 near the bottom there should be some links doris put)
If you have any issues please feel free to comment them below 😄 

# Resources:
Couldn't have done this without doris, they've helped me out so much.
https://hackaday.io/project/177034-spot-spotify-in-a-4th-gen-ipod-2004
https://github.com/dupontgu/retro-ipod-spotify-client
https://www.youtube.com/watch?v=KciKqGX8g94 
https://github.com/dupontgu/retro-ipod-spotify-client/issues/41
https://github.com/dupontgu/retro-ipod-spotify-client/issues/22
https://github.com/dupontgu/retro-ipod-spotify-client/issues/34
https://github.com/dupontgu/retro-ipod-spotify-client/issues/65
https://github.com/dupontgu/retro-ipod-spotify-client/issues/23
https://github.com/dupontgu/retro-ipod-spotify-client/issues/66
http://rsflightronics.com/spotifypod
https://sigmdel.ca/michel/ha/rpi/bluetooth_n_buster_01_en.html
https://github.com/dtcooper/raspotify/wiki/Play-via-Bluetooth-Speaker
https://github.com/bablokb/pi-btaudio

# Wiring

Here is the wiring of the hardware, as of revision 1. Note that the pin numbers correlate to those referenced in [click.c](./clickwheel/click.c)

![Wiring Diagram](./.docs/sPot_schematic.png)
