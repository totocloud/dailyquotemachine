# Daily Quote Machine with Raspberry Pi

![pi quote machine](https://github.com/totocloud/dailyquotemachine/blob/0eb85a4e08da129583b6d5d084836053f889fa93/quote-machine-in-action.JPG)

Get your dose of inspiration with the Daily Quote Machine! Display a fun or inspiring daily quote, joke, or word-of-the-day on a 2.13" Waveshare eInk screen with a Raspberry Pi Zero.

I'm a beginner at Raspberry Pi tinkering, so this project is aimed at newbies and for my learning. The goal for this project was to display a quote or joke on an eInk screen. I used a 2.13-inch Waveshare eInk screen. Pay attention to the sticker on your version and get the correct drivers from Waveshare. 

Note about the 2.13-inch Waveshare eink/epaper screen: My device was flickering and gave me the "busy" signal when I ran the test file, but then I placed both the epd2in13_V4.py and epdconfig.py files in the same folder, which solved the problem for me. Many people have reported similar issues, thinking their device was a dud when it was a file location/version issue on the Waveshare end (which is not uncommon, according to the forums I read).

This project aims for simplicity. Once it's set up, it just plug and go! The Quote Machine will load a new daily quote, joke, or whatever you want from the quotes.json file. The screen will then go to sleep and update again when you choose the next day. The awesome thing about eInk technology is that it's like an Etch-a-Sketch; the words will remain on the screen until refreshed and generally use very little power.

## Hardware I'm Using

- Raspberry Pi Zero 2W
- Waveshare 2.13 inch e-ink HAT display (epd2in13_V4 - check the sticker on yours)
- Micro SD card
- Power for your Pi Zero

## Setup Instructions

### 1. Installing the Waveshare eInk HAT
To install the Waveshare eInk HAT on your Raspberry Pi Zero, follow these steps:

Align the Pins: Carefully align the 40-pin GPIO connector of the HAT with the GPIO pins on the Raspberry Pi. Ensure that the pin 1 (usually marked with a square solder pad or a white dot) on the HAT matches pin 1 on the Raspberry Pi's GPIO header.

Attach the HAT: Gently press the HAT down onto the GPIO pins until it is securely connected. The eInk display should be facing outward, with the screen visible and the HAT's components seated firmly on the Raspberry Pi. The display connectors should be aligned with the screen and not upside down. I made that mistake when I couldn't figure out the driver issue, and things got a bit "smokey," but fortunately, nothing was damaged.

Secure the HAT: Ensure the HAT is firmly attached and all pins are properly connected. This helps prevent any loose connections that might cause issues.

### 2. Run The Raspberry Pi Imager

Download and run the Raspberry Pi Imager from [here](https://www.raspberrypi.com/software/).

Use the most recent version of the Pi OS without a desktop. The Pi Zero 2W is very memory-constrained, and using a desktop OS may cause reliability issues with the display driver.
- Use the gear icon to add extra settings to your install:
  - Add your WIFI credentials.
  - Set a hostname (e.g., `quotemachine.local`) and username/password (e.g., `pi@quotemachine.local/`) which will be used for connecting.
  - Enable SSH. This is required so you can configure the quote machine remotely without a traditional display.
- Run the tool, and it will wipe the SD card and install the image.

### 3. Establish SSH Connection

- Install the SD card in the Pi, and connect it to power. You should see blinking lights as it completes the first boot.
- After a minute or so, open up Windows Power Shell, Terminal, or whatever command line tool you want to use.
- Connect to your Pi Zero via:
  ```
  ssh pi@quotemachine.local
  ```
Your computer will connect to the Pi Zero and do some security stuff like ask for your password. If you have errors here, there's usually an easy fix if you Google it.

If the OS was installed correctly, you will connect and be greeted with the Pi Zero's command line.

### 3. Configure Via SSH

Follow these commands to set up the quote machine with the files it needs to run the program:
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














