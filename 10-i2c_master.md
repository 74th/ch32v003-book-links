# 10. I2Cマスター

## 10.1. CH32V003のI2C

## 10.2. サンプル回路

## 10.3. Arduino

> https://github.com/74th/ch32v003-book-code/tree/main/i2c_master-arduino_core_ch32

```c
Wire.begin();
```

```c
// 送信の開始
// 第1引数: スレーブのアドレス
void Wire.beginTransmission(uint8_t address)
// 1バイトのデータ送信
void Wire.write(uint8_t data)
// 送信の終了
void Wire.endTransmission()
```

```c
Wire.beginTransmission(0x44);
Wire.write(0x24);
Wire.write(0x00);
Wire.endTransmission();
```

```c
// スレーブからのデータ送信のリクエスト
Wire.requestFrom(uint8_t address, uint8_t num)
// 1バイトのデータ受信
uint8_t Wire.read()
```

```c
Wire.requestFrom(SHT31_I2C_ADDR, 6);
for (i = 0; i < 6; i++)
{
  dac[i] = Wire.read();
}

// 最初2バイトが温度データ
int t = (dac[0] << 8) | dac[1];
// 温度データを、℃に変換（小数点切り捨て）
uint32_t temperature = (((uint32_t)(t) * 175) >> 16) - 45;
// 4、5バイトが湿度データ
int h = (dac[3] << 8) | dac[4];
// 湿度データを、%に変換（小数点切り捨て）
uint32_t humidity = ((uint32_t)(h)*100) >> 16;
```

## 10.4. WCH SDK

> https://github.com/74th/ch32v003-book-code/tree/main/i2c_master-wch_sdk

### GPIO、I2C、AFIOへのクロック供給の有効化

```c
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
RCC_APB1PeriphClockCmd(RCC_APB1Periph_I2C1, ENABLE);
// リマップを使う場合
RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
```

### GPIOの設定

```c
GPIO_InitTypeDef GPIO_InitStructure = {0};

// PC1: SDA, PC2: SCL
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_1 | GPIO_Pin_2;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_OD;
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
GPIO_Init(GPIOC, &GPIO_InitStructure);
```

```c
// I2Cのリマップ
// Remap 1: GPIO_PartialRemap_I2C1
// Remap 2: GPIO_FullRemap_I2C1
GPIO_PinRemapConfig(GPIO_FullRemap_I2C1, ENABLE);
```

### I2Cの設定と有効化

```c
// クロック速度
I2C_InitTypeDef I2C_InitTSturcture = {0};
I2C_InitTSturcture.I2C_ClockSpeed = 100000;
// モード（固定値）
I2C_InitTSturcture.I2C_Mode = I2C_Mode_I2C;
// デューティサイクル
// I2C_DutyCycle_16_9: 16/9
// I2C_DutyCycle_2: 2/1
I2C_InitTSturcture.I2C_DutyCycle = I2C_DutyCycle_2;
// Ackを有効にする
I2C_InitTSturcture.I2C_Ack = I2C_Ack_Enable;
// Ackアドレスのビット数
I2C_InitTSturcture.I2C_AcknowledgedAddress = I2C_AcknowledgedAddress_7bit;
// 設定の有効化
I2C_Init(I2C1, &I2C_InitTSturcture);

// I2Cの有効化
I2C_Cmd(I2C1, ENABLE);
I2C_AcknowledgeConfig(I2C1, ENABLE);
```

### 送信

```c
// 送信
void send_i2c_data(uint8_t address, uint8_t *data, uint8_t length)
{
  // I2Cのバスがビジーでなくなるまで待つ
  while (I2C_GetFlagStatus(I2C1, I2C_FLAG_BUSY) != RESET)
    ;

  // 通信の開始の送信
  I2C_GenerateSTART(I2C1, ENABLE);

  // マスターモードに準備できるまで待つ
  while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_MODE_SELECT))
    ;

  // スレーブアドレスをマスターからスレーブへの通信として送信
  I2C_Send7bitAddress(I2C1, address << 1, I2C_Direction_Transmitter);

  // トランスミッターモードに準備できるまで待つ
  while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED))
    ;

  for (int i = 0; i < length; i++)
  {
    // 送信バッファが空になるまで待つ
    while (I2C_GetFlagStatus(I2C1, I2C_FLAG_TXE) == RESET)
      ;

    // 1バイトのデータ送信
    I2C_SendData(I2C1, data[i]);
  }

  // 送信完了イベントを待つ
  while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTED))
    ;

  // 通信の終了の送信
  I2C_GenerateSTOP(I2C1, ENABLE);
}
```

```c
uint8_t buf[2] = {0x24, 0x00};
send_i2c_data(0x44, buf, 2);
```

### 受信

```c
// 受信
void read_i2c_data(uint8_t address, uint8_t *data, uint8_t length)
{
    // I2Cのバスがビジーでなくなるまで待つ
    while (I2C_GetFlagStatus(I2C1, I2C_FLAG_BUSY) != RESET)
        ;

    // 通信の開始の送信
    I2C_GenerateSTART(I2C1, ENABLE);

    // マスターモードに準備できるまで待つ
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_MODE_SELECT))
        ;

    // スレーブアドレスをスレーブからマスターへの通信として送信
    I2C_Send7bitAddress(I2C1, address << 1, I2C_Direction_Receiver);

    // レシーバーモードに準備できるまで待つ
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_RECEIVER_MODE_SELECTED))
        ;

    for (int i = 0; i < length; i++)
    {
        // 受信完了イベントを待つ
        while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_RECEIVED))
            ;

        // 受信データの取得
        data[i] = I2C_ReceiveData(I2C1);
    }

    // 通信の終了の送信
    I2C_GenerateSTOP(I2C1, ENABLE);
}
```

```c
uint8_t dac[6];
uint32_t t, h;
uint32_t temperature, humidity;

read_i2c_data(0x44, dac, sizeof(dac));

t = (dac[0] << 8) | dac[1];
temperature = (((uint32_t)(t) * 175) >> 16) - 45;
h = (dac[3] << 8) | dac[4];
humidity = (((uint32_t)(h) * 100) >> 16);
```

## 10.5. レジスタ操作

> https://github.com/74th/ch32v003-book-code/tree/main/i2c_master-register

### GPIO、I2C、AFIOへのクロック供給の有効化

```c
RCC->APB2PCENR |= RCC_APB2Periph_GPIOC;
RCC->APB1PCENR |= RCC_APB1Periph_I2C1;
// リマップを使う場合
RCC->APB2PCENR |= RCC_APB2Periph_AFIO;
```

### GPIO、リマップの設定

```c
// PC1: SDA
GPIOC->CFGLR &= ~(0xf << (4 * 1));
GPIOC->CFGLR |= (GPIO_Speed_50MHz | GPIO_CNF_OUT_OD_AF) << (4 * 1);

// PC2: SCL
GPIOC->CFGLR &= ~(0xf << (4 * 2));
GPIOC->CFGLR |= (GPIO_Speed_50MHz | GPIO_CNF_OUT_OD_AF) << (4 * 2);
```

```c
// リマップ設定リセット
AFIO->PCFR1 &= ~(AFIO_PCFR1_I2C1_REMAP | AFIO_PCFR1_I2C1_HIGH_BIT_REMAP);
// リマップ設定
// Remap1: AFIO_PCFR1_I2C1_REMAP
// Remap2: AFIO_PCFR1_I2C1_HIGH_BIT_REMAP
AFIO->PCFR1 |= AFIO_PCFR1_I2C1_HIGH_BIT_REMAP;
```

### I2Cの設定と有効化

```c
uint16_t tempreg;

// I2Cの転送速度
uint32_t clkrate = 400000;
// I2Cの回路のクロックスピード
uint32_t prerate = clkrate * 2;

// 回路のクロック周波数を設定
tempreg = I2C1->CTLR2;
tempreg &= ~I2C_CTLR2_FREQ;
tempreg |= (FUNCONF_SYSTEM_CORE_CLOCK / prerate) & I2C_CTLR2_FREQ;
I2C1->CTLR2 = tempreg;

// I2Cの転送速度の設定
tempreg = I2C1->CKCFGR;
tempreg &= ~I2C_CKCFGR_CCR;
tempreg = (FUNCONF_SYSTEM_CORE_CLOCK / (25 * clkrate)) & I2C_CKCFGR_CCR;
I2C1->CKCFGR = tempreg;
```

```c
I2C1->CKCFGR |=
  // デューティーサイクル(16:9の場合1、2:1の場合は設定なし)
  I2C_CKCFGR_DUTY |
  // ファストモード（100kHzより大きい速度の場合指定）
  I2C_CKCFGR_FS;
```

```c
I2C1->CTLR1 |=
  // I2Cの有効化
  I2C_CTLR1_PE |
  // ACKの有効化
  I2C_CTLR1_ACK;
```

### 送信

```c
uint8_t check_i2c_event(uint32_t event_mask)
{
  uint32_t status = I2C1->STAR1 | (I2C1->STAR2 << 16);
  return (status & event_mask) == event_mask;
}
```

```c
// 送信
uint8_t send_i2c_data(uint8_t address, uint8_t *data, uint8_t length)
{
  int32_t timeout;

  // I2Cのバスがビジーでなくなるまで待つ
  timeout = TIMEOUT_MAX;
  while (I2C1->STAR2 & I2C_STAR2_BUSY)
    ;

  // 通信の開始の送信
  I2C1->CTLR1 |= I2C_CTLR1_START;

  // マスターモードに準備できるまで待つ
  timeout = TIMEOUT_MAX;
  while (!check_i2c_event(I2C_EVENT_MASTER_MODE_SELECT))
    ;

  // スレーブアドレスをマスターからスレーブへの通信として送信
  I2C1->DATAR = address << 1;

  // トランスミッターモードに準備できるまで待つ
  timeout = TIMEOUT_MAX;
  while (!check_i2c_event(I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED))
    ;

  for (int i = 0; i < length; i++)
  {
    // 1バイトのデータ送信
    timeout = TIMEOUT_MAX;
    while (!(I2C1->STAR1 & I2C_STAR1_TXE))
      ;

    // 1バイト送信
    I2C1->DATAR = data[i];
  }

  // 送信完了イベントまで待つ
  timeout = TIMEOUT_MAX;
  while (!check_i2c_event(I2C_EVENT_MASTER_BYTE_TRANSMITTED))
    ;

  // 通信の終了の送信
  I2C1->CTLR1 |= I2C_CTLR1_STOP;

  return 0;
}
```

### 受信

```c
int read_i2c_data(uint8_t address, uint8_t *buf, uint8_t length)
{
  int32_t timeout;

  I2C1->CTLR1 |= I2C_CTLR1_ACK;

  // I2Cのバスがビジーでなくなるまで待つ
  while (I2C1->STAR2 & I2C_STAR2_BUSY)
    ;

  // 通信の開始の送信
  I2C1->CTLR1 |= I2C_CTLR1_START;

  // マスターモードに準備できるまで待つ
  while (!check_i2c_event(I2C_EVENT_MASTER_MODE_SELECT))
    ;

  // スレーブアドレスをスレーブからマスターへの通信として送信
  I2C1->DATAR = address << 1 | 0x1;

  // レシーバーモードに準備できるまで待つ
  while (!check_i2c_event(I2C_EVENT_MASTER_RECEIVER_MODE_SELECTED))
    ;

  for (int i = 0; i < length; i++)
  {
    // 最後のバイトの受信の前にACKを降ろす
    if (length - 1 == i)
    {
      I2C1->CTLR1 &= ~I2C_CTLR1_ACK;
    }

    // 受信完了イベントを待つ
    while (!check_i2c_event(I2C_EVENT_MASTER_BYTE_RECEIVED))
      ;

    // 受信データの受信
    buf[i] = I2C1->DATAR;
  }

  // 通信の終了の送信
  I2C1->CTLR1 |= I2C_CTLR1_STOP;

  return 0;
}
```
