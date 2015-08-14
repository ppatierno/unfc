# unfc

uNFC - NFC library for .Net platforms

uNFC library allow to use NFC integrated circuit connected to your PC (via serial) or to embedded system based on Windows Embedded Compact or .Net Micro Framework.

It supports all three types of .Net Framework :

* .Net Framework 4.0 for PC based on Windows 7 / 8 or embedded systems based on Windows Embedded Standard 7 and Windows Embedded 8 Standard;
* .Net Compact Framework 3.5 / 3.9 for embedded systems based on Windows Embedded Compact 7 and Windows Embedded Compact 2013;
* .Net Micro Framework 4.2 and 4.3 for embedded systems like Netduino boards and .Net Gadgeteed boards;

The library supports NXP PN532 chip but it also defines a little framework so that you can develop a managed driver for a new chip and change it without modifying the upper layers and interface to the user application. The support for PN532 chip provides all three possible communication channels for it : I2C, SPI and HSU. Tipically the HSU (High Speed UART) channel is used for connecting to a serial port on PC or Windows Embedded Compact based system. I2C and SPI connections are better for .Net Micro Framework based boards.

The development and testing was made using RFID/NFC Breakout board from Elecfreaks.
More information [here](http://www.elecfreaks.com/store/rfid-nfc-c-82.html)

The reference for PN532 managed driver is the official NXP user manual available [here](http://www.nxp.com/documents/user_manual/141520.pdf)
