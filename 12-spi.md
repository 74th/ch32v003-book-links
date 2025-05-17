# 12. SPI

## 12.1. CH32V003のSPI

## 12.2. サンプル回路

## 12.3. Arduino

> https://github.com/74th/ch32v003-book-code/tree/main/spi_master-arduino_core_ch32

```c
#include <SPI.h>

void loop() {
  uint16_t raw;
  int32_t raw_int;

  // 連続読み取りモードの開始
  SPI.beginTransaction(SPISettings(1000000, MSBFIRST, SPI_MODE0));
  // コマンドの送信
  SPI.transfer(0x54);
  SPI.endTransaction();

  delay(500);

  // 2バイト読み取り
  SPI.beginTransaction(SPISettings(1000000, MSBFIRST, SPI_MODE0));
  uint8_t b1 = SPI.transfer(0);
  uint8_t b2 = SPI.transfer(0);
  uint16_t raw = b1 << 8 | b2;
  SPI.endTransaction();
}
```

```h
// speedMaximum: 通信速度
// dataOrder: データの順序（MSBFIRST or LSBFIRST）
// dataMode: SPIモード（SPI_MODE0, SPI_MODE1, SPI_MODE2, SPI_MODE3）
SPISettings mySetting(speedMaximum, dataOrder, dataMode)
```

```h
// トランザクションの開始
void beginTransaction(SPISettings settings);
// トランザクションの終了
void endTransaction();
// データの送受信
uint8_t transfer(uint8_t data);
```

## 12.4. ch32fun

> extralibs/ch32v003_SPI.h

```h
// funconfig.h
// 通信速度
#define CH32V003_SPI_SPEED_HZ 1000000

// SPIのモードを設定
#define CH32V003_SPI_CLK_MODE_POL0_PHA0
// #define CH32V003_SPI_CLK_MODE_POL0_PHA1
// #define CH32V003_SPI_CLK_MODE_POL1_PHA0
// #define CH32V003_SPI_CLK_MODE_POL1_PHA1

// SPIのデータの順序を設定
// 送信のみ
// #define CH32V003_SPI_DIRECTION_1LINE_TX
// 送受信
#define CH32V003_SPI_DIRECTION_2LINE_TXRX

// CS信号の制御の設定
#define CH32V003_SPI_NSS_HARDWARE_PC1
```

```c
int main()
{
  SystemInit();

  // SPIの初期化
  SPI_init();

  // 通信の開始
  SPI_begin_8();

  // 送信（ADT7310の高速読み取りモードの開始）
  SPI_transfer_8(0x54);

  // 通信の終了
  SPI_end();

  Delay_Ms(500);

  while (1)
  {
    printf("loop\\r\\n");

    SPI_begin_8();

    // 2バイトのデータを受信
    uint8_t b1 = SPI_transfer_8(0x00);
    uint8_t b2 = SPI_transfer_8(0x00);
    uint16_t raw = b1 << 8 | b2;

    SPI_end();

    Delay_Ms(1000);
  }
}
```

## 12.5. WCH SDK

> https://github.com/74th/ch32v003-book-code/tree/main/spi_master-wch_sdk

### GPIO、SPIへのクロック供給の開始

```c
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC | RCC_APB2Periph_SPI1, ENABLE);
```

### GPIOの設定

```c
GPIO_InitTypeDef GPIO_InitStructure = {0};

// PC1: NSS(CS)
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_1;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_30MHz;
GPIO_Init(GPIOC, &GPIO_InitStructure);

// PC5: SCK
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_5;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_30MHz;
GPIO_Init(GPIOC, &GPIO_InitStructure);

// PC7: MISO
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_7;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;
GPIO_Init(GPIOC, &GPIO_InitStructure);

// PC6: MOSI
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_30MHz;
GPIO_Init(GPIOC, &GPIO_InitStructure);
```

### SPIの設定と適用

```c
SPI_InitTypeDef SPI_InitStructure = {0};

// 通信方向の指定
// 全2重: SPI_Direction_2Lines_FullDuplex
// 全2重受信のみ: SPI_Direction_2Lines_RxOnly
// 受信のみ: SPI_Direction_1Line_Rx
// 送信のみ: SPI_Direction_1Line_Tx
SPI_InitStructure.SPI_Direction = SPI_Direction_2Lines_FullDuplex;
// マスターモード
SPI_InitStructure.SPI_Mode = SPI_Mode_Master;
// データサイズ（8bit or 16bit）
SPI_InitStructure.SPI_DataSize = SPI_DataSize_8b;
// SPIモード
SPI_InitStructure.SPI_CPOL = SPI_CPOL_Low;
SPI_InitStructure.SPI_CPHA = SPI_CPHA_1Edge;
// NSSピンの制御方法
SPI_InitStructure.SPI_NSS = SPI_NSS_Soft;
// 通信速度となるプリスケーラ
SPI_InitStructure.SPI_BaudRatePrescaler = SPI_BaudRatePrescaler_256;
// データ順序（MSB or LSB）
SPI_InitStructure.SPI_FirstBit = SPI_FirstBit_MSB;
// CRCの計算を行うか
SPI_InitStructure.SPI_CRCPolynomial = 7;
// 設定の適用
SPI_Init(SPI1, &SPI_InitStructure);

// SPIの有効化
SPI_Cmd(SPI1, ENABLE);

// NSSピンのセット
GPIO_ResetBits(GPIOC, GPIO_Pin_1);
```

### 通信

```c
uint8_t transfer_spi(uint8_t data)
{
  // 常に書き込みと読み込みを両方行う
  // 送信バッファが空になる（書込可能）のを待つ
  while (SPI_I2S_GetFlagStatus(SPI1, SPI_I2S_FLAG_TXE) == RESET)
    ;
  // 1バイトの送信
  SPI_I2S_SendData(SPI1, data);

  // 受信バッファにデータが入る（読込可能）のを待つ
  while (SPI_I2S_GetFlagStatus(SPI1, SPI_I2S_FLAG_RXNE) == RESET)
    ;

  // 1バイトの受信
  return SPI_I2S_ReceiveData(SPI1);
}

// 送信
void send_spi_data(uint8_t *data, uint8_t length)
{
  for (int i = 0; i < length; i++)
  {
    transfer_spi(data[i]);
  }
}

// 受信
void read_spi_data(uint8_t *data, uint8_t length)
{
  for (int i = 0; i < length; i++)
  {
    data[i] = transfer_spi(0);
  }
}
```

```c
// 連続変換モードの開始
uint8_t buf2[1] = {0x54};
send_spi_data(buf2, sizeof(buf2));

while (1)
{
  uint8_t buf[2] = {0x00, 0x00};
  // 測定結果の2バイトの受信
  read_spi_data(buf, sizeof(buf));

  uint16_t raw = (buf[0] << 8) | buf[1];

  Delay_Ms(1000);
}
```

## 12.6. レジスタ操作

> https://github.com/74th/ch32v003-book-code/tree/main/spi_master-register

### GPIO、SPIへのクロック供給の開始

```c
RCC->APB2PCENR |= RCC_APB2Periph_GPIOC | RCC_APB2Periph_SPI1;
```

### GPIOの設定

```c
// PC1: NSS
GPIOC->CFGLR &= ~(0xf << (4 * 1));
GPIOC->CFGLR |= (GPIO_Speed_50MHz | GPIO_CNF_OUT_PP_AF) << (4 * 1);

// PC5: SCK
GPIOC->CFGLR &= ~(0xf << (4 * 5));
GPIOC->CFGLR |= (GPIO_Speed_50MHz | GPIO_CNF_OUT_PP_AF) << (4 * 5);

// PC6: MOSI
GPIOC->CFGLR &= ~(0xf << (4 * 6));
GPIOC->CFGLR |= (GPIO_Speed_50MHz | GPIO_CNF_OUT_PP_AF) << (4 * 6);

// PC7: MISO
GPIOC->CFGLR &= ~(0xf << (4 * 7));
GPIOC->CFGLR |= (GPIO_Speed_In | GPIO_CNF_IN_FLOATING) << (4 * 7);
```

### SPIの設定と適用

```c
// SPI設定
// SPI設定をリセット
SPI1->CTLR1 = 0;
SPI1->CTLR1 |=
    // プリスケーラ
    (SPI_CTLR1_BR & SPI_BaudRatePrescaler_32) |
    // SPIモード
    SPI_CPOL_Low | SPI_CPHA_1Edge |
    // NSSの制御モード
    SPI_NSS_Hard |
    // マスターモード
    SPI_Mode_Master;

// SPIの開始
SPI1->CTLR2 |= SPI_CTLR2_SSOE;
```

### SPIの通信

```c
void spi_begin_8()
{
  SPI1->CTLR1 &= ~(SPI_CTLR1_DFF);
  SPI1->CTLR1 |= SPI_CTLR1_SPE;
}

void spi_end()
{
  SPI1->CTLR1 &= ~(SPI_CTLR1_SPE);
}
```

```c
uint8_t spi_transfer(uint8_t data)
{
  // 送信バッファが空になる（書込可能）のを待つ
  while (!(SPI1->STATR & SPI_STATR_TXE))
    ;
  // 1バイトの送信
  SPI1->DATAR = data;

  // 受信バッファにデータが入る（読込可能）のを待つ
  while (!(SPI1->STATR & SPI_STATR_RXNE))
    ;
  // 1バイトの受信
  return SPI1->DATAR;
}
```

```c
void main()
{
  // 略 初期化コード

  // ADT7310の連続変換モードの開始
  spi_begin_8();
  spi_transfer(0x54);
  spi_end();

  Delay_Ms(500);

  while (1)
  {
    printf("loop\r\n");

    // 測定結果の受信
    spi_begin_8();
    uint8_t b1 = spi_transfer(0x00);
    uint8_t b2 = spi_transfer(0x00);
    spi_end();

    uint16_t raw = b1 << 8 | b2;

    Delay_Ms(1000);
  }
}
```
