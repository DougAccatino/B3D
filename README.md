![image](https://github.com/DougAccatino/B3D/blob/main/Images/Cad_Modelv2.png?raw=true)
# B3D
***
<h2 align="center">!!Work in progress!!</h2>

 <h1 align="center">Modular 3D printer bed system</h1>

 This project originally started with the intention of utilizing Klipper firmware
 in order to develop a modular bed system that deploys multiple grid based beds. An
 Ender 5 plus is subject zero for this, however, it is not limited to just this printer. In the current state of the project, implementing a Marlin firmware is not in the current scope.

 There will be a fair bit of working with tools and hardware. Always remember that safety is your priority when working with any kind of tools, and if you dont know how to do something properly, seek assistance. 

 The primary intention is to combat unnecessary power usage of larger beds by only toggling the needed space, and also minimize the amount of warping on larger beds
 by evenly distributing heat on multiple smaller beds, with an expansion gap of <= 1mm to reduce the overall stress across each mini bed for a mutual distribution of distortion.

***

 The goal for subject zero is to use 
 9 - 120mm x 120mm x 6mm MIC6 cast aluminum plates
 9 - 100mm x 100m 24VDC/30W silicone heat pads
 9 - NTC 100k thermistors
 9 - 120mm x 120mm Magnetic sheets
 1 - Spring steel PEI sheet or Glass

 There are no current mainboards that support enough thermistors, so an expansion board will be necessary. My current mainboard is a BTT Octopus paired to a Raspberry pi 4. Finding enough analog inputs can be intimidating, but a goal of this project is to implement ease of use for many. 

 *Note: The board featured below isnt the only one that will work. Any MCU supported by Klipper with at least 8 analog inputs will work. 

 An Adafruit Metro M4 Grand Central carries a ATSAMD51P20A MCU with 
 - 32-bit, 3.3V logic and power
 - 70 pins in total
 - Dual 1 MSPS DAC (A0 and A1)
 - Dual 1 MSPS ADC (15 analog pins)
 - 8 x hardware SERCOM (can be I2C, SPI or UART)
 - 22 x PWM outputs

 The important aspect of the Grand Central is that it can be flashed with Klipper rather easily via USB, and contains 15 Analog inputs. However, these analog inputs do not contain resistors on the board's internal circuitry, so until a further solution can be found, a DIY circuit must be constructed. 
 
 This circuit will contain a voltage divider with one 100k Ohm resistor and one of the 100k NTC thermistors. 

***

When I flashed Klipper to the grand central, I plugged the Grand Central board directly to the Raspberry pi via USB, and proceeded with the follow steps:

    *To compile the micro-controller code, start by running these commands on the Raspberry Pi:

            cd ~/klipper/
            make menuconfig

    Quoted directly from @FHeilmann from his post on the klipper github issue#1729
        "configure the bootloader size during the make menuconfig step to a size of 16KiB and make sure to set the correct MCU variant for the grand central (SAMD51P20).

        Setting the Bootloader Size to 16 KiB ensures that the uf2 bootloader is not overwritten during the flashing process. You can then double tap reset again to e.g. upload arduino code to the grand central again.

        The uf2 bootloader also accepts regular flashing via bossac, so you can use the method described in the klipper docs to flash the board."

    *Select the appropriate micro-controller and review any other options provided. Once configured, run:

            make

    *It is necessary to determine the serial port connected to the micro-controller. For micro-controllers that connect via USB, run the following:


            ls /dev/serial/by-id/*

    *It should report something similar to the following:


            /dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0

    *It's common for each printer to have its own unique serial port name. This unique *name will be used when flashing the micro-controller. It's possible there may be *multiple lines in the above output - if so, choose the line corresponding to the *micro-controller (see the FAQ for more information).

    *For common micro-controllers, the code can be flashed with something similar to:


            sudo service klipper stop
            make flash FLASH_DEVICE=/dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
            sudo service klipper start
    *Be sure to update the FLASH_DEVICE with the printer's unique serial port name.

    *When flashing for the first time, make sure that OctoPrint is not connected *directly to the printer (from the OctoPrint web page, under the "Connection" *section, click "Disconnect").

***

Once the flash is completed, you have to configure your printer.cfg to recognize and use the Grand Central mcu. *Note: your serial will be different. Do not just copy and paste that, it wont work. The lines I've added to printer.cfg are as followed

  [You can name this however you would like]
      |
      V
[mcu m4]
serial: /dev/serial/by-id/usb-Klipper_samd51p20a_C8A1EC224E37355320202036472203FF-if00
baud:250000
restart_method = command

You can find the pin names in the included image document. This is referenced in the Klipper docs;To use a pin, lets say PC1 which is analog4, you would put:

[temperature_sensor analog4]
sensor_type: Generic 3950
sensor_pin: m4:PC1
pullup_resistor: 100000







