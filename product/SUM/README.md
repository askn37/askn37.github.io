# SERIAL/UPDI-MANAGER (SUM)

> 写真は試作品

Microchip AVRシリーズ（UPDI採用世代）用のプログラミング機能を併せ持つ、USBシリアル通信アダプター

- シリアル通信（UART）とプログラム機能（UPDI）は、DTR信号でホストPCからの自動切り替えが可能
  - 搭載コントローラーによる2回路バス選択切替
  - DTR信号論理は Arduino IDE 互換で、シリアルモニター使用時は DTR=LOW、プログラム時は DTR=HIGH
  - DIPスイッチで自動切替を無効にし、UARTまたは UPDIいずれかのモードに常時固定することも可能
- SerialUPDIプロトコルによる UPDIプログラミング
  - AVRDUDE 7.3以降を推奨[^1][^2]
- Arduinoブートローダープロトコルによる シリアルプログラミング
  - urbootプロトコルも使用可能
- ESD保護対策 USB 2.0 TYPE-Cコネクタと、WCH CH340X シリアル変換チップ
  - Windows Update対応のドライバーインストール
  - macosは標準対応
  - UART時最大2Mbps、プログラム時最大450kbps
- プッシュスイッチまたは RTS信号での、ターゲットAVRリモートリセット操作
  - リセット操作は UPDI通信で行われるため、物理リセットピンが未設定の tinyAVRシリーズに対しても有効[^3]
  - RTS信号での自動リセットにより UARTモードで、ターゲットAVRのブートローダーを Arduino IDEで起動可能[^4]
- ターゲットAVRへの電源供給は、DIPスイッチによる5Vと3.3Vから選択[^5]
- ターゲットコネクタの排他的選択取り付け（要ハンダ付け）により、複数の接続状態が選択可能[^6]
  - 細ピンヘッダによるブレッドボードへの脱着取り付け[^7]
  - 水平または垂直2x3Pコネクタ[^8]による IDC6ピンフラットケーブル接続
  - 水平または垂直1x6Pコネクタ[^9]による 6Pフラットケーブルまたは（バラ線）ジャンパーワイヤー接続
  - ピンヘッダによるユニバーサル基板への直接固定
  - M3マウントホールによる TAMIYAユニバーサルボードへの固定[^10]

[^1]: 起動オプションに`-cserialupdi`と`-xrtsdtr=high`が必要\
[^2]: 7.2以前は AVR-EA/EB/DUシリーズ非対応\
[^3]: UPDIピン機能とRESETピン機能の両方が、FUSEで無効化されている場合は非対応\
[^4]: UPDI固定モードでは RTS信号操作は無視\
[^5]: 5Vは最大1A、3.3Vは最大300mA\
[^6]: 水平2x3Pコネクタと他のコネクタ／ピンヘッダは同時装着不可\
[^7]: 電源レーンは排他選択で正位置／半穴違いに対応\
[^8]: MIL/6P AVR-ICSP UPDI仕様、ただし microUPDI仕様とはUARTモード非互換\
[^9]: SIP/6P 512643配列、28P以上のICパッケージと直結可能な同一配列\
[^10]: 基板サイズ 31x41mm、取付ピッチ 25x35mm

## 対応AVR品種

- megaAVR-0 系統
  - ATmega808 ATmega1608 ATmega3208 __ATmega4808__
  - ATmega809 ATmega1609 ATmega3209 __ATmega4809__
- tinyAVR-0 系統
  - __ATtiny202__ ATtiny402 
  - ATtiny204 ATtiny404 ATtiny804 ATtiny1604 
  - ATtiny406 ATtiny806 ATtiny1606 
  - ATtiny807 ATtiny1607
- tinyAVR-1 系統
  - ATtiny212 __ATtiny412__
  - ATtiny214 ATtiny414 ATtiny814 __ATtiny1614__
  - ATtiny416 ATtiny816 __ATtiny1616__
  - ATtiny417 ATtiny817 ATtiny1617
- tinyAVR-2 系統
  - ATtiny424 __ATtiny824__ ATtiny1624 ATtiny3224
  - ATtiny426 ATtiny826 __ATtiny1626__ __ATtiny3226__
  - ATtiny427 ATtiny827 ATtiny1627 ATtiny3227
- AVR_DA 系統
  - AVR32DA28 AVR64DA28 __AVR128DA28__
  - __AVR32DA32__ AVR64DA32 AVR128DA32
  - AVR32DA48 AVR64DA48 AVR128DA48
  - AVR32DA64 AVR64DA64 AVR128DA64
- AVR_DB 系統
  - AVR32DB28 AVR64DB28 AVR128DB28
  - AVR32DB32 AVR64DB32 __AVR128DB32__
  - AVR32DB48 AVR64DB48 __AVR128DB48__
  - AVR32DB64 AVR64DB64 AVR128DB64
- AVR_DD 系統
  - AVR16DD14 __AVR32DD14__ AVR64DD14
  - AVR16DD20 AVR32DD20 AVR64DD20
  - AVR16DD28 AVR32DD28 AVR64DD28
  - AVR16DD32 AVR32DD32 __AVR64DD32__
- AVR_DU 系統（2024/04時点で未発売）
  - *AVR64DU28*
  - *AVR64DU32*
- AVR_EA 系統
  - AVR16EA28 AVR32EA28 AVR64EA28
  - AVR16EA32 AVR32EA32 __AVR64EA32__
  - AVR16EA48 AVR32EA48 AVR64EA48
- AVR_EB 系統
  - AVR16EB14
  - AVR16EB20
  - AVR16EB28
  - __AVR16EB32__

> __太字__ は動作検証済\
> AVRDUDE 7.2以前は AVR-EA/EB/DUシリーズ非対応

## AVR-ICSP端子配列

![ICSP-Pinout](https://askn37.github.io/product/SUM/Images/Image-1.drawio.svg)

> HV機能未対応である以外は、UPDI4AVRと同一機能\
> 端子ピッチは 2.54mm（100mil）

## Copyright and Contact

Twitter: [@askn37](https://twitter.com/askn37) \
BlueSky Social: [@multix.jp](https://bsky.app/profile/multix.jp) \
GitHub: [https://github.com/askn37/](https://github.com/askn37/) \
Product: [https://askn37.github.io/](https://askn37.github.io/)

Copyright (c) askn (K.Sato) multix.jp \
Released under the MIT license \
[https://opensource.org/licenses/mit-license.php](https://opensource.org/licenses/mit-license.php) \
[https://www.oshwa.org/](https://www.oshwa.org/)
