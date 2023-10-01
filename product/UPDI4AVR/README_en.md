# [MULTIX UPDI4AVR Programmer] HV compatible program writer for modernAVR generation

Switch Language [(en-US)](https://askn37.github.io/product/UPDI4AVR/README_en.html) [(ja-JP)](https://askn37.github.io/product/UPDI4AVR/)

![Set contents](https://askn37.github.io/product/UPDI4AVR/images/IMG_3530.png)

## Overview

- New generation AVR 8 bit MCU dedicated program writing device with UPDI programmer.
- Rewriting/backup/verification of Flash/EEPROM/FUSE, etc.
- For tinyAVR-0/1/2, megeAVR-0, AVR DA/DB/DD/EA series only
- HV (high voltage) control for tinyAVR-0/1/2, AVR_DD/EA series
- Providing reset button function to target AVR devices
- LOCK_BIT Lock/unlock
- USERROW special write to locked AVR device
- UART pass-through (assuming use of Arduino IDE serial monitor)
- avrdude compatible (Windows / macOS / Linux / etc)
- Arduino IDE compatible, JTAG2UPDI compatible
- Open source software and hardware

## Special features

- Supports two types of high voltage programming: 12V and 8.2V
   - Especially the writing device that can handle 8.2V system is unique.
- Blind write to USERROW area of locked AVR device
   - Because the control procedure is special, no other implementation examples can be found.

## Precautions when purchasing

- Not compatible with older generation AVR (non-UPDI) and SAM generation AVR (JTAG).
- Trace & break debugging is not possible. (Proprietary from Microchip)
- Expensive choice if HV control is not required.
- Because it is handmade, there may be individual differences in appearance.

## Compatible AVR types

- megaAVR-0 series (not HV)
   - ATmega808 ATmega1608 ATmega3208 __ATmega4808__
   - ATmega809 ATmega1609 ATmega3209 __ATmega4809__
- tinyAVR-0 series (HV=12V to PA0)
   - __ATtiny202__ ATtiny402
   - ATtiny204 ATtiny404 ATtiny804 ATtiny1604
   - ATtiny406 ATtiny806 ATtiny1606
   - ATtiny807 ATtiny1607
- tinyAVR-1 series (HV=12V to PA0)
   - ATtiny212 __ATtiny412__
   - ATtiny214 ATtiny414 ATtiny814 __ATtiny1614__
   - ATtiny416 ATtiny816 __ATtiny1616__
   - ATtiny417 ATtiny817 ATtiny1617
- tinyAVR-2 series (HV=12V to PA0)
   - ATtiny424 __ATtiny824__ ATtiny1624 ATtiny3224
   - ATtiny426 ATtiny826 __ATtiny1626__ __ATtiny3226__
   - ATtiny427 ATtiny827 ATtiny1627 ATtiny3227
- AVR_DA series (not HV)
   - AVR32DA28 AVR64DA28 __AVR128DA28__
   - __AVR32DA32__ AVR64DA32 AVR128DA32
   - AVR32DA48 AVR64DA48 AVR128DA48
   - AVR32DA64 AVR64DA64 AVR128DA64
- AVR_DB series (not HV)
   - AVR32DB28 AVR64DB28 AVR128DB28
   - AVR32DB32 AVR64DB32 __AVR128DB32__
   - AVR32DB48 AVR64DB48 __AVR128DB48__
   - AVR32DB64 AVR64DB64 AVR128DB64
- AVR_DD series (HV=8.2V to PF6)
   - AVR16DD14 __AVR32DD14__ AVR64DD14
   - AVR16DD20 AVR32DD20 AVR64DD20
   - AVR16DD28 AVR32DD28 AVR64DD28
   - AVR16DD32 AVR32DD32 __AVR64DD32__
- AVR_EA series (HV=8.2V to PF6)
   - AVR16EA28 AVR32EA28 AVR64EA28
   - AVR16EA32 AVR32EA32 __AVR64EA32__
   - AVR16EA48 AVR32EA48 AVR64EA48
- AVR_EB series (not released as of 2023/07 scheduled to be supported)
   - *AVR16EB14*
   - *AVR16EB20*
   - *AVR16EB28*
   - *AVR16EB32*

> __bold__ indicates operation has been verified, *italics* indicates future support

## Specifications

- Board dimensions: 66mm x 31mm (FRISK case size)
- Board fixing hole: x4 M3 (pitch 25mm x 60mm)
- Protective case dimensions: 69mm x 34mm x 12mm (excluding protrusions)
- Main controller: ATtiny1626 (or equivalent)
- Operating voltage: 3.1V~5.3V
- AVR-ICSP connection: IDC/6P ribbon cable, Dupont loose wire 6P
- VDD supply capacity: 5V or 3.3V, max. 100mA (with short circuit protection)
- PC connection: USB Type-C (2.0) WCH-CH340E

> Parts used, external color, etc. may change depending on production time.

### Product contents

- x1 UPDI4AVR main board
- x1 protective case set (x2 piece configuration, x2 fixing screws)
- x1 IDC/6P ribbon cable
- x1 JP1 jumper plug

> There are no accessories included when the main board is sold separately.

> Please prepare the USB cable (Type-C) separately.

## Component placement

![](https://askn37.github.io/product/UPDI4AVR/images/Image-2.drawio.svg)

## AVR-ICSP terminal arrangement

![](https://askn37.github.io/product/UPDI4AVR/images/Image-1.drawio.svg)

> Terminal pitch is 2.54mm (100mil)

## Example of use

![Connection with IDC ribbon cable](https://askn37.github.io/product/UPDI4AVR/images/IMG_3529.png)

![Connection with breadboard](https://askn37.github.io/product/UPDI4AVR/images/IMG_3527.png)

## Related Links

- [Usage guide](https://askn37.github.io/product/UPDI4AVR/1_Usage_en.html)
- [Key points for modifying avrdude.conf](https://askn37.github.io/product/UPDI4AVR/2_Configuration_en.html)
- [avrdude command operation example](https://askn37.github.io/product/UPDI4AVR/3_Oparation_en.html)

## Open source software and hardware

- [askn37/multix-zinnia-updi4avr-firmware-builder](https://github.com/askn37/multix-zinnia-updi4avr-firmware-builder) - for UPDI4AVR Firmware
- [Schematic diagram](https://askn37.github.io/product/UPDI4AVR/2306_UPDI4AVR/2306_Zinnia-UPDI4AVR-MZU2406B2.pdf) (PDF)
- [Board diagram](https://askn37.github.io/product/UPDI4AVR/2306_UPDI4AVR/2306_Zinnia-UPDI4AVR-MZU2306B7_layers.svg) (SVG)
   - [Top side of board](https://askn37.github.io/product/UPDI4AVR/2306_UPDI4AVR/2306_Zinnia-UPDI4AVR-MZU2306B7_top.svg) (SVG)
   - [Bottom side of board](https://askn37.github.io/product/UPDI4AVR/2306_UPDI4AVR/2306_Zinnia-UPDI4AVR-MZU2306B7_bottom.svg) (SVG)
- [Gerber file](https://github.com/askn37/askn37.github.io/tree/main/product/UPDI4AVR/2306_UPDI4AVR/PCBA/) (ZIP)
- [Protective case](https://github.com/askn37/askn37.github.io/tree/main/product/UPDI4AVR/2306_UPDI4AVR/3DP/) (STL)

## Copyright and Contact

Twitter: [@askn37](https://twitter.com/askn37) \
GitHub: [https://github.com/askn37/](https://github.com/askn37/) \
Product: [https://askn37.github.io/](https://askn37.github.io/)

Copyright (c) askn (K.Sato) multix.jp \
Released under the MIT license
[https://opensource.org/licenses/mit-license.php](https://opensource.org/licenses/mit-license.php) \
[https://www.oshwa.org/](https://www.oshwa.org/)
