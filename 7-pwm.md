# 7. PWM

## 7.1. PWMが使えるピン

## 7.2. Arduino

> https://github.com/74th/ch32v003-book-code/tree/main/pwm-arduino_core_ch32

```c
pinMode(PD2, OUTPUT);
```

```c
void loop() {
  for(int i=1;i<=32;i++){
    analogWrite(PD2, 128*i-1);
    delay(31);
  }

  for(int i=1;i<=32;i++){
    analogWrite(PD2, 128*(33-i)-1);
    delay(31);
  }
}
```

## 7.3. WCH SDK

> https://github.com/74th/ch32v003-book-code/tree/main/pwm-wch_sdk

### GPIO、Timer、AFIOのクロック供給の有効化

```c
// GPIO、Timerのクロック有効化
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA | RCC_APB2Periph_GPIOC | RCC_APB2Periph_GPIOD | RCC_APB2Periph_TIM1, ENABLE);
// AFIOクロック有効化
RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
```

### GPIOのピン設定

```c
// TIM1_CH1 -> PD2
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_2;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_30MHz;
GPIO_Init(GPIOD, &GPIO_InitStructure);
```

### AFIOのリマップ設定

```c
// 第1引数にリマップの種類を指定
// - GPIO_PartialRemap1_TIM1
// - GPIO_PartialRemap2_TIM1
// - GPIO_FullRemap_TIM1
GPIO_PinRemapConfig(GPIO_PartialRemap1_TIM1, ENABLE);
```

### Timerと、Timerの出力の設定

```c
// Timer1を、毎クロックカウントし、256でリセット
TIM_TimeBaseInitStructure.TIM_Period = (256 - 1);
TIM_TimeBaseInitStructure.TIM_Prescaler = 1;
TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;
TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;
TIM_TimeBaseInit(TIM1, &TIM_TimeBaseInitStructure);
```

```c
// Timer1 CH1をPWMで有効化
TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;
TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;
TIM_OCInitStructure.TIM_Pulse = 0;
TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;
TIM_OC1Init(TIM1, &TIM_OCInitStructure);
```

```c
// カウンタの値を即時反映するため、プリロード機能をDisableにする
TIM_OC1PreloadConfig(TIM1, TIM_OCPreload_Disable);
TIM_OC4PreloadConfig(TIM1, TIM_OCPreload_Disable);
TIM_ARRPreloadConfig(TIM1, ENABLE);
```

### PWM出力と、Timerの開始

```c
// PWM出力の有効化
TIM_CtrlPWMOutputs(TIM1, ENABLE);
// Timerの有効化
TIM_Cmd(TIM1, ENABLE);
```

### デューティー比の変更

```c
while (1)
{
  for (int i = 1; i <= 32; i++)
  {
    TIM_SetCompare1(TIM1, 8 * i - 1);
    Delay_Us(1000000 / 32);
  }
  for (int i = 1; i <= 32; i++)
  {
    TIM_SetCompare1(TIM1, 8 * (33 - i) - 1);
    Delay_Us(1000000 / 32);
  }
}
```

## 7.4. レジスタ操作

> https://github.com/74th/ch32v003-book-code/tree/main/pwm-register

### GPIO、Timer、AFIOのクロック供給の有効化

```c
// TIM、GPIOのクロック供給の有効化
RCC->APB2PCENR |= RCC_APB2Periph_GPIOD | RCC_APB2Periph_GPIOC |
          RCC_APB2Periph_TIM1;
// AFIOのクロック供給の有効化
RCC->APB2PCENR |= RCC_APB2Periph_AFIO;
```

### GPIOのピン設定

```c
// TIM1_CH1 -> PD2
GPIOD->CFGLR &= ~(0xf << (4 * 2));
GPIOD->CFGLR |= (GPIO_Speed_10MHz | GPIO_CNF_OUT_PP_AF) << (4 * 2);
```

### AFIOのリマップ設定

```c
// - AFIO_PCFR1_TIM1_REMAP_PARTIALREMAP1;
// - AFIO_PCFR1_TIM1_REMAP_PARTIALREMAP2;
// - AFIO_PCFR1_TIM1_REMAP_FULLREMAP;
AFIO->PCFR1 &= ~AFIO_PCFR1_TIM1_REMAP;
AFIO->PCFR1 |= AFIO_PCFR1_TIM1_REMAP_PARTIALREMAP1;
```

### Timer、Timerの出力の設定

```c
// Timerの設定
// Timer1を256周期で有効化
TIM1->PSC = 1;
TIM1->ATRLR = 255;
// 繰り返す
TIM1->SWEVGR |= TIM_UG;
```

```c
// TIM1_CH1
// CH1の比較アウトプットを有効化
TIM1->CCER |= TIM_CC1E | TIM_CC1P;
// PWMモード (CC1S = 00, OC1M = 110)
TIM1->CHCTLR1 |= TIM_OC1M_2 | TIM_OC1M_1;
// CH1をアウトプットで有効化
TIM1->CH1CVR = 0;
```

### PWM出力、Timerの開始

```c
// TIM1の出力を有効化
TIM1->BDTR |= TIM_MOE;
// TIM1を有効化
TIM1->CTLR1 |= TIM_CEN;
```

### デューティー比の変更

```c
while (1)
{
  for (int i = 1; i <= 32; i++)
  {
    TIM1->CH1CVR = 8 * i - 1;   // CH1
    Delay_Us(1000000 / 32);
  }

  for (int i = 1; i <= 32; i++)
  {
    TIM1->CH1CVR = 8 * (33 - i); // CH1
    Delay_Us(1000000 / 32);
  }
}
```
