# 9. UART

## 9.1. CH32V003のUART

## 9.2. サンプル回路

## 9.3. Arduino

> https://github.com/74th/ch32v003-book-code/tree/main/uart-arduino_core_ch32

```c
Serial.begin(9600);
```

```c
// 1バイトの送信
size_t Serial.write(uint8_t val)
// テキストの送信
size_t Serial.write(char *str)
// バイト列の送信
// 第1引数にバッファ、第2引数にバッファの長さを指定する
size_t Serial.write(uint8_t buf[], size_t len)
```

```c
uint8_t CMD_READ_CO2_CONNECTION[9] =⏎
  { 0xFF, 0x01, 0x86, 0x00, 0x00, 0x00, 0x00, 0x00, 0x79 };

Serial.write(CMD_READ_CO2_CONNECTION, sizeof(CMD_READ_CO2_CONNECTION));
```

```c
// 1バイトの受信
// 受信できるまでブロックする
uint8_t Serial.read()
// 複数バイトの受信
// 第1引数にバッファ、第2引数にバッファの長さを指定する
// 戻り値は受信したバイト数
// タイムアウト機能があり、受信できない場合は0を返す
size_t readBytes(uint8_t *buf, size_t len)
// 複数バイトの受信
// readBytesの機能に加え、第1引数に終了文字を指定できる
size_t readBytesUntil(uint8_t terminator, uint8_t *buf, size_t len)
```

```c
uint8_t read_buf[9] = { 0 };
uint32_t len = Serial.readBytes(read_buf, sizeof(read_buf));
// MH-Z19Cの応答からデータの抽出
uint16_t co2 = read_buf[2] * 256 + read_buf[3];
```

## 9.4. WCH SDK

> https://github.com/74th/ch32v003-book-code/tree/main/uart-wch_sdk

### クロック供給の有効化

```c
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC | RCC_APB2Periph_GPIOD |⏎
    RCC_APB2Periph_USART1, ENABLE);
RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
```

### GPIOのピン設定と、AFIOのリマップ設定

```c
GPIO_InitTypeDef GPIO_InitStructure = {0};

// PD5: TX
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_5;
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_30MHz;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
GPIO_Init(GPIOD, &GPIO_InitStructure);

// PD6: RX
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;
GPIO_Init(GPIOD, &GPIO_InitStructure);
```

```c
// - GPIO_PartialRemap1_USART1
// - GPIO_PartialRemap2_USART1
// - GPIO_FullRemap_USART1
GPIO_PinRemapConfig(GPIO_FullRemap_USART1, ENABLE);
```

### UARTの設定と有効化

```c
USART_InitTypeDef USART_InitStructure = {0};

// ボーレート
USART_InitStructure.USART_BaudRate = 9600;
// 8bitデータ
USART_InitStructure.USART_WordLength = USART_WordLength_8b;
// ストップビットは1bit
USART_InitStructure.USART_StopBits = USART_StopBits_1;
// パリティなし
USART_InitStructure.USART_Parity = USART_Parity_No;
// フロー制御はなし
USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
// 送信(Tx)と受信(Rx)の両方を有効化
USART_InitStructure.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;

// 設定の反映
USART_Init(USART1, &USART_InitStructure);
// UARTの有効化
USART_Cmd(USART1, ENABLE);
```

### 送信

```c
void write_uart(uint8_t *buf, uint16_t len)
{
  for (uint16_t i = 0; i < len; i++)
  {
    // 準備完了まで待つ
    while (USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET)
      ;

    // 送信データをセット
    USART_SendData(USART1, buf[i]);
  }

  // 送信完了まで待つ
  while (USART_GetFlagStatus(USART1, USART_FLAG_TC) == RESET)
    ;
}
```

```c
uint8_t CMD_READ_CO2_CONNECTION[9] =⏎
    {0xFF, 0x01, 0x86, 0x00, 0x00, 0x00, 0x00, 0x00, 0x79};
write_uart(CMD_READ_CO2_CONNECTION, sizeof(CMD_READ_CO2_CONNECTION));
```

### 受信

```c
#define TIMEOUT_MAX 100000

uint16_t read_uart_with_timeout(uint8_t *buf, uint16_t len)
{
  for (uint16_t i = 0; i < len; i++)
  {
    int32_t timeout = TIMEOUT_MAX;
    // 受信完了まで待つ
    while (timeout-- && USART_GetFlagStatus(USART1, USART_FLAG_RXNE) == RESET)
      ;

    if (timeout < 0)
    {
      return i;
    }

    // 受信データを読み込む
    buf[i] = USART_ReceiveData(USART1);
  }

  return len;
}
```

```c
uint8_t read_buf[9] = {0};

read_len = read_uart_with_timeout(&read_buf, 7);
if (read_len < 7)
{
    printf("timeout\r\n");
    Delay_Ms(1000);
}

uint16_t co2 = read_buf[2] * 256 + read_buf[3];
```

## 9.5. レジスタ操作

> https://github.com/74th/ch32v003-book-code/tree/main/uart-register

### クロック供給の有効化

```c
RCC->APB2PCENR |= RCC_APB2Periph_GPIOC | RCC_APB2Periph_GPIOD |⏎
    RCC_APB2Periph_USART1;
RCC->APB2PCENR |= RCC_APB2Periph_AFIO;
```

### GPIOのピン設定と、AFIOのリマップ設定

```c
// PD5: TX
GPIOD->CFGLR &= ~(0xf << (4 * 5));
GPIOD->CFGLR |= (GPIO_Speed_10MHz | GPIO_CNF_OUT_PP_AF) << (4 * 5);
// PD6: RX
GPIOD->CFGLR &= ~(0xf << (4 * 6));
GPIOD->CFGLR |= (GPIO_Speed_In | GPIO_CNF_IN_FLOATING) << (4 * 6);
```

```c
// REMAPの設定を削除
AFIO->PCFR1 &= ~(AFIO_PCFR1_USART1_REMAP | AFIO_PCFR1_USART1_REMAP_1);
// Partial Remap 1を使う場合
AFIO->PCFR1 |= AFIO_PCFR1_USART1_REMAP;
// Partial Remap 2を使う場合
AFIO->PCFR1 |= AFIO_PCFR1_USART1_HIGH_BIT_REMAP;
// Full Remapを使う場合
AFIO->PCFR1 |= AFIO_PCFR1_USART1_REMAP | AFIO_PCFR1_USART1_REMAP_1;
```

### UARTの設定と有効化

```c
// UARTの設定
USART1->CTLR1 =
  // バイト長8bit
  USART_WordLength_8b |
  // パリティなし
  USART_Parity_No |
  // TX有効化
  USART_Mode_Tx |
  // RX有効化
  USART_Mode_Rx;
// ストップビット1
USART1->CTLR2 = USART_StopBits_1;
// フロー制御なし
USART1->CTLR3 = USART_HardwareFlowControl_None;
// ボーレート9600
USART1->BRR = (((FUNCONF_SYSTEM_CORE_CLOCK) + (9600) / 2) / (9600));
// 有効化
USART1->CTLR1 |= CTLR1_UE_Set;
```

### 送信

```c
void write_uart(uint8_t *buf, uint16_t len)
{
  for (uint16_t i = 0; i < len; i++)
  {
    // バッファが空いて次のデータが入れられるまで待つ
    while (!(USART1->STATR & USART_FLAG_TXE))
      ;

    // 送信データをバッファにセット
    USART1->DATAR = buf[i];
  }

  // 送信完了まで待つ
  while (!(USART1->STATR & USART_FLAG_TC))
    ;
}
```

### 受信

```c
uint16_t read_uart_with_timeout(uint8_t *buf, uint16_t len)
{
  for (uint16_t i = 0; i < len; i++)
  {
    int32_t timeout = TIMEOUT_MAX;
    // 受信データが来るまで待つ
    while (timeout-- && !(USART1->STATR & USART_FLAG_RXNE))
      ;

    if (timeout < 0)
    {
      return i;
    }

    // 受信データを読み込む
    buf[i] = USART1->DATAR;
  }

  return len;
}
```
