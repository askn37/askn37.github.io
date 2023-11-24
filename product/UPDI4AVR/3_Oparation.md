# [MULTIX UPDI4AVR Programmer] avrdude コマンド操作例

Switch Language [(en-US)](3_Oparation_en.html) [(ja-JP)](3_Oparation.html)

> 以下の用例は avrdude コマンドにパスが通されている前提で記述されています。

UPDI4AVR を使用するには、acrdude.conf に以下の設定を追加してください。
JTAG2UPDI 用の設定がすでにされている場合は、
それを `-c jtag2updi` として選ぶこともできます。

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

> 任意の avrdude.conf を直接指定する場合は `-C` オプションを使います。

> avrdude 6.x 以前（それはArduino IDEに付属しているものです）では`jtagmkii_updi`が定義されておらず、
代わりに`jtagmkii_pdi`を指定する必要があります。
この場合各`part`セクションに`has_updi = yes`指示がなければなりません。
また`eeprom`設定には`page_size = 1`指示が必要になります。

> JTAG2UPDIの規定速度は`-b 115200`に等価です。

`programmer` 指定は `-c updi4avr` オプションで与えます。
UPDI4VAR のシリアルポート（COM）は`-P`オプションで、
対象AVRデバイスの形式は`-p`オプションで、指定します。
少なくともこの3つ組は必要です。

```sh
$ avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 -v
```

`-v`オプションも加えて実行すると、
UPDI4AVR ファームウェアバージョンと、
UPDI4AVR ハードウェア固有のシリアル番号を確認することができます。

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

`-p`オプションに指定できる文字列は avrdude.conf に依存しますが、
概ね次のルールで、型番から読み替えて指定します。

|Family|Device|-p Symbol|
|-|-|-|
|tinyAVR|ATtiny202|t202|
|megaAVR|ATmega4808|m4808|
|AVR|AVR32DD14|avr32dd14|

> `-p`指定は HV制御時に特に重要です。
その値に依存して制御電圧と、それを印加する対象端子が自動的に決まります。
正しくない電圧が正しくない端子に印加された場合、
対象デバイスに不可逆的なダメージを与える場合があります。

> `-p`指定が実際の対象AVRデバイス署名と一致しない（あるいはデバイスから署名を読み出せない）場合、
通常はそれより先の処理を進めずにコマンド実行を中断します。
これを許可して先へ進めるには`-F`オプションの付加が必要です。

> HV制御が必要な状態では 通常UPDI制御ができないので
デバイス署名を読むこともできませんが、
Bootloaderが動作しているのであればそれによって
デバイス署名を確認することができます。
つまり `-c arduino -p m808` 等としてデバイス署名をテストし、
その結果を `-c updi4avr -p XXX` に渡すことで
間違いのない HV制御ができる可能性があります。

実際のメモリ読み出し／書き込み操作は、`-U`オプションで与えます。
メモリタイプには次の種類があります。

|Type|Read|Write|Verify|説明|
|-|-|-|-|-|
|signature|r||v|デバイス署名|
|prodsig|r||v|デバイス固有情報全体|
|sernum|r||v|デバイス固有SN|
|fuses|r|w|v|FUSE領域全体|
|fuseN|r|w|v|FUSE個別(番号)|
|lock|r|w|v|LOCK_BITS領域(1または4バイト)|
|flash|r|w|v|フラッシュ領域|
|eeprom|r|w|v|EEPROM領域|
|userrow|r*|w|v*|USERROW領域|

> userrowの特殊書き込みでは、読み出しと検証はできません。
UPDI4AVR はダミーの結果を返します。

例えば EEPROM領域を Intel-Hex形式ファイルに保存するには、次のように指定します。

```sh
$ avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
  -U eeprom:r:EEPROM.HEX:i
```

FUSEのように1バイト単位で指定できるものは直接、値を書き込むことができます。

```sh
$ avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
  -U fuse0:w:0x00:w
```

> Intel-Hex形式ファイルで書く場合、ファイル中各行の相対的アドレス指定が有効です。
従ってフラッシュ領域の途中だけを書くようなことも、
ページ粒度境界でファイル分割して書くこともできます。
BIN形式ファイルの場合はアドレス指定が（conf記述でしか）できないため、
常にメモリタイプで規定されたアドレスを起点に書き込まれます。

Bootloaderを書き込むような場合は、複数の領域を同時に一回の操作で設定するべきです。
次の例は`-e`オプションでフラッシュ領域を初期化消去し、
Bootloadrを書き込み、それが正しく起動できるように FUSEを設定しています。

```sh
# FUSE と Bootloader を書き込む
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

FLASH 領域は、消去操作がページ粒度単位（`page_size`）ちょうどでなければならないため、
書き込み単位は 32 / 64 / 128 / 512 バイト（デバイス依存）のいずれかでなければなりません。
また 128KiB品種ではアドレス幅が 17ビット以上必要であるため
avrdude 7.x 未満の場合の
avrdude.conf の当該デバイス設定には`load_ext_addr`指示がなければなりません。

> 2023/07時点では 256 指定を必要とする 対象AVRデバイス品種は知られていません。

> UPDI4AVR は`-e`ではなく`-D`が指示されている場合、ブロック消去書込を支援します。
このため他の一般的書込器と違い、
ブロック境界で複数に分割されたアドレス指定付きメモリファイルを
実デバイス上で正しく結合できます。
代わりに`-e`指示の場合より処理速度は制限されます。

### eeprom

EEPROM 領域は 1バイト単位で読み書きができます。
しかし一度に書けるページ粒度は対象AVRデバイス内蔵のコントローラ仕様に依存するため、
最大値は 1 / 2 / 8 / 32 / 64 バイト（デバイス依存）のいずれかであるべきです。

FUSE の EEPROMセーブビットがセットされている時の `-e` デバイス消去操作は
EEPROM 領域を全体消去しません。

### userrow

USERROW は `-e` デバイス消去操作の対象になりません。

tinyAVR と megaAVR シリーズでは、この領域は EEPROM に等価です。
よってバイト単位で読み書きができます。ページ消去操作はありません。

AVR_Dx シリーズ以降では、この領域は特別な FLASH 保存領域です。
全体を一括でしか消去初期化できないため、書くときは全体を単独の 1回の操作で行ってください。

> AVR_Dx シリーズ以降での対話ターミナルモードでは
USERROW にバイト単位で部分的に書けるように見えますが、
通常の操作と異なり、結果は既存値とのビット単位論理積になります。
従って正しく検証はできません。
また UPDI4AVR 固有の機能として、先頭の1バイトに 0xFF を書くことで
USERROW 領域初期化ができます。

USERROW はデバイスが施錠されていても特別な操作で書くことができます。
（後述）

### 特別なデバイス署名応答

UPDI制御が何らかの原因で制限されている場合、
UPDI4AVR は特別なデバイス署名（device signature）を avrdude に返します。

```sh
# avrdude 応答例
avrdude: AVR device initialized and ready to accept instructions
avrdude: device signature = 0xffffff (probably .avr8x) (retrying)
avrdude: device signature = 0xffffff (probably .avr8x) (retrying)
avrdude: device signature = 0xffffff (probably .avr8x)
avrdude main() error: Yikes!  Invalid device signature.
```

|signature|説明|
|-|-|
|0x 00 00 00|期待されない UPDI応答、配線にノイズが乗っている|
|0x 1E 41 32|施錠された AVR_DA/DB/DD デバイス|
|0x 1E 41 33|施錠された AVR_EA デバイス|
|0x 1E 41 34|施錠された AVR_DU デバイス|
|0x 1E 41 34|施錠された AVR_EB デバイス|
|0x 1E 6D 30|施錠された megaAVR-0 デバイス|
|0x 1E 74 30|施錠された tinyAVR-0/1/2 デバイス|
|0x FF FF FF|配線が外れている、あるいは UPDIが応答しない (要HV制御)|

> tinyAVR シリーズで UPDI端子が GPIO用に使われている場合は、
0x000000 を返すこともあります。

## 対話ターミナルモード

対話ターミナルモードを起動するには `-t`オプションを付加します。
ただしそのままでは通信タイムアウトが無効になりません。
対話操作中に操作不能になるのを避けるには、
`-b`オプションで特別な通信速度 `38400` または `666666` を指定してください。

```sh
# タイムアウトを無効にした対話ターミナルモードで起動する
avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 -t -b 38400
# 終了は q または quit コマンド
```

対話ターミナルモードにはリダイレクトを用いて直接対話コマンドを渡せるので、
SERNUM や FUSE の確認をしたいだけの場合は次のようにするのが便利です。
この例では `-b` を指定する必要はありません。

```sh
# SERNUM をメモリダンプする
echo 'dump sernum' | avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 -t -q
```

結果は例えば次のように表示されるでしょう。

```log
avrdude: AVR device initialized and ready to accept instructions
avrdude: device signature = 0x1e953b (probably avr32dd14)
avrdude> dump sernum
0000  42 14 91 78 71 00 16 24  01 32 00 94 00 00 00 00  |B..xq..$.2......|

avrdude> 

avrdude done.  Thank you.
```

> dump コマンドに渡せるメモリタイプは`-U`と同じです。
flash や eeprom 全体を見渡すには、始点オフセットと、そこからのバイト数も指定してください。

> avrdude コマンド終了後も UPDI4AVR が待機状態に戻らない場合は、
SW1ブレークボタンを押すか、いちど電源を切ってください。

> HV制御が必要な状態で対話ターミナルモードを始めるには JP1をショートし、常に -F を付加します。

## HV制御

FUSEの設定を誤ると UPDI制御ピンが無効化されてしまい、以後の一切の操作ができなくなることがあります。
UPDI4AVR ではこの場合は `-U -F -e` の3つ組を使用することで HV制御を有効化し、
正しい FUSE を書き込めるようにすることで、その状態から復旧させることができます。
デバイス毎に固有の正しい FUSE 値（あるいは工場出荷値）は、個別データシートを参照してください。

```sh
# HV制御でFUSEを復旧する
avrdude -C avrdide.conf.UPDI4AVR -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
  -U fuse5:w:0xD9:m -F -e
```

> 添付の構成ファイルを使用するよう`-C avrdide.conf.UPDI4AVR`を指定してください。`AVRDUDE 7.3`時点では標準の構成ファイルでは HV制御を有効にできません。

> 意図的にUPDI制御を禁止しても、Bootloaderでのメモリ読み書きは禁止されません。

> tinyAVR シリーズは（tinyAVR-2 シリーズの 20端子以上品種を除き）
UPDI端子と GPIOあるいは RESET端子の機能が必ず排他選択となります。
従って UPDI端子機能を無効化した tinyAVR シリーズを、
再び UPDI制御可能状態に戻すには HV制御が必須です。

> tinyAVR シリーズで PA0/UPDI端子を OUTPUT用途の GPIOに設定してしまうと
HV制御権の獲得は電気的に困難となるため、推奨されません。
UPDI用途以外に変更するならば、RESET用か、INPUT専用の GPIOとすべきです。

> 一部の品種（AVR_EAシリーズ等）では`erasing chip`操作中に応答を失うことがあります。
この場合はもう一度同じ操作を繰り返してください。

### JP1ジャンパーショート

通常、HV制御を開始するにはフェイルセーフのため `-U -F -e` の3つ組を指定しなければなりません。
`-e`によるデバイス全消去初期化を伴いたくない場合は、JP1ジャンパを短絡状態にしておくことで、
用例によっては `-e` がなくとも HV制御を自動機能させることができます。
これは対話ターミナルモードで特に有用です。

> UPDI制御が普通にできる限り、HV制御は不要のため能動的には発生しません。

ただし `-p` 指定が間違っていても指示された通りの対象AVRデバイスと見做して
強制的に制御が進んでしまうため、誤った指定は不可逆的な結果を招く場合があります。

> JP1ジャンパーは保護ケース内に隠れています。使用する場合はケースから取り出してください。

> 基板単体販売では JP1ジャンパープラグは付属していません。
別途用意するか、ジャンパーピンを半田付けしてください。

## LOCK_BITS デバイス施錠

LOCK_BITS領域を規定外の値に書き換えると（FUSE書換とは別に）UPDI制御を有効化したまま、
対象AVRデバイスのメモリ読み書きを禁止することができます。
これは フラッシュや EEPROM領域からのデータ抜き取りを阻止する
プロプライエタリ用途に使えます。

> 意図的にデバイスを施錠しても、Bootloader 経由でのメモリ読み書きは禁止されません。

```sh
# LOCK_BITSを壊して デバイスを施錠
avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
  -U lock:w:0x00:m -D
```

施錠されたデバイスを解錠するには、`-e -F`でのデバイス全体消去が必要です。解錠に成功すると、電源を切るまで一時的に解錠状態が維持されます。永続的に解錠するには LOCK_BITSを既定値に書き直してください。

```sh
# デバイス解錠
avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
  -eF

# 永続的解錠 tinyAVR と megaAVR の場合
avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
  -eFU lock:w:0xc5:m

# 永続的解錠 AVR_Dx/Ex の場合
avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
  -eFU lock:w:0x5c,0xc5,0xc5,0x5c:m
```

> LOCK_BITS の永続的解錠値は、tinyAVR / megaAVR シリーズと、AVR_DA/DB/DD/EA シリーズとでは異なります。

> 施錠されているデバイスへのメモリ操作に、HV制御は必須ではありません。
施錠と UPDI禁止 FUSEを同時に行っていた場合は、解錠には HV制御が必要です。

> デバイス解錠操作は`-e`で必ずデバイス全体消去しなければならないため、
施錠されたデバイスからフラッシュ領域を UPDI制御で読み出す方法はありません。
しかし Bootloader 経由による読み書き動作は禁止されません。

> 正しく解錠できるのであれば、同時操作で FUSE や Flash を書き換えることもできます。

## 施錠されたデバイスへの USERROW 特殊書込

USERROW領域は、実装コードからは tinyAVR / megaAVR シリーズでは EEPROMと、
AVR_DA/DB/DD/EA シリーズでは Flash領域と同様に扱える不揮発メモリ領域です。
この領域は、施錠されたデバイスであっても UPDI制御で書き換えることができます。
ただし施錠されている間は、UPDI制御で記録内容を読み戻すことはできません。
これは施錠された（プロプライエタリな）デバイスに特別な出荷時情報を
記憶させたい用途に使用できます。

> USERROWは -e での全体消去の影響を受けません。

施錠されたデバイスへの、USERROWへの書き込みは次のように
`-U -D -F -V`オプションの4つ組を使用します。
USERROW領域は全体を1回の単独の操作で書いてください。

```sh
# USERROW の書き換え
avrdude -p avr32dd14 -c updi4avr -P /dev/cu.wchusbserial230 \
  -DFVU userrow:w:USERROW.hex:i
```

> `-D`は`-e`がなくても、`-V`は検証確認に失敗しても、コマンド操作を先に進めさせます。

USERROWを初期化消去したい場合は、全体を 0xFF で上書きしてください。

> 施錠されたデバイスへの対話ターミナルモードでのバイト単位 USERROW 書き換えは推奨されません。一方で`erase userrow`コマンドを用いることで初期状態に戻せます。

## AVR_EB シリーズの補足

AVR_EB シリーズ対応は UPDI4AVR 0.2.8 時点ではまだ実験的です。0.3.0 以降で正式対応の予定です。

## avrdude オプションの補足

### -F

avrdude は実行されるとまず UPDI制御を有効化して、
最初にデバイス署名を実機から読み取り、
`-p`で与えられたデバイスタイプと一致するか調べ、
異なるならそれ以上の実行を中断します。
`-F`はこれを覆し、実行を継続させることができます。

対象デバイスが施錠されている状況、あるいは
UPDI端子がその機能を失っていて HV制御が必要とされる状況では、
実機からデバイス署名を読み取ることができないため、
作業を続行するために`-F`の付加が必須となります。

事前に JP1ジャンパがショートされている場合は、
最初の UPDI制御失敗再試行段階で HV制御を有効化せしめます。
ただし対象デバイスが施錠されていると解る場合は、HV制御は暗黙的には行われません。
なぜならデバイス署名は読めなくとも、UPDI制御自体は成功するからです。

### -e

`-U`で Flash への書き込みを指示した場合、
それに先立ってメモリ消去を行わなければなりません。
`-e`も`-D`も指示されていない場合、avrdude は警告文を出力し、
必要なら暗黙の`-e`を実行しようとします。

`-U`が FUSE や LOCK_BIT への書き込みを指示している場合、
暗黙の`-e`は実行されません。

一方で施錠されたデバイスを開錠するには`-e`でのデバイス消去が必ず実行されなければなりません。
従ってデバイス解錠作業には`-e -F -U`の三つ組みオプション指定が必須です。
更に施錠と同時にデバイスの UPDI機能が失活している場合もあるので、
この三つ組みオプションは HV制御も有効化します。

`-e`指定なし（つまり メモリ消去なし・デバイス開錠必要なし）で
HV制御を有効化したい場合は、事前に JP1をショートしておきます。
それは最初の UPDI制御失敗再試行段階で HV制御を有効化せしめます。

`-e`指定なしでの Flash書き込みは、
UPDI4AVR では部分ブロック消去モードで実行されます。
このため`-e`指定済の場合よりも、書込速度が制限されます。
`-e`付加時は最初に一括消去を行うため、最速効率でのメモリ書込ができます。

### -V

`-U`でのメモリ書き換え後の、暗黙のベリファイ検証を禁止します。
USERROWの特殊書込では、メモリの読み戻し操作が不可能であるため
`-V`の付加が推奨されます。

それ以外の用途での`-V`付加は常に推奨されません。

### -D

暗黙の`-e`が実行されうる場合、それを明示的に禁止します。
つまり全体メモリ消去を実行させません。
Flashメモリ一括書き換えをしない、あるいはデバイス開錠以外の作業しか行わない場合、
つまり EEPROM や USERROW 取り扱い限定時は安全のために
`-U`と共に`-D`が指示されることが望まれます。

> UPDI4AVRでは Flashメモリに対して部分ブロック消去を行えるため、
`-D`を付加することでブロック粒度を最小単位としての、分割領域書換ができます。
（そうでなければ暗黙の`-e`により、デバイス初期化がされるでしょう）

> UPDI4AVRでは EEPROMメモリに対しては常にバイト単位消去/バイト単位書換を行います。
`-e` での EEPROM消去は、それ以前の FUSE値に影響されます。

> UPDI4AVRでは USERROWメモリに対しては一括消去/一括書換を行います。
必ず USERROW領域全体を 1回の操作で書き換えてください。
バイト単位書換（ビット単位論理積）は対話ターミナルモードでのみ限定的に行えます。

## Copyright and Contact

Twitter(X): [@askn37](https://twitter.com/askn37) \
BlueSky Social: [@multix.jp](https://bsky.app/profile/multix.jp) \
GitHub: [https://github.com/askn37/](https://github.com/askn37/) \
Product: [https://askn37.github.io/](https://askn37.github.io/)

Copyright (c) 2023 askn (K.Sato) multix.jp \
Released under the MIT license \
[https://opensource.org/licenses/mit-license.php](https://opensource.org/licenses/mit-license.php) \
[https://www.oshwa.org/](https://www.oshwa.org/)
