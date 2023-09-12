# [MULTIX UPDI4AVR Programmer] avrdude.conf 修正の要点

Switch Language [(en-US)](2_Configuration_en.html) [(ja-JP)](2_Configuration.html)

古い avrdude に付属している conf ファイルには、
最新のデバイスを操作するには欠けている設定がいくつかあります。
正しく制御できない場合は下記の情報を参考に修正してください。

## programmer "updi4avr" の追加

UPDI4AVR は JTAG2UPDI 互換ですが、`-p` 指定用の設定を追加すればより高速に動作します。
`baudrate` は `-b` オプションの省略時規定値です。

```conf
programmer
  id   = "updi4avr";
  desc = "JTAGv2 to UPDI bridge";
  type = "jtagmkii_pdi";
  connection_type = serial;
  baudrate = 230400;
;
```

> JTAG2UPDI の最高速度は `-b 115200` でそれ以上は推奨されません。

> UPDI4AVR 自体は `-b 1000000` でも動作しますが、
実際に使用できる最大速度は PC側の処理性能に依存します。
処理性能が不足していると通信タイムアウトが多発、あるいは中断されます。

> avrdude.conf 7.0 以降の拡張書式に、
UPDI4AVR は FW=7.53 時点では依存せず、
その有無に影響されません。

## memory "boot" 設定の不足

この設定が不足していると、
Flash書き込み時に「"boot"設定がない」という余分な警告が表示されることがあります。
これを避けるには `memory "boot"` に `size = 0` を与えてください。

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

## AVR128Dx シリーズでの load_ext_addr 設定不足

avrdude 6.x 以前でかつ、フラッシュ容量が 128KiB 品種の場合、
この`load_ext_addr`設定が不足しているとフラッシュ領域の上位アクセスに際し、
avrdude コマンドがアドレスオーバーフローによって正常動作しないことがあります。

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

> アドレスオーバーフローが発生していると前半 64KiB領域に、
avrdude は後半64KiB領域を 上書きしようとします。
その結果、領域内容は不正に破壊されて、検証にも失敗します。

> 少なくとも avrdude 7.1 以降では追記する必要はないはずです。
追記しても害はありません。

## AVR_Dx シリーズでの適切な EEPROMページ粒度

AVR_Dx シリーズ以降で EEPROM領域が正しく書けないない場合は、
`memory "eeprom"`の`page_size`値が小さすぎるか、大きすぎます。
この値は `1` であってはなりません。
AVR_DA/DB/DD シリーズでは `0x10`（16）を、
AVR_EA シリーズでは `0x08`（8）を、それぞれ指定してください。

```conf
    memory "eeprom"
        size      = 0x100;
        offset    = 0x1400;
        page_size = 0x10;       # HERE
        readsize  = 0x100;
    ;
```

> tinyAVR シリーズでは 0x20（32）が、megaAVR シリーズでは 0x40（64）が推奨値です。
> これらの違いは対象AVRに搭載されている NVMCTRL 制御器のバージョンや動作特性に依存します。

> 範囲外の値では
`warning: timeout/error communicating with programmer`
がレポートされます。

## memory "data" 設定不足

USERROW 特殊書き込みでは
`memory "data"` の `offset` が正しく SRAM先頭アドレスを示していなければなりません。
指定する値は個別データシートを参照して決定してください。
特殊ではない普通の（施錠されていないデバイスへの）USERROW書き換えでは、この設定は使われません。

```conf
    memory "data"
    # USERROW special write address for locked device.
        offset    = 0x7000;
    ;
```

> [参考 : modernAVR 周辺機能比較一覧 - 内蔵SRAM量目](https://github.com/askn37/askn37.github.io/wiki/Peripheral#内蔵sram量目)

> avrdude 7.2 時点の規定の avrdude.conf には、これらの記述は一切ありません。
使用する場合は必ず追記する必要があります。

> この指定がなかったり誤っていても UPDI4AVR はエラーを返しませんが、
書込操作は暗黙のうちに失敗します。

## Copyright and Contact

Twitter: [@askn37](https://twitter.com/askn37) \
GitHub: [https://github.com/askn37/](https://github.com/askn37/) \
Product: [https://askn37.github.io/](https://askn37.github.io/)

Copyright (c) askn (K.Sato) multix.jp \
Released under the MIT license \
[https://opensource.org/licenses/mit-license.php](https://opensource.org/licenses/mit-license.php) \
[https://www.oshwa.org/](https://www.oshwa.org/)
