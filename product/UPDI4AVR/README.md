# [MULTIX UPDI4AVR Programmer] modernAVR世代専用HV対応プログラム書込器

Switch Language [(en-US)](README_en.html) [(ja-JP)](index.html)

![セット内容](https://askn37.github.io/product/UPDI4AVR/images/IMG_3530.png)

## 概要

- UPDI制御方式の新世代 AVR 8bit MCU 専用プログラム書込装置
- Flash / EEPROM / FUSE等の書換・バックアップ・検証
- tinyAVR-0/1/2、megeAVR-0、AVR DA/DB/DD/EA シリーズ専用
- tinyAVR-0/1/2、AVR_DD/EA シリーズの HV（高電圧）制御
- 対象AVRデバイスへのリセットボタン機能の提供
- LOCK_BIT 施錠・開錠
- 施錠されたAVRデバイスへの USERROW 特殊書込
- UARTパススルー（Arduino IDE シリアルモニター利用想定）
- avrdude 対応（Windows / macOS / Linux / etc）
- Arduino IDE対応、JTAG2UPDI互換
- オープンソースソフトウェア・ハードウェア

## 購入時の注意

- 旧世代 AVR（非UPDI）および SAM世代 AVR（JTAG）には対応しません。
- トレース＆ブレークデバッグはできません。（Microchip社プロプライエタリ）
- HV制御が不要なら高価な選択になります。
- ハンドメイドのため外観仕上に個体差があります。

## 対応AVR品種

- megaAVR-0 系統（not HV）
  - ATmega808 ATmega1608 ATmega3208 __ATmega4808__
  - ATmega809 ATmega1609 ATmega3209 __ATmega4809__
- tinyAVR-0 系統（HV=12V to PA0）
  - __ATtiny202__ ATtiny402 
  - ATtiny204 ATtiny404 ATtiny804 ATtiny1604 
  - ATtiny406 ATtiny806 ATtiny1606 
  - ATtiny807 ATtiny1607
- tinyAVR-1 系統（HV=12V to PA0）
  - ATtiny212 __ATtiny412__
  - ATtiny214 ATtiny414 ATtiny814 __ATtiny1614__
  - ATtiny416 ATtiny816 __ATtiny1616__
  - ATtiny417 ATtiny817 ATtiny1617
- tinyAVR-2 系統（HV=12V to PA0）
  - ATtiny424 __ATtiny824__ ATtiny1624 ATtiny3224
  - ATtiny426 ATtiny826 __ATtiny1626__ __ATtiny3226__
  - ATtiny427 ATtiny827 ATtiny1627 ATtiny3227
- AVR_DA 系統（not HV）
  - AVR32DA28 AVR64DA28 __AVR128DA28__
  - __AVR32DA32__ AVR64DA32 AVR128DA32
  - AVR32DA48 AVR64DA48 AVR128DA48
  - AVR32DA64 AVR64DA64 AVR128DA64
- AVR_DB 系統（not HV）
  - AVR32DB28 AVR64DB28 AVR128DB28
  - AVR32DB32 AVR64DB32 __AVR128DB32__
  - AVR32DB48 AVR64DB48 __AVR128DB48__
  - AVR32DB64 AVR64DB64 AVR128DB64
- AVR_DD 系統（HV=8.2V to PF6）
  - AVR16DD14 __AVR32DD14__ AVR64DD14
  - AVR16DD20 AVR32DD20 AVR64DD20
  - AVR16DD28 AVR32DD28 AVR64DD28
  - AVR16DD32 AVR32DD32 __AVR64DD32__
- AVR_EA 系統（HV=8.2V to PF6）
  - AVR16EA28 AVR32EA28 AVR64EA28
  - AVR16EA32 AVR32EA32 __AVR64EA32__
  - AVR16EA48 AVR32EA48 AVR64EA48
- AVR_EB 系統（2023/07時点で未発売・対応予定）
  - *AVR16EB14*
  - *AVR16EB20*
  - *AVR16EB28*
  - *AVR16EB32*

> __太字__ は動作検証済、*斜体* は今後対応予定

## 諸元

- 基板寸法 : 66mm x 31mm（FRISKケース大）
- 基板固定穴 : x4 M3（ピッチ25mm x 60mm）
- 保護ケース寸法 : 69mm x 34mm x 12mm（突起部含まず）
- 主制御器 : ATtiny1626（または同等品）
- 動作電圧 : 3.1V〜5.3V
- AVR-ICSP接続 : IDC/6Pリボンケーブル、Dupontバラ線6P
- VDD供給能力 : 5Vまたは3.3V、最大100mA（短絡保護有）
- PC接続 : USB Type-C (2.0) WCH-CH340E

> 使用部品、外観色等は生産時期により変わることがあります。

### 商品内容

- x1 UPDI4AVR 本体基板
- x1 保護ケースセット（x2 ピース構成、x2 固定ビス）
- x1 IDC/6Pリボンケーブル
- x1 JP1ジャンパープラグ

> 本体基板単品販売では、付属品はありません。

> USBケーブル（Type-C）は別途御用意ください。

## コンポーネント配置

![](https://askn37.github.io/product/UPDI4AVR/images/Image-2.drawio.svg)

## AVR-ICSP端子配列

![](https://askn37.github.io/product/UPDI4AVR/images/Image-1.drawio.svg)

> 端子ピッチは 2.54mm（100mil）

## 使用例

![IDCリボンケーブルでの接続](https://askn37.github.io/product/UPDI4AVR/images/IMG_3529.png)

![ブレッドボードとの接続](https://askn37.github.io/product/UPDI4AVR/images/IMG_3527.png)

## 関連リンク

- [使い方の手引き](https://askn37.github.io/product/UPDI4AVR/1_Usage.html)
- [avrdude.conf 修正の要点](https://askn37.github.io/product/UPDI4AVR/2_Configuration.html)
- [avrdude コマンド操作例](https://askn37.github.io/product/UPDI4AVR/3_Oparation.html)

## オープンソースソフトウェア・ハードウェア

- [askn37 / multix-zinnia-updi4avr-firmware-builder](https://github.com/askn37/multix-zinnia-updi4avr-firmware-builder) - for UPDI4AVR Firmware
- [回路図](https://askn37.github.io/product/UPDI4AVR/2306_UPDI4AVR/2306_Zinnia-UPDI4AVR-MZU2406B2.pdf)（PDF）
- [基板線図](https://askn37.github.io/product/UPDI4AVR/2306_UPDI4AVR/2306_Zinnia-UPDI4AVR-MZU2306B7_layers.svg)（SVG）
  - [基板表面](https://askn37.github.io/product/UPDI4AVR/2306_UPDI4AVR/2306_Zinnia-UPDI4AVR-MZU2306B7_top.svg)（SVG）
  - [基板裏面](https://askn37.github.io/product/UPDI4AVR/2306_UPDI4AVR/2306_Zinnia-UPDI4AVR-MZU2306B7_bottom.svg)（SVG）
- [ガーバーファイル](https://github.com/askn37/askn37.github.io/tree/main/product/UPDI4AVR/2306_UPDI4AVR/PCBA/)（ZIP）
- [保護ケース](https://github.com/askn37/askn37.github.io/tree/main/product/UPDI4AVR/2306_UPDI4AVR/3DP/)（STL）

## Copyright and Contact

Twitter: [@askn37](https://twitter.com/askn37) \
GitHub: [https://github.com/askn37/](https://github.com/askn37/) \
Product: [https://askn37.github.io/](https://askn37.github.io/)

Copyright (c) askn (K.Sato) multix.jp \
Released under the MIT license \
[https://opensource.org/licenses/mit-license.php](https://opensource.org/licenses/mit-license.php) \
[https://www.oshwa.org/](https://www.oshwa.org/)
