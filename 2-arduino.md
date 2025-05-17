# 2. Arduinoでの開発

## 2.1. Arduinoでの開発の準備

```
https://raw.githubusercontent.com/robinjanssens/WCH32V_board_manager_files/main/⏎
  package_ch32v_index.json
```

## 2.2. まずLチカを成功させる

```c
void setup() {
  pinMode(PC0, OUTPUT);
}

void loop() {
  digitalWrite(PC0, HIGH);
  delay(1000);
  digitalWrite(PC0, LOW);
  delay(1000);
}
```

## 2.3. Arduinoでのファームウェアの開発

### Arduinoでのピンの名前の定義

### ログ出力

```c
void setup()
{
  Serial.begin(115200);
  Serial.println("start");
}

int count = 0;

void loop()
{
  Serial.print("loop ");
  Serial.println(count++);
  delay(500);
}
```

## 2.4. Arduinoのコードからレジスタを操作する

```c
uint32_t pin_name = digitalPinToPinName(PC1);
GPIO_TypeDef *gpio = get_GPIO_Port(CH_PORT(pin_name));
```

```c
  uint32_t pin_name = digitalPinToPinName(PC1);
  uint32_t pin_mask = CH_GPIO_PIN(pin_name);
```

```c
uint32_t pin_name = digitalPinToPinName(PC1);
GPIO_TypeDef *gpio = get_GPIO_Port(CH_PORT(pin_name));
uint32_t pin_mask = CH_GPIO_PIN(pin_name);
gpio->BSHR = pin_mask;
```

## 2.5. まとめ
