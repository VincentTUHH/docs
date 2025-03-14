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


Reach Alpha 5 Arm
******************
serial-to-USB adapter, then just connecting it via USB to the Raspberry Pi should be enough to establish communication. The Raspberry Pi will automatically recognize it as a serial device.

If only one USB-to-serial device is connected, it will usually be /dev/ttyUSB0.


UART
****

UART is an asynchronous serial communication protocol, meaning that it takes bytes of data and transmits the individual bits in a sequential fashion.
Asynchronous transmission allows data to be transmitted without the sender having to send a clock signal to the receiver. Instead, the sender and receiver agree on timing parameters in advance and special bits called 'start bits' are added to each word and used to synchronize the sending and receiving units.
On a Raspberry Pi conveniently controlled over GPIO's (General Purpose Input/Output)

UART is very simple and only uses two wires between transmitter and receiver to transmit and receive in both directions.

UARTs (Universal Asynchronous Receiver-Transmitters) are serial communication interfaces. The Raspberry Pi has multiple UARTs, each mapped to specific GPIO pins. The UART number (e.g., UART2, UART3, etc.) is significant because it determines:
	1.	Which GPIO pins are used for Tx (Transmit) and Rx (Receive).
	2.	How the Linux kernel identifies the UART device (e.g., /dev/serial1).

Each UART corresponds to specific GPIO pins.
	•	UART0 → Primary UART, but often assigned to Bluetooth on some Raspberry Pi models.
	•	UART1 → Mini UART (not full-featured, clock-dependent).
	•	UART2 - UART5 → Secondary hardware UARTs, mapped to specific GPIO pins.

The Linux kernel assigns different device names based on the UART number. UART5 will show up as something like /dev/ttyAMA2, /dev/serial2, etc.

For UARTs, the kernel (kernel is the core of the operating system that directly interacts with the hardware) assigns device names (e.g., /dev/serial0, /dev/ttyAMA0, etc.), so software can communicate with them.

TX and RX are always defined from the perspective of the Raspberry Pi. For example, if the guide says “UART5 TX is on GPIO12,” that means:
- GPIO12 on the Raspberry Pi sends data (TX).
- GPIO13 on the Raspberry Pi receives data (RX).
So if you connect a PX4, you must:
- Connect Pi’s TX (GPIO12) to PX4’s RX.
- Connect Pi’s RX (GPIO13) to PX4’s TX.

Yes, the Raspberry Pi’s UARTs are generally mapped to fixed GPIO pins unless changed in the device tree overlay (dtoverlay in config.txt).

fe201400.serial refers to the hardware address of UART2. These addresses tell the kernel where each UART physically exists in the Pi’s memory. When you enable dtoverlay=uart5, the kernel knows that UART5 corresponds to fe201a00.serial and assigns a device like /dev/ttyAMA2. /dev/serial0, /dev/serial1 → Aliases that point to actual UART devices.

1.	UARTs Must Be Enabled First
 - By default, some UARTs on the Raspberry Pi are disabled to free up GPIOs for other functions.
 - You must enable UARTs in /boot/firmware/config.txt using:
 - dtoverlay=uart5
 - This tells the Linux kernel to activate UART5.
2.	Once Enabled, the Kernel Recognizes the UART
 - After rebooting, the kernel will now list the active UART as a device.
 - You can check which UARTs are available by running:
 - ls -l /dev/serial*
 - If UART5 is enabled, you should see a device like: /dev/ttyAMA1 -> fe201a00.serial
 - This confirms that UART5 (physically mapped to fe201a00.serial) is now accessible via /dev/ttyAMA1.
3. If you physically connect a device (like PX4) to GPIO12 (TX) and GPIO13 (RX) on the Raspberry Pi, it will receive data on /dev/ttyAMA1.
4. Any program (Python, C++, ROS 2 nodes, etc.) can open /dev/ttyAMA1 to send/receive data.
5. If dtoverlay=uart5 is missing or disabled in config.txt, /dev/ttyAMA1 won’t exist, and no program can use it.
6. The presence of /dev/ttyAMA1 does not mean something is connected. It only means that the UART5 hardware is available for use.
7. How to Check if a Device is Connected?: cat /dev/ttyAMA1
8. Loopback Test (Check if UART is Working Without a Device)
 - If you want to test without another device, you can connect TX and RX together (GPIO12 to GPIO13 for UART5):
 - run: 
   echo "Hello" > /dev/ttyAMA1
   cat /dev/ttyAMA1
 - If it prints “Hello,” the UART is working.


Understanding the UART Rule (/etc/udev/rules.d/50-serial.rules)
***************************************************************
This section in the tutorial configures udev rules, which:
- Assign user permissions (GROUP="dialout").
- Create symlinks (e.g., /dev/fcu_data instead of /dev/ttyAMA1).
- Ensure that UART devices always have the same consistent names.
- Instead of referring to UART5 as /dev/ttyAMA1, it will be accessible as /dev/fcu_data.
- This avoids issues where device names (e.g., /dev/ttyAMA1) might change across reboots.
- Programs can use /dev/fcu_data or /dev/fcu_debug instead of /dev/ttyAMA*. (especially as the number of ttyAMA* changes depending on how many UART's are active and is not linked to the number of the UART)
- Verify everything works: ls -l /dev/fcu_* (must see all symlinks)

udev does more than just create symlinks for UART devices. It is a device manager for the Linux kernel that handles hardware detection, naming, and access permissions dynamically. 

GROUP="dialout" in your rules allows non-root users to access serial ports.

Triggers Scripts or Commands When a Device is Detected
- You can configure udev to run a script automatically when a device is plugged in.
- Example: If PX4 is connected via UART5, udev could start a logging service.

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

Teensy Baudrate
***************
The baud rate refers to the speed of serial communication, measured in bits per second (bps). It determines how fast data is transmitted between the Raspberry Pi and the Teensy microcontroller over the UART interface. Both the Raspberry Pi and Teensy must use the same baud rate; otherwise, communication will be unreliable or fail completely. For example, if the Teensy is configured to transmit at 115200 bps but the Raspberry Pi expects 9600 bps, the received data will be garbled or lost.




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
