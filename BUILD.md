# Building instructions

This is a detailed guide for building the firmware of MMDVM_HS from the source code. This
software runs on STM32F103 microcontroller. You can build this code using Arduino IDE with
STM32duino package, or using "make" with ARM GCC tools. Also, Arduino with 3.3 V I/O (Arduino
Due and Zero) and Teensy (3.1, 3.2, 3.5 or 3.6) are supported.

# Index

- ZUMSpot RPi
- ZUMSpot Libre Kit
- ZUMSpot USB
- Makefile options
- Config.h options
- Pinout definitions
 
# ZUMSpot RPi

Download latest Raspbian image and install to a micro SD

* See: https://www.raspberrypi.org/documentation/installation/installing-images/
* Configure your SD before booting. Useful article: https://styxit.com/2017/03/14/headless-raspberry-setup.html

Boot your Raspberry Pi. Run raspi-config and configure according to your preferences:

    sudo raspi-config

Select at least:

* Expand filesystem
* Change default password
* Enable "Wait for Network at Boot"
* Disable Desktop GUI if you don't plan to use it

### Enable serial port in Raspberry Pi 3 or Pi Zero W

Edit /boot/cmdline.txt:

    sudo nano /boot/cmdline.txt
    (remove the text: console=serial0,115200)

Disable services:

    sudo systemctl disable serial-getty@ttyAMA0.service
    sudo systemctl disable bluetooth.service

Edit /boot/config.txt

    sudo nano /boot/config.txt

and add the following lines at the end of /boot/config.txt:

    enable_uart=1
    dtoverlay=pi3-disable-bt

Reboot your RPi:

    sudo reboot

### Enable serial port in Raspberry Pi 2

Edit /boot/cmdline.txt:

    sudo nano /boot/cmdline.txt
    (remove the text: console=serial0,115200)

Disable the service:

    sudo systemctl disable serial-getty@ttyAMA0.service

Reboot your RPi:

    sudo reboot

### Build de firmware and upload to ZUMSpot RPi

Install the necessary software tools:

    sudo apt-get update
    sudo apt-get install gcc-arm-none-eabi gdb-arm-none-eabi libstdc++-arm-none-eabi-newlib libnewlib-arm-none-eabi
    
    cd ~
    git clone https://git.code.sf.net/p/stm32flash/code stm32flash
    cd stm32flash
    make
    sudo make install

Download the firmware sources:

    cd ~
    git clone https://github.com/juribeparada/MMDVM_HS
    cd MMDVM_HS/
    git clone https://github.com/juribeparada/STM32F10X_Lib

(Please do not download any different code inside MMDVM_HS folder)

Edit Config.h

    nano Config.h
    
and enable:

    #define PI_HAT_7021_REV_03
    #define ENABLE_ADF7021
    #define BIDIR_DATA_PIN
    #define ADF7021_14_7456
    #define STM32_USART1_HOST
    #define ENABLE_SCAN_MODE

Build the firmware:

    make

Upload the firmware to ZUMSpot RPi board:

    sudo make zumspot-pi

Install MMDVMHost:

    cd ~
    git clone https://github.com/g4klx/MMDVMHost/
    cd MMDVMHost/
    make

Edit MMDVM.ini according your preferences

    nano MMDVM.ini
    (use Port=/dev/ttyAMA0 in [Modem])

Execute MMDVMHost:

    ./MMDVMHost MMDVM.ini

# ZUMSpot Libre Kit

## Windows

Download and install the Arduino IDE:

    https://www.arduino.cc/en/Main/Software

Run Arduino IDE. On the Tools menu, select the Boards manager, and install the "Arduino SAM" from the list of available boards.

Download STM32duino (Arduino for STM32) from this URL:

    https://github.com/rogerclarkmelbourne/Arduino_STM32/tree/ZUMspot

Unzip and change the extracted folder name "Arduino_STM32-ZUMSpot" to "Arduino_STM32"

Copy Arduino_STM32 folder in:

    My Documents/Arduino/hardware

Install the USB bootloader to STM32F103. Follow the instructions:

    https://github.com/rogerclarkmelbourne/Arduino_STM32/wiki/stm32duino-bootloader

Connect the ZUMSpot Libre Kit to your PC. Install the USB Mapple driver using the bat file 
(you may also check http://wiki.stm32duino.com/index.php?title=Windows_driver_installation): 

    My Documents/Arduino/hardware/Arduino_STM32/drivers/win/install_drivers.bat

You have to be sure that Windows detect your ZUMSpot as an USB serial device COMx (please
see Windows Device Manager).

Download the source (zip file) of MMDVM_HS from:

    https://github.com/juribeparada/MMDVM_HS

Do not download or install the STM32F103 library (STM32F10X_Lib) this is not necessary
under STM32duino.

Unzip MMDVM_HS-master.zip and change the folder name to "MMDVM_HS". The path name to this
folder can't have spaces.

Start the Arduino IDE. Open the MMDVM_HS.ino file in the MMDVM_HS folder.

Under the menu "Tools" select "Board" and then select:

    Board: Generic STM32F103C Series
    Variant: STM32F103C8 (20k RAM, 64k Flash)
    CPU Speed: 72 MHz (Normal)
    Upload method: STM32duino bootloader (you have transfered the USB bootloader before)
    Serial port: COMx (Maple Mini)

Edit Config.h:

    #define ADF7021_CARRIER_BOARD
    #define ENABLE_ADF7021
    #define BIDIR_DATA_PIN
    #define ADF7021_14_7456
    #define STM32_USB_HOST
    #define ENABLE_SCAN_MODE

Click the Upload button in the IDE and wait for the transfer.

Once the transfer is completed, press the RESET button of the board or disconnect and
connect the USB cable. You will see the LED (PC13) of the blue pill blinking. Once you connect
with MMDVMHost, the LED will blink fast.

## Linux Raspbian

Install the necessary software tools:

    sudo apt-get update
    sudo apt-get install gcc-arm-none-eabi gdb-arm-none-eabi libstdc++-arm-none-eabi-newlib libnewlib-arm-none-eabi

Download the sources:

    cd ~
    git clone https://github.com/juribeparada/MMDVM_HS
    cd MMDVM_HS/
    git clone https://github.com/juribeparada/STM32F10X_Lib

(Please do not download any different code inside MMDVM_HS folder)

Edit Config.h:

    nano Config.h

and enable:

    #define ADF7021_CARRIER_BOARD
    #define ENABLE_ADF7021
    #define BIDIR_DATA_PIN
    #define ADF7021_14_7456
    #define STM32_USB_HOST
    #define ENABLE_SCAN_MODE

Build the firmware with bootloader support:

    make bl

Upload bootloader and firmware to ZUMSpot Libre Kit, using serial port first (you
are using an USB-serial converter with device name /dev/ttyUSB0). Move BOOT0
jumper to 1, next press and release RESET and execute:

    sudo make serial-bl devser=/dev/ttyUSB0

Move BOOT0 jumper to 0.

For following firmware updates, you could use the USB port directly:

    sudo make dfu devser=/dev/ttyACM0

Install MMDVMHost:

    cd ~
    git clone https://github.com/g4klx/MMDVMHost/
    cd MMDVMHost/
    make

Edit MMDVM.ini according your preferences

    nano MMDVM.ini
    (use Port=/dev/ttyACM0 in [Modem])

Execute MMDVMHost:

    ./MMDVMHost MMDVM.ini
    
# ZUMSpot USB

## Windows

Download and install the Arduino IDE:

    https://www.arduino.cc/en/Main/Software

Run Arduino IDE. On the Tools menu, select the Boards manager, and install the "Arduino SAM" from the list of available boards.

Download STM32duino (Arduino for STM32) from this URL:

    https://github.com/rogerclarkmelbourne/Arduino_STM32/tree/ZUMspot

Unzip and change the extracted folder name "Arduino_STM32-ZUMSpot" to "Arduino_STM32"

Copy Arduino_STM32 folder in:

    My Documents/Arduino/hardware

Connect the ZUMSpot USB to your PC. Install the USB Mapple driver using the bat file: 

    My Documents/Arduino/hardware/Arduino_STM32/drivers/win/install_drivers.bat
    (you may also check: http://wiki.stm32duino.com/index.php?title=Windows_driver_installation)

You have to be sure that Windows detect your ZUMSpot as an USB serial device COMx (please
see Windows Device Manager)

Download the source (zip file) of MMDVM_HS from:

    https://github.com/juribeparada/MMDVM_HS

Do not download or install the STM32F103 library, STM32F10X_Lib, this is not necessary
under STM32duino.

Unzip MMDVM_HS-master.zip and change the folder name to "MMDVM_HS". The path name to this
folder can't have spaces.

Start the Arduino IDE. Open the MMDVM_HS.ino file in the MMDVM_HS folder.

Under the menu "Tools" select "Board" and then select:

    Board: Generic STM32F103C Series
    Variant: STM32F103C8 (20k RAM, 64k Flash)
    CPU Speed: 72 MHz (Normal)
    Upload method: STM32duino bootloader (you have transfered the USB bootloader before)
    Serial port: COMx (Maple Mini)

Edit Config.h:

    #define PI_HAT_7021_REV_03
    #define ENABLE_ADF7021
    #define BIDIR_DATA_PIN
    #define ADF7021_14_7456
    #define STM32_USB_HOST
    #define ENABLE_SCAN_MODE

Click the Upload button in the IDE and wait for the transfer.

Once the transfer is completed, press the RESET button of the board or disconnect and
connect the USB cable.

## Linux Raspbian

Install the necessary software tools:

    sudo apt-get update
    sudo apt-get install gcc-arm-none-eabi gdb-arm-none-eabi libstdc++-arm-none-eabi-newlib libnewlib-arm-none-eabi

Download the sources:

    cd ~
    git clone https://github.com/juribeparada/MMDVM_HS
    cd MMDVM_HS/
    git clone https://github.com/juribeparada/STM32F10X_Lib

(Please do not download any different code inside MMDVM_HS folder)

Edit Config.h

    nano Config.h

and enable:

    #define PI_HAT_7021_REV_03
    #define ENABLE_ADF7021
    #define BIDIR_DATA_PIN
    #define ADF7021_14_7456
    #define STM32_USB_HOST
    #define ENABLE_SCAN_MODE

Build the firmware with bootloader support:

    make bl

Upload the firmware to ZUMSpot USB:

    sudo make dfu devser=/dev/ttyACM0

Install MMDVMHost:

    cd ~
    git clone https://github.com/g4klx/MMDVMHost/
    cd MMDVMHost/
    make

Edit MMDVM.ini according your preferences

    nano MMDVM.ini
    (use Port=/dev/ttyACM0 in [Modem])

Execute MMDVMHost:

    ./MMDVMHost MMDVM.ini

# Makefile options

- make clean: delete all objects files *.o, for starting a new firmware building.

- make: it builds a standard firmware (without USB bootloader support).

- make bl: it builds a firmware with USB bootloader support.

- make zumspot-pi: upload the firmware to a ZUMSpot RPi version (using internal RPi serial port) 

- make dfu [devser=/dev/ttyXXX]: upload firmware using USB bootloader. "devser" is optional,
and it corresponds to the USB serial port device name. This option permits to perform a reset
to enter to booloader mode (DFU). If you don't use "devser", you have to press the reset button
of the ZUMSpot just before using this command.

- make serial devser=/dev/ttyXXX: upload standard firmware using serial port bootloader method.

- make serial-bl devser=/dev/ttyXXX: upload firmware with USB bootloader support using serial
port bootloader method.

- make stlink: upload standard firmware using ST-Link interface.

- make stlink-bl: upload firmware with USB bootloader support using ST-Link interface.

- make ocd: upload standard firmware using ST-Link interface. This method uses a local
openocd installation.

- make ocd-bl: upload firmware with USB bootloader support using ST-Link interface. This
method uses a local openocd installation.

## Common Makefile commands

Serial programming (first programming, transfer the USB bootloader):

    make clean
    make bl
    sudo make serial-bl devser=/dev/ttyUSB0

USB programming (you have already transfered the USB bootloader):

    make clean
    make bl
    sudo make dfu (reset ZUMSpot before) or
    sudo make dfu devser=/dev/ttyACM0 (/dev/ttyACM0 is the device name of ZUMSpot USB under Raspbian)

ZUMSpot RPi (no USB support needed):

    make clean
    make
    sudo make zumspot-pi

# Config.h options

- #define PI_HAT_7021_REV_02: enable pinouts for first revision of ZUMSpot RPi. In general is
not used.

- #define PI_HAT_7021_REV_03: enable pinouts support for ZUMSpot RPi or ZUMSpot USB. You have
to enable this option if you have one of these products.

- #define ADF7021_CARRIER_BOARD: enable this option if you have a ZUMSpot Libre Kit (Board with
modified RF7021SE and Blue Pill STM32F103).

- #define ENABLE_ADF7021: add support for ADF7021 (all boards, enabled by default).

- #define ADF7021_N_VER: enable support for narrow band version of ADF7021 (ADF7021N). Disabled
by default, in general all boards will have just ADF7021.

- #define DUPLEX: enable duplex mode with dual ADF7021. It is still under development.

- #define BIDIR_DATA_PIN: enable Standard TX/RX Data Interface of ADF7021 (enabled by default, 
needed for scanning mode detection feature).

- #define ADF7021_14_7456: select this option if your board uses a 14.7456 MHz (enabled by default).

- #define ADF7021_12_2880: select this option if your board uses a 12.2880 MHz.

- #define ADF7021_ENABLE_4FSK_AFC: enable AFC support for DMR, YSF and P25. This is experimental,
depending on your frequency offset this option will improve or not your BER reception.

- #define ADF7021_AFC_POS: enable this option if you can not receive any signal after enable the
ADF7021_ENABLE_4FSK_AFC option.

- #define STM32_USART1_HOST: enable direct serial host communication with ZUMSpot (using USART1
PA9 and PA10 pins). Disable STM32_USB_HOST if you enable this option. Enable this if you have
a ZUMSpot RPi. You don't need to enable this option if you will transfer the bootloader.

- #define STM32_USB_HOST: enable USB host communication with ZUMSpot (using STM32F103 USB
interface). Disable STM32_USART1_HOST if you enable this option. Enable this if you have
a ZUMSpot USB or ZUMSpot Libre Kit.

- #define ENABLE_SCAN_MODE: enable automatic mode detection in ZUMSpot. This is based on
scanning over all enabled modes, and you could have some detection delay. Enabled by default.

- #define SEND_RSSI_DATA: enable RSSI reports to MMDVMHost. It is already converted to dBm.

- #define SERIAL_REPEATER: enable a second serial port (USART2, pins PA2 and PA3) for Nextion
LCD display.

- #define ENABLE_P25_WIDE: enable support for Motorola Wide P25 mod/demod in XTS3000 radios. Using
this mode improves RX BER. You need to enable this mode in your radio for each conventional
personalities.

# Pinout definitions

## Pinout definitions for ZUMSpot Libre Kit

This is the carrier board or any board with RF7021SE + STM32F103.

Main RF7021SE board:

    CE             PC14
    SLE            PB8
    SREAD          PB7
    SDATA          PB6
    SCLK           PB5
    DATA           PB4 (TxRxData)*
    DCLK           PB3 (TxRxCLK)*
    CLKOUT         PA15 (jumper wire in RF7021SE, not needed with BIDIR_DATA_PIN enabled)
    PAC            PB14 (PTT LED)
    VCC            3.3 V
    GND            Ground

Second RF7021SE board (duplex mode, experimental):

    SLE            PA6
    DATA           PA4 (TxRxData)*
    DCLK           PA5 (TxRxCLK)*
    PAC            NC
    CLKOUT         NC
    VCC            3.3 V
    GND            Ground

    SDATA, SREAD, SCLK and CE are shared with the main ADF7021.

Serial ports:

    TXD            PA9 (serial port host communication)
    RXD            PA10 (serial port host communication)
    DISP_TXD       PA2 (Nextion LCD serial repeater)
    DISP_RXD       PA3 (Nextion LCD serial repeater)

Status LEDs:

    COS_LED        PB15
    PTT_LED        PB14
    P25_LED        PB0
    YSF_LED        PB1
    DMR_LED        PB13
    DSTAR_LED      PB12

Misc pins:

    PIN_LED        PC13 (status led)
    PIN_DEB        PB9 (debugging pin)

You could install a serie resistor (10 - 100 ohms) in each TxRxData and TxRxCLK lines, for
reducing EMI.

## Pinout definitions for Arduino Due/Zero + RF7021SE

Use Arduino IDE with SAM support for building the code.

Main RF7021SE board:

    CE            12
    SLE            6
    SREAD          5
    SDATA          4   // 2 in Arduino Zero Pro
    SCLK           3 
    DATA           7 (TxRxData)*
    DCLK           8 (TxRxCLK)*
    CLKOUT         2   // 4 in Arduino Zero Pro  (jumper wire in RF7021SE, not needed with BIDIR_DATA_PIN enabled)
    PAC            9 (PTT LED)
    VCC            3.3 V
    GND            Ground

Serial ports:

    USB Arduino Programming Port (host communication)

Status LEDs:

    COS_LED       10
    PTT_LED        9
    P25_LED       17
    YSF_LED       16
    DMR_LED       15
    DSTAR_LED     14

Misc pins:

    PIN_LED       13
    PIN_DEB       11

You could install a serie resistor (10 - 100 ohms) in each TxRxData and TxRxCLK lines, for
reducing EMI.

## Pinout definitions for Teensy (3.1, 3.2, 3.5 or 3.6) + RF7021SE:

Use Teensyduino + Arduino IDE for building the code.

Main RF7021SE board:

    CE             6
    SLE            5
    SREAD          4
    SDATA          3
    SCLK           2
    DATA           7 (TxRxData)*
    DCLK           8 (TxRxCLK)*
    CLKOUT         22 (jumper wire in RF7021SE, not needed with BIDIR_DATA_PIN enabled)
    PAC            14 (PTT LED)
    VCC            3.3 V
    GND            Ground

Serial ports:

    Teensy USB Port (host communication)
    DISP_TXD       1 (Nextion LCD serial repeater)
    DISP_RXD       0 (Nextion LCD serial repeater)

Status LEDs:

    COS_LED       15
    PTT_LED       14
    P25_LED       19
    YSF_LED       18
    DMR_LED       17
    DSTAR_LED     16

Misc pins:

    PIN_LED       13
    PIN_DEB       23

You could install a serie resistor (10 - 100 ohms) in each TxRxData and TxRxCLK lines, for
reducing EMI.