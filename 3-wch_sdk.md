# 3. WCH SDKでの開発

## 3.1. MounRiver Studio 2を使った環境セットアップ

### インストール

> MounRiver Studio<br/>http://www.mounriver.com/download

### プロジェクトセットアップ

## 3.2. ソースコードの構成

```c
//#define SYSCLK_FREQ_8MHz_HSI    8000000
//#define SYSCLK_FREQ_24MHZ_HSI   HSI_VALUE
#define SYSCLK_FREQ_48MHZ_HSI   48000000
//#define SYSCLK_FREQ_8MHz_HSE    8000000
//#define SYSCLK_FREQ_24MHz_HSE   HSE_VALUE
//#define SYSCLK_FREQ_48MHz_HSE   48000000
```

## 3.3. MounRiver Studioでのビルドと書き込み

## 3.4. MounRiver Studioでのダウンロード設定

## 3.5. SDI-Printによるログ出力

```c
int main(void)
{
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_1);
    SystemCoreClockUpdate();
    Delay_Init();
#if (SDI_PRINT == SDI_PR_OPEN)
    SDI_Printf_Enable();
#else
    USART_Printf_Init(115200);
#endif
```

## 3.6. PlatformIOを使った環境セットアップ

> https://github.com/Community-PIO-CH32V/platform-ch32v.git

> examples/blinky-none-os/src/main.c<br/>https://github.com/Community-PIO-CH32V/platform-ch32v/blob/develop/examples/blinky-none-os/src/main.c

```c
// src/main.c
#include "stdio.h"

// 以下のどちらかをmain関数の中に追加

// SDI-Printfの有効化
SDI_Printf_Enable();

// USART1 printfの有効化
USART_Printf_Init(115200)
```

```ini
// platformio.ini
[env:genericCH32V003F4P6]
platform = ch32v
board = genericCH32V003F4P6
framework = noneos-sdk
monitor_speed = 115200
```

## 3.7. PlatformIOでのビルドと書き込み

## 3.8. PlatformIOでのSDI-Printの有効化

```ini
; platformio.ini
[env:genericCH32V003F4P6]
platform = ch32v
board = genericCH32V003F4P6
framework = noneos-sdk
monitor_speed = 115200
; ビルドフラグ
build_flags = -DSDI_PRINT=1
; 書き込みと同時にSDI-Printの有効化
upload_protocol = wlink
upload_command = wlink flash --chip CH32V003 --enable-sdi-print $SOURCE
```
