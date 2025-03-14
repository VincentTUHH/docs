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
1    -                          -                   
2    :code:`fe201400.serial`    :code:`GPIO0/GPIO1`  
3    :code:`fe201600.serial`    :code:`GPIO4/GPIO5`  
4    :code:`fe201800.serial`    :code:`GPIO8/GPIO9`  
5    :code:`fe201a00.serial`    :code:`GPIO12/GPIO13`
==== ========================== =====================


The Linux kernal then creates a device entry :code:`/dev/ttyAMA*`for every active UART.
Those device entries are the serial device names that programs use to send and receive data. However, the specific :code:`/dev/ttyAMA*`` assignment may change across reboots and if multiple UARTs are acitavted, leading to inconsistencies.

To avoid this issue, we define a fixed device name using a udev rule. This ensures that the UART is always accessible under a predictable name, regardless of how :code:`/dev/ttyAMA*`` numbers are assigned.

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
The udev system dynamically manages device nodes in Linux. By default, it applies standard rules that determine permissions, naming, and symbolic links for connected devices. Custom udev rules allow assigning consistent names to devices, ensuring that :code:`/dev/ttyAMA*`` numbers remain predictable across reboots, as device names like :code:`/dev/ttyAMA2` may change on reboot. To assign a persistent name:
   #. Create a new udev rule:
   .. code-block:: console

      sudo nano /etc/udev/rules.d/50-serial.rules
   #. For the :code:`main` paste:
   .. code-block:: console

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
      lrwxrwxrwx 1 root root 7 Dec 11 14:57 /dev/fteensy_data -> ttyAMA3
   
   A udev rule consists of several components:

   * :code:`KERNEL``: Matches device names like :code:`ttyAMA[0-9]*`.

   * :code:`GROUP``: Assigns the device to a user group (e.g., dialout allows serial access without root privileges).

   * :code:`ENV{SERIAL_MARKER}``: A custom environment variable to distinguish devices.

   * :code:`SUBSYSTEM``: Specifies the device type (e.g., tty for serial devices).

   * :code:`KERNELS``: Matches the hardware address (fe201a00.serial for UART5).

   * :code:`SYMLINK+=""``: Creates a custom, fixed name for the device.


USB Configuartion Guide
=======================
As for UART, we can also set udev rules for USB connections. We use that for the Reach Alpha Arm, where a serial (RS232) to USB adapter is used.



























BlueROV Teather
===============
The yellow teather from BlueRobotics only uses two wires for communication depite more wires given. Note which wires those are and where they are connected to as they are not labled and easy to identify. 

Since only two wires are supported a connection of 100Mbit/s is possible only. To avoid a switching unwantedly between Mbit/s and Gbit/s from Pi side which results in unstable connection, we manually set the advertised link mode to 100 MBit/s in the :file:`/etc/network/if-pre-up.d/eththool`. Add

.. code-block:: sh
   :linenos:
   :caption: /etc/network/if-pre-up.d/eththool

   $ETHTOOL --change eth0 advertise 0x008

behind ... in that file. Thus the check if that gloabl variable exists is performed first before setting it (Is that right?)


Buddy Overview
==============
The buddy solely has the ReachAlpha 5 Arm connected. It might be of interest to see what to also do for an additional top-down camera as it is installed on the other BlueROV. Create a new picture/file for the Pinout of the buddy! Also make sure the Heartbeat LED is connected.


Main Overview
=============
The barometer, Pixhawk 6c (FCU with Px4), teensy are all connected to the main. With the hardare setup they should be running by themself by just connecting those to the correct pins. Just make sure the pins are configured right (eq. UART).