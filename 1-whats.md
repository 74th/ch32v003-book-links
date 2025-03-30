# 1. CH32V003の魅力と基礎知識

## 1.2. CH32V003の魅力と基礎知識

> 50 ピース/ロット CH32V003 工業グレード MCU、RISC-V2A、単線シリアル デバッグ インターフェイス、システム周波数 48MHz - AliExpress<br/>https://ja.aliexpress.com/item/1005005036714708.html

> 32ビットRISC-Vマイコン CH32V003F4P6: 半導体 秋月電子通商-電子部品・ネット通販<br/>https://akizukidenshi.com/catalog/g/g118061/

## 1.3. CH32V003の魅力と基礎知識

### ファームウェア書き込み機

> WCH-LinkEエミュレーター: 開発ツール・ボード 秋月電子通商-電子部品・ネット通販<br/>https://akizukidenshi.com/catalog/g/g118065/

> WCH-LinkE オンライン ダウンロード デバッガー サポート WCH RISC-V アーキテクチャ MCU/SWD インターフェイス ARM チップ 1 シリアル ポートから USB チャネルへ - AliExpress<br/>https://ja.aliexpress.com/item/1005004881582037.html

> WCH-LinkEクローン USB-C,SWD10Pin(¥2,500) [74TH-G016] - 74th Books & Gadgets - BOOTH<br/>https://74th.booth.pm/items/5022813

### ファームウェアライブラリ

#### 公式SDK (C/C++)

> https://github.com/openwch/ch32v003/

#### Arduino (C)

> https://github.com/openwch/arduino_core_ch32

> https://github.com/AlexanderMandera/arduino-wch32v003

#### ch32fun (C)

> Open source minimal stack for the ch32 line of WCH processors, including the ch32v003, a 10¢ 48 MHz RISC-V Microcontroller - as well as many other chips within the ch32v/x line.<br/>https://github.com/cnlohr/ch32fun

#### ch32-rs (Rust)

> Embedded Rust device crates for WCH's RISC-V and Cortex-M microcontrollers<br/>https://github.com/ch32-rs/ch32-rs

> ch32-rs/ch32v00x-hal: HAL for the CH32V003 family of microcontrollers<br/>https://github.com/ch32-rs/ch32v00x-hal

### エディタ・IDE

#### MounRiver Studio 2

> MounRiver Studio<br/>http://www.mounriver.com/

### 書き込みツール

#### WCH-Link Utility

> WCH-LinkUtility.ZIP - 南京沁恒微电子股份有限公司<br/>https://www.wch.cn/downloads/WCH-LinkUtility_ZIP.html

#### wlink

> ch32-rs/wlink: An open source WCH-Link library/command line tool written in Rust.<br/>https://github.com/ch32-rs/wlink

> Releases · ch32-rs/wlink<br/>https://github.com/ch32-rs/wlink/releases

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

```
minichlink -T
```

## 1.4. CH32V003の魅力と基礎知識

### 電源オン時のフラッシュ消去

```
wlink erase --chip CH32V003 --method power-off
```

```
minichlink -u
```

### WCH-LinkEには、DapLinkモードがある

```
# CH32V用のRVモードにする
wlink mode-switch --rv

# ARM用のDAPモードにする
wlink mode-switch --dap
```
