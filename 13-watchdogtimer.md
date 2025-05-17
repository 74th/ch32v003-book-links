# 13. Watchdog Timer

## 13.1. CH32V003のWDT

## 13.2. WCH SDK

> https://github.com/74th/ch32v003-book-code/tree/main/watchdogtimer-wch_sdk

### IWDGの設定と有効化

```c
// Independent Watchdog Timerを2sのカウンタでセット
IWDG_WriteAccessCmd(IWDG_WriteAccess_Enable);
// LSIは128kHzにつき、128分周でミリ秒になる
IWDG_SetPrescaler(IWDG_Prescaler_128);
// 2,000ms = 2s でリセット
IWDGSetReload(2000);
// 最初のカウンタリロード
IWDG_ReloadCounter();
// IWDGを有効化
IWDG_Enable();
```

### IWDGのカウンタリセット

```c
IWDG_ReloadCounter();
```

## 13.3. レジスタ操作

> https://github.com/74th/ch32v003-book-code/tree/main/watchdogtimer-register

### IWDGの設定と有効化

```c
// Independent Watchdog Timerを2sのカウンタでセット
IWDG->CTLR = IWDG_WriteAccess_Enable;
// LSIは128kHzにつき、128分周でミリ秒になる
IWDG->PSCR = IWDG_Prescaler_128;
// 2,000ms = 2s でリセット
IWDG->RLDR = 2000 & 0xfff;
// 最初のカウンタリロード
IWDG->CTLR = CTLR_KEY_Reload;
// IWDGの開始
IWDG->CTLR = CTLR_KEY_Enable;
```

### IWDGのカウンタリセット

```c
IWDG->CTLR = CTLR_KEY_Reload;
```

## 13.4. 筆者の組み込みの実例

```c
void on_write(uint8_t reg, uint8_t length)
{
  IWDG->CTLR = CTLR_KEY_Reload;
}

void on_read(uint8_t reg)
{
  IWDG->CTLR = CTLR_KEY_Reload;
}
```
