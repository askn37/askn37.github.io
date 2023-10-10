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
    id                     = "updi4avr";
    desc                   = "JTAGv2 to UPDI bridge";
    type                   = "jtagmkii_updi";
    prog_modes             = PM_UPDI;
    connection_type        = serial;
    baudrate               = 230400;
    hvupdi_support         = 0, 1, 2;
;
```

> avrdude 6.x 以前（それはArduino IDEに付属しているものです）では
`jtagmkii_updi`が定義されておらず、
代わりに`jtagmkii_pdi`を指定する必要があります。
この場合各`part`セクションに`has_updi = yes`指示がなければなりません。
また`eeprom`設定には`page_size = 1`指示が必要になります。

> JTAG2UPDI の最高速度は `-b 115200` でそれ以上は推奨されません。

> UPDI4AVR 自体は `-b 1000000` でも動作しますが、
実際に使用できる最大速度は PC側の処理性能に依存します。
処理性能が不足していると通信タイムアウトが多発、あるいは中断されます。

> avrdude.conf 7.0 以降の拡張書式に、
UPDI4AVR は FW=7.53 時点では依存せず、
その有無に影響されません。

## memory "boot" 設定の不足

avrdude 6.x 以前でこの設定が不足していると、
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

## 適切な EEPROMページ粒度

特に AVR_Dx シリーズ以降で EEPROM領域が正しく書けないない場合は、
`memory "eeprom"`セクションの`page_size`値が小さすぎるか、大きすぎます。
AVR_DA/DB/DD シリーズでは 16 を、
AVR_EA/EB シリーズでは 8を、それぞれ指定してください。

```conf
    memory "eeprom"
        size      = 256;
        offset    = 0x1400;
        page_size = 16;       # HERE
        readsize  = 256;
    ;
```

tinyAVR シリーズでは 32 が、
かつ一部の品種では 64 が、
megaAVR シリーズでは 64 が推奨値です。
これらの違いは対象AVRに搭載されている`NVMCTRL`制御器の特性に依存します。

制限範囲を超える値では
`warning: timeout/error communicating with programmer`
がレポートされるでしょう。
これは *avrdude* 側の送信後受信待機タイムアウト制限時間内に
一回の書込操作が終わらなかったことを意味します。
同時にそれは1バイト単位の低速動作へフォールバックするでしょう。
この問題を解消して可能な限り早く効率的に書き込むには、
`page_size`を適切な大きさまで小さくしなければなりません。

> tinyAVR と megaAVR シリーズでは`NVMCTRL`制御器が
SRAMとは別の十分大きな独自の緩衝メモリを持っているため、
他のシリーズより大きなブロックサイズを一回の操作で書くことができます。
ただしメモリアライメントを跨ぐ場合は操作を分割しなければなりません。
従って有効な`page_size`の最大値は 32か 64になります。
なおこれらのシリーズでは`userrow`に対しても同様です。

> AVR_DA/DB/DD シリーズは`NVMCTRL`制御器が緩衝メモリを持たず、
1バイト単位で`eeprom`を書き換えるしかないため、
他のシリーズに比べて低速にならざるを得ません。
多くの場合その上限は 16です。

> AVR_EA/EB シリーズでは`NVMCTRL`制御器の持つ
EEPROM用の緩衝メモリが8バイト（メモリアライメント）であるため、
一回の操作は8バイトまでにしなければなりません。

どのシリーズでも確実に動作するであろう`page_size`は1です。
しかし*abrdude.conf*を他の書込器と共有する場合、
`page_size=2`以上でなければその書込器が正常動作しない場合があります。
これは`page_size=1`が暗黙の`FUSE`書換動作として実装されている場合に生じます。

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
