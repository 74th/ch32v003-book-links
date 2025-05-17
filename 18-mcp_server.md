# 18. 本書のMCP Serverを使おう

### CH32V003開発ガイドブックMCP Serverとは

```
名前: read_ch32v003_guide_book

概要:
CH32V003の実装方法を説明したドキュメントを返します。
CH32V003を開発する際には参照してください。
CH32V003の開発フレームワークには、WCH SDKとch32funがあります。
利用する場合には指定してください。
作りたいものに応じて、ADC、DMA、GPIO、I2C、neopixel、OLED、PWM、SleepStandby、SPI、
    Timer、UART、watchdogtimerを指定してください。

引数:
1. framework: フレームワーク(WCH SDK, ch32fun)
2. item: 内容(ADC、DMA、GPIO、I2C、neopixel、OLED、PWM、SleepStandby、SPI、Timer、
             UART、watchdogtimer)

返答内容:
以下の情報を含むマークダウンテキスト
- ch32fun、WCH SDKのフレームワークの章
- itemの章
- itemのサンプルコード
```

> CH32V003 MCUのファームウェアの開発です
>
> フレームワークはch32funを使います
>
> ch32funでの実装方法は、Tool read_ch32v003_guide_bookを参照してください

> UARTを使う準備をして。ボーレート115200で1バイト読み書きするコードも用意して
>
> PC0、PC1を使いたいよ

> SPIで8バイトずつ送受信するコードを作成して

### ダウンロード

> ch32v003-guidebook-mcpserver
>
> https://github.com/74th/ch32v003-guidebook-mcpserver

### 書籍購入者向けには本書のテキストも
