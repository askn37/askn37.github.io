# [MULTIX UPDI4AVR Programmer] avrdude command operation example

Switch Language [(en-US)](3_Oparation_en.html) [(ja-JP)](3_Oparation.html)

> The following example is written assuming that the path is passed to the avrdude command.

To use UPDI4AVR, add the following settings to acrdude.conf.
If you have already set up JTAG2UPDI,
You can also select it as `-c jtag2updi`.

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

> Use the `-C` option to directly specify any avrdude.conf.

> avrdude 6.x and earlier (which is the one that comes with the Arduino IDE) does not define `jtagmkii_updi`,
You should specify `jtagmkii_pdi` instead.
In this case each `part` section must have a `has_updi = yes` directive.
Also, the `page_size = 1` instruction is required for the `eeprom` setting.

> The specified speed of JTAG2UPDI is equivalent to `-b 115200`.

The `programmer` specification is given with the `-c updi4avr` option.
The serial port (COM) of UPDI4VAR is the `-P` option,
Specify the format of the target AVR device using the `-p` option.
At least these three sets are required.

```sh
$ avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 -v
```

If you run it with the `-v` option,
UPDI4AVR firmware version and
You can find the unique serial number of your UPDI4AVR hardware.

```log
JTAG ICE mkII sign-on message:
Communications protocol version: 1
M_MCU:
  boot-loader FW version:        1
  firmware version:              2.01
  hardware version:              1
S_MCU:
  boot-loader FW version:        1
  firmware version:              7.53
  hardware version:              1
Serial number:                   4a:34:b4:46:34:34
Device ID:                       JTAGICE mkII
```

The string that can be specified for the `-p` option depends on avrdude.conf, but
Generally, specify the model number according to the following rules.

|Family|Device|-p Symbol|
|-|-|-|
|tinyAVR|ATtiny202|t202|
|megaAVR|ATmega4808|m4808|
|AVR|AVR32DD14|avr32dd14|

> The `-p` specification is especially important during HV control.
Depending on that value, the control voltage and the target terminal to which it is applied are automatically determined.
If an incorrect voltage is applied to the incorrect terminal,
This may cause irreversible damage to the target device.

> If the `-p` specification does not match the actual target AVR device signature (or the signature cannot be read from the device),
Normally, command execution is interrupted without proceeding further.
To allow this and proceed further, you need to add the `-F` option.

> Normally UPDI control is not possible when HV control is required.
It also cannot read device signatures,
If the Bootloader is running, then
You can check the device signature.
That is, test the device signature with `-c arduino -p m808` etc.
By passing the result to `-c updi4avr -p XXX`
It is possible to perform error-free HV control.

Actual memory read/write operations are given with the `-U` option.
The memory types are:

|Type|Read|Write|Verify|Description|
|-|-|-|-|-|
|signature|r||v|Device signature|
|prodsig|r||v|Entire device-specific information|
|sernum|r||v|Device-specific SN|
|fuses|r|w|v|Entire FUSE area|
|fuseN|r|w|v|FUSE individual (number)|
|lock|r|w|v|LOCK_BITS area (1 or 4 bytes)|
|flash|r|w|v|flash area|
|eeprom|r|w|v|EEPROM area|
|userrow|r*|w|v*|USERROW area|

> Reading and verification are not possible with special write of userrow.
UPDI4AVR returns dummy results.

For example, to save the EEPROM area to an Intel-Hex format file, specify as follows.

```sh
$ avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
   -U eeprom:r:EEPROM.HEX:i
```

For items like FUSE that can be specified in 1-byte units, values can be written directly.

```sh
$ avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
   -U fuse0:w:0x00:w
```

> When writing in an Intel-Hex format file, relative addressing for each line in the file is effective.
Therefore, it is possible to write only part of the flash area.
You can also write by dividing files at page granularity boundaries.
For BIN format files, addresses can only be specified (with conf description), so
Data is always written starting from the address specified by the memory type.

When writing a Bootloader, you should set multiple areas at the same time in one operation.
The following example initializes and erases the flash area with the `-e` option,
He has written a Bootloadr and configured FUSE so that it can boot properly.

```sh
# Write FUSE and Bootloader
avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
   -U fuse0:w:0x00:m \
   -U fuse1:w:0x00:m \
   -U fuse2:w:0x00:m \
   -U fuse5:w:0xD9:m \
   -U fuse6:w:0x08:m \
   -U fuse7:w:0x00:m \
   -U fuse8:w:0x01:m \
   -U flash:w:bootloader.hex:i \
   -U flash:w:userprog.hex:i \
   -D -e
```

### flash

FLASH areas require erasure operations to be exactly in page granularity (`page_size`), so
Write unit must be 32 / 64 / 128 / 512 bytes (device dependent).
In addition, the 128KiB product requires an address width of 17 bits or more.
For avrdude below 7.x
The device configuration in avrdude.conf must have the `load_ext_addr` directive.

> As of July 2023, there are no known target AVR device types that require 256 specification.

> UPDI4AVR supports block erase/write if `-D` is specified instead of `-e`.
For this reason, unlike other general writing devices,
Addressed memory files divided into multiple blocks at block boundaries
It can be combined correctly on the real device.
Instead, processing speed is more limited than with the `-e` directive.

### eeprom

The EEPROM area can be read and written in 1-byte units.
However, the page granularity that can be written at once depends on the controller specifications built into the target AVR device, so
The maximum value should be 1 / 2 / 8 / 32 / 64 bytes (device dependent).

The `-e` device erase operation when the FUSE EEPROM save bit is set is
Does not erase the entire EEPROM area.

### userrow

USERROW is not subject to `-e` device erase operations.

For tinyAVR and megaAVR series, this area is equivalent to EEPROM.
Therefore, you can read and write in bytes. There is no page erase operation.

For AVR_Dx series and later, this area is a special FLASH storage area.
Since you can only erase and initialize the entire memory at once, when writing, please perform the entire operation in one operation.

> In interactive terminal mode on AVR_Dx series and later
It looks like you can partially write to USERROW in bytes, but
Unlike normal operations, the result is a bitwise AND with the existing value.
Therefore, it cannot be verified correctly.
Also, as a feature unique to UPDI4AVR, by writing 0xFF to the first byte,
You can initialize the USERROW area.

USERROW can be written with special operations even if the device is locked.
(described later)

### Special device signature response

If UPDI control is restricted for some reason,
UPDI4AVR returns a special device signature to avrdude.

```sh
# avrdude response example
avrdude: AVR device initialized and ready to accept instructions
avrdude: device signature = 0xffffff (probably .avr8x) (retrying)
avrdude: device signature = 0xffffff (probably .avr8x) (retrying)
avrdude: device signature = 0xffffff (probably .avr8x)
avrdude main() error: Yikes! Invalid device signature.
```

|signature|description|
|-|-|
|0x 00 00 00|Unexpected UPDI response, noise on the wiring|
|0x 1E 41 32|Locked AVR_DA/DB/DD device|
|0x 1E 41 33|Locked AVR_EA device|
|0x 1E 41 34|Locked AVR_DU device|
|0x 1E 41 34|Locked AVR_EB device|
|0x 1E 6D 30|Locked megaAVR device|
|0x 1E 74 30|Locked tinyAVR device|
|0x FF FF FF|Wiring is disconnected or UPDI does not respond (HV control required)|

> If the UPDI terminal is used for GPIO in the tinyAVR series,
It may also return 0x000000.

## Interactive terminal mode

Append the `-t` option to start interactive terminal mode.
However, communication timeout will not be disabled if this is done.
To avoid getting stuck during interaction,
Specify the special communication speed `38400` or `666666` with the `-b` option.

```sh
# Start in interactive terminal mode with timeout disabled
avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 -t -b 38400
# Quit with q or quit command
```

You can pass interactive commands directly to interactive terminal mode using redirection, so
If you just want to check SERNUM and FUSE, it is convenient to do the following.
There is no need to specify `-b` in this example.

```sh
# Dump SERNUM memory
echo 'dump sernum' | avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 -t -q
```

The results may look like this, for example:

```log
avrdude: AVR device initialized and ready to accept instructions
avrdude: device signature = 0x1e953b (probably avr32dd14)
avrdude> dump sernum
0000  42 14 91 78 71 00 16 24  01 32 00 94 00 00 00 00  |B..xq..$.2......|

avrdude> 

avrdude done.  Thank you.
```

> The memory types that can be passed to the dump command are the same as `-U`.
To look across flash or eeprom, also specify the starting offset and number of bytes from there.

> If UPDI4AVR does not return to standby state even after the avrdude command ends,
Please press the SW1 break button or turn off the power.

> To start interactive terminal mode when HV control is required, short JP1 and always add -F.

## HV control

If FUSE is incorrectly set, the UPDI control pin may be disabled and all further operations may not be possible.
In UPDI4AVR he enables HV control by using the `-U -F -e` triplet in this case,
You can recover from that state by writing the correct FUSE.
Please refer to the individual data sheet for the correct FUSE value (or factory default value) specific to each device.

```sh
# Restore FUSE using HV control
avrdude -C avrdide.conf.UPDI4AVR -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
  -U fuse5:w:0xD9:m -F -e
```

> Specify `-C avrdide.conf.UPDI4AVR` to use the attached configuration file. As of `AVRDUDE 7.3`, HV control cannot be enabled in the standard configuration file.

> Even if you intentionally prohibit UPDI control, reading and writing memory in the Bootloader is not prohibited.

> The tinyAVR series (excluding the tinyAVR-2 series with 20 or more terminals)
The functions of the UPDI pin and GPIO or his RESET pin are always exclusive selections.
Therefore, the tinyAVR series with the UPDI terminal function disabled,
HV control is required to return to UPDI controllable state again.

> If you set the PA0 / UPDI terminal to his GPIO for OUTPUT on the tinyAVR series
Obtaining HV control rights is electrically difficult and is therefore not recommended.
If you change it to something other than UPDI, it should be his GPIO for RESET or INPUT only.

> Some products (AVR_EA series, etc.) may lose response during the `erasing chip` operation.
In this case, please repeat the same operation again.

### JP1 jumper short

Normally, to start HV control, you must specify the `-U -F -e` triplet for fail-safety.
If you do not want to initialize the entire device by `-e`, short-circuit the JP1 jumper.
Depending on the usage, HV control can be activated automatically without `-e`.
This is especially useful in interactive terminal mode.

> As long as UPDI control can be performed normally, HV control is unnecessary and will not occur actively.

However, even if the `-p` specification is incorrect, it will be treated as the target AVR device as specified.
Incorrect specification may lead to irreversible consequences as control will be forced forward.

> JP1 jumper is hidden inside the protective case. Please remove it from the case when using it.

> JP1 jumper plug is not included when selling the board alone.
Please prepare separately or solder the jumper pin.

## LOCK_BITS Device lock

If you rewrite the LOCK_BITS area to a non-standard value (apart from rewriting FUSE), UPDI control will remain enabled,
You can prohibit reading and writing the memory of the target AVR device.
This prevents data extraction from flash or his EEPROM area
Can be used for proprietary purposes.

> Intentionally locking the device does not prevent memory reading and writing via the Bootloader.

```sh
# Break LOCK_BITS and lock the device
avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
   -U lock:w:0x00:m -D
```

To unlock a locked device, you need to erase the entire device with `-e -F`. Upon successful unlocking, LOCK_BITS will revert to its correct device-dependent default value.

```sh
# Unlock device
avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
  -eF
```

> The unlock value of LOCK_BITS is different between the tinyAVR/megaAVR series and the AVR_DA/DB/DD/EA series, but there is no need to be aware of it using the above method.

> HV control is not required for memory operations on locked devices.
If locking and UPDI prohibition FUSE are performed at the same time, HV control is required to unlock.

> To unlock the device, you must use `-e` to erase the entire device, so
There is no way to read the flash area from a locked device using UPDI control.
However, reading and writing operations via the Bootloader are not prohibited.

> If you can unlock it correctly, you can also rewrite FUSE and Flash at the same time.

## USERROW special write to locked device

From the implementation code, the USERROW area is EEPROM and EEPROM in tinyAVR / megaAVR series.
In the AVR_DA/DB/DD/EA series, this is a non-volatile memory area that can be handled in the same way as the Flash area.
This area can be rewritten using UPDI control even if the device is locked.
However, while the door is locked, the recorded contents cannot be read back using UPDI control.
This adds special factory information to locked (proprietary) devices.
You can use it for purposes you want to memorize.

> USERROW is not affected by erasure with -e.

Writing to USERROW on a locked device is as follows:
Use the quadruplet of `-U -D -F -V` options.
Write the entire USERROW area in one single operation.

```sh
# Rewrite USERROW
avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
  -DFVU userrow:w:USERROW.hex:i
```

> `-D` allows the command operation to proceed even without `-e`, and `-V` allows the command operation to proceed even if validation checks fail.

If you want to initialize and erase USERROW, overwrite the entire file with 0xFF.

> Byte-by-byte USERROW rewriting in interactive terminal mode to locked devices is not recommended.

## Supplement for AVR_EB series

AVR_EB series support is still experimental as of UPDI4AVR 0.2.8. Official support is planned for 0.3.0 or later.

## Supplement for avrdude options

### -F

When avrdude is executed, it first enables UPDI control and
First, read the device signature from the actual device,
Check if it matches the device type given with `-p`,
If they are different, further execution is aborted.
`-F` can override this and allow execution to continue.

The target device is locked, or
In situations where the UPDI terminal has lost its function and HV control is required,
Since the device signature cannot be read from the actual device,
Adding `-F` is required to continue the work.

If the JP1 jumper is shorted in advance,
Enables HV control at the first UPDI control failure retry stage.
However, if the target device is found to be locked, HV control will not be performed implicitly.
This is because even if the device signature cannot be read, UPDI control itself will succeed.

### -e

If you specify writing to Flash with `-U`,
Prior to that, you must erase the memory.
If neither `-e` nor `-D` is specified, avrdude prints a warning and
Attempts to perform an implicit `-e` if necessary.

If `-U` instructs to write to FUSE or LOCK_BIT,
Implicit `-e` is not executed.

On the other hand, to unlock a locked device, you must always erase the device with `-e`.
Therefore, it is necessary to specify the triple option `-e -F -U` to unlock the device.
Furthermore, the UPDI function of the device may be deactivated at the same time as the lock is locked.
This triple option also enables his HV control.

Without `-e` specification (i.e. no memory erasure, no need to unlock the device)
If he wants to enable HV control, he shorts JP1 in advance.
It enables HV control at the first UPDI control failure retry stage.

Flash writing without the `-e` specification is
UPDI4AVR runs in partial block erase mode.
For this reason, the writing speed is more limited than when `-e` is specified.
When `-e` is added, the data is erased all at once, so memory can be written with the highest efficiency.

### -V

Prohibits implicit verification after memory rewriting with `-U`.
Because the special write of USERROW does not allow memory readback operations
Adding `-V` is recommended.

Adding `-V` for other purposes is always discouraged.

### -D

Explicitly disallows implicit `-e` if it can be executed.
In other words, the entire memory will not be erased.
If you do not rewrite the Flash memory all at once, or if you only perform tasks other than unlocking the device,
In other words, for safety when handling only EEPROM or USERROW,
It is preferred that `-D` be specified along with `-U`.

> UPDI4AVR allows partial block erasure of Flash memory, so
By adding `-D`, you can rewrite divided areas using block granularity as the minimum unit.
(Otherwise, the implicit `-e` will initialize the device)

> UPDI4AVR always erases/rewrites EEPROM memory in bytes.
EEPROM erasure with `-e` is affected by the previous FUSE value.

> UPDI4AVR performs batch erasing/batch rewriting for USERROW memory.
Be sure to rewrite the entire USERROW area in one operation.
Byte-by-byte rewriting (bit-by-bit logical AND) can only be performed in interactive terminal mode.

## Copyright and Contact

Twitter(X): [@askn37](https://twitter.com/askn37) \
BlueSky Social: [@multix.jp](https://bsky.app/profile/multix.jp) \
GitHub: [https://github.com/askn37/](https://github.com/askn37/) \
Product: [https://askn37.github.io/](https://askn37.github.io/)

Copyright (c) 2023 askn (K.Sato) multix.jp \
Released under the MIT license \
[https://opensource.org/licenses/mit-license.php](https://opensource.org/licenses/mit-license.php) \
[https://www.oshwa.org/](https://www.oshwa.org/)
