Short Manual 
############

.. attention::

   This manual is intended for a Raspberry Pi 4. Most instructions most likely also apply to older or newer versions, but everything it is not guaranteed at all. Changes might be neccessary to be make.

   To control the brightness of the spotlights and the servo motor of the front camera (both run on the main) software PWM is used. The :code:`pigpio` library is utilized to do so. However this library only supports Pi 4 hardware!

.. note:: Naming Convention Pi's

   The underwater vehicle has two tubes, a top and bottom tube. In this workgroup we call the Raspberry Pi in the top tube **main** and in the bottom tube **buddy**.


Flash SD Card 
=============

.. attention::

   For a Raspberry Pi 5, Ubuntu 24.04 or newer is required. The Pi 5 needs a more recent kernel and firmware, as well as a compatible boot partition, which are not supported by Ubuntu 22.04 or earlier. If an unsupported image is used, the Pi will not boot, which is indicated by a constantly lit green LED.

To flash the SD card with Ubuntu 24.04. it is recommended to use the Raspberry Pi Imager. A download is available for Windows, MacOS and Linux `here <https://www.raspberrypi.com/software/>`_.

Insert the SD card into your computer and open the Raspberry Pi Imager. A window pops up were you need to make some settings:

- **Choose OS**:
   #. *Other general-purpose OS*
   #. *Ubuntu*
   #. *Ubuntu Server 24.04 (64-bit)*

- **Choose Storage**: select your SD card

- **pre-defined settings**:
   #. activate SSh key and generate a key
   #. login + password 
   - or select no pre-defined settings

Modify Cloud-Init
*****************

Modify :code:`user-data` on the `system-boot` partition to your liking. The SD card must be inserted into your computer. For MacOS users you access the partion from terminal:

.. code-block::

   cd /Volumes/system-boot && \
   sudo nano user-data

An example configuration is provided below. Things you should alter:
- *hostname* and *name*: Keep :code:`name: pi`. Relevant for deploying a ssh connection: :code:`ssh name@hostname.local`
- *password*: Is later used to login onto the pi
- *ssh_authorized_keys*: If you already have an ssh key paste it here, otherwise wait for the steps below. Delete the existing ssh keys, if you don't know to whom they belong to

.. code-block:: yaml
   :caption: user-data

   #cloud-config
   
   hostname: hippo-main-04
   manage_etc_hosts: true
   locale: en_US.UTF-8
   timezone: Europe/Berlin
   
   users:
       - name: pi
         groups: sudo, dialout, video
         sudo: "ALL=(ALL) NOPASSWD:ALL"
         # false -> allow password login
         lock_passwd: false
         shell: /bin/bash
         plain_text_passwd: "hippocampus"
         ssh_authorized_keys:
         - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIF0xs6V9Lp2xzh/+hs+S919KpwAj9VHWO5NeHEuTYTpQ thies.lennart.alff@tuhh.de
         - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDXZYQ3V6tmAQ7RhfCkmy9406FmKKgAVF0qDkeDJ7XroyYC10kLyEOWXFMbemqrQU3SHJ2ly0ujizZuJLtEYY60dWBCzIG1jHenPX7az0Vf/0Mj5m7lyttt/81RblfLzIDrFlFmdY/GI7fBB9IaMiFNQfTEV2IP7SsGQrrB8Ki7FkoHKZvV9iVn7+taO+g/MLs17JTtZICkTGZpzgxJfQepZMNt2/D5G7eo5YeTUnGT5SxidTxkhjcVqepdriDfy1lBZ7j/bD4tXh17024Pj/GI3gAO+CU67/mAvujaoDm2/LmiRkNvPYsSF1wyDVhfAu+Wdk/g/pbOAuas6boiQvPAM64I7cdjr2yOsmEJoADfaO2fxQHCwR7eW+gz2vY35XDdaAHfxGqTlLAvmS2rMjvKRPbC6chCyfozoBVQq2jZ/vq3IqqzLDt8Mb7LyiLm0ohxnD6o/8AbAOPL1un2mFm3kVYfIKuP00YrcBb4lDUDdE1YoxFlEeqGiTxsUIoiW0hyWmVRQa6aHpZte8AMoPev2JKk/WRLzcj5Pyf7/UG4zO7XrSb6baZ+r+Kqq0oHoE9zCG8On44vO751IWxmLeCXxliDbORhS12Ke+kMDzUz1gz+4wi38uJCcAKqzTSeNuchGmvTdoRGCNHGcqjFSGNfDVDdwOi+1+59fCTIgN9Ldw== lennartalff-yubikey
         - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAID1jzFjfirr762nLh2CAwCoyrjfIezMH4it5nd4by+Bn nathalie_bauschmann@hotmail.de
   
   # allow password ssh login
   ssh_pwauth: true

   package_update: true
   package_upgrade: true
   packages:
       - avahi-daemon

   power_state: 
       mode: reboot
       delay: now
       condition: True






General Setup on Pi 
===================

Let's start the pi for the first time. First insert all devices for powering it on: insert the SD card, connect keyboard and mouse via USB, connect a Display via HDMI and connect the pi via Ethernet to the local network. Now power the pi with a USB-C charging cable. 

.. note::

   A typical 5A smartphone charger is most likely to weak to power the pi.

Deactivate Password for :code:`sudo` for root User 
**************************************************

.. note::

   This manual always uses :code:`nano` as text editor. When asked to make changes or create a new file you are expected to always save it. Press :code:`Strg-X` - :code:`Y` - :code:`Enter` to save the file and get back to the terminal.

.. code-block:: console

   sudo nano /etc/sudoers

Add the following line to the end:

.. code-block:: sh

   pi ALL=(ALL:ALL) NOPASSWD:ALL

Rebbot to apply the changes:

.. code-block:: console

   sudo reboot

Now follow the steps:

.. code-block:: console

   sudo apt update && sudo apt upgrade -y && \
   sudo apt install net-tools

Change Keyboard Layout 
**********************

.. code-block:: console

   sudo apt-get install console-data && \
   sudo dpkg-reconfigure keyboard-configuration

According to your keyboard select for instance :code:`Fujitsu`, :code:`German`, :code:`AltGr` and apply changes:

.. code-block:: console

   sudo reboot

Change Font Size permanentely 
*****************************

.. code-block:: console

   sudo nano /etc/default/console-setup

Change the follwoing line and save the file:

.. code-block:: sh

   FONTSIZE="16x32"

You need to apply the changes:

.. code-block:: console

   sudo setupcon

Change Name of Pi
*****************

.. note::

   These steps are not neccessary when the **Cloud-Init** has already been modified.

.. code-block:: console

   sudo nano /etc/hostname

and change the hostname to :code:`bluerov-02-main`

.. code-block:: console

   sudo nano /etc/hosts

and change the name after IP address to :code:`bluerov-02-main`. :code:`reboot` your system.

In addition do that, to avoid needing a static IP:

.. code-block:: console

   sudo apt install avahi-daemon -y && \
   sudo systemctl enable avahi-daemon && \
   sudo systemctl start avahi-daemon

To adapt the username you first have to create a temporary user:

.. code-block:: console

   sudo adduser tempuser

Set a password and press :code:`Enter` to select all default settings.

.. code-block:: console

   sudo usermod -aG sudo tempuser

Now switch to the new user from where you can change the username to *pi*. You need to adapt :code:`old_user` accordingly to your system.

.. code-block:: console

   su - tempuser

.. code-block:: console

   sudo usermod -l pi -d /home/pi -m old_user


You can now reach the pi via SSH by *ssh username@hostname.local*, so in this case *ssh pi@bluerov-02-main.local*. When you first add your SSH key, you don't have to enter a password to verify yourself.



SSH Verification
****************

Create a SSH-key on your own computer. Open a terminal and type the following. Don't forget to replace :code:`your-email@example.com`. This can be any of your email, that simply adds another level of security.

.. code-block:: console

   ssh-keygen -t rsa -b 4096 -C "your-email@example.com"

You are prompted among other things to select a file location. Always press :code:`Enter` to select the default.

.. attention::

   When you already have a SSH key, skip this. At last you will notice that you already have a key, when a prompt tells you that *this file already exists*. We advise you to not override your existing SSH-key, so press :code:`N`.

Now you need to copy your SSH key to the pi. Both, your computer and the pi must be in the same local network, then you do it from a terminal of your computer with *ssh-copy-id pi@<raspberry_pi_ip>*, eg.:

.. code-block:: console

   ssh-copy-id pi@bluerov-02.local


















Quality-of-Life Features
========================

Before we start, let's install some quality-of-life features to improve the user experience.

.. code-block:: console

   $ sudo apt install -y zsh git curl wget byobu vim\
   && sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

Choose :code:`zsh` as default by hitting enter.

.. code-block:: console

   $ mkdir -p "$HOME/.zsh" && \
   git clone https://github.com/sindresorhus/pure.git "$HOME/.zsh/pure" && \
   echo 'fpath+=$HOME/.zsh/pure \nautoload -U promptinit; promptinit \nprompt pure' | cat - ~/.zshrc > temp && mv temp ~/.zshrc && \
   sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME=""/' ~/.zshrc && \
   git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions && \
   sed -i 's/plugins=(git)/plugins=(git zsh-autosuggestions)/' ~/.zshrc && \
   echo "zstyle ':prompt:pure:path' color 075\nzstyle ':prompt:pure:prompt:success' color 214\nzstyle ':prompt:pure:user' color 119\nzstyle ':prompt:pure:host' color 119\nZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=161'" >> ~/.zshrc && \
   echo "zstyle ':prompt:pure:path' color 075\nzstyle ':prompt:pure:prompt:success' color 214\nzstyle ':prompt:pure:user' color 119\nzstyle ':prompt:pure:host' color 119\nZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=161'" >> ~/.zshrc && \
   echo "export TERM=xterm-256color" >> ~/.zshrc && \
   source ~/.zshrc && \
   byobu-enable












Hardware Features 
=================


Pinout Raspberry Pi 
*******************

.. note::

   This pinout is the same for a Raspberry Pi 4 and Raspberry Pi 5.

The following provides an overview on the pinout used for the main and buddy Pi.

To-Do: Bild davon wie herum man die Pinbelegung verstehen muss.


.. tabs::         

   .. tab:: BlueROV (main)

      .. image:: /res/images/pi_pinout_bluerov.svg
         :width: 30000
         :align: center

   .. tab:: BlueROV (buddy)

      .. todo:: Create the pinout.



GPIO Settings 
*************

We need to make some hardware changes to the pi, thus we enable specific functinoalities on the GPIO pins.

Open the :code:`config.txt` file:

.. code-block:: console

   sudo nano /boot/firmware/config.txt

Comment out :code:`dtparam=spi=on` and add the lines below an :code:`[all]`: section:

.. note:: 

   Those tree overlay settings are in accordance to the pinout of main and buddy and which devices are connected to which pins for serial communictaion (uart) for instance.

.. tabs::

   .. code-tab:: console main

      dtoverlay=i2c4,pins_6_7

      dtoverlay=uart2
      dtoverlay=uart3
      dtoverlay=uart4
      dtoverlay=uart5

      dtoverlay=gpio-led,gpio=11,label=led11-heartbeat,trigger=heartbeat

      dtoverlay=pwm,pin=20,func=4
      dtoverlay=pwm,pin=21,func=4

      TO-DO correct the pins ans uart that I actually use!!!!!
      AND inform the reader about the pinout that is relevnt here

   .. code-tab:: console buddy
      
      To-Do !!!!!!!!!!!!!!!!!   

Save and apply changes with :code:`reboot`.

.. note:: GPIO Meanings

   **dtoverlay=i2c4,pins_6_7**: adds an additional I2C bus 4 (I2C4) and assigns it to :code:`GPIO 6` (SDA) and :code:`GPIO 7` (SCL). SDA stands for data line, SCL for clock line. In additioin to the SDA and SCL pins, the I2C device must also be connected to any GND and VCC (3.3V/5V) pin.

   **dtoverlay=uart2**: activates all UART2 that vill be used as serial connection.

   **dtoverlay=gpio-led,gpio=11,label=led11-heartbeat,trigger=heartbeat**: configures :code:`GPIO pin 11` as an LED controlled by the Linux system. The :code:`trigger=heartbeat` setting means that the LED will blink to indicate system activity (like a heartbeat). This setup is used for status indication. Once the Raspberry Pi boots, it will automatically start blinking the LED according to the system heartbeat.

   **dtoverlay=pwm,pin=20,func=4**: assigns a PWM signal to GPIO20.

UART
****

Each UART on the Raspberry Pi 5 is mapped to specific GPIO pins and a corresponding hardware address, which is a unique identifier assigned to each UART inside the Raspberry Pi's system memory. This mapping is standard and cannot be changed.

==== ========================== =====================
UART KERNELS / HARDWARE ADDRESS Tx/Rx GPIOs          
==== ========================== =====================
0                               :code:`GPIO14/GPIO15`
1    --                         --                   
2    :code:`fe201400.serial`    :code:`GPIO0/GPIO1`  
3    :code:`fe201600.serial`    :code:`GPIO4/GPIO5`  
4    :code:`fe201800.serial`    :code:`GPIO8/GPIO9`  
5    :code:`fe201a00.serial`    :code:`GPIO12/GPIO13`
==== ========================== =====================

The Linux kernal then creates a device entry :code:`/dev/ttyAMA*` for every active UART.
Those device entries are the serial device names that programs use to send and receive data. However, the specific :code:`/dev/ttyAMA*` assignment may change across reboots and if multiple UARTs are acitavted, leading to inconsistencies.

To avoid this issue, we define a fixed device name using a udev rule. This ensures that the UART is always accessible under a predictable name, regardless of how :code:`/dev/ttyAMA*` numbers are assigned.

.. note::

   TX and RX are defined from the perspective of the Raspberry Pi. Example for *UART5*:

   - *TX (GPIO12)* → *RX of the external device*
   - *RX (GPIO13)* → *TX of the external device*
   - *GND* must also be connected between devices.

UDEV Rules - Persistent :code:`/dev` names 
******************************************

The *udev* system dynamically manages device nodes in Linux. By default, it applies standard rules that determine permissions, naming, and symbolic links for connected devices. Custom udev rules allow assigning consistent names to devices, ensuring that :code:`/dev/ttyAMA*` numbers remain predictable across reboots, as device names like :code:`/dev/ttyAMA2` may change on reboot. Namin the rule's file is important as higher-numbered rules (e.g., 99-custom.rules) are applied last, overriding lower-numbered default rules. To assign a persistent name create a new file (you are free to change the name):

.. code-block:: console

   sudo nano /etc/udev/rules.d/99-serial.rules

.. tabs::

   .. tab:: main

      .. code-block:: sh

         KERNEL=="ttyAMA[0-9]*", GROUP="dialout", ENV{SERIAL_MARKER}="serial_marker"

         # uart2
         ENV{SERIAL_MARKER}=="serial_marker",  SUBSYSTEM=="tty", KERNELS=="fe201400.serial", SYMLINK+="teensy_data"
         # uart4
         ENV{SERIAL_MARKER}=="serial_marker",  SUBSYSTEM=="tty", KERNELS=="fe201800.serial", SYMLINK+="fcu_debug"
         # uart5
         ENV{SERIAL_MARKER}=="serial_marker",  SUBSYSTEM=="tty", KERNELS=="fe201a00.serial", SYMLINK+="fcu_data"

      For instance the teensy is now callable with :code:`/dev/teensy_data`.

   .. code-tab:: sh buddy
      
      To-Do !!!!!!!!!!!!!!!!!   

To-D0: what about these other udev rules that are on the main for hindering that something mistakens the teensy or so for a modem or so...

Save and apply new rules:

.. code-block:: console

   sudo udevadm control --reload-rules && \
   sudo udevadm trigger

You should see the new device names when testing:

.. code-block:: console

   ls /dev/* -l


USB Configuration and UDEV Rules 
********************************

.. note::

   For now not relevant, as only one device is connected to each pi respectively (so both will be called /ttyUSB0). The second USB on the main is a camera device, hence it will not interfere with the USB* naming.

.. As for UART, we can also set udev rules for USB connections. We use that for the Reach Alpha Arm, where a serial (RS232) to USB adapter is used. We want to ensure the Reach Alpha 5 Arm always appears as the same device, no matter which USB port we use. For that we must define a rule including the manipulator's **serial number (:code:`ATTRS{serial}`)**. This number is unique to the device itself, not the ports on the raspberry pi. 

.. Want do this for:
.. - **main**: Pixhawk 6c | front camera
.. - **buddy**: Reach Alpha Arm (reach_alpha)

.. ===== ================= ============
.. pi    device            symlink     
.. ===== ================= ============
.. main  Pixhawk 6c        pixhawk     
.. main  front camera      front_camera
.. buddy Reach Alpha 5 Arm reach_alpha 
.. ===== ================= ============

.. .. attention:: Wo Pixhawk 

..    Wo wird die Adresse vom Pixhawk und Front camera benutzt, dass man das auch ändert?

.. Disconnect all USB devices but the one we want to set up. Now list all USB devices:

.. .. code-block:: sh

..    ls /dev/ttyUSB*

.. Unplug your device, repeat the listing and note which device identifier belongs to your device, eg. :code:`/dev/ttyUSB0`. Once connected again do according to the device identifier:

.. .. code-block:: console

..    udevadm info -a -n /dev/ttyUSB0 | grep -E "idVendor|idProduct|serial"

.. You look for the following three things, they shoud look like:

.. .. code-block:: console 

..    ATTRS{idVendor}=="2341"
..    ATTRS{idProduct}=="0042"
..    ATTRS{serial}=="A1B2C3D4"

.. Repeat those steps for all your USB devices.

.. Now we assign persistent names for which we create a new file:

.. .. code-block:: console

..    sudo nano /etc/udev/rules.d/99-usb.rules

.. Add the following line for each device (2 devices when setting up the main and 1 device when setting up the buddy), but **REPLACE** *idVendor*, *idProduct*, and *serial* with your values and the symlink names according to the tabel above:

.. .. code-block:: sh

..    SUBSYSTEM=="tty", ATTRS{idVendor}=="2341", ATTRS{idProduct}=="0042", ATTRS{serial}=="A1B2C3D4", SYMLINK+="reach_alpha", MODE="0666"

.. Save and apply new rules:

.. .. code-block:: console

..    sudo udevadm control --reload-rules && \
..    sudo udevadm trigger


.. note:

..    Maybe I have to add the modem blocker here as well for the fcu or other objects connected, when I notice, those are blocked (see comments.rst)



Ethernet 
********

.. attention::

   Sometimes the ethernet interface of the Raspberry Pi is lagging. We have not ultimately identified the cause, but the following steps might help with that.

.. note::

   The Bluerobotics Switch is a 100Mbit/s switch. So there is no need to configure the connection to be 100Mbit/s only, because it is 100Mbit/s anyway.

Most likely the Gigabit connection via our manually crimped RJ45 connectors is not stable. Hence, the connection speed (100 Mbit/s vs Gbit) is repetitively negotiated which results in an unstable connection. We try to avoid this by manually setting the advertised link mode to 100 MBit/s.

.. code-block:: console

   sudo nano /etc/network/if-pre-up.d/eththool

Add the line :code:`$ETHTOOL --change eth0 advertise 0x008` to look like:

.. code-block:: sh

   TO-DO Wie sieht dieses File denn nun aus?!?!?!?!?



PWM devices 
***********

.. attention:: pigpio library 

   The pigpio library works solely for the Raspberry Pi 4 hardware. For teh Raspberry Pi 5 a different library like `rpi_hardware_pwm <https://pypi.org/project/rpi-hardware-pwm/>`_ is neccessary. Code in ... servo and spotlights github files, must be altered.
   To-Do obiges anpassen









Automated Restart Settings 
==========================
Open :code:`needrestart.conf`:

.. code-block:: console

   sudo nano /etc/needrestart/needrestart.conf

We want to change those settings to automatically restart services after package upgrades and detect neccessyra rebbots de to kernal upgrades. Find the following commands in the file, uncomment and modify the value accordingly to match:

.. code-block::sh

   $nrconf{restart} = 'a';
   $nrconf{kernelhints} = 0;









ROS Installation 
================

.. _ros-installation:

ROS Installation
################

.. attention::
   We are moving to ROS2. This guide and the following pages are still in the process of being updated. There might be some construction works still going on here and there.

.. note::
   This guide assumes `Ubuntu 24.04 <https://releases.ubuntu.com/24.04/>`_ is used as OS.


We use ROS2 Jazzy.
The following installations steps work for the Ubuntu 24.04 arm64 server image for the Raspberry Pi.

Preparation
***********

#. Make sure you have a UTF-8 supported locale with
   
   .. code-block:: console
      
      $ locale
   
   If not, refer to the `ROS documentation <https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debians.html#set-locale>`__.

#. Enable universe repository
   
   .. code-block:: console
      
      $ sudo apt install software-properties-common \
      && sudo add-apt-repository universe

#. Add the key

   .. code-block:: console

      $ sudo apt update && sudo apt install curl -y \
      && sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

#. Add sources

   .. code-block:: console

      $ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null




#. Update

   .. warning:: This is critial!
   

   .. code-block:: console

      $ sudo apt update && sudo apt upgrade -y

Installation
************

Use the more lightweight installation for Raspberry Pis. 

#. Install ROS

   .. code-block:: console

      sudo apt install ros-jazzy-perception

#. Install development tools

   .. code-block:: console

      $ sudo apt install ros-dev-tools python3-pip

rosdep Initialization
*********************

.. code-block:: console

   $ sudo rosdep init && rosdep update

.. note:: Do **not** execute :code:`rosdep update` with root privileges. This would lead to permission issues.

Source the ROS Setup
********************

.. code-block:: console

   $ echo 'source /opt/ros/jazzy/setup.zsh' >> ~/.zshrc \
   && . ~/.zshrc



























Pre-built packages
==================

For convenience, we provide our own packages as pre-built binaries at `repositories.hippocampus-robotics.net <repositories.hippocampus-robotics.net>`__.
The advantage is that we do **not** need to compile the packages we are not developing actively ourselfs.

.. note::

   Packages in a local workspace have a higher priority than the ones installed in :file:`/opt/ros/` via debian packages.
   So we can install all our packages via `apt install` and still use a custom/modifed version of them by having them included in our workspace.

.. note:: 

   In case that we do not wish to install the pre-built binaries, it is perfectly fine to skip this part and clone all the required packages into our underlay workspace and build them on our own.

List of pre-build packages (as of 27.03.2025)
*********************************************

Those summarize all packages that will be loaded from the :code:`hippo_robot` binary, when looking for the dependencies.

- interface packages
   - acoustic_msgs
   - alpha_msgs
   - buttons_msgs
   - dvl_msgs
   - gantry_msgs
   - hippo_control_msgs
   - hippo_msgs
   - px4_msgs
   - rapid_trajectories_msgs
   - state_estimation_msgs
   - uvms_msgs

- all other packages
   - dvl
   - esc
   - gantry
   - hardware
   - hippo_common
   - hippo_control
   - mjpeg_cam
   - path_planning
   - qualisys_bridge
   - remote_control
   - visual_localization



Add Sources
***********

#. Adding the key

   .. code-block:: console

      $ sudo curl https://repositories.hippocampus-robotics.net/hippo-archive.key -o /etc/apt/keyrings/hippocampus-robotics.asc

#. Adding the sources

   .. code-block:: console

      $ echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/hippocampus-robotics.asc] https://repositories.hippocampus-robotics.net/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/hippocampus.list

#. Updating ``apt``

   .. code-block:: console

      $ sudo apt update
   

``rosdep``
**********

#. Add keys for ``rosdep`` so it knows that our packages can be resolved via ``apt install ros-${ROS_DISTRO}-<pkg-name>``.

   .. attention::

      Make sure that ``ROS_DISTRO`` is set to the installed ROS version.
      If you followed the previous section, this should already be done.
      It can be checked by

      .. code-block:: console

         $ echo $ROS_DISTRO
         jazzy # (or whatever ROS version is used.)

      If it not set, it can be set manually by

      .. code-block:: console

         $ export ROS_DISTRO=jazzy
      
   .. code-block:: console

      $ echo "yaml https://raw.githubusercontent.com/HippoCampusRobotics/hippo_infrastructure/main/rosdep-${ROS_DISTRO}.yaml" | sudo tee /etc/ros/rosdep/sources.list.d/50-hippocampus-packages.list

#. Apply the changes

   .. code-block:: console

      $ rosdep update

Installation
************

On Raspberry Pis (deployed with the Ubuntu server image), the ``hippo_robot`` package without graphical dependencies is what we are going for

.. code-tab:: console

   $ sudo apt install ros-${ROS_DISTRO}-hippo-robot





































Create ROS Workspaces 
=====================

.. note::
   This guide assumes `Ubuntu 24.04 <https://releases.ubuntu.com/24.04/>`_ is used as OS. We use ROS2 `jazzy <https://docs.ros.org/en/jazzy/index.html>`_.

We need ROS workspaces because they provide a structured place to develop, build, and test our own packages without affecting the system-wide ROS installation. A workspace is basically a large directory where you keep your own code, along with any third-party packages that aren't available as pre-built binaries and need to be built from source.

We use two types of workspaces: an underlay and an overlay.
The **underlay** contains core packages that provide essential functionality for the system. You don’t usually modify these packages — they form the foundation for your work.
The **overlay** is where your current project lives. It includes the packages you are actively developing and updating. This workspace is built on top of the underlay and depends on it.

.. code-block:: console

   mkdir -p ~/ros2/src &&  \
   mkdir -p ~/ros2_underlay/src


Getting the external packages
-----------------------------

AprilTag-ROS
************

We currently use a slightly adapted version of the ROS2 port of the :code:`apriltag_ros` package by `Christian Rauch <https://github.com/christianrauch/apriltag_ros>`__.

Download our version:

.. tabs::

   .. code-tab:: console ssh

      $ cd ~/ros2_underlay/src \
      && git clone -b hippo git@github.com:HippoCampusRobotics/apriltag_ros.git

   .. code-tab:: console https
      
      $ cd ~/ros2_underlay/src \
      && git clone -b hippo https://github.com/HippoCampusRobotics/apriltag_ros.git




Building the Workspaces
-----------------------

With :code:`colcon`, the new build tool for ROS2, you cannot build your custom workspace when it is sourced.
This would mean that you either cannot source your workspace in :file:`.zshrc`, or you have to manually make sure to run the build command in an environment where you only source workspaces outside the workspace you want to build. 

Since this is very tedious, we define some aliases. Add these lines into your :file:`.zshrc` by executing the following in the terminal:

.. code:: console

   echo "alias build_ros=\"env -i HOME=\$HOME USER=\$USER TERM=xterm-256color bash -l -c 'source \$HOME/ros2_underlay/install/setup.bash && cd \$HOME/ros2 && colcon build --symlink-install --cmake-args -DCMAKE_EXPORT_COMPILE_COMMANDS=ON'\"" >> ~/.zshrc
   source ~/.zshrc
   echo "alias build_underlay=\"env -i HOME=\$HOME USER=\$USER TERM=xterm-256color bash -l -c 'source /opt/ros/jazzy/setup.bash && cd \$HOME/ros2_underlay && colcon build'\"" >> ~/.zshrc
   source ~/.zshrc
   echo "alias rosdep-ros2=\"env -i HOME=$HOME USER=$USER TERM=xterm-256color bash -l -c 'source $HOME/ros2_underlay/install/setup.bash && cd $HOME/ros2 && rosdep install --from-paths src -y --ignore-src'\"" >> ~/.zshrc
   source ~/.zshrc
   echo "alias rosdep-underlay=\"env -i HOME=$HOME USER=$USER TERM=xterm-256color bash -l -c 'source /opt/ros/jazzy/setup.bash && cd $HOME/ros2_underlay && rosdep install --from-paths src -y --ignore-src'\"" >> ~/.zshrc
   source ~/.zshrc

.. note::

   :code:`echo <content> >> ~/.zshrc` adds the *content* to the end of the file that is at *~/.zshrc*.

.. important::

   Make sure to source the :file:`.zshrc` in your terminal - as we do right now - each time you make changes to it

   .. code-block::
      
      source ~/.zshrc


Underlay Workspace
******************

.. attention::
   This is a (mostly) full list of our packages. If you installed the pre-built packages, you probably do not want to clone all these packages. (The ones installed via pre-build packages are comment out)

.. tabs::

   .. code-tab:: console ssh

      $ cd ~/ros2_underlay/src \
      && git clone git@github.com:HippoCampusRobotics/state_estimation.git

      # && git clone git@github.com:HippoCampusRobotics/hippo_sim.git \
      # && git clone git@github.com:HippoCampusRobotics/hippo_gz_plugins.git \
      # && git clone --recursive git@github.com:HippoCampusRobotics/hippo_common.git \
      # && git clone --recursive git@github.com:HippoCampusRobotics/hippo_msgs.git \
      # && git clone git@github.com:HippoCampusRobotics/hippo_control_msgs.git \
      # && git clone --recursive git@github.com:HippoCampusRobotics/esc.git \
      # && git clone git@github.com:HippoCampusRobotics/hippo_control.git \
      # && git clone git@github.com:HippoCampusRobotics/visual_localization.git \
      # && git clone git@github.com:HippoCampusRobotics/remote_control.git


   .. code-tab:: console https
      
      $ cd ~/ros2/src \
      && git clone https://github.com/HippoCampusRobotics/state_estimation.git

      # && git clone --recursive https://github.com/HippoCampusRobotics/hippo_common.git \
      # && git clone --recursive https://github.com/HippoCampusRobotics/hippo_msgs.git \
      # && git clone https://github.com/HippoCampusRobotics/hippo_control_msgs.git \
      # && git clone https://github.com/HippoCampusRobotics/hippo_sim.git \
      # && git clone https://github.com/HippoCampusRobotics/hippo_gz_plugins.git \
      # && git clone --recursive https://github.com/HippoCampusRobotics/esc.git \
      # && git clone https://github.com/HippoCampusRobotics/hippo_control.git \
      # && git clone https://github.com/HippoCampusRobotics/remote_control.git \
      # && git clone https://github.com/HippoCampusRobotics/visual_localization.git && \


These packages have some more dependencies. Let's resolve them by executing

.. code:: console

   $ rosdep-underlay

And to build using our defined alias:

.. code:: console

   $ build_underlay

Note that you do not have to be inside the respective workspace directory to build by executing the defined alias. Very convenient!

Add sourcing the ROS installation in your :code:`.zshrc`. Execute in your terminal:

.. code:: console

   $ echo "source /opt/ros/${ROS_DISTRO}/setup.zsh" >> ~/.zshrc && \
   source ~/.zshrc

After a successful build, we can source this workspace in the :file:`.zshrc`, so that our main, overlayed workspace will find it. For that execute in a terminal:

.. code:: console

   $ echo 'source $HOME/ros2_underlay/install/setup.zsh' >> ~/.zshrc && \
   source ~/.zshrc


Main / Overlay Workspace
************************

Packages for the uvms (BlueROV + Alpha Arm) are not yet provided as binaries. We have to build them from source and as we might need to work on them we add them to the overlayed workspace. Go to

.. code-block:: console

   $ cd ~/ros2/src

From there we clone the packages:

.. tabs::

   .. code-tab:: console ssh

      $ git clone git@github.com:HippoCampusRobotics/uvms.git \
      && git clone git@github.com:HippoCampusRobotics/alpha_arm.git


   .. code-tab:: console https
      
      $ git clone https://github.com/HippoCampusRobotics/uvms.git \
      && git clone https://github.com/HippoCampusRobotics/alpha_arm.git



We can now build the ovberlayed workspace ros2. But first, let's check for unresolved dependencies. Make sure that the underlay workspace containing external packages is sourced for this.

.. code:: console

   $ rosdep-ros2

Then, we can build this workspace using our defined alias.

.. code:: console

   $ build_ros

Now, source this workspace in your :file:`.zshrc`, too, using the local setup this time:

.. code:: console

   $ echo 'source $HOME/ros2/install/local_setup.zsh' >> ~/.zshrc && \
   source ~/.zshrc

Note that since this workspace overlays the :file:`ros2_underlay` workspace, this setup file needs to be sourced afterwards.


Auto-Complete
*************

.. todo::

   This might have changed for Ubuntu 24.04.
   Check and complete this todo!

ROS2 command line tools do not autocomplete as of this `GitHub Issue <https://github.com/ros2/ros2cli/issues/534>`_. While this issue has since been closed, the problem still occurs. To fix this

.. code-block:: console
   
   $ echo "eval \"\$(register-python-argcomplete ros2)\"" >> ~/.zshrc
   $ echo "eval \"\$(register-python-argcomplete colcon)\"" >> ~/.zshrc

Auto-completing topic names seems to work only after an execution of `ros2 topic list`. Before the auto-complete gets stuck and has to be canceled by :kbd:`Ctrl` + :kbd:`C`.

Sourcing :file:`install/setup.zsh` might reset this. Better source :file:`install/local_setup.zsh`.


Final Check
***********

Your :file:`.zshrc` should look similar to this now:

.. code:: sh 
   
   ...


   alias build_ros="env -i HOME=$HOME USER=$USER TERM=xterm-256color bash -l -c 'source $HOME/ros2_underlay/install/setup.bash && cd $HOME/ros2 && colcon build --symlink-install --cmake-args --no-warn-unused-cli -DCMAKE_EXPORT_COMPILE_COMMANDS=ON'"
   alias build_underlay="env -i HOME=$HOME USER=$USER TERM=xterm-256color bash -l -c 'source /opt/ros/jazzy/setup.bash && cd $HOME/ros2_underlay && colcon build --symlink-install --cmake-args --no-warn-unused-cli -DCMAKE_EXPORT_COMPILE_COMMANDS=ON'"

   alias rosdep-ros2="env -i HOME=$HOME USER=$USER TERM=xterm-256color bash -l -c 'source $HOME/ros2_underlay/install/setup.bash && cd $HOME/ros2 && rosdep install --from-paths src -y --ignore-src'"
   alias rosdep-underlay="env -i HOME=$HOME USER=$USER TERM=xterm-256color bash -l -c 'source /opt/ros/jazzy/setup.bash && cd $HOME/ros2_underlay && rosdep install --from-paths src -y --ignore-src'"

   source /opt/ros/jazzy/setup.zsh
   source $HOME/ros2_underlay/install/local_setup.zsh
   source $HOME/ros2/install/local_setup.zsh

   eval "$(register-python-argcomplete ros2)"
   eval "$(register-python-argcomplete colcon)




















































Reach Alpha 5 Arm Mount 
=======================

To-Do:
- stl files zum download für die Grundplatte und die Klemme (Könenn 3D gedruckt werden)
- dazu schreiben dass man zusätzlich dieses extra Einspann Sachen braucht, die aber beim Kit auch eigentlich dabei sind. Das hintere von den Teilen lässt sich stufenweise einstellen, so montieren wie auf Bild.
- welche Schrauben und Muttern man dafür braucht und wo 
- Wie die Bohrungen machen (8mm grundböhrung für Schraube, 8,5 mm Bohrung zum leicht aufbohren + senken, wo dann die Muttern reingepresst werden)
- Bohr/ Grundplatte so anhalten, dass dass es mittig ist und dann kurze seitig bündig mit diesem Teil, wo ne Röhre drin ist.
- getting startet with the Alpha Arm: simple python code examples to read joint data and to write joint commands