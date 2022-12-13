# Multix Zinnia Product SDK [*AVR] for Arduino IDE

## 導入方法

- Arduino IDE の「環境設定」「追加のボードマネージャーのURL」に以下のリンクを追加
  - [`https://askn37.github.io/package_multix_zinnia_index.json`](https://askn37.github.io/package_multix_zinnia_index.json)
- 「ボードマネージャー」ダイアログパネルを開き、検索欄に "multix" と入力
- 目的のアーキテクチャを選択して「インストール」\
   `megaAVR` `modernAVR` `reduceAVR`

## 対応するホストOS

- Windows (32bit/64bit)
- macOS (64bit)
- Linux (主にintel系64bit)

## 対応AVRアーキテクチャ

現在この SDK は複数の異なる対象アーキテクチャ向けにリポジトリを分けて提供される。\
ボードメニュー/サブメニュー以下の選択はそれぞれを参照のこと。

- __Multix Zinnia Product SDK [megaAVR]__ [->HERE](https://github.com/askn37/multix-zinnia-sdk-megaAVR)
  - megaAVR-0 と tinyAVR-0/1/2 系統。（Atmelブランド世代）
- __Multix Zinnia Product SDK [modernAVR]__ [->HERE](https://github.com/askn37/multix-zinnia-sdk-modernAVR)
  - AVR DA/DB/DD 系統。（Microchipブランド世代）
- __Multix Zinnia Product SDK [reduceAVR]__ [->HERE](https://github.com/askn37/multix-zinnia-sdk-reduceAVR)
  - 旧世代AVRのうち TPI方式に対応した系統。（Atmelブランド世代）

> 共通基盤の AVR-GCC/AVR-LIBC toolchain は既知の AVR 8bit 系全種に対応している。\
> makeコマンドを別途用意すれば大抵の事はできるはず。（Windowsでも）

## 対象AVR

### [megaAVR] tinyAVR-0/1/2

|系統|pin|2KiB|4KiB|8KiB|16KiB|32KiB
|-|-|-|-|-|-|-|
|tinyAVR-0
||8 |__ATtiny202__|ATtiny402
||14|ATtiny204|ATtiny404|ATtiny804|ATtiny1604
||20|         |ATtiny406|ATtiny806|ATtiny1606
||24|         |         |ATtiny807|ATtiny1607
|tinyAVR-1
||8 |ATtiny212|__ATtiny412__
||14|ATtiny214|ATtiny414|__ATtiny814__|__ATtiny1614__
||20|         |ATtiny416|ATtiny816|__ATtiny1616__|__ATtiny3216__
||24|         |ATtiny417|ATtiny817|ATtiny1617|ATtiny3217
|tinyAVR-2
||14|         |ATtiny424|__ATtiny824__|ATtiny1624|ATtiny3224
||20|         |ATtiny426|ATtiny826|ATtiny1626|ATtiny3226
||24|         |ATtiny427|ATtiny827|ATtiny1627|ATtiny3227

> __太字__ は実物確認済

### [megaAVR] megaAVR-0

|系統|pin|8KiB|16KiB|32KiB|48KiB
|-|-|-|-|-|-|
|megaAVR-0
||28|ATmega808|ATmega1608|ATmega3208|__ATmega4808__
||32|ATmega808|ATmega1608|ATmega3208|ATmega4808
||40|         |          |          |__ATmega4809__
||48|ATmega809|ATmega1609|ATmega3209|__ATmega4809__

> __太字__ は実物確認済

### [modernAVR] AVR DA/DB/DD

|系統|pin|8KiB|16KiB|32KiB|64KiB|128KiB
|-|-|-|-|-|-|-|
|AVR_DA
||28|        |         |AVR32DA28|AVR64DA28|AVR128DA28
||32|        |         |__AVR32DA32__|AVR64DA32|AVR128DA32
||48|        |         |AVR32DA48|AVR64DA48|AVR128DA48
||64|        |         |         |AVR64DA64|AVR128DA64
|AVR_DB
||28|        |         |AVR32DB28|AVR64DB28|AVR128DB28
||32|        |         |AVR32DB32|AVR64DB32|__AVR128DB32__
||48|        |         |AVR32DB48|AVR64DB48|__AVR128DB48__
||64|        |         |         |AVR64DB64|AVR128DB64
|AVR_DD
||14|        |AVR16DD14|__AVR32DD14__|AVR64DD14
||20|        |AVR16DD20|AVR32DD20|AVR64DD20
||28|        |AVR16DD28|AVR32DD28|AVR64DD28
||32|        |AVR16DD32|AVR32DD32|__AVR64DD32__
|AVR_EA
||28|*AVR8EA28*|*AVR16EA28*|*AVR32EA28*|*AVR64EA28*
||32|*AVR8EA32*|*AVR16EA32*|*AVR32EA32*|*AVR64EA32*
||48|*AVR8EA48*|*AVR16EA48*|*AVR32EA48*|*AVR64EA48*

> __太字__ は実物確認済\
> *斜体* は今後対応予定（未定）

### [reduceAVR]

- ATtiny4 ATtiny5 ATtiny9 __ATtiny10__
- *ATtiny20* *ATtiny40*
- *ATtiny102* *ATtiny104*

> __太字__ は実物確認済\
> *斜体* は現在未対応（Variant選択なし）

## 許諾

各構成要素はそれぞれ異なる配布ライセンスに属する。条件はそれぞれの規約に従う。

- BSD License
  - avr-libc
- GNU General Public License v2.0
  - avr-gcc
  - avrdude
- MIT License
  - other original document and code

## 著作表示

Twitter: [@askn37](https://twitter.com/askn37) \
GitHub: [https://github.com/askn37/](https://github.com/askn37/) \
Product: [https://askn37.github.io/](https://askn37.github.io/)

Copyright (c) askn (K.Sato) multix.jp \
Released under the MIT license \
[https://opensource.org/licenses/mit-license.php](https://opensource.org/licenses/mit-license.php) \
[https://www.oshwa.org/](https://www.oshwa.org/)
