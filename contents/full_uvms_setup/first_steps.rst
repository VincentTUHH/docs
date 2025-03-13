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

UART und USB Config komplett geskippt. Nur das Ethernet gemacht. Das sind wohl diese rules files, da mal auf dem klopsi gucken, welche das waren, dann weiß ich auch auf welchem pi ich das ändern muss. Herausfinden, wofür die Pin Belegung ist, und ob ich da was ändern muss/ anpassen muss?

Dann Quality of Life Features Kapitel gemacht. Und anschließend stück für stück jetzt die ganzen "Getting Started" Kapitel durch gearbeitet.

Der check mit turtle sim bringt hier nichts, weil ich keine graphic user interface besitze mit dem pi.

Next: Pre build packages!!!!