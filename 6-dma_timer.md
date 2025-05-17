# 6. TimerとDMA

## 6.1. TimerとDMAとは

## 6.2. WCH SDKでTimerを使う

> https://github.com/74th/ch32v003-book-code/tree/main/timer_blink-wch_sdk

```c
// TIM1のクロックを有効化する
RCC_APB2PeriphClockCmd(RCC_APB2Periph_TIM1, ENABLE);
// TIM2のクロックを有効化する
RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);
```

```c
TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure = {0};

// カウントするクロックの周期
// クロックが48MHzのため、48000 - 1を設定しておけば
// 1kHz (1ms)でカウントすることになる
TIM_TimeBaseInitStructure.TIM_Prescaler = 48000 - 1;

// カウンターをリセットする周期
// 1000 - 1をセットすれば、上と併せて、1Hz (1秒)でリセットされる
TIM_TimeBaseInitStructure.TIM_Period = 1000 - 1;

// カウンタの方向
TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;

// 設定の適用
TIM_TimeBaseInit(TIM1, &TIM_TimeBaseInitStructure);
```

```c
NVIC_InitTypeDef NVIC_InitStructure = {0};

// 割込フラグのクリア
TIM_ClearITPendingBit(TIM1, TIM_IT_Update);

// 割込の設定（優先度、サブ優先度、割込の有効化）
NVIC_InitStructure.NVIC_IRQChannel = TIM1_UP_IRQn;
NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 0;
NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;
NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
NVIC_Init(&NVIC_InitStructure);

// 割込の有効化
TIM_ITConfig(TIM1, TIM_IT_Update, ENABLE);
```

```c
// Timerの開始
TIM_Cmd(TIM1, ENABLE);
```

```c
void TIM1_UP_IRQHandler(void) __attribute__((interrupt("WCH-Interrupt-fast")));
```

```c
// LEDの状態
uint8_t ledState = 0;

void TIM1_UP_IRQHandler(void)
{
  // 割込のステータスの確認
  if (TIM_GetITStatus(TIM1, TIM_IT_Update) == SET)
  {
    printf("1sec event\r\n");
  }
  // 割込フラグクリア
  TIM_ClearITPendingBit(TIM1, TIM_IT_Update);

  // PC1に繋いだLEDの点滅
  GPIO_WriteBit(GPIOC, GPIO_Pin_1, ledState);
  ledState ^= 1;
}
```

## 6.3. レジスタ操作でTimerを使う

> https://github.com/74th/ch32v003-book-code/tree/main/timer_blink-register

```c
// プリスケーラの設定
TIM1->PSC = 48000 - 1;
// カウンターのオートリロード値（周期）
TIM1->ATRLR = 1000 - 1;
// プリスケーラとカウンターの値を即時反映
TIM1->SWEVGR = TIM_UG;
```

```c
// 割込フラグクリア
TIM1->INTFR = (uint16_t)~TIM_IT_Update;

// 割込設定
// Preemption Priority = 0, Sub Priority = 1
NVIC->IPRIOR[TIM1_UP_IRQn] = 0 << 7 | 1 << 6;
```

```c
// 割込の有効化
NVIC->IENR[((uint32_t)(TIM1_UP_IRQn) >> 5)] |= (1 << ((uint32_t)(TIM1_UP_IRQn) & 0x1F));
TIM1->DMAINTENR |= TIM_IT_Update;
```

```c
void TIM1_UP_IRQHandler(void) __attribute__((interrupt("WCH-Interrupt-fast")));
```

```c
// LEDの状態
uint8_t ledState = 0;

void TIM1_UP_IRQHandler(void)
{
  // フラグ確認
  if (((TIM1->INTFR & TIM_IT_Update) != 0) &&
      ((TIM1->DMAINTENR & TIM_IT_Update) != 0))
  {
    printf("1sec event\r\n");
  }

  // フラグのクリア
  TIM1->INTFR &= (uint16_t)~TIM_IT_Update;
  TIM1->DMAINTENR &= (uint16_t)~TIM_IT_Update;

  // PC1のLEDの点滅
  if (ledState)
  {
    GPIOC->BSHR = 1 << 1;
  }
  else
  {
    GPIOC->BSHR = (1 << (16 + 1));
  }
  ledState ^= 1;
}
```

```c
// Timerの開始
TIM1->CTLR1 |= TIM_CEN;
```

## 6.4. DMAを使った7セグLEDのダイナミック点灯とは

```c
uint32_t gpioa_bshr_buf[4] = {0};
uint32_t gpioc_bshr_buf[4] = {0};
uint32_t gpiod_bshr_buf[4] = {0};
```

## 6.5. WCH SDKでDMAを使う

> https://github.com/74th/ch32v003-book-code/tree/main/timer_dma-wch_sdk

### 使うDMAのチャンネルを選ぶ

### DMAへのクロック供給

```c
RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);
```

### DMAの設定

```c
DMA_InitTypeDef DMA_InitStructure = {0};

DMA_StructInit(&DMA_InitStructure);
// データ転送方向
// - DMA_DIR_PeripheralDST: Memory -> Peripheral
// - DMA_DIR_PeripheralSRC: Peripheral -> Memory
DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralDST;
// メモリ間転送の場合
DMA_InitStructure.DMA_M2M = DMA_M2M_Disable;

// Peripheral側のデータサイズ
// Byte (8bit), HalfWord (16bit), Word (32bit)を設定できます
DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_Word;
// メモリ側のデータサイズ
DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_Word;

// 転送先のPeripheralアドレス（GPIOAのBit Set/Resetレジスタ）
DMA_InitStructure.DMA_PeripheralBaseAddr = (uint32_t)&GPIOA->BSHR;
// Peripheral側のアドレスはインクリメントしない（常にGPIOA->BSHRに書き込む）
DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Disable;

// 転送元のメモリアドレス（転送データを格納した配列を指定）
DMA_InitStructure.DMA_MemoryBaseAddr = (uint32_t)gpioa_bshr_buf;
// メモリ側のアドレスはインクリメントして、配列の次の要素を読む
DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;

// 転送するデータ数（配列の要素数）
DMA_InitStructure.DMA_BufferSize = sizeof(gpioa_bshr_buf) / sizeof(gpioa_bshr_buf[0]);

// DMAのモード
// DMA_Mode_Normal: 1回の転送シーケンスで終了
// DMA_Mode_Circular: 転送シーケンス完了後、最初から繰り返す
DMA_InitStructure.DMA_Mode = DMA_Mode_Circular;

// 優先度
DMA_InitStructure.DMA_Priority = DMA_Priority_VeryHigh;
```

```c
// DMA1_Channel2として有効化
DMA_DeInit(DMA1_Channel2);
DMA_Init(DMA1_Channel2, &DMA_InitStructure);
DMA_Cmd(DMA1_Channel2, ENABLE);
```

### Timerの設定

```c
TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure = {0};

// クロック48MHzを480分周 => 100kHz (10us)でカウント
TIM_TimeBaseInitStructure.TIM_Prescaler = 480 - 1;
// 10カウントでリセット => 10 * 10us = 100usごとにトリガ
TIM_TimeBaseInitStructure.TIM_Period = 10 - 1;
TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;
TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;
TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;
TIM_TimeBaseInit(TIM1, &TIM_TimeBaseInitStructure);
```

### Timerの出力の設定

```c
TIM_OCInitTypeDef TIM_OCInitStructure = {0};

// PWMモード1で出力設定する
TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;
TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;
TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_Low;
// パルス幅（デューティ比）を指定（トリガ生成には影響しない）
TIM_OCInitStructure.TIM_Pulse = 6;
TIM_OC1Init(TIM1, &TIM_OCInitStructure);
TIM_CtrlPWMOutputs(TIM1, ENABLE);
```

```c
// TIM1_CH1 (DMA1_Channel2に接続)からのDMA要求を有効化
TIM_DMACmd(TIM1, TIM_DMA_CC1, ENABLE);
```

### 最後にTimerの有効化

```c
TIM_Cmd(TIM1, ENABLE);
```

## 6.6. レジスタ操作でDMAを使う

> https://github.com/74th/ch32v003-book-code/tree/main/timer_dma-register

### DMAの設定

```c
DMA1_Channel2->CFGR =
  // データ転送方向
  // Peripheral -> Memory: セットなし
  // Memory -> Peripheral: DMA_CFGR1_DIR
  DMA_CFGR1_DIR |

  // メモリ側のデータサイズ (Word)
  // Byte: セットなし
  // HalfWord: DMA_CFGR1_MSIZE_0
  // Word: DMA_CFGR1_MSIZE_1
  DMA_CFGR1_MSIZE_1 |

  // Peripheral側のデータサイズ (Word)
  // Byte: セットなし
  // HalfWord: DMA_CFGR1_PSIZE_0
  // Word: DMA_CFGR1_PSIZE_1
  DMA_CFGR1_PSIZE_1 |

  // アドレスインクリメント
  // メモリ側: DMA_CFGR1_MINC
  // Peripheral側: DMA_CFGR1_PINC
  DMA_CFGR1_MINC |

  // DMAのモード (Circular)
  DMA_CFGR1_CIRC |

  // 優先度 (Very High)
  DMA_CFGR1_PL;

// 転送元メモリアドレス
DMA1_Channel2->MADDR = (uint32_t)gpioa_bshr_buf;
// 転送先Peripheralアドレス (GPIOA->BSHR)
DMA1_Channel2->PADDR = (uint32_t)&GPIOA->BSHR;

// 転送データ数
DMA1_Channel2->CNTR = sizeof(gpioa_bshr_buf) / sizeof(gpioa_bshr_buf[0]);

// DMAチャンネル有効化
DMA1_Channel2->CFGR |= DMA_CFGR1_EN;
```

### Timerの設定（アウトプットチャンネル含む）

```c
// プリスケーラ (10us周期のカウントクロック)
TIM1->PSC = 480 - 1; // 480分周
// カウンターのオートリロード値 (100us周期でリセット)
TIM1->ATRLR = 10 - 1; // 10カウント
// プリスケーラとカウンターの値を即時反映 (Update Generation)
TIM1->SWEVGR = TIM_UG;
// PWMモード1, 出力有効, 極性Low
TIM1->CCER = TIM_CC1E | TIM_CC1P;
TIM1->CHCTLR1 = TIM_OC1M_2 | TIM_OC1M_1;
// パルス幅 (デューティ比)
TIM1->CH1CVR = 6;
// メイン出力有効化
TIM1->BDTR = TIM_MOE;
// Timer有効化
TIM1->CTLR1 = TIM_CEN;
// TIM1_CH1からのDMA要求を有効化
TIM1->DMAINTENR = TIM_CC1DE;
```
