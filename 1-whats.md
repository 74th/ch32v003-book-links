# 1. CH32V003の魅力と基礎知識

## 1.1. CH32V003とは

## 1.2. CH32V003のMCU単体の入手方法

> 50 ピース/ロット CH32V003 工業グレード MCU、RISC-V2A、単線シリアル デバッグ インターフェイス、システム周波数 48MHz - AliExpress<br/>https://ja.aliexpress.com/item/1005005036714708.html

> 32ビットRISC-Vマイコン CH32V003F4P6: 半導体 秋月電子通商-電子部品・ネット通販<br/>https://akizukidenshi.com/catalog/g/g118061/

## 1.3. 最小限の回路

## 1.4. ファームウェア開発の方法

### ファームウェアプログラマ

> WCH-LinkEエミュレーター: 開発ツール・ボード 秋月電子通商-電子部品・ネット通販<br/>https://akizukidenshi.com/catalog/g/g118065/

> メイン周波数48MHz,1本/キット,ch32v003f4p6 qingke RISC-V2A - AliExpress<br/>https://ja.aliexpress.com/item/1005004895791296.html

> WCH-LinkEクローン USB-C,SWD10Pin(¥2,500) [74TH-G016] - 74th Books & Gadgets - BOOTH<br/>https://74th.booth.pm/items/5022813

### ファームウェアライブラリ

#### 公式SDK (C/C++)

> https://github.com/openwch/ch32v003/

#### Arduino Core CH32 (C)

> https://github.com/openwch/arduino_core_ch32

#### ch32fun (C)

> Open source minimal stack for the ch32 line of WCH processors, including the ch32v003, a 10¢ 48 MHz RISC-V Microcontroller - as well as many other chips within the ch32v/x line.<br/>https://github.com/cnlohr/ch32fun

#### ch32-rs (Rust)

> Embedded Rust device crates for WCH's RISC-V and Cortex-M microcontrollers<br/>https://github.com/ch32-rs/ch32-rs

> ch32-rs/ch32v00x-hal: HAL for the CH32V003 family of microcontrollers<br/>https://github.com/ch32-rs/ch32v00x-hal

#### arduino-wch32v003 (C)

> https://github.com/AlexanderMandera/arduino-wch32v003

### エディタ・IDE

#### MounRiver Studio 2

> MounRiver Studio<br/>http://www.mounriver.com/

#### Arduino IDE

#### PlatformIO (VS Code)

#### ch32funはVS Codeでも開発できる

### ライブラリ、開発環境の使い分け

### 書き込みツール

#### WCH-Link Utility

> WCH-LinkUtility.ZIP - 南京沁恒微电子股份有限公司<br/>https://www.wch.cn/downloads/WCH-LinkUtility_ZIP.html

#### OpenOCD

#### wlink

> ch32-rs/wlink: An open source WCH-Link library/command line tool written in Rust.<br/>https://github.com/ch32-rs/wlink

> Releases · ch32-rs/wlink<br/>https://github.com/ch32-rs/wlink/releases

```sh
wlink flash ch32v003.bin
```

#### minichlink

```sh
git clone https://github.com/cnlohr/ch32fun.git
cd ch32fun/minichlink
make
```

```
minichlink -w ch32v003.bin flash -b
```

#### 使い分け

## 1.5. ひとまず動く開発ボードが欲しい

### WCH社公式評価ボード

> メイン周波数48MHz,1本/キット,ch32v003f4p6 qingke RISC-V2A - AliExpress<br/>https://ja.aliexpress.com/item/1005004895791296.html

### 筆者作成のProMicro型ボード

> CH32V003 ProMicroサイズ開発ボードキット (3個入¥1,500) [74TH-G015] - 74th Books & Gadgets - BOOTH<br/>https://74th.booth.pm/items/4645948

> ch32v-dev-boards/ch32v003-promicro at main · 74th/ch32v-dev-boards<br/>https://github.com/74th/ch32v-dev-boards/tree/main/ch32v003-promicro

### UIAPduino Pro Micro CH32V003

> UIAPduino Pro Micro CH32V003 V1.4 - UIAP - BOOTH<br/>https://uiap.booth.pm/items/5845791

### その他の開発ボード

> WA00007 [Weact]CH32V003F4U6マイコンボード — ビット・トレード・ワン 公式オンラインショップ BTOS<br/>https://btoshop.jp/products/wa00007

## 1.6. CH32V003開発で知っておくと良いこと

### SOP-8のCH32V003J4M6のUSART TXとSWIOのピンの共用

### 電源オン時のフラッシュ消去

#### WCH-Link Utilityの場合

#### wlinkの場合

```
wlink erase --chip CH32V003 --method power-off
```

#### minichlinkの場合

```
minichlink -u
```

### CPU命令セットに整数の乗算・除算がないが、ソフトウェアで使える

### WCH-LinkEには、DapLinkモードがある

```
# CH32V用のRVモードにする
wlink mode-switch --rv

# ARM用のDAPモードにする
wlink mode-switch --dap
```

## 1.7. 後継、類似モデル

### CH32V002、CH32V006

### 他のCH32シリーズ

### USB、WiFi対応CH570
