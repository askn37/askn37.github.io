# SERIAL/UPDI-MANAGER (SUM)

> 写真は試作品

Microchip AVRシリーズ（UPDI対応世代）用のプログラミング機能を併せ持つ、USBシリアル通信アダプター

- 動作電圧任意選択：5V、3.3V
- 自動切替モード（要対応アプリケーション）
  - シリアル通信（UART）、Arduinoシリアルモニター／コンソール対応
  - プログラム機能（UPDI）、SerialUPDIプロトコル対応
- UPDIプログラム機能固定モード
  - SerialUPDIプロトコル専用
- UARTシリアル通信固定モード
  - Arduinoシリアルモニター／コンソール
  - Arduinoブートローダー（自動リセット）機能対応
    - UR-boot（urclock）も使用可能
- ターゲットリセットボタン
  - リセット端子未設定の tinyAVRシリーズもリセット可能
  - UART固定モードでは tinyAVRシリーズでも、Arduino IDE/CLIでブートローダー起動が可能
- 用途に応じて選べるターゲット接続コネクタ（別売：要ハンダ付け）
  - 縦型/横型の MIL/6Pコネクタ - IDC/6P フラットケーブル
  - SIP/6Pコネクタ - Dupontケーブル、ジャンパーワイヤー
  - ピンヘッダ - ブレッドボード脱着、ユニバーサル基板直接取付

## 使用例

- ケース、ピンヘッダ、ケーブル類は頒布物に含まない

## 基本仕様

- 形式：MZU2301A
- 基板サイズ：31x41mm - 突起部含まず
- 動作電圧：5V／3.3V、300mA
- ホスト接続コネクタ：USB 2.0 TYPE-C - 5V専用
- ホストブリッジ（UARTドライバー）：CH340X - 最大通信速度 1Mbps
- 搭載コントローラ：ATtiny1616 - または同等品
- インジケーター：LED 1灯
- マウントホール：M3-4穴、ピッチ25x35mm - TAMIYAユニバーサルボード使用可能

## 頒布物内容

- SUMモジュール基板本体（1個）
- 細ピンヘッダー（要加工／ハンダ付け）

## 使用方法

- CN1：ターゲットコネクタパッド（2x3P）- IDC/6Pフラットケーブル用
- CN2：ターゲットコネクタパッド（1x6P）- ピンヘッダ／ピンソケット用
  - CN1とCN2の同時使用は不可
- CN3：USB TYPE-Cコネクタ - ホストPCに接続（ケーブル別売）
- SW1：ターゲットリセットボタン
  - 押し下げ中、ターゲットMCUのリセット状態を維持（LEDブリンク）
  - プログラミング中には押さないこと（強制中断）
- SW2：DIPスイッチ
  - 1：VDDに対し、ON=5V供給、OFF=3.3V供給
    - 外部からVDDへ任意の電源（3.0V〜5.5V）を供給する場合は、必ずON
  - 2：ON=自動切替モード（次項無効）、OFF=固定選択モード（次項有効）
  - 3：ON=UPDI通信固定、OFF=UART通信固定
- TP1：ファームウェアアップデートパッド（ハーフピッチコンスルー6ピン専用）
- TP2、TP3：ブレッドボード用電源レール端子用パッド
- TP4、TP5：ユニバーサル基板直付用固定／電源端子用パッド

## 対応アプリケーション

- Arduino IDE/CLI - 後述の対応SDKインストールが別途必要
  - AVRDUDEまたはpymcuprogを同梱
  - 一部を除き、UARTあるいはUPDI固定モードのいずれかで使用
- AVRDUDE（少なくとも7.1以上）
  - SerialUPDIプロトコル`-cserialupdi`
    - 自動切替モード対応`-xrtsdtr=high`
    - またはUPDI固定モードで使用
  - Arduinoプロトコル`-carduino`
    - 自動切替不可：UART固定モードで使用
  - urclockプロトコル`-curclock`
    - 自動切替不可：UART固定モードで使用
- pymcuprog
  - 自動切替不可：UPDI固定モードで使用

## AVR-ICSP端子配列

![ICSP-Pinout](https://askn37.github.io/product/SUM/Images/Image-1.drawio.svg)

> HTCR=Host-TxD to Client-RxD\
> HRCT=Host-RxD from Client-TxD\
> HV非対応である以外は、UPDI4AVRと同等\
> 端子ピッチは 2.54mm（100mil）

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
  - AVR32DB28 AVR64DB28 __AVR128DB28__
  - AVR32DB32 AVR64DB32 __AVR128DB32__
  - AVR32DB48 AVR64DB48 __AVR128DB48__
  - AVR32DB64 AVR64DB64 AVR128DB64
- AVR_DD 系統
  - AVR16DD14 __AVR32DD14__ AVR64DD14
  - AVR16DD20 AVR32DD20 AVR64DD20
  - AVR16DD28 AVR32DD28 __AVR64DD28__
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

> 掲出以外の形式は非対応、__太字__ は動作検証済\
> AVRDUDE 7.2以前は AVR-EA/EB/DUシリーズ非対応

## 関連リンク

- オープンハードウェア資料
  - [https://github.com/askn37/askn37.github.io/tree/main/product/SUM](https://github.com/askn37/askn37.github.io/tree/main/product/SUM)
  - 回路図
  - PCB製造用ガーバーデータ
  - PCBA製造用BOM/CPLファイル（JLCPCB規格）
  - 専用ケース図面、3Dプリンタ製造ファイル
- オープンソースファームウェア
  - [__MultiX Zinnia UPDI4AVR Firmware Builder__](https://github.com/askn37/multix-zinnia-updi4avr-firmware-builder/) に付属
  - ボード／スケッチ選択: SERIAL/UPDI-MANAGER (SUM)
- AVRプログラミングツール（SerialUPDI対応）
  - [AVRDUDE](https://github.com/avrdudes/avrdude/releases)
  - [pymcuprog](https://github.com/microchip-pic-avr-tools/pymcuprog)
- Arduino IDE/CLI 用 AVRシリーズ SDK（SerialUPDI対応）
  - [MCUdude/MegaCoreX](https://github.com/MCUdude/MegaCoreX) (megaAVR系統専用、Arduino API互換)
  - [SpenceKonde/megaTinyCore](https://github.com/SpenceKonde/megaTinyCore) (tinyAVR系統専用、Arduino API互換)
  - [SpenceKonde/DxCore](https://github.com/SpenceKonde/DxCore) (Microchip AVR系統専用、Arduino API互換)
  - [askn37/megaAVR](https://github.com/askn37/multix-zinnia-sdk-megaAVR) (tinyAVRおよびmegaAVR系統用、Standard C/C++)（自動切替モード対応）
  - [askn37/modernAVR](https://github.com/askn37/multix-zinnia-sdk-modernAVR) (Microchip AVR系統専用、Standard C/C++)（自動切替モード対応）

> 各SDKは、Arduino IDE 1.8.19での使用を推奨。2.x では正常に使用できない可能性が高い（非公認サードパーティソフトウェア全般）

## Copyright and Contact

Twitter: [@askn37](https://twitter.com/askn37) \
BlueSky Social: [@multix.jp](https://bsky.app/profile/multix.jp) \
GitHub: [https://github.com/askn37/](https://github.com/askn37/) \
Product: [https://askn37.github.io/](https://askn37.github.io/)

Copyright (c) askn (K.Sato) multix.jp \
Released under the MIT license \
[https://opensource.org/licenses/mit-license.php](https://opensource.org/licenses/mit-license.php) \
[https://www.oshwa.org/](https://www.oshwa.org/)
