# 15. NeoPixelとOLED SSD1306の制御

## 15.1. NeoPixel

### Arduino

> https://github.com/74th/ch32v003-book-code/tree/main/neopixel-arduino_core_ch32

```c
// NeoPixel制御関数
// digital_pin: NeoPixelのデータピン
// data: NeoPixelのデータ (GRB順、各8ビット)
// led_num: NeoPixelのLEDの数
void write_neopixel(uint32_t digital_pin, uint8_t *data, uint32_t led_num);
```

```c
void setup()
{
  pinMode(PC0, OUTPUT);
}
```

```c
#define LED_NUM 6

// 各LEDの色データを格納する配列 (GRB)
uint8_t data[LED_NUM * 3];

for (int i = 0; i < LED_NUM; i++) {
  switch (i % 3) {
    case 0: // Green
      data[i * 3] = 0x20;
      data[i * 3 + 1] = 0x00;
      data[i * 3 + 2] = 0x00;
      break;
    case 1: // Red
      data[i * 3] = 0x00;
      data[i * 3 + 1] = 0x20;
      data[i * 3 + 2] = 0x00;
      break;
    case 2: // Blue
      data[i * 3] = 0x00;
      data[i * 3 + 1] = 0x00;
      data[i * 3 + 2] = 0x20;
      break;
  }
}
```

```c
write_neopixel(PC0, data, LED_NUM);
```

### ch32fun

> https://github.com/74th/ch32v003-book-code/tree/main/neopixel-ch32fun

```c
// 各LEDの色データを格納する配列
uint32_t data[LED_NUM];

// 指定されたLED番号に対応する色データを返すコールバック関数
// 戻り値は uint32_t 型で、(Green << 16) | (Red << 8) | Blue の形式
uint32_t WS2812BLEDCallback(int ledno)
{
  return data[ledno];
}
```

> ch32fun/examples/ws2812bdemo/color_utilities.h<br/>https://github.com/cnlohr/ch32fun/blob/master/examples/ws2812bdemo/⏎<br/>color_utilities.h

```c
static uint32_t EHSVtoHEX( uint8_t hue, uint8_t sat, uint8_t val );
```

```c
#include "ch32fun.h"
#include "ws2812b_dma_spi_led_driver.h"
#include "color_utilities.h"

#define LED_NUM 6
uint32_t data[LED_NUM];

// 指定されたLED番号に対応する色データを返すコールバック関数
// 戻り値は uint32_t 型で、(Green << 16) | (Red << 8) | Blue の形式
uint32_t WS2812BLEDCallback(int ledno)
{
  return data[ledno];
}

int main()
{
  int k;
  SystemInit();

  // 初期化
  WS2812BDMAInit();

  uint32_t count = 0;

  while (1)
  {
    // 転送終了まで待つ
    while (WS2812BLEDInUse)
      ;

    for (int i = 0; i < LED_NUM; i++)
    {
      // HSV形式で色を指定し、ライブラリ形式に変換
      data[i] = EHSVtoHEX((count + i) * 16 & 0xff, 255, 3);
    }

    // 転送開始
    WS2812BDMAStart(LED_NUM);

    count++;
    Delay_Ms(100);
  }
}
```

### WCH SDK

## 15.2. OLED SSD1306

### Arduino

> https://github.com/74th/ch32v003-book-code/tree/main/i2c_oled-arduino_core_ch32

> OLED SSD1306 - SH1106 - Arduino Reference<br/>https://reference.arduino.cc/reference/en/libraries/oled-ssd1306-sh1106/

```c
#include <oled.h>

// OLEDオブジェクトの定義
OLED oled = OLED(
  PC1, // SDAピン
  PC2, // SCLピン
  NO_RESET_PIN, // リセットピン
  OLED::W_128, // 幅
  OLED::H_32, // 高さ
  OLED::CTRL_SSD1306, // コントローラ
  0x3c // I2Cアドレス
);

void setup()
{
  // OLEDの初期化
  oled.begin();
}
```

```c
// 描画バッファをクリア
oled.clear();
// 座標を指定して文字列を描画
oled.draw_string(60, 8, "@74th");
// カーソル位置を設定してprintf形式で文字列を描画
oled.setCursor(60,16);
oled.printf("CH32V003");
// 矩形を描画
oled.draw_rectangle(50, 6, 120, 26);
// 描画バッファの内容をOLEDに転送
oled.display();
```

> image2cpp<br/>https://javl.github.io/image2cpp/

```c
// モノクロビットマップデータ (縦方向、1ピクセル1ビット)
static const uint8_t BUNCHO_74TH[] =
  {
    0xff, 0xff, 0xff, 0xff,
    // ... (データ省略) ...
    0xb7, 0xb7, 0xbf, 0xff};

```

```c
// ビットマップ画像を描画バッファに描画 (PROGMEMから読み込み)
oled.draw_bitmap_P(0, 0, 32, 32, BUNCHO_74TH);
// 描画バッファの内容をOLEDに転送
oled.display();
```

### ch32fun

> https://github.com/74th/ch32v003-book-code/tree/main/i2c_oled-ch32fun

```c
// 使用するOLEDのサイズに合わせて定義
// #define SSD1306_64X32
// #define SSD1306_128X32
#define SSD1306_128X64

#include "ch32fun.h"
#include <stdio.h>
#include "ssd1306_i2c.h"
#include "ssd1306.h"

// モノクロビットマップデータ (例)
const unsigned char BUNCHO[] = { /* ... データ ... */ };

int main()
{
	SystemInit();
  // I2Cの初期化
  ssd1306_i2c_init();
  // SSD1306コントローラの初期化
  ssd1306_init();

  //... (描画処理など)
}
```

```c
// 描画バッファをクリア (0で埋める)
ssd1306_setbuf(0);

// ビットマップ画像を描画バッファに描画
ssd1306_drawImage(0, 0, BUNCHO, 64, 64, 0);
// 文字列を描画バッファに描画
ssd1306_drawstr(80, 20, "@74th", 1);
ssd1306_drawstr(80, 34, "CH32V", 1);
ssd1306_drawstr(96, 44, "003", 1);
// 矩形を描画バッファに描画
ssd1306_drawRect(75, 15, 128 - 70 - 5, 45, 1);

// 描画バッファの内容をOLEDに転送
ssd1306_refresh();
```

### WCH SDK
