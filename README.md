# Daily Quote Machine with Raspberry Pi

Get your dose of inspiration with the Daily Quote Machine! Display a fun or inspiring daily quote, joke, or word-of-the-day on a 2.13" Waveshare eInk screen with a Raspberry Pi Zero.

I'm a very beginner Raspberry Pi tinkerer, so this project is aimed at newbies and for my own learning. The goal for this project was to display a quote or joke on an eInk screen. I used a 2.13-inch Waveshare eInk screen. Pay attention to the sticker on your version and get the correct drivers from Waveshare. My device was flickering and gave me the "busy" signal, but I placed both the epd2in13_V4.py and epdconfig.py file in the same folder and it solved the problem for me. Many people have reported similar issues thinking their device was a dud, when it was a file/version issue on the Waveshare end.

This project aims for simplicity. Once it's set up, it just plug-and-go! The Quote Machine will load a new daily quote, joke, or whatever you want from the quotes.json file. The screen will then go to sleep and update again at the time of your choosing the next day. The awesome thing about eInk technology is that it's like an Etch-a-Sketch; the words will remain on the screen until refreshed and use very little power in general.

![Place Quote Machine Picture Here](dailyquotemachine/quote machine in action.JPG)

## Hardware I'm Using

- Raspberry Pi Zero 2W
- Waveshare 2.13 inch e-ink display (epd2in13_V4 - check the sticker on yours)
- Micro SD card
- Power for your Pi Zero

## Setup Instructions

### 1. Run The Raspberry Pi Imager

Download and run the Raspberry Pi Imager from [here](https://www.raspberrypi.com/software/).

- Use the no-desktop version of the most recent version of the Pi OS. The Pi Zero 2W is very constrained for memory. You may have reliability issues with the display driver if you use a desktop OS.
- Use the gear icon to add extra settings to your install:
  - Add your WIFI credentials.
  - Set a hostname (e.g., `quotemachine.local`) and username/password (e.g., `pi@quotemachine.local/`) which will be used for connecting.
  - Enable SSH. This is required so you can configure the typewriter remotely, without a traditional display.
- Run the tool, and it will wipe the SD card and install the image.

### 2. Establish SSH Connection

- Install the SD card in the Pi, and connect it to power. You should see blinking lights as it completes the first boot.
- After a minute or so, open up Windows Power Shell, Terminal, or whatever command line tool you want to use.
- Connect to your Pi Zero via:
  ```
  ssh zero@zerowriter.local
  ```
- Your computer will connect to the Pi Zero and do some security stuff—if you have errors here, there's usually an easy fix if you google it.
- You will connect and be greeted with the Pi Zero's command line.

### 3. Configure Via SSH

Follow these commands in order to set up the quote machine with the files it needs to run the program:
```
sudo apt update
sudo apt full-upgrade
```

#### Turn on SPI (which allows the Pi to use the display)
sudo raspi-config
Interfacing Options -> SPI -> Enable

#### Install dependencies:
```
sudo apt-get install python3-pip python3-pil python3-numpy git
pip3 install RPi.GPIO spidev
```
#### Install the Waveshare drivers:
```
git clone https://github.com/waveshare/e-Paper.git ~/e-Paper
pip3 install ~/e-Paper/RaspberryPi_JetsonNano/python/
```
#### Clone the code from GitHub to your Pi for the Quote Machine to run:
```
git clone https://github.com/totocloud/dailyquotemachine.git ~/dailyquotemachine
```
#### Run the program:

```
python ~/dailyquotemachine/quotemachine.py
```
#### And there you have it. Your very own Quote Machine!

### Autorun on Reboot

#### Create a new service configuration file:
```
sudo nano /etc/systemd/system/quotemachine.service
```
#### Copy and paste the following into the service configuration file and change any settings to match your environment:
```
[Unit]
Description=QuoteMachine
After=network.target

[Service]
ExecStart=/usr/bin/python3 -u /home/pi/dailyquotemachine/quotemachine.py
WorkingDirectory=/home/pi/dailyquotemachine
StandardOutput=inherit
StandardError=inherit
Restart=always
User=pi 

[Install]
WantedBy=multi-user.target
```

#### Enable the service so that it starts whenever the RPi is rebooted:
```
sudo systemctl enable quotemachine.service
```

#### Start the service:
```
sudo systemctl start quotemachine.service
```

If you need to troubleshoot, you can use the logging configurations of this program (mentioned below). Alternatively, you can check to see if there is any output in the system service logging:
```
sudo journalctl -f -u quotemachine.service
```

### Optional: Install SMB for File Access

If you want to easily access and manage your files from another computer, you should install SMB on your Pi. I used it to install the Josefin_Sans TrueType fonts under /usr/share/fonts/truetype/JosefinSans/JosefinSans-ExtraLight.ttf. Check out the instructions [here](https://pimylifeup.com/raspberry-pi-samba/).














