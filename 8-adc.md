# 8. ADC

## 8.1. CH32V003のADC

## 8.2. サンプルコードの回路

## 8.3. Arduino

> https://github.com/74th/ch32v003-book-code/tree/main/adc-arduino_core_ch32

```c
pinMode(PA1, INPUT_ANALOG);
pinMode(PA2, INPUT_ANALOG);
```

```c
// 0～1023の範囲で値を取得
uint32_t x = analogRead(PA1);
uint32_t y = analogRead(PA2);
```

## 8.4. ch32fun

> https://github.com/74th/ch32v003-book-code/tree/main/adc-ch32fun

### 標準ライブラリ

```c
// GPIO有効化
funGpioInitA();
// ADC初期化
funAnalogInit();

// ピン設定
funPinMode(PA1, GPIO_CNF_IN_ANALOG);
```

```c
uint16_t x = funAnalogRead(1);
```

### スタティックマクロch32v003_GPIO_branchless.h

```c
#define VRX_PIN GPIOv_from_PORT_PIN(GPIO_port_A, 1)
```

```c
// GPIO有効化
GPIO_port_enable(GPIO_port_A);
// ADC初期化
GPIO_ADCinit();

// ピン設定
GPIO_pinMode(VRX_PIN, GPIO_pinMode_I_analog, GPIO_Speed_In);
```

```c
uint16_t x = GPIO_analogRead(GPIO_Ain1_A1);
```

## 8.5. WCH SDKで1回ADCを変換する

> https://github.com/74th/ch32v003-book-code/tree/main/adc-wch_sdk

### クロック供給の有効化

```c
RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1 | RCC_APB2Periph_GPIOA, ENABLE);
```

### GPIOピンをADC入力用に設定

```c
GPIO_InitTypeDef GPIO_InitStructure = {0};

GPIO_InitStructure.GPIO_Pin = GPIO_Pin_A1;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
GPIO_Init(GPIOA, &GPIO_InitStructure);
```

### ADCの設定

```c
ADC_InitTypeDef ADC_InitStructure = {0};

ADC_DeInit(ADC1);
// 独立モード（各チャンネルの変換は独立して行う）
ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;
ADC_InitStructure.ADC_ScanConvMode = DISABLE;
// 連続変換モード無効
ADC_InitStructure.ADC_ContinuousConvMode = DISABLE;
// 外部トリガ無効
ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
// 10bitの変換結果を16bitレジスタに格納
// データアライメント（右揃え）
ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
// 変換するチャンネル数
ADC_InitStructure.ADC_NbrOfChannel = 1;

// ADC設定を適用
ADC_Init(ADC1, &ADC_InitStructure);
```

### ADCの有効化とADCキャリブレーションの実行

```c
// ADCキャリブレーション電圧を設定
ADC_Calibration_Vol(ADC1, ADC_CALVOL_50PERCENT);
// ADC有効化
ADC_Cmd(ADC1, ENABLE);

// キャリブレーションリセット
ADC_ResetCalibration(ADC1);
while (ADC_GetResetCalibrationStatus(ADC1))
    ;
// キャリブレーション開始
ADC_StartCalibration(ADC1);
while (ADC_GetCalibrationStatus(ADC1))
    ;
```

### ADC変換チャンネルと順序の設定、変換の実行、結果の取得

```c
// 変換順序1番目にADC1_CH1を設定
// サンプリング時間を241サイクルに設定
ADC_RegularChannelConfig(ADC1, ADC_Channel_1, 1, ADC_SampleTime_241Cycles5);
```

```c
// ソフトウェアトリガで変換開始
ADC_SoftwareStartConvCmd(ADC1, ENABLE);
// 変換完了(EOCフラグ)まで待機
while (!ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC))
    ;
```

```c
uint16_t x = ADC_GetConversionValue(ADC1);
```

## 8.6. DMAを用いた連続ADC変換

> https://github.com/74th/ch32v003-book-code/tree/main/adc_dma-wch_sdk

### 変換結果格納用バッファ

```c
uint16_t adc_buf[2] = {0};
```

### GPIO、ADC、DMAへのクロック供給有効化

```c
// GPIOとADCにクロック供給
RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1 | RCC_APB2Periph_GPIOA, ENABLE);
// DMAにクロック供給
RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);
```

### GPIOピンをADC入力用に設定

### ADCの設定

```c
ADC_InitTypeDef ADC_InitStructure = {0};

// ADC初期化
ADC_DeInit(ADC1);
ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;
// スキャンモード有効
ADC_InitStructure.ADC_ScanConvMode = ENABLE;
// 連続変換モード有効
ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;
// 外部トリガ無効
ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
// 変換するチャンネル数
ADC_InitStructure.ADC_NbrOfChannel = 2;
// ADC設定を適用
ADC_Init(ADC1, &ADC_InitStructure);
```

### ADC変換チャンネルと順序の設定

```c
ADC_RegularChannelConfig(ADC1, ADC_Channel_1, 1, ADC_SampleTime_241Cycles);
ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 2, ADC_SampleTime_241Cycles);
```

### DMAの設定

```c
DMA_InitTypeDef DMA_InitStructure = {0};

// DMAチャンネル1初期化
DMA_DeInit(DMA1_Channel1);

// 転送方向: Peripheral(ADC) -> メモリ
DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralSRC;
DMA_InitStructure.DMA_PeripheralBaseAddr = (uint32_t)&ADC1->RDATAR;
DMA_InitStructure.DMA_MemoryBaseAddr = (uint32_t)adc_buf;
// 転送データ数（バッファサイズ）
DMA_InitStructure.DMA_BufferSize = sizeof(adc_buf) / sizeof(adc_buf[0]);
// メモリアドレスインクリメント有効
DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Disable;
DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;
// Peripheralデータサイズ: 16bit (HalfWord)
DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_HalfWord;
// メモリデータサイズ: 16bit (HalfWord)
DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_HalfWord;
// DMAモード: 循環モード
DMA_InitStructure.DMA_Mode = DMA_Mode_Circular;
// 優先度: 最高
DMA_InitStructure.DMA_Priority = DMA_Priority_VeryHigh;
DMA_InitStructure.DMA_M2M = DMA_M2M_Disable;

// DMA設定を適用
DMA_Init(DMA1_Channel1, &DMA_InitStructure);
```

### ADCの有効化とADCキャリブレーションの実行

```c
// ADC有効化とキャリブレーション
ADC_Calibration_Vol(ADC1, ADC_CALVOL_50PERCENT);
ADC_Cmd(ADC1, ENABLE);

ADC_ResetCalibration(ADC1);
while (ADC_GetResetCalibrationStatus(ADC1))
    ;
ADC_StartCalibration(ADC1);
while (ADC_GetCalibrationStatus(ADC1))
    ;
```

### ADCとDMAの連携設定と、DMAの有効化

```c
// ADCのDMAリクエスト有効化
ADC_DMACmd(ADC1, ENABLE);
// DMAチャンネル1有効化
DMA_Cmd(DMA1_Channel1, ENABLE);
```

### ADC変換の開始（ソフトウェアトリガ）

```c
// ソフトウェアトリガで変換開始
ADC_SoftwareStartConvCmd(ADC1, ENABLE);
```

## 8.7. レジスタ直接操作による単発ADC変換

> https://github.com/74th/ch32v003-book-code/tree/main/adc-register

### GPIOとADCへのクロック供給有効化

```c
RCC->APB2PCENR |= RCC_APB2Periph_GPIOA | RCC_APB2Periph_ADC1;
```

### GPIOピンをADC入力用に設定

```c
// PA1をADC用に設定
GPIOA->CFGLR &= ~(0xf << (4 * 1));
GPIOA->CFGLR |= (GPIO_Speed_In | GPIO_CNF_IN_ANALOG) << (4 * 1);
```

### ADCの設定

```c
// シーケンスレジスタ初期化
ADC1->RSQR1 = 0;
ADC1->RSQR2 = 0;
ADC1->RSQR3 = 0;

// ソフトウェアトリガを選択
ADC1->CTLR2 |= ADC_EXTSEL;
```

### ADC有効化とキャリブレーション

```c
// ADC有効化 (ADON=1)
ADC1->CTLR2 |= ADC_ADON;

// キャリブレーションリセット (RSTCAL=1)
ADC1->CTLR2 |= ADC_RSTCAL;
while (ADC1->CTLR2 & ADC_RSTCAL)
  ;
// キャリブレーション開始 (CAL=1)
ADC1->CTLR2 |= ADC_CAL;
while (ADC1->CTLR2 & ADC_CAL)
  ;
```

### ADC変換チャンネルと順序の設定

```c
// シーケンス1番目にチャンネル1 (ADC_Channel_1) を設定
ADC1->RSQR3 = ADC_Channel_1 << (5 * 0);

// チャンネル1のサンプリング時間を241サイクルに設定
ADC1->SAMPTR2 = ADC_SampleTime_241Cycles5 << (3 * 1); // CH1はSMP1[2:0]

// 変換するチャンネル数を1に設定
ADC1->RSQR1 = (1 - 1) << 20;
```

### ADC変換の開始と結果取得

```c
// ソフトウェアトリガで変換開始
ADC1->CTLR2 |= ADC_SWSTART;
// 変換完了(EOC=1)まで待機
while (!(ADC1->STATR & ADC_EOC))
  ;

uint16_t x = ADC1->RDATAR;
```

## 8.8. レジスタ直接操作によるDMAを用いた連続ADC変換

> https://github.com/74th/ch32v003-book-code/tree/main/adc_dma-register

### GPIO、DMA、ADCへのクロック供給有効化

```c
RCC->APB2PCENR |= RCC_APB2Periph_GPIOA | RCC_APB2Periph_ADC1;
RCC->AHBPCENR |= RCC_AHBPeriph_DMA1;
```

### GPIOピンをADC入力用に設定

### ADCの設定

```c
// スキャンモード有効
ADC1->CTLR1 |= ADC_SCAN;
ADC1->CTLR2 |=
  // 連続変換モード有効
  ADC_CONT |
  // ソフトウェアトリガを選択
  ADC_EXTSEL |
  // DMAリクエスト有効化
  ADC_DMA;
```

### ADC変換チャンネルと順序の設定

```c
// 変換するチャンネル数を2に設定
ADC1->RSQR1 = (2 - 1) << 20;
ADC1->RSQR2 = 0;
// シーケンス1番目にCH1、2番目にCH0を設定
ADC1->RSQR3 = (ADC_Channel_1 << (5 * 0)) | (ADC_Channel_0 << (5 * 1));
// CH1とCH0のサンプリング時間を241サイクルに設定
ADC1->SAMPTR2 = (ADC_SampleTime_241Cycles5 << (3 * 1)) |⏎
    (ADC_SampleTime_241Cycles5 << (3 * 0));
```

### DMAの設定

```c
// 転送元アドレス (Peripheral): ADCデータレジスタ
DMA1_Channel1->PADDR = (uint32_t)&ADC1->RDATAR;
// 転送先アドレス (メモリ): バッファ
DMA1_Channel1->MADDR = (uint32_t)adc_buf;
// 転送データ数
DMA1_Channel1->CNTR = sizeof(adc_buf) / sizeof(adc_buf[0]);
DMA1_Channel1->CFGR =
  // 転送方向: Peripheral -> メモリ
  DMA_DIR_PeripheralSRC |
  DMA_M2M_Disable |
  // 優先度: 最高
  DMA_Priority_VeryHigh |
  // メモリデータサイズ: 16bit
  DMA_MemoryDataSize_HalfWord |
  // Peripheralデータサイズ: 16bit
  DMA_PeripheralDataSize_HalfWord |
  // メモリアドレスインクリメント有効
  DMA_MemoryInc_Enable |
  // 循環モード有効
  DMA_Mode_Circular;
```

### ADCの有効化とADCキャリブレーションの実行

### DMA有効化とADC変換開始

```c
// DMAチャンネル1有効化 (EN=1)
DMA1_Channel1->CFGR |= DMA_CFGR1_EN;

// ソフトウェアトリガで変換開始
ADC1->CTLR2 |= ADC_SWSTART;
```
