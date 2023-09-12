# [MULTIX UPDI4AVR Programmer] Key points for modifying avrdude.conf

Switch Language [(en-US)](2_Configuration_en.html) [(ja-JP)](2_Configuration.html)

The conf file that comes with older avrdude contains
There are some settings that are missing to operate modern devices.
If it cannot be controlled correctly, please refer to the information below and make corrections.

## Add programmer "updi4avr"

UPDI4AVR is JTAG2UPDI compatible, but it will run faster if you add the `-p` setting.
`baudrate` is the default value for the `-b` option.

```conf
programmer
   id = "updi4avr";
   desc = "JTAGv2 to UPDI bridge";
   type = "jtagmkii_pdi";
   connection_type = serial;
   baudrate = 230400;
;
```

> The maximum speed of JTAG2UPDI is `-b 115200` and higher speeds are not recommended.

> UPDI4AVR itself works with `-b 1000000`, but
The maximum speed that can actually be used depends on the processing performance of the PC.
If processing performance is insufficient, communication timeouts occur frequently or are interrupted.

> In the extended format of avrdude.conf 7.0 and later,
UPDI4AVR does not depend on FW=7.53,
It is not affected by its presence or absence.

## Missing memory "boot" setting

If this setting is missing,
When writing to Flash, an extra warning may be displayed stating that "boot" setting is missing.
To avoid this, give `memory "boot"` `size = 0`.

```conf
part
    id    = ".avr8x";
    desc  = "AVR8X family common values";
    has_updi  = yes;
    nvm_base  = 0x1000;
    ocd_base  = 0x0F80;

    memory "boot"
        size = 0;               # HERE
        offset = 0x4000;
    ;
```

## Insufficient load_ext_addr setting on AVR128Dx series

If you are using avrdude 6.x or earlier and the flash capacity is 128KiB,
If this `load_ext_addr` setting is insufficient, when accessing the upper level of the flash area,
The avrdude command may not work properly due to address overflow.

```conf
part parent    ".avrdx"
    id        = "avr128da64";
    desc      = "AVR128DA64";
    signature = 0x1E 0x97 0x07;

    memory "flash"
        size      = 0x20000;
        offset    = 0x800000;
        page_size = 0x200;
        readsize  = 0x100;
        load_ext_addr   = "  0   1   0   0      1   1   0   1",
                          "  0   0   0   0      0   0   0   0",
                          "  0   0   0   0      0   0   0 a16",
                          "  0   0   0   0      0   0   0   0";
    ;
```

> If an address overflow occurs, the first half of the 64KiB area will be
avrdude tries to overwrite the latter 64KiB area.
As a result, the region contents are corrupted and validation fails.

> At least for avrdude 7.1 or later, there should be no need to add this.
There is no harm in adding it.

## Proper EEPROM page granularity in AVR_Dx series

If the EEPROM area cannot be written correctly on AVR_Dx series or later,
The `page_size` value of `memory "eeprom"` is too small or too large.
This value must not be `1`.
For AVR_DA/DB/DD series, `0x10` (16),
For the AVR_EA series, specify `0x08` (8).

```conf
     memory "eeprom"
         size = 0x100;
         offset = 0x1400;
         page_size = 0x10; # HERE
         readsize = 0x100;
     ;
```

> Recommended value is 0x20 (32) for tinyAVR series and 0x40 (64) for megaAVR series.
> These differences depend on the version and operating characteristics of the NVMCTRL controller installed in the target AVR.

> For values outside the range
`warning: timeout/error communicating with programmer`
will be reported.

## Insufficient memory "data" setting

USERROW special write
`offset` of `memory "data"` must correctly indicate the SRAM start address.
Please refer to the individual data sheet to determine the value to specify.
For normal (non-special) USERROW rewrites (to unlocked devices), this setting is not used.

```conf
     memory "data"
     # USERROW special write address for locked device.
         offset = 0x7000;
     ;
```

> [Reference: modernAVR peripheral function comparison list - Built-in SRAM amount](https://github.com/askn37/askn37.github.io/wiki/Peripheral#%E5%86%85%E8%94%B5sram%E9%87%8F%E7%9B%AE)

> The default avrdude.conf as of avrdude 7.2 does not have any of these descriptions.
If you use it, you must add it.

> Even if this specification is missing or incorrect, UPDI4AVR will not return an error, but
Write operations fail implicitly.

## Copyright and Contact

Twitter: [@askn37](https://twitter.com/askn37) \
GitHub: [https://github.com/askn37/](https://github.com/askn37/) \
Product: [https://askn37.github.io/](https://askn37.github.io/)

Copyright (c) askn (K.Sato) multix.jp \
Released under the MIT license
[https://opensource.org/licenses/mit-license.php](https://opensource.org/licenses/mit-license.php) \
[https://www.oshwa.org/](https://www.oshwa.org/)
