# 5. GPIO

## 5.1. Arduino

> https://github.com/74th/ch32v003-book-code/tree/main/blink_and_read-arduino_core_ch32

```c
void setup()
{
  // 出力
  pinMode(PC0, OUTPUT);

  // 入力
  // INPUT: フローティング
  // INPUT_PULLUP: プルアップ
  // INPUT_PULLDOWN: プルダウン
  pinMode(PA1, INPUT_PULLUP);
}
```

```c
void loop() {
  // ボタンが押されるとLOWになる
  bool btn = digitalRead(PA1);
  if (!btn)
  {
    delay(1000);
    return;
  }

  digitalWrite(PC0, HIGH);
  delay(500);
  digitalWrite(PC0, LOW);
  delay(500);
}
```

## 5.2. CH32V003のGPIOの基本

## 5.3. ch32fun

> https://github.com/74th/ch32v003-book-code/tree/main/blink_and_read-ch32fun

### 標準のマクロ

#### クロックの供給

```c
// GPIO有効化
funGpioInitAll();
```

#### ピンの設定

```c
// PC0 出力
funPinMode(LED_PIN, GPIO_Speed_10MHz | GPIO_CNF_OUT_PP);

// PA1 入力
funPinMode(BUTTON_PIN, GPIO_CNF_IN_PUPD);
// プルアップ/プルダウンの設定は、funDigitalWrite() で行います。
funDigitalWrite(BUTTON_PIN, FUN_HIGH);
```

```c
// ピンの設定
// 第2引数には以下を設定
// - GPIO_CNF_IN_FLOATING: フローティング
// - GPIO_CNF_IN_PUPD: プルアップ、プルダウン
// - GPIO_CNF_OUT_PP: プッシュプル出力
// - GPIO_CNF_OUT_OD: オープンドレイン出力
// 第2引数には出力の場合はクロックスピードも | で一緒に指定
// - GPIO_Speed_2MHz
// - GPIO_Speed_10MHz
// - GPIO_Speed_50MHz
void funPinMode(pin, mode);

// GPIOのプルアップ、プルダウンの指定
// 第2引数には以下を設定
// - FUN_HIGH(1): プルアップ
// - FUN_LOW(0): プルダウン
void funDigitalWrite(pin, value);
```

#### Read/Write

```c
while(1)
{
  // PA1 のボタンが押されていれば点灯しない
  uint8_t btn = funDigitalRead(PA1);
  if (btn == FUN_LOW)
  {
    Delay_Ms(1000);
    continue;
  }

  // LEDの点灯
  funDigitalWrite(PC0, FUN_HIGH);
  Delay_Ms(500);
  funDigitalWrite(PC0, FUN_LOW);
  Delay_Ms(500);
}
```

```c
// GPIOのWrite
void funDigitalWrite(pin, value);

// GPIOのRead
uint16_t funDigitalRead(pin);
```

### スタティックマクロ extralibs/ch32v003_GPIO_branchless.h

```c
#include "ch32v003_GPIO_branchless.h"
```

```c
// LED PC0
#define LED_PIN GPIOv_from_PORT_PIN(GPIO_port_C, 0)
// ボタン PA1
#define BUTTON_PIN GPIOv_from_PORT_PIN(GPIO_port_A, 1)
```

```c
GPIO_port_enable(GPIO_port_C);
GPIO_port_enable(GPIO_port_A);
```

```c
// PC0 出力
GPIO_pinMode(LED_PIN, GPIO_pinMode_O_pushPull, GPIO_Speed_10MHz);

// PA1 入力
GPIO_pinMode(BUTTON_PIN, GPIO_pinMode_I_pullUp, GPIO_Speed_In);
```

```c
// PA1 のボタンが押されていれば点灯しない
uint8_t btn = GPIO_digitalRead(BUTTON_PIN);
if (btn == low)
{
  Delay_Ms(1000);
  continue;
}

GPIO_digitalWrite_high(LED_PIN);
Delay_Ms(500);
GPIO_digitalWrite_low(LED_PIN);
Delay_Ms(500);
```

```h
// GPIOの初期化（クロック供給）
void GPIO_port_enable(port);

// ピンの設定
// 第2引数には以下のモード設定
// - GPIO_pinMode_I_floating: フローティング入力
// - GPIO_pinMode_I_pullUp: プルアップ付き入力
// - GPIO_pinMode_I_pullDown: プルダウン付き入力
// - GPIO_pinMode_O_pushPull: プッシュプル出力
// - GPIO_pinMode_O_openDrain: オープンドレイン出力
// 第3引数には以下のGPIOのスピード設定
// - GPIO_Speed_In: 入力の場合
// - GPIO_Speed_2MHz: 出力の場合2MHz
// - GPIO_Speed_10MHz: 出力の場合10MHz
// - GPIO_Speed_50MHz: 出力の場合50MHz
void GPIO_pinMode(pin, pin_mode, gpio_speed);

// GPIOのRead
uint8_t GPIO_digitalRead(pin);

// GPIOのWrite
void GPIO_digitalWrite_high(pin);
void GPIO_digitalWrite_low(pin);
```

## 5.4. WCH SDK

> https://github.com/74th/ch32v003-book-code/tree/main/blink_and_read-wch_sdk

### クロックの供給

```c
// GPIOにクロック供給
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOD, ENABLE);
```

### GPIOの設定

```c
// LEDのピン番号 PC0
#define LED_PIN GPIO_Pin_0
// ボタンのピン番号 PA1
#define BUTTON_PIN GPIO_Pin_1

GPIO_InitTypeDef GPIO_InitStructure = {0};

// 出力
GPIO_InitStructure.GPIO_Pin = LED_PIN;
// GPIO_Mode_Out_PP に設定
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
GPIO_Init(GPIOC, &GPIO_InitStructure);

// 入力
GPIO_InitStructure.GPIO_Pin = BUTTON_PIN;
// GPIO_Mode_IPU: プルアップ
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
// GPIO_Mode_IPD: プルダウン
// GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPD;
// GPIO_Mode_IN_FLOATING: フローティング
// GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;
GPIO_Init(GPIOA, &GPIO_InitStructure);
```

### 入出力

```c
while (1)
{
    // ボタンの状態を読み取り。0、1のbool値で受け取る
    uint8_t btn = GPIO_ReadInputDataBit(GPIOA, BUTTON_PIN);
    if (btn == 0)
    {
        Delay_Ms(1000);
        continue;
    }

    // LEDの点灯。0、1のbool値で設定する
    Delay_Ms(500);
    GPIO_WriteBit(GPIOC, LED_PIN, Bit_SET);
    Delay_Ms(500);
    GPIO_WriteBit(GPIOC, LED_PIN, Bit_RESET);
}
```

```c
// ピン単位でReadする
uint8_t GPIO_ReadInputDataBit(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);

// GPIO単位でReadする。戻り値の各bitが各ピンの状態となる。
uint16_t GPIO_ReadInputData(GPIO_TypeDef *GPIOx);

// ピン単位でWriteする
void GPIO_WriteBit(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, BitAction BitVal);

// GPIO単位でWriteする。PortValの各bitが各ピンの状態となる。
void GPIO_Write(GPIO_TypeDef *GPIOx, uint16_t PortVal);
```

## 5.5. レジスタ操作

> https://github.com/74th/ch32v003-book-code/tree/main/blink_and_read-register

### クロックの供給

```c
RCC->APB2PCENR |= RCC_APB2Periph_GPIOA | RCC_APB2Periph_GPIOC;
```

### ピンの設定

```c
// PC0 LED
#define LED_PIN_NO 0
// PA1 ボタン
#define BUTTON_PIN_NO 1

// PC0 出力
// 設定リセット
GPIOC->CFGLR &= ~(0xf << (4 * LED_PIN_NO));
// GPIO_CNF_OUT_PP 出力プッシュプル
GPIOC->CFGLR |= (GPIO_Speed_10MHz | GPIO_CNF_OUT_PP) << (4 * LED_PIN_NO);

// PA1 入力
// 設定リセット
GPIOA->CFGLR &= ~(0xf << (4 * BUTTON_PIN_NO));
// GPIOA->CFGLR GPIO設定
//   GPIO_CNF_IN_FLOATING フローティング
//   GPIO_CNF_IN_PUPD プルアップorプルダウン
GPIOA->CFGLR |= (GPIO_Speed_In | GPIO_CNF_IN_PUPD) << (4 * BUTTON_PIN_NO);
```

```c
// PA1をプルアップ
GPIOA->OUTDR |= 0x1 << BUTTON_PIN_NO;

// PA1をプルダウン
GPIOA->OUTDR &= ~(1 << BUTTON_PIN_NO);
```

### GPIOのRead

```c
uint8_t btn = (GPIOA->INDR & (1 << BUTTON_PIN_NO)) == (1 << BUTTON_PIN_NO);
if (btn == 0)
{
  printf("button pressed\r\n");
  Delay_Ms(1000);
  continue;
}
```

### GPIOのWrite

```c
// PC0 に1を出力
GPIOC->BSHR |= 1 << LED_PIN_NO;
Delay_Ms(500);

// PC0 に0を出力
GPIOC->BSHR |= 1 << (16 + LED_PIN_NO);
Delay_Ms(500);
```
