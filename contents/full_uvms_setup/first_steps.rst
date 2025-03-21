First steps 
###########

I got a new Raspberry Pi 5 and a 32 GB SD card on which Ubuntu 24.04. must be flashed.

To flash the SD card with Ubuntu 24.04. it is recommended to use the Raspberry Pi Imager. A download is available for Windows, MacOS and Linux `here <https://www.raspberrypi.com/software/>`_.

Then insert th SD card into your PC.

Now open the Raspberry Pi Imager and 
- Click “Choose OS” → Scroll down and select “Other general-purpose OS” → Choose Ubuntu.
- Select Ubuntu Server 24.04 (64-bit)
- “Choose Storage” → Select your SD card.

Then when asked to make pre-defined setting:
- activate SSH and generate a key
- setup login + password (vincent.lenz, hippocampus)

Then inserted SD to raspberry Pi, connected via ethernet to local network, connected via HDMI a display and via USB mouse and keyboard.

Steps:
- sudo apt update && sudo apt upgrade -y
- sudo apt install net-tools (damit man ifconfig aufrufen kann)

Change keyboard layout
- sudo apt-get install console-data
- sudo dpkg-reconfigure keyboard-configuration
- select keyboard model: eg. Fujitsu
- select layout: eg. German
- select variant: eg AltGr
- once done: sudo reboot to apply changes

Change Fontsize permanentely:
- sudo nano /etc/default/console-setup
- FONTSIZE="16x32"
- Ctrl + O, Enter, Ctrl + X (to save the file)
- sudo setupcon ( to apply changes)


Deactivate password for sudo for user pi
****************************************
sudo nano /etc/sudoers

add line to the end: pi ALL=(ALL:ALL) NOPASSWD:ALL

save and sudo reboot

Tackling SSH:
*************
- sudo nano /etc/hostname
 - change name to bluerov-02
- sudo nano /etc/hosts
 - change name behind IP address to bluerov-02
 - sudo reboot
Without needing a static IP: Do the follwoing on the Pi:
- sudo apt install avahi-daemon -y
- sudo systemctl enable avahi-daemon
- sudo systemctl start avahi-daemon
- then from your local computer, when both in the same local network, you should reach the pi via ssh: ssh username@hostname.local, here ssh pi@bluerov-02.local

Change username afterwards by creating a temporary user first:
- sudo adduser tempuser
- Set a password and press Enter for default values.
- sudo usermod -aG sudo tempuser (give it sudo permissions)
- su - tempuser (switch to new user)
- sudo usermod -l pi -d /home/pi -m vincent.lenz (to change from old=vincent.lenz to new=pi. It changes the username and home directory)
- sudo groupmod -n pi vincent.lenz (Rename the primary group from vincent.lenz to pi)


Create ssh key on your machine, if not already existing:

ssh-keygen -t rsa -b 4096 -C "your-email@example.com"

hit enter (selecting the default) when asked where to save the key.
If a prompt appears that this file already exist, you already have a ssh key and we advise you to not ovveride the file! Hit "n" and jump to "how to copy the key to the pi".
If you are not confronted with that prompt, you dont have a ssh key yet, so hit enter for every follwing prompt, always selecting the default, then go to how to copy to pi.

If it promt

Copy that key to the pi (your computer and pi must be in the same local network):
ssh-copy-id pi@<raspberry_pi_ip>
here:
ssh-copy-id pi@bluerov-02.local





Da ich bei dem Pi Imager ein SSH erstellt habe, muss ich daraufhin kein Passwort eingeben, sonst müsste man das Passowrt für den Pi eingeben.



Jetzt haben wir das ganze mit dem SSH auch auf dem Pi an Einstellungen vorgenommen. Man kann das auch bevor man die SD Karte in den Pi steckt sondern auf dem lokalen Rechner, die SD Karte sich anschaut und eine entsprechene Datei findet und die Schritte in "Ubuntu 24.04 Server 64bit" ausführt.
Danach wie auch hier dann die ganzen Getting Startet Sachen machen mit Quality-of Life Feature und ROS Installation ... 



Jetzt in der HippoCampus Docs weiter gemacht in Kapitel: Ubuntu 24.04 Server 64bit

In /boot/firmware/config.txt we added the lines:
- dtoverlay=i2c4,pins_6_7
- dtoverlay=uart2
- dtoverlay=uart3
- dtoverlay=uart4
- dtoverlay=uart5

In /etc/needrestart/needrestart.conf we uncomment
- $nrconf{restart} = ...
- $nrconf{kernelhints} = ...
And change those to:
- $nrconf{restart} = 'a';
- $nrconf{kernelhints} = 0

UART und USB Config komplett geskippt. Nur das Ethernet gemacht. By disabling Gigabit mode, the Raspberry Pi will only connect at 100 Mbit/s, preventing unnecessary speed renegotiation and improving connection stability. ethtool eth0 | grep Speed (check the current speed)

 Das sind wohl diese rules files, da mal auf dem klopsi gucken, welche das waren, dann weiß ich auch auf welchem pi ich das ändern muss. Herausfinden, wofür die Pin Belegung ist, und ob ich da was ändern muss/ anpassen muss?

Dann Quality of Life Features Kapitel gemacht. Und anschließend stück für stück jetzt die ganzen "Getting Started" Kapitel durch gearbeitet.

Der check mit turtle sim bringt hier nichts, weil ich keine graphic user interface besitze mit dem pi.

Next: Pre build packages!!!!





GPIO pin / pinout on Raspberry Pi
*********************************
Each GPIO pin has a dedicated function, such as:
- UART TX/RX (serial communication)
- I2C SDA/SCL (for sensors, some controllers)
- SPI MOSI/MISO/SCK (for fast serial peripherals)
- PWM (for motor controllers, servos)
- General-purpose GPIO (can be used as digital inputs/outputs)

I2C 
***
another type of protocol.


pinout
******
The raspberry Pi 5 has a 40-pin GPIO header mapping out how the pins are assigned to different functionalities.
cross-reference between the contacts, or pins, of an electrical connector or electronic component, and their functions
**bluerov-02-main**
- power pins
 - 5V (Pins 2, 4): Direct power supply, useful for powering external devices.
 - 3.3V (Pins 1, 17): Regulated 3.3V output.
 - Ground (GND) (Pins 6, 9, 14, 20, 25, 30, 34, 39): Common ground connections.
- GPIO Pins (General Purpose Input/Output)
 - Can be programmed as inputs or outputs.
 - Some are assigned to specific functions, like UART, I2C, SPI, or PWM.
- UART (Universal Asynchronous Receiver/Transmitter)
 - Used for serial communication.
 - GPIO 14 (TX0) (Pin 8) and GPIO 15 (RX0) (Pin 10) are the default serial interface.
 - GPIO 0 (TX2) (Pin 27) and GPIO 1 (RX2) (Pin 28) for Teensy.
 - GPIO 8 (TX4) (Pin 24) and GPIO 9 (RX4) (Pin 21) for FCU Debug.
 - GPIO 12 (TX5) (Pin 32) and GPIO 13 (RX5) (Pin 33) for FCU Telemetry (TL2).
- I2C (Inter-Integrated Circuit):
 - Used for communicating with sensors (e.g., barometer).
 - GPIO 2 (SDA) (Pin 3) and GPIO 3 (SCL) (Pin 5).
- SPI (Serial Peripheral Interface)
 - Not used in this setup but available on specific GPIOs.
- PWM (Pulse Width Modulation):
 - GPIO 20 (Pin 38) for PWM Camera Servo.
 - GPIO 21 (Pin 40) for PWM Lights.

By default, the Raspberry Pi uses UART0 (GPIO 14 & 15) for the console/login shell.
I2C (For Sensors Like the Barometer). Needs to be enabled via: sudo raspi-config. Go to Interfacing Options > I2C and enable it.
Raspberry Pi 5 supports hardware PWM, but you might need to enable it via: sudo dtoverlay=pwm-2chan. GPIO 20 & 21 are likely assigned as PWM outputs, but double-check with: gpio readall
If using multiple UARTs, you might need to enable extra UARTs in /boot/config.txt: dtoverlay=uart2
dtoverlay=uart4
dtoverlay=uart5

Make sure to match voltage levels of peripherals to 3.3V logic (Raspberry Pi GPIOs are not 5V tolerant).

If you are using heartbeat LEDs, make sure to set the corresponding GPIOs as outputs in your script:
import RPi.GPIO as GPIO

GPIO.setmode(GPIO.BCM)
GPIO.setup(11, GPIO.OUT)  # Signal Heartbeat LED
GPIO.output(11, GPIO.HIGH)  # Turn it on

If using Teensy for communication, ensure its baud rate matches the Raspberry Pi UART configuration.


PWM
***
Run: ls /sys/class/pwm/. If it shows something like pwmchip0, then PWM is enabled. If nothing appears, you may need to enable hardware PWM in /boot/config.txt: sudo nano /boot/config.txt
Add this line at the bottom: dtoverlay=pwm-2chan
Then save (CTRL+X, Y, Enter) and reboot: sudo reboot






full duplex vs half duplex communication
****************************************
- full: receive and send data between to devices simultaneously
- half: can only send or receive but not both at the same time 


Battery Management System (BMS)
*******************************
- monitors individual cell voltages and can cut off power when the voltage drops too low.
- should be paired with a safe shutdown script to avoid abrupt shutdown
Example:
- Use a voltage divider circuit or an INA219 power monitor to measure battery voltage via the Pi’s GPIO.
- Run a Python script that continuously checks voltage levels and executes sudo shutdown -h now when a low threshold is reached.
- Optionally, add a Supercapacitor or small backup battery to keep the Pi running for a few seconds after main power is cut.
Combine:
- A BMS or a low-voltage cutoff relay for battery protection.
- A voltage monitoring circuit (e.g., INA219) connected to the Raspberry Pi.
- A shutdown script that detects low voltage and powers down the system safely before the cutoff is triggered.
