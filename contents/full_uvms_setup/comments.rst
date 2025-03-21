Comments and Notes
##################

Pinout 
======
I can declare a difffernt pinout as I like. The only things that are predefined are the pins for the UART. I am free in changing the orthers, but have to make changes in the config files as well.


UART Configuration Guide
========================

Overview
********
Universal Asynchronous Receiver-Transmitter (UART) is a simple, asynchronous serial communication protocol using only two wires: **TX (Transmit)** and **RX (Receive)**. It operates without a clock signal, relying on predefined baud rates for synchronization.

On the **Raspberry Pi 5**, UART communication is available through multiple hardware UARTs, each mapped to specific **GPIO pins**.

Mandatory Setup Steps
*********************
#. Enable UARTs in :file:`config.txt`

   * By default, some UARTs are disabled. To enable them:
   .. code-block:: console

      sudo nano /boot/firmware/config.txt

   * Append the following lines. **Note**: Those are the UART's used for the current pinout of bluerov-02-main (for FCU Debug, Teensy and FCU Telemetry). Those three devices must then also be connected to the corresponding GPIO pins as defined in the table below. **For the buddy this should be different**
   .. code-block:: sh

      dtoverlay=uart2
      dtoverlay=uart4
      dtoverlay=uart5
   * Save and reboot:
   .. code-block:: console

      sudo reboot


#. Verify UART Availability

   * After rebooting, check the available serial devices:
   .. code-block:: console

      ls -l /dev/serial*
   * If :code:`uart5` is enabled, you should see something like:
   .. code-block:: console

      /dev/ttyAMA2 -> fe201a00.serial

#. UART Kernel-to-Pin Mapping

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

#. Wiring Guidelines

   * TX and RX are defined **from the perspective of the Raspberry Pi**. Example for **UART5**:
      * **TX (GPIO12)** → Connect to **RX of the external device**
      * **RX (GPIO13)** → Connect to **TX of the external device**
      * **GND** must also be connected between devices.

#. Check if the Connection is Working

   * To check if a device is connected and sending data:
   .. code-block:: console

      cat /dev/ttyAMA2  # Replace with your actual UART device

   * For a **loopback test** (without an external device), connect **TX to RX** and run:
   .. code-block:: console

      echo "Hello" > /dev/ttyAMA2
      cat /dev/ttyAMA2
   
   * If "Hello" is displayed, UART is working correctly.

#. Set Up Persistent Device Naming

   The **udev** system dynamically manages device nodes in Linux. By default, it applies standard rules that determine permissions, naming, and symbolic links for connected devices. Custom udev rules allow assigning consistent names to devices, ensuring that :code:`/dev/ttyAMA*` numbers remain predictable across reboots, as device names like :code:`/dev/ttyAMA2` may change on reboot. Namin the rule's file is important as higher-numbered rules (e.g., 99-custom.rules) are applied last, overriding lower-numbered default rules. To assign a persistent name:

   #. Create a new udev rule:

      .. code-block:: console

         sudo nano /etc/udev/rules.d/99-serial.rules
   #. For the :code:`main` paste:

      .. code-block:: sh

         KERNEL=="ttyAMA[0-9]*", GROUP="dialout", ENV{SERIAL_MARKER}="serial_marker"

         # uart2
         ENV{SERIAL_MARKER}=="serial_marker",  SUBSYSTEM=="tty", KERNELS=="fe201400.serial", SYMLINK+="teensy_data"
         # uart4
         ENV{SERIAL_MARKER}=="serial_marker",  SUBSYSTEM=="tty", KERNELS=="fe201800.serial", SYMLINK+="fcu_debug"
         # uart5
         ENV{SERIAL_MARKER}=="serial_marker",  SUBSYSTEM=="tty", KERNELS=="fe201a00.serial", SYMLINK+="fcu_data"

   #. Apply the new rule:

      .. code-block:: console

         sudo udevadm control --reload-rules && sudo udevadm trigger

      * From now on, use :code:`/dev/fcu_data` and all the other device addresses for a stable reference.

   #. Check rules applied:

      .. code-block:: console

         ls /dev/fcu* -l
         ls /dev/teensy* -l

      The output should show symbolic links for the serial devices:

      .. code-block:: console

         lrwxrwxrwx 1 root root 7 Dec 11 14:57 /dev/fcu_debug -> ttyAMA1
         lrwxrwxrwx 1 root root 7 Dec 11 14:57 /dev/fcu_tele -> ttyAMA2
         lrwxrwxrwx 1 root root 7 Dec 11 14:57 /dev/teensy_data -> ttyAMA3
   
   A udev rule consists of several components:

   * :code:`KERNEL``: Matches device names like :code:`ttyAMA[0-9]*`.

   * :code:`GROUP``: Assigns the device to a user group (e.g., dialout allows serial access without root privileges).

   * :code:`ENV{SERIAL_MARKER}``: A custom environment variable to distinguish devices.

   * :code:`SUBSYSTEM``: Specifies the device type (e.g., tty for serial devices).

   * :code:`KERNELS``: Matches the hardware address (fe201a00.serial for UART5).

   * :code:`SYMLINK+=""``: Creates a custom, fixed name for the device.


USB Configuartion Guide
=======================
As for UART, we can also set udev rules for USB connections. We use that for the Reach Alpha Arm, where a serial (RS232) to USB adapter is used. We want to ensure the Reach Alpha 5 Arm always appears as the same device, no matter which USB port we use. For that we must define a rule including the manipulator's **serial number (:code:`ATTRS{serial}`)**. This number is unique to the device itself, not the ports on the raspberry pi. 

 #. Identify the device:
   .. code-block:: console

      ls /dev/ttyUSB* /dev/ttyACM*

   Depending on the raspberry pi's architecture, USB devices either appear as :code:`USB*` or :code:`ACM*`. Say the manipulator appears as :code:`/dev/ttyUSB0` get more information with:

   .. code-block:: console

      udevadm info -a -n /dev/ttyUSB0 | grep -E "idVendor|idProduct|serial"

   **Note**: change device identifier to the device you reveived.

   In the output look for **serial** number, vendor ID (**idVendor**), and product ID (**idProduct**) in the output. Example:

   .. code-block:: console

      ATTRS{idVendor}=="2341"
      ATTRS{idProduct}=="0042"
      ATTRS{serial}=="A1B2C3D4"

   #. Create new udev rule:

      Open a new rule file:

      .. code-block:: console

         sudo nano /etc/udev/rules.d/99-reach-alpha.rules
      
      Add the following line, but **REPLACE** idVendor, idProduct, and serial with your actual values:

      .. code-block:: sh

         SUBSYSTEM=="tty", ATTRS{idVendor}=="2341", ATTRS{idProduct}=="0042", ATTRS{serial}=="A1B2C3D4", SYMLINK+="reach_alpha", MODE="0666"

      :code:`Mode="0666"` grant read and write access to all users without the need for sudo.

      Whenever the Reach Alpha 5 Arm is connected, it now appears as :code:`/dev/reach_alpha`. The real USB device (:code:`/dev/ttyUSB0` or :code:`/dev/ttyACM0`) remains unchanged, but :code:`/dev/reach_alpha` will always point to it.

   #. Apply the rules:

      Reload and trigger the new rule:

      .. code-block:: console

         sudo udevadm control --reload-rules && sudo udevadm trigger
      Check rules applied:

      .. code-block:: console

         ls -l /dev/reach_alpha

      The output should show symbolic links for the serial devices like:

      .. code-block:: console

         lrwxrwxrwx 1 root root 7 Mar 14 12:00 /dev/reach_alpha -> ttyUSB0

      From now on use :code:`/dev/reach_alpha` in any program to read and write to the manipulator.

Do the same for the **FCU (PixHawk 6C)** which is also connected via USB. If you have more than one device connectde via USB it helps to unplug all but one device to correctly identify the **serial** number, vendor ID (**idVendor**), and product ID (**idProduct**).

Create the file:

.. code-block:: console

   sudo nano /etc/udev/rules.d/99-fcu.rules

and add the follwoing line with according changes to idVendor, idProduct, and serial:

.. code-block:: sh

   SUBSYSTEM=="tty", ATTRS{idVendor}=="1d6b", ATTRS{idProduct}=="0002", ATTRS{serial}=="A1B2C3D4", SYMLINK+="fcu_usb", MODE="0666"

Don't forget to apply the rules and to check them:

.. code-block:: console

   sudo udevadm control --reload-rules && sudo udevadm trigger && \
   ls -l /dev/fcu_sub

Note: The idVendor is a 16-bit identifier assigned to a manufacturer by the USB Implementers Forum, while the idProduct is a 16-bit identifier assigned by the manufacturer to distinguish different models. The serial number is a unique identifier for each individual device, ensuring that even if multiple devices share the same vendor and product ID, they can still be distinguished. In udev rules, using only idVendor and idProduct applies to all matching devices, whereas including the serial ensures that only a specific device is recognized, preventing mix-ups.


00-teensy.rules on klopsi-main-00
=================================

The file reads:

.. code-block:: sh

   ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="04*", ENV{ID_MM_DEVICE_IGNORE}="1", ENV{ID_MM_PORT_IGNORE}="1"
   ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="04[789a]*", ENV{MTP_NO_PROBE}="1"
   KERNEL=="ttyACM*", ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="04*", MODE:="0666", RUN:="/bin/stty -F /dev/%k raw -echo"
   KERNEL=="hidraw*", ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="04*", MODE:="0666"
   SUBSYSTEMS=="usb", ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="04*", MODE:="0666"
   KERNEL=="hidraw*", ATTRS{idVendor}=="1fc9", ATTRS{idProduct}=="013*", MODE:="0666"
   SUBSYSTEMS=="usb", ATTRS{idVendor}=="1fc9", ATTRS{idProduct}=="013*", MODE:="0666"


* **KERNEL=="hidraw*"**: (USB) HID devices. Those are Human Intercae Devices, like mice or keyboard, that allow for human interaction with the computer

* **KERNEL=="ttyACM*"**: serial devices that are connected via USB.

* **ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="04*", ENV{ID_MM_DEVICE_IGNORE}="1", ENV{ID_MM_PORT_IGNORE}="1"**: ModemManager (ID_MM_DEVICE_IGNORE) is a Linux service that manages mobile broadband modems (e.g., 3G/4G LTE USB sticks). When a new USB device is detected, ModemManager probes it to see if it is a modem. Some USB serial devices (like microcontrollers, including the Teensy) can be mistaken for modems, causing unwanted interference (e.g., delays, unexpected disconnections). If ModemManager tries to probe the Teensy, it may lock the port, making it inaccessible or causing connection issues. We want to prohibt this by ensuring the ModemManager ignores the teensy.

* **ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="04[789a]*", ENV{MTP_NO_PROBE}="1"**: MTP (Media Transfer Protocol) (MTP_NO_PROBE) is a protocol used to transfer media files between devices (e.g., Android phones and Windows/Linux). Some microcontrollers that present USB storage or serial communication might be incorrectly detected as an MTP device. This rule tells the system not to attempt MTP probing on specific Teensy devices.



KERNEL=="ttyACM*", ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="04*", MODE:="0666", RUN:="/bin/stty -F /dev/%k raw -echo"

What is this telling. Also why are they using kernal here and not just subsystem? What is the meaning behind it?





Teensy Baudrate
===============
The baud rate refers to the speed of serial communication, measured in bits per second (bps). It determines how fast data is transmitted between the Raspberry Pi and the Teensy microcontroller over the UART interface. Both the Raspberry Pi and Teensy must use the same baud rate; otherwise, communication will be unreliable or fail completely. For example, if the Teensy is configured to transmit at 115200 bps but the Raspberry Pi expects 9600 bps, the received data will be garbled or lost.


**TO-DO** What are the steps to verify same baud rate

#. raspberry pi side:

   Check current baud rate on the UART that teensy uses (replace device accordingly. When udev rules active from above it should be :code:`/dev/teensy_data`)

   .. code-block:: console

      stty -F /dev/teensy_data

   Look for the line:

   .. code-block:: console

      speed 115200 baud; line = 0;
   
   Change baud rate in python file if it doesn't match the teensy baud rate???

#. teensy side:

   **TO-DO**







:code:`/boot/firmware/config.txt`
=================================

Apart from the things I add (only those uart, that I also use), **must add it under the [all] section:

.. code-block:: sh
   [all]
   ...
   dtoverlay=i2c4,pins_6_7
   dtoverlay=uart2
   dtoverlay=uart3
   dtoverlay=uart4
   dtoverlay=uart5

The difference to the default, that is on the old pi is:

.. code-block:: sh

   # dtparam=spi=on

   dtoverlay=gpio-led,gpio=11,label=led11-heartbeat,trigger=heartbeat
   # must also be under [all]


The new default file additionally has these lines:

.. code-block:: sh

   # Enable KMS ("full" KMS) graphics overlay, leavinf GPU memory as the 
   # default (the kernal is in control of graphics memory with ful KMS)
   dtoverlay=vc4-kms-v3d
   disbale_fw_kmms_setup=1

And the order is slightly different.





We will make some chnages to the Raspberry Pi's hardware configurations. It will have immediate impact on the usage of the pins we see in the pinout. For that we will open the respected :code:`config.txt` file:

.. code-block:: console

   sudo nano /boot/firmware/config.txt

#. Deactivate Serial Peripheral Interface (SPI) as we don't use SPI devices. Commennt out in the default :code:`config.tet`:

   .. code-block:: sh

      # dtparam=spi=on

#. Add to the end of the file:

   .. code-block:: sh

      dtoverlay=gpio-led,gpio=11,label=led11-heartbeat,trigger=heartbeat

   This line enables a device tree overlay that configures :code:`GPIO pin 11` as an LED controlled by the Linux system. The :code:`trigger=heartbeat` setting means that the LED will blink to indicate system activity (like a heartbeat). This setup is used for status indication. Once the Raspberry Pi boots, it will automatically start blinking the LED according to the system heartbeat.

   **Note**: TO-DO: how is the LED cuircuit looking, also according to the pinout.

#. Activate all UART's that vill be used as serial connection, by adding:

   .. code-block:: sh

      dtoverlay=uart2

   The default :code::`uart0` is active by default.

#. Add additional I2C bus 4 (I2C4) and assign it to :code:`GPIO 6` (pin 31) (SDA) and :code:`GPIO 7` (pin 7) (SCL). It then appears as :code:`/dev/i2c-4`. (SDA = data line, SCL = clock line):

   .. code-block:: sh

      dtoverlay=i2c4,pins_6_7

   By default, when :code:`dtparam=i2c_arm=on`, :code:`GPIO 2` (SDA) and :code:`GPIO 3` (SCL) are used. It appears as :code:`/dev/i2c-1`

   In additioin to the SDA and SCL pins, the I2C device must also be connected to any :code:`GND` pin (9, 14, 20, 25, 30, 34, 39) and voltage supply, meaning :code:`VCC (3.5V/5V)` on either pin 1, 17 or 2, 4.

#. Enable hardware PWM on specific GPIO (here: GPIO 20 and 21), by adding:

   .. code-block:: sh

      dtoverlay=pwm,pin=20,func=4
      dtoverlay=pwm,pin=21,func=4

   :code:`func=4` assigns the correct alternate function for PWM. Next to the PWM in the PWM devices (servo motor or LED) must also be connected to other pins as well for power supply (**TO-DO**: figure out how servo and headlights are connected for power)

   Must be :code:`GPIO 20` for front camera tilt and :code:`GPIO 21` for headlight brightness, as defined in hippocampus hardware package in camera_servo_node and spotlight_node `hardware GITHUB <https://github.com/HippoCampusRobotics/hardware.git/>`_

   Check for pwm availability:

   .. code-block:: console

      ls /sys/class/pwm/

In the end always rebbot the system to apply changes:

.. code-block:: console

   sudo reboot



PWM Devices 
===========

The servo motor to control the camera angle or the brightnes to control the headlights requires a PWM signal. Apart from that they also require power supply.

After enabling PWM in the :code:`config.txt` the PWM signals are controlled through a special system file.

:code:`camera_servo_node` operates the PWM with the pigpiod library, that allows for precise hardware PWM control, even on GPIO pins that don't normally spuuport hardware PWM.

pigpio is NOT pre-installed on Ubuntu (only on Raspbian = Raspberry Pi OS, the default operating system for Raspberry Pi). Hence, the pigpio package is often times missing from standard repositories in Ubuntu and must be manually installed, as well as pigpiod (the daemon of pigpio) must be enabled.

Beides mal ausprobieren, so wie ChatGPT:

.. code-block:: console

   cd && \
   git clone https://github.com/joan2937/pigpio.git && \
   cd pigpio && \
   make && \
   sudo make install

.. code-block:: console

   sudo apt update && \
   sudo apt install unzip

or as in Hippo docs:

.. code-block::console

   wget https://github.com/joan2937/pigpio/archive/master.zip && \
   sudo apt update && \
   sudo apt install unzip && \
   unzip master.zip && \
   sudo rm master.zip && \
   cd pigpio-master && \
   make && \
   sudo make install

Then create a :code:`pigpiod.service` file at :code:`/etc/systemd/system`:

.. code-block:: console

   sudo nano /etc/systemd/system/pigpiod.service

.. code-block:: sh

   [Unit]
   Description=Pigpio daemon

   [Service]
   Type=forking
   PIDFile=pigpio.pid
   ExecStart=/usr/local/bin/pigpiod

   [Install]
   WantedBy=multi-user.target

Hit Strg-0, Enter Strg-X to save and return to console.

The enable the system permanently to run at boot:

.. code-block:: console

   sudo systemctl enable pigpiod.service

And run it for the first time:

.. code-block:: console

   sudo systemctl start pigpiod.service



Make sure the pigpio deamon is running:

.. code-block:: console

   sudo systemctl status pigpiod


needrestart.conf 
================
needrestart, a tool used on Linux systems to check whether services need to be restarted after package updates.

Open the file:

.. code-block:: console

   sudo nano /etc/needrestart/needrestart.conf

Uncomment and modify the following two lines in that file.

enables automatic service restarts after package updates:

.. code-block:: sh

   $nrconf{restart} = 'a';

disables kernel hints, meaning needrestart will not notify you if a system reboot is required due to a kernel update

.. code-block:: sh

   $nrconf{kernelhints} = 0;

Apply changes:

.. code-block:: console

   sudo systemctl restart needrestart
   



systemd service file 
====================


You cannot name the sections arbitrarily in systemd service files. Systemd only recognizes specific predefined section names. If you use an invalid section name, systemd will ignore it or produce an error.

systemd service file:
========= ===================================================================
SECTION   PURPOSE          
========= ===================================================================
[Unit]    Defines metadata, description, and dependencies.      
[Service] Specifies how the service starts, stops, and runs.   
[Install] Controls when and how the service is enabled (e.g., start at boot).   
[Socket]  Used when defining socket-based activation (optional). 
[Timer]   Used for timed execution of services (optional). 
[Path]    Used for path-based activation (optional).
========= ===================================================================


/boot/firmware/config.txt
=========================

boot config.txt file. [] sections control which settings apply to which hardware
======= ===================================================================
SECTION   PURPOSE          
======= ===================================================================
[all]   Runs on all Raspberry Pi models. 
[pi4]   Only applies to Raspberry Pi 4 (or Pi 5 if not using [pi5]).
[pi5]   Only applies to Raspberry Pi 5.
[cm4]   Only applies to the Compute Module 4 (CM4).
======= ===================================================================
If a setting appears in [pi5], it overrides conflicting [all] settings on a Pi 5.
Since I am solely using this config.txt for this specific Pi 5, I can put everything under under [all] or [pi5] — both will work for your Raspberry Pi 5.
**Einmal gucken, was unter Pi 4 oder cm4 war, ob das relevant ist?!**





If the deamon were running:

❯ sudo systemctl status pigpiod
● pigpiod.service - Pigpio daemon
     Loaded: loaded (/etc/systemd/system/pigpiod.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2025-03-17 12:06:18 CET; 5h 47min ago
   Main PID: 683 (pigpiod)
      Tasks: 4 (limit: 9244)
     Memory: 772.0K
        CPU: 26min 50.676s
     CGroup: /system.slice/pigpiod.service
             └─683 /usr/local/bin/pigpiod

Mar 17 12:06:18 klopsi-main-00 systemd[1]: Starting Pigpio daemon...
Mar 17 12:06:18 klopsi-main-00 systemd[1]: Started Pigpio daemon.





Code example in python how to use it:

.. code-block:: sh

   import pigpio

   servo_pin = 20  # GPIO 20 (Pin 38)

   pi = pigpio.pi()  # Connect to the pigpio daemon
   pi.set_mode(servo_pin, pigpio.OUTPUT)

   # Servo expects pulses between ~500µs (left) and ~2500µs (right), with ~1500µs as center
   pi.set_servo_pulsewidth(servo_pin, 1500)  # Neutral position








BlueROV Teather
===============
The yellow teather from BlueRobotics only uses two wires for communication depite more wires given. Note which wires those are and where they are connected to as they are not labled and easy to identify. 

Since only two wires are supported a connection of 100Mbit/s is possible only. To avoid a switching unwantedly between Mbit/s and Gbit/s from Pi side which results in unstable connection, we manually set the advertised link mode to 100 MBit/s in the :file:`/etc/network/if-pre-up.d/eththool`. Add

.. code-block:: sh
   :linenos:
   :caption: /etc/network/if-pre-up.d/eththool

   $ETHTOOL --change eth0 advertise 0x008

behind ... in that file. Thus the check if that gloabl variable exists is performed first before setting it (Is that right?)

Power Supply 
============
Pin 4 (5V) and pin 6 (GND) are reserved for the external power supply, which is provided by the 4 cell LiIon battery. As the battery voltage is too high and varies over time a SBEC is connected in between. It takes the battery volate as input and outputs 5A with 5V or 6V. The position of the 2-pin jumper on the output side of the SBEC determines the output voltage, which must be 5V.

Camera
======

The top-own camera in the bottom tube has 4 wires which are connected to a USB port. This connection provides both power to the camera and data transmission. The raspberry pi's USV ports supply 5V power to connected devices.

Camera devices should be detected as:

.. code-block:: console

   ls /dev/video*

The first detected camera always appears as :code:`/dev/video0` and the second as :code:`/dev/video1`. As the main pi is only connected to the front camera and the buddy pi only to the vertical camera, both will appear on their respected pi as :code:`/dev/video0`. Hence this device name is hard coded in the launch file, that start the mjpeg_cam_node (see below).

The hardware setup of the camera is done, once it is connected via USB and is recognized by it. To process the image data, use the above device name in a program.


v4l2 
****

Video capturing, processing and noticing video devices, eg :code:`/dev/video0` is done by the Video4Linux (v4l2) subsystem, which is native for Ubuntu 24.04. and built into the kernal. Without it

.. code-block:: console

   ls /dev/video*

would'nt create an output.

:code:`v4l2-ctl` is an additional tool to handle video devices and gather information. It is not native however, and must be installed first:

.. code-block:: console

   sudo apt update && sudo apt install v4l-utils -y

To use v42l-ctl to list available video devices

.. code-block:: console

   v4l2-ctl --list-devices

or to read /dev/video0 specifications:

.. code-block:: console

   v4l2-ctl --list-formats-ext

Here it shows the supported video formats (MJPEG should be one of them, as we use it in the node / launch file below). It also list the possible resolution (eg. :code:`Size Discrete 1280x720` them 1280 and 720 for witdh and height, or rather as seen below the resolution is selected index based, in that example discrete_size = 1, would choose the first resolution in that list) and corresponding fps that are possible with those resolutions. Those settings must be then made in the mjpeg_cam_node when defining the camera device, because when the camera is "opened" for the first time with those settings, the camera then sends image data with those settings respectively.

/hardware/launch/bluerov.launch.py
**********************************

**By the way**: this file also starts the PWM for the front camera servo and spotlight LED.

.. code-block:: sh

   def add_jpeg_camera_node():
      action = Node(
         executable='mjpeg_cam_node',
         package='mjpeg_cam',
         name='front_camera',
         namespace='front_camera',
         parameters=[
               {
                  'device_id': 0,
                  'discrete_size': 1,
                  'fps': 30,
                  'publish_nth_frame': 3,
               },
         ],
      )
      return action


/hardware/launch/bluerob_buddy.launch.py
****************************************

.. code-block:: sh

   def include_vertical_camera_node():
      source = launch_file_source('mjpeg_cam', 'ov9281.launch.py')
      args = LaunchArgsDict()
      args.add_vehicle_name_and_sim_time()
      args['camera_name'] = 'vertical_camera'
      return IncludeLaunchDescription(source, launch_arguments=args.items())

The :code:`ov9281.launch.py` then starts the mjep_cam_node as for the front camera. Here the camera data (device_id, discrete_size, fps, publish_nth_frame) is not provided directly, but through a confi file :code:`ov9281.yaml` which also declares the camera as :code:`device_id=0`.

**ov9281** is the name of the camera that is build in the robot.


mjpeg_cam package 
*****************
Everything is started from the laucnh file :code:`ov9281.lkaunch.py`. They are only called from within the :code:`hardware` package in :code:`hippo.launch.py` and :code:`bluerov.launch.py`. And in :code:`bluerov_buddy.launch.py` the :code:`mjpeg_cam_node` is executed directly.

This package defines the interface betweem the camera hardware and processing of image data on software side. All of this is defined in the class MjpegCam. In the header :code:`mjepeg_cam.hpp` a method DeviceName() returns a string of the pi device the camera is using:

.. code-block:: sh

   std::string DeviceName() {
    return "/dev/video" + std::to_string(params_.device_id);
   }

The initialization is in :code:`mjepeg_cam.cpp`. It calls the constructur of teh Device class that initializes a camera device like :code:`/dev/video0`, the resolution (width and height) and the frames per seconds (fps).

.. code-block:: sh

   camera_ = std::make_shared<Device>(DeviceName(), frame_size.first,
                                     frame_size.second, params_.fps);

The core of the camera interface, responsible for opening, configuring, capturing, and controlling a USB camera using the V4L2 (Video4Linux2) API lies in :code:`device.cpp`. This file is implementing the :code:`Device` class, which interacts with a USB camera via the V4L2 interface. 


External Sensors 
================

The barometer has 4 wires: two for power supply and two for data transmission via I2C. Those four must be connected accordingly to a any 3.3V (or 5V depending on sensor) pin, any GND pin, and an active I2C port (SDA + SCL)


Buddy Overview
==============
The buddy solely has the ReachAlpha 5 Arm connected. It might be of interest to see what to also do for an additional top-down camera as it is installed on the other BlueROV. Create a new picture/file for the Pinout of the buddy! Also make sure the Heartbeat LED is connected.


Main Overview
=============
The barometer, Pixhawk 6c (FCU with Px4), teensy are all connected to the main. With the hardare setup they should be running by themself by just connecting those to the correct pins. Just make sure the pins are configured right (eq. UART).


Cooling Fan 
===========

We have two wire cooling fans (+ and - wire). We can either directly connect them to 5V and GND on teh pin board, but then they would run all the time, or control them with respect to CPU tempertaure.

For that change the tree:

.. code-block:: console

   sudo nano /boot/firmware/config.txt

Add the line to the section [all]:

.. code-block:: sh

   dtoverlay=gpio-fan,gpiopin=18,temp=55000

Any GPIO pin is possible, but GPIO18, GPIO17, GPIO22, GPIO27 are suited best. Select a desired CPU temperature, at which the pin goes high (3.3V) (55000 is 55°C). A PGIO pin does not provide enough power to power the fan. Thus use a npn-trasnistor as switch. Use the same circuit as shown `here <https://raspberrypi.stackexchange.com/questions/130583/is-it-possible-to-control-the-fan-from-gpio>`_

TO-DO
=====


Visualization laucnh to start the april tag localization, was brauche ich dafür 


 qualisys setup and what else I might require to read mocap data (is mocap data just odometry data of the frames it detects or also camera data and the qualisys bridge is handling that?)

 SOS Leack Sensor Installation BlueRobotics 
 -> QGroundControl brauche ich wohl dafür, von Nathalie zeigen lassen!!!


Wie Latex mit svg einbinden direkt, Nathalie zeigen lassen, was sie da an tricks hatte 

Malte 3d Druck schnalle für Alpha Arm 

Bohrung am Bluerov freihand und dann einfach Muttern reingedrückt? 

qualisys bridge angucken, was geht rein, was geht raus, brauche ich einen ros treiber (was ist ein ros treiber) was sind netzwerk voraussetzungen?


ASC Anmelden Segelschein 

pgiio ersatz, wenn man raspberry pi 5 nutzt. Man könnte/ müsste dann die vorgesehenen hardware PWM nodes nutzen:
https://pypi.org/project/rpi-hardware-pwm/

 Epoxy adapter selber machen BlueRobotics 