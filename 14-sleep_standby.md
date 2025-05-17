# 14. 省エネルギーモード

## 14.1. CH32V003の省エネルギーモード

## 14.2. スリープモード WCH SDK

> https://github.com/74th/ch32v003-book-code/tree/main/sleep_mode_blink-wch_sdk

### なんらかのCPU割込を有効にする

### スリープモードを有効にする

### 割込待ちをする

```c
void TIM2_IRQHandler(void) __attribute__((interrupt("WCH-Interrupt-fast")));

// Timer割込ハンドラ
void TIM1_UP_IRQHandler(void)
{
  // フラグを降ろすのみ
  TIM_ClearITPendingBit(TIM1, TIM_IT_Update);
}

int main()
{
  // 初期化は省略

  uint8_t ledState = 0;

  while (1)
  {
    // LEDをトグル
    GPIO_WriteBit(GPIOC, GPIO_Pin_1, ledState);
    ledState ^= 1;

    // 割込待ちでスリープモード突入
    __WFI();

    // Timer割込でループする
  }
}
```

## 14.3. スリープモード レジスタ操作

> https://github.com/74th/ch32v003-book-code/tree/main/sleep_mode_blink-register

### スリープモードを有効化する

```c
PFIC->SCTLR &= ~(1 << 2 | 1 << 3);
PWR->CTLR &= PWR_CTLR_PDDS;
```

### 割込待ちをする

```c
__ASM volatile("wfi");
```

```c
__WFI();
```

## 14.4. スタンバイモード WCH SDK

> https://github.com/74th/ch32v003-book-code/tree/main/standby_mode_blink-wch_sdk

### LSIの有効化と、AWUへのクロック供給

```c
RCC_LSICmd(ENABLE);
RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR, ENABLE);
```

### AWUの設定、有効化

```c
PWR_AWU_SetPrescaler(PWR_AWU_Prescaler_10240);
PWR_AWU_SetWindowValue(25);
PWR_AutoWakeUpCmd(ENABLE);
```

### スタンバイモードに突入する

```c
PWR_EnterSTANDBYMode(PWR_STANDBYEntry_WFE);
```

## 14.5. スタンバイモード レジスタ操作

> https://github.com/74th/ch32v003-book-code/tree/main/sleep_mode_blink-register

### LSIの有効化と、AWUへのクロック供給

```c
RCC->APB1PCENR |= RCC_APB1Periph_PWR;
// LSIを有効化
RCC->RSTSCKR |= (1 << 0);
```

```c
while (!(RCC->RSTSCKR & (1 << 1)))
  ;
```

### AWUの設定、有効化

```c
EXTI->EVENR |= EXTI_Line9;
EXTI->FTENR |= EXTI_Line9;
```

```c
// プリスケーラ
PWR->AWUPSC = PWR_AWU_Prescaler_10240;
// カウンタ
PWR->AWUWR = 25;
// AWUの有効化
PWR->AWUCSR |= (1 << 1);
```

### 割込待ちをして、スタンバイモードに突入する

```c
// スタンバイモード
PWR->CTLR &= CTLR_DS_MASK;
PWR->CTLR |= PWR_CTLR_PDDS;

NVIC->SCTLR |=
    // WFIでディープスリープモードへ
    (1 << 2) |
    // WFEとして動作
    (1 << 3);
```

```c
__ASM volatile("wfi");
```

```c
// 設定の解除
NVIC->SCTLR &= ~((1 << 2) | (1 << 3));
```

## 14.6. スリープモード、スタンバイモードのLチカの消費電力
