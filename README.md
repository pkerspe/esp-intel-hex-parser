# ESP-IntelHex-Parser
 This library provides the needed functions to parse a file in Intel HEX format.

 **Note:** this class currently does not validate the CRC checksum part of the records in the HEX file, since this is not needed directly for the intended use case.
 
 Initially intended to be used to parse Intel Hex files generated by Arduino IDE when compiling an Arduino Firmware.
 
 This initial use case for this library was to extract the pure data needed for flashing an arduino firmware using the STK500 protocol via serial (UART) in pages of defined size (default = 128). 
 
 For mor details of this use case see: [https://github.com/pkerspe/ESP-UARTArduinoFirmwareUpdate](https://github.com/pkerspe/ESP-UARTArduinoFirmwareUpdate)
  
 For additional Information on the Intel HEX Foramte see also: [https://en.wikipedia.org/wiki/Intel_HEX](https://en.wikipedia.org/wiki/Intel_HEX)
  
 ## About the Intel HEX format:
 A Intel HEX files contains multiple lines (records), each following the structure:

 ```<start character ":"><byte count><address><record type><data><checksum>```
 
 **Example:**
 
 the following record
 ```:0300300002337A1E```
 
 decodes to:
 ```
  ------------------------------------------------------------------------------
 | :   |    03    |   0030  |    00       |        02 33 7A        |   1E       |
 |start|byte count| address | record type |      data section      |checksum    |
 |     |  8 bit   | 16 bit  |   0 = dat   |3 bytes 0x02, 0x33, 0x7A|            |
 | b0  |  bit 1-2 | bit 3-6 | bit 7-8     | bit 9-14               | last 2 bits|
  ------------------------------------------------------------------------------
 ```

**Byte count:** two hex digits (one hex digit pair), indicating the number of bytes (hex digit pairs) in the data field.
 The maximum byte count is 255 (0xFF). 8 (0x08),[6] 16 (0x10)[6] and 32 (0x20) are commonly used byte counts.
 In case of the generated HEX files from Arduino, 16 bytes per record are used
 
**Address:** 
 Four hex digits, representing the 16-bit beginning memory address offset of the data.
 The physical address of the data is computed by adding this offset to a previously established base address,
 thus allowing memory addressing beyond the 64 kilobyte limit of 16-bit addresses.
 The base address, which defaults to zero, can be changed by various types of records.
 Base addresses and address offsets are always expressed as big endian values.
 Currently we do not use this address information and rather calculate address in the flashing process itself
 
**Record Types:** (_only relevant types for firmware update process when flashing an Arduino via Serial are listed_)
    00	Data
    01	End Of File		Must occur exactly once per file in the last record of the file. The byte count is 00, the address field is typically 0000 and the data field is omitted.

**Data:** 
 A sequence of n bytes of data, represented by 2n hex digits.
 Some records omit this field (n equals zero). The meaning and interpretation of data bytes depends on the application.
 (4-bit data will either have to be stored in the lower or upper half of the bytes, that is, one byte holds only one addressable data item)

**Checksum** (_ignored in this implementation_): two hex digits, a computed value that can be used to verify the record has no errors.
