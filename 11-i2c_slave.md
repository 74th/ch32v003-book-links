# 11. I2C スレーブ

## 11.1. I2C通信のメッセージボックス実装

## 11.2. Arduino

> https://github.com/74th/ch32v003-book-code/tree/main/i2c_slave-arduino_core_ch32

```c
// レジスタ
volatile uint8_t i2c_registers[0x30] = { 0x00 };
volatile uint8_t position = 0;

// マスターからスレーブへの送信（受信）
void on_receive(int length) {
  for (int i = 0; i < length; i++) {
    if (i == 0) {
      // 最初のバイトはレジスタアドレスとする
      position = Wire.read();
    } else {
      // 2バイト目以降はレジスタアドレスに書き込む
      if (position + (i - 1) < sizeof(i2c_registers)) {
        i2c_registers[position + (i - 1)] = Wire.read();
      } else {
        Wire.read();
      }
    }
  }
}

// スレーブからマスターへの送信（送信）
void on_request() {
  int i;

  for (i = 0; i < 4; i++) {
    // レジスタアドレスのデータを送信する
    if (position + (i - 1) < sizeof(i2c_registers)) {
      Wire.write(i2c_registers[position + i]);
    } else {
      Wire.write(0x00);
    }
  }
}
```

```c
void setup() {
  Serial.begin(115200);

  Wire.onReceive(on_receive);
  Wire.onRequest(on_request);
  Wire.begin(I2C_ADDRESS);
}
```

## 11.3. ch32fun

> https://github.com/74th/ch32v003-book-code/tree/main/i2c_slave-ch32fun

```c
#include "ch32fun.h"
#include "i2c_slave.h"

// マスターからスレーブへの送信（受信）後に呼ばれる
void on_write(uint8_t reg, uint8_t length)
{
}

// スレーブからマスターへの送信（送信）後に呼ばれる
void on_read(uint8_t reg)
{
}

volatile uint8_t i2c_registers[0x30] = {0x00};

int main()
{
  SystemInit();
  funGpioInitAll();

  // SDA, SCLのピンの設定
  funPinMode(PC1, GPIO_CFGLR_OUT_10Mhz_AF_OD);
  funPinMode(PC2, GPIO_CFGLR_OUT_10Mhz_AF_OD);

  // I2Cスレーブの初期化
  SetupI2CSlave(0x10, i2c_registers, sizeof(i2c_registers), on_write, on_read, false);

  while (1)
  ;
}
```

```c
// I2Cスレーブの初期化
// address: I2Cスレーブのアドレス
// registers: レジスタとなる配列のアドレス
// size: レジスタのサイズ
// write_callback: マスターからスレーブへの送信（受信）後に呼ばれるコールバック関数
// read_callback: スレーブからマスターへの送信（送信）後に呼ばれるコールバック関数
// read_only: マスターからスレーブへの送信専用の場合true
void SetupI2CSlave(
  uint8_t              address,
  volatile uint8_t*    registers,
  uint8_t              size,
  i2c_write_callback_t write_callback,
  i2c_read_callback_t  read_callback,
  bool                 read_only);
```

## 11.4. WCH SDK

> https://github.com/74th/ch32v003-book-code/tree/main/i2c_slave-wch_sdk

### クロック供給、GPIOの設定

### I2Cスレーブと、割込の設定と、有効化

```c
NVIC_InitTypeDef NVIC_InitStructure = {0};

// I2C1の割込チャンネル
NVIC_InitStructure.NVIC_IRQChannel = I2C1_EV_IRQn;
// 優先度0（最高）
NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 0;
// サブ優先度0（最高）
NVIC_InitStructure.NVIC_IRQChannelSubPriority = 0;
// 有効化
NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
// 設定の適用
NVIC_Init(&NVIC_InitStructure);

// I2C1のエラーチャンネル
NVIC_InitStructure.NVIC_IRQChannel = I2C1_ER_IRQn;
// 設定の適用
NVIC_Init(&NVIC_InitStructure);
```

```c
I2C_InitTypeDef I2C_InitTSturcture = {0};

I2C_InitTSturcture.I2C_Mode = I2C_Mode_I2C;
// I2Cスレーブのアドレス
I2C_InitTSturcture.I2C_OwnAddress1 = I2C_ADDRESS << 1;
// ACKの有効化
I2C_InitTSturcture.I2C_Ack = I2C_Ack_Enable;
I2C_InitTSturcture.I2C_AcknowledgedAddress = I2C_AcknowledgedAddress_7bit;
I2C_Init(I2C1, &I2C_InitTSturcture);
```

```c
// I2Cの有効化
I2C_Cmd(I2C1, ENABLE);
// I2C ACKの有効化
I2C_AcknowledgeConfig(I2C1, ENABLE);

// I2Cの割込の有効化
I2C_ITConfig(I2C1, I2C_IT_EVT, ENABLE);
I2C_ITConfig(I2C1, I2C_IT_ERR, ENABLE);
I2C_ITConfig(I2C1, I2C_IT_BUF, ENABLE);
NVIC_EnableIRQ(I2C1_EV_IRQn);
NVIC_SetPriority(I2C1_EV_IRQn, 2 << 4);
NVIC_EnableIRQ(I2C1_ER_IRQn);
```

### 割込ハンドラ

```c
void I2C1_EV_IRQHandler(void) __attribute__((interrupt("WCH-Interrupt-fast")));

// レジスタとなる配列
volatile uint8_t i2c_registers[0x30] = {0x00};
// 送信、受信したレジスタアドレス
volatile uint8_t i2c_start_position = 0;
// 次に送信、受信するレジスタアドレス
volatile uint8_t i2c_position = 0;
// 最初の受信を判定するフラグ
volatile uint8_t i2c_first_receive = 0;

void I2C1_EV_IRQHandler(void)
{
  uint32_t event = I2C_GetLastEvent(I2C1);

  // ACKを有効化
  I2C_AcknowledgeConfig(I2C1, ENABLE);

  // マスターからスレーブへの送信（受信）要求
  if (event == I2C_EVENT_SLAVE_RECEIVER_ADDRESS_MATCHED)
  {
    // 最初のイベントとしてフラグを立てておくと良い
    i2c_first_receive = 1;
  }

  // マスターからスレーブへの送信（受信）を受け取った
  if (event == I2C_EVENT_SLAVE_BYTE_RECEIVED)
  {
    // 1バイト受信
    uint8_t v = I2C_ReceiveData(I2C1);
    if (i2c_first_receive)
    {
      // 1バイト目の受信。レジスタアドレスとして記録。
      i2c_position = v;
      i2c_start_position = v;
      i2c_first_receive = 0;
    }
    else if (i2c_position < sizeof(i2c_registers))
    {
      // 2バイト目以降の受信。レジスタに格納。
      i2c_registers[i2c_position] = v;
      i2c_position++;
    }
    else
    {
      // 2バイト目以降の受信、レジスタ範囲外
      // 何もしない
    }
  }

  // スレーブからマスターへの送信（送信）要求
  // または、スレーブからマスターへの送信（送信）の完了
  if (event == I2C_EVENT_SLAVE_TRANSMITTER_ADDRESS_MATCHED ||
      event == I2C_EVENT_SLAVE_BYTE_TRANSMITTED)
  {
    // 1バイト送信する
    I2C_SendData(I2C1, i2c_registers[i2c_position]);
    i2c_position++;
  }
}
```

```c
void I2C1_ER_IRQHandler(void) __attribute__((interrupt("WCH-Interrupt-fast")));

// エラーハンドラ
void I2C1_ER_IRQHandler(void)
{
  if (I2C_GetITStatus(I2C1, I2C_IT_AF))
  {
    // フラグを降ろす
    I2C_ClearITPendingBit(I2C1, I2C_IT_AF);
  }
}
```

## 11.5. レジスタ操作

> https://github.com/74th/ch32v003-book-code/tree/main/i2c_slave-register

### クロック供給、GPIOの設定

### I2Cスレーブと、割込の設定と、有効化

```c
// I2Cアドレス設定
I2C1->OADDR1 = 0x10 << 1;

// 割込
I2C1->CTLR2 |= I2C_CTLR2_ITBUFEN;
I2C1->CTLR2 |= I2C_CTLR2_ITEVTEN; // イベント割込
I2C1->CTLR2 |= I2C_CTLR2_ITERREN; // エラー割込
NVIC_EnableIRQ(I2C1_EV_IRQn);     // イベント割込
NVIC_SetPriorty(I2C1_EV_IRQn, 2 << 4);
NVIC_EnableIRQ(I2C1_ER_IRQn); // エラー割込

// I2C、ACK有効化
I2C1->CTLR1 |= I2C_CTLR1_PE | I2C_CTLR1_ACK;
```

### 割込ハンドラ

```c
void I2C1_EV_IRQHandler(void)
{
  uint16_t STAR1, STAR2 __attribute__((unused));
  STAR1 = I2C1->STAR1;
  STAR2 = I2C1->STAR2;

  I2C1->CTLR1 |= I2C_CTLR1_ACK;

  if (check_i2c_event(I2C_EVENT_SLAVE_RECEIVER_ADDRESS_MATCHED)) // 0x0002
  {
    // 最初のイベント
    i2c_first_receive = 1;
  }

  if (check_i2c_event(I2C_EVENT_SLAVE_BYTE_RECEIVED))
  {
    // 1byte の受信イベント（master -> slave）
    uint8_t v = I2C1->DATAR;
    if (i2c_first_receive)
    {
      // 最初の1バイトはレジスタアドレスとする
      i2c_start_position = v;
      i2c_position = v;
      i2c_first_receive = 0;
    }
    else if (i2c_position < sizeof(i2c_registers))
    {
      // 2バイト目以降
      i2c_registers[i2c_position] = v;
      i2c_position++;
      i2c_receive_available += 1;
    }
    else
    {
      // 何もしない
    }
  }

  if (STAR1 & I2C_STAR1_TXE)
  {
    // 1byte の送信イベント（master -> slave）
    if (i2c_position < sizeof(i2c_registers))
    {
      I2C1->DATAR = i2c_registers[i2c_position];
      i2c_position++;
      i2c_request_available += 1;
    }
    else
    {
      // ゼロ値を送る
      I2C1->DATAR = 0x00;
    }
  }
}
```

```c
void I2C1_ER_IRQHandler(void)
{
  uint16_t STAR1 = I2C1->STAR1;

  if (STAR1 & I2C_STAR1_AF)
  {
    I2C1->STAR1 &= ~(I2C_STAR1_AF);
  }
}
```

## 11.6. 筆者の実例
