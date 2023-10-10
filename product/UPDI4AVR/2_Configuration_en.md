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
    id                     = "updi4avr";
    desc                   = "JTAGv2 to UPDI bridge";
    type                   = "jtagmkii_updi";
    prog_modes             = PM_UPDI;
    connection_type        = serial;
    baudrate               = 230400;
    hvupdi_support         = 0, 1, 2;
;
```

> In avrdude 6.x and earlier (that's what comes with the Arduino IDE)
`jtagmkii_updi` is not defined and
You should specify `jtagmkii_pdi` instead.
In this case each `part` section must have a `has_updi = yes` directive.
Also, the `page_size = 1` instruction is required for the `eeprom` setting.

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

## Appropriate EEPROM page granularity

Especially if the EEPROM area cannot be written correctly on AVR_Dx series or later,
The `page_size` value in the `memory "eeprom"` section is either too small or too large.
16 for AVR_DA/DB/DD series,
Please specify 8 for AVR_EA/EB series.

```conf
    memory "eeprom"
        size      = 256;
        offset    = 0x1400;
        page_size = 16;       # HERE
        readsize  = 256;
    ;
```

32 for the tinyAVR series,
and 64 in some varieties;
64 is the recommended value for megaAVR series.
These differences depend on the characteristics of the `NVMCTRL` controller installed in the target AVR.

For values exceeding the limit range
`warning: timeout/error communicating with programmer`
will be reported.
This is within the timeout limit for waiting for reception after sending on the *avrdude* side.
This means that one write operation was not completed.
At the same time it will fall back to single byte slow operation.
To eliminate this problem and write as quickly and efficiently as possible,
You must reduce `page_size` to an appropriate size.

> In tinyAVR and megaAVR series, `NVMCTRL` control is
Because it has its own buffer memory that is separate from SRAM and is large enough,
Larger block sizes can be written in one operation than other series.
However, when spanning memory alignment, operations must be split.
Therefore, the maximum valid `page_size` value is 32 or 64.
The same applies to `userrow` in these series.

> In the AVR_DA/DB/DD series, the `NVMCTRL` controller does not have buffer memory,
Since the only option is to rewrite `eeprom` in 1-byte units,
It has to be slower than other series.
The upper limit is often 16.

> In the AVR_EA/EB series, the `NVMCTRL` controller has
Since the buffer memory for EEPROM is 8 bytes (memory alignment),
A single operation must be no more than 8 bytes.

`page_size` is 1 which will definitely work for any series.
However, if you share *abrdude.conf* with other writers,
Unless `page_size=2` or more, the writer may not work properly.
This occurs when `page_size=1` is implemented as an implicit `FUSE` rewrite behavior.

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
