Multosensor — User Manual & Technical Guide

A comprehensive HMI system for water quality monitoring and industrial automation based on Raspberry Pi / Linux, Modbus RTU (RS485) interface, FastAPI web framework, pywebview graphical shell, and TailwindCSS styling.

This system is designed to collect 24/7 real-time data from industrial water quality sensors, and fully supports automatic calibration, measurement logging, external actuator control, and automatic sensor cleaning features.

1. Major System Updates (Latest Release)

In this version, the following core technologies have been improved and integrated to ensure stable operation even in extreme industrial environments.

Linux RS-485 Communication Optimization: The timeout for the io_worker and modbus_worker threads is strictly fixed at 0.7 seconds to prevent buffer flush issues with the Linux usb-serial driver and maximize communication speed.

I/O Module Expansion & Custom UI Application:

Digital I/O Module (Default Address: 02): Expanded the UI to display all 12 DI and 12 DO ports on the screen, improving intuitiveness by explicitly stating the module address at the top of the manual control panel.

Analog I/O Module (Default Address: 01): Supports 4 AI (4-20mA input) and 4 AO (4-20mA output) ports, featuring a dynamic UI that allows expansion in increments of 4 ports in the future. The address is explicitly stated at the top left of the panel (e.g., 'A/O: 01').

Intelligent Module Scan (AUTO DETECT) Algorithm: Introduces a hardware feature identification method using FC04 (Analog Read) and FC01 (Relay Status Read) to completely prevent address conflicts.

I/O Module ID Setup Integrity: Fixed a bug where the ID would reset to 0 when closing the setup window. It has been improved to permanently save only when the [APPLY ID] button is directly clicked.

Dashboard Module Address Display: Updated so that the current address of each sensor and output device can be intuitively checked in the 'System Status' window on the main monitoring screen.

2. Hardware Requirements & Wiring Guide

2.1 Hardware Requirements

Controller (Host): Raspberry Pi 3B+ / 4B / Compute Module 4, or a Linux OS (Debian/Ubuntu) based PC.

Display: Touchscreen supporting a resolution of 1024x600 px or higher.

Communication Interface: Isolated USB to RS485 converter or SPI to RS485 expansion HAT.

Supported Sensors: MLSS, UV254 (TOC/COD), DO, ORP, Oil-in-Water, pH, EC, Turbidity, etc.

I/O Boards: Modbus RTU compatible 12-channel relay board, 4-channel AI/AO analog board, etc.

2.2 RS485 and I/O Wiring (Very Important)

RS485 communication lines MUST be wired in a Daisy Chain topology.

[Raspberry Pi / USB-RS485]
       |
       +=== A (+) ===================== A (+) Sensor 1 ==== A (+) Sensor 2 ...
       +=== B (-) ===================== B (-) Sensor 1 ==== B (-) Sensor 2 ...
       +=== GND (Shield) ============== GND Sensor 1 ====== GND Sensor 2 ...


Digital Input (DI) Wiring Precautions:
The DI board linked to this system uses the "NPN Input (Low Level Trigger)" method. A 1 (ON) signal is recognized only when the contact is connected (shorted) to GND (negative). Please pay attention to the wiring, as it will not work if connected to a PNP sensor that applies positive (+) voltage.

3. Program Initial Installation Detailed Guide

3.1 OS Preparation

Install Raspberry Pi OS (64-bit, Desktop) on an SD card using Raspberry Pi Imager.

Connect to the internet (Wi-Fi/Ethernet) after booting.

3.2 Install Required Libraries & System Components

Open the Terminal and run the commands below in order:

sudo apt update
sudo apt install -y python3-pip python3-venv python3-dev \
                    libgtk-3-dev libwebkit2gtk-4.0-dev \
                    gobject-introspection libgirepository1.0-dev


3.3 Program Copy & Virtual Environment (venv) Setup (USB Installation Method)

Considering security and site conditions, copy the program (compressed file) via USB memory instead of downloading it from the internet (GitHub).

Copy the compressed program file (e.g., new-project.zip) from the USB memory to the Raspberry Pi's home directory (~).

Open the Terminal and run the commands below in order to extract it and set up the virtual environment:

cd ~
unzip new-project.zip -d new-project
cd new-project
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt


(Note: The Raspberry Pi must be temporarily connected to the internet to install Python packages via pip install.)

3.4 Grant Communication Port (USB) Access Permissions

Set the program to access the USB-RS485 port without Root privileges.

sudo usermod -a -G dialout $USER


(Caution: You must reboot the system using sudo reboot after entering the command for it to take effect.)

4. Desktop Shortcut & Auto-start (Kiosk) Setup

Create a desktop icon that can be launched with a single click upon device boot. Copy and paste the command below into the terminal.

cat <<EOF > ~/Desktop/Multisensor.desktop
[Desktop Entry]
Name=Multisensor
Comment=Start Multisensor
Exec=bash -c "cd ~/Multisensor && source venv/bin/activate && python src/main.py"
Icon=utilities-terminal
Terminal=false
Type=Application
Categories=Utility;
EOF

chmod +x ~/Desktop/Multisensor.desktop


Auto-start Setup: To have the program launch automatically upon power-up, add the following commands:

mkdir -p ~/.config/autostart
cp ~/Desktop/Multisensor.desktop ~/.config/autostart/


5. Kiosk UI Interface & Key Features Guide

5.1 Monitoring Dashboard

Displays real-time measurement values and the communication status (OK, WAIT, ERR) of registered sensors.

System Status: You can intuitively check the Modbus address assigned to each device on the right panel. The unique ID of the currently communicating device is specified to the right of the sensor and I/O equipment name (e.g., ID: 15, ID: 02).

5.2 Trends

Analyze the trends of measurement values using multi-line graphs.

5.3 Logs / Data

View recorded 5-minute/1-hour average data and alarm history.

EXPORT TO FILE: Select the dates and sensor columns to extract data to a USB as a CSV file.

5.4 Control

The screen to monitor and control external I/O equipment. The Modbus address of the corresponding module is specified at the top of each control panel to allow intuitive device identification, even in environments with multiple connected devices.

DI / MANUAL RELAY OVERRIDE (12 Port): Check the 12-channel digital input/output status and perform manual control. The relay module's address is displayed at the top of the panel (e.g., ID: 02).

AI (4 Port): Measure the real-time 4-20mA current values input from external sources. The analog input module's address is displayed at the top of the panel (e.g., A/I: 01).

MANUAL ANALOG OUTPUT (4 Port): Manually output arbitrary 4-20mA currents to external equipment. The analog output module's address is displayed at the top of the panel (e.g., A/O: 01).

5.5 Setup

Add/Delete Sensor: Set the sensor ID and display name to be linked to the system communication network.

CAL (Calibration): Perform HMI software and sensor hardware (EEPROM) calibration.

I/O SETUP: Set the communication addresses for the relay and analog boards and press the [APPLY ID] button to apply them.

6. Device Communication Address (Modbus ID) Change Tool Guide

This is an independent tool used to change the initial ID to a desired address when installing or replacing equipment on-site.

[Warning] To prevent address conflicts, you MUST connect only 1 device you want to change to the Raspberry Pi.

Close the main Kiosk program.

Open the terminal, activate the virtual environment, and run the tool:

cd ~/Multisensor
source venv/bin/activate
python tools/sensor_tool.py


Scan the device using the [AUTO DETECT] button.

Enter the desired address in NEW ID and click [CHANGE ID].

When 'SUCCESS' is displayed, power off the sensor and power it on again (reboot).

7. Frequently Asked Questions & Troubleshooting

Q. Sensor values display as "ERROR" and the system lags.
If unwired equipment is left turned on in the SETUP screen, communication delays occur due to timeouts. Please deactivate (OFF) any equipment without physical wiring in the SETUP screen.

Q. The I/O module ID in the setup tab resets to 0 after rebooting the equipment.
It is not saved if you just close the popup window. After entering the ID, you must explicitly click the [APPLY ID] button at the bottom of the window to save it permanently.

Q. The AI and AO blocks in the Control tab show up as a red "Err".
This is an analog I/O board communication failure. Verify that the ANALOG OUTPUT ID specified in the I/O SETUP of the Setup tab matches the actual installed dip-switch address, and check if the A(+) and B(-) polarities are correct.

Q. A /dev/ttyUSB0 Permission Denied error occurs.
The USB port control privilege is missing. Please grant permissions using the sudo usermod -a -G dialout $USER command and reboot the system.

Q. I connected a switch to the Digital Input (DI), but the CH signal does not turn on.
The linked relay board uses the NPN (negative contact) method. It will not operate if positive voltage (PNP) is applied, so modify the wiring to short with GND (negative).
