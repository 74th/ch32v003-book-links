# 4. ch32funでの開発とレジスタ操作

## 4.1. ch32funとは

## 4.2. 開発環境の構築

> Installation · cnlohr/ch32fun Wiki<br/>https://github.com/cnlohr/ch32fun/wiki/Installation

## 4.3. プロジェクトのセットアップ

```makefile
all : flash

TARGET:=main

TARGET_MCU?=CH32V003
include ../ch32fun/ch32fun/ch32fun.mk

flash : cv_flash
clean : cv_clean
```

### ch32funのリポジトリから独立させる

```
git clone https://github.com/cnlohr/ch32fun.git
cd ch32fun/minichlink
make all
sudo cp minichlink /usr/local/bin/
```

```bash
CH32FUN=/Users/nnyn/ghq/github.com/cnlohr/ch32fun

# プロジェクトのテンプレート
cp -rf ${CH32FUN}/examples/template/* ${CH32FUN}/examples/template/.* ./

# ch32funのコアライブラリ
mkdir ch32fun
cp ${CH32FUN}/ch32fun/ch32fun.* ./ch32fun/
cp ${CH32FUN}/ch32fun/ch32v003hw.h ./ch32fun/
cp -rf ${CH32FUN}/LICENSE ./ch32fun/

# ビルド、デバッグに必要なファイル
mkdir misc
cp ${CH32FUN}/misc/libgcc.a ./misc/
cp ${CH32FUN}/misc/CH32V003xx.svd ./misc/
cp ${CH32FUN}/misc/LIBGCC_LICENSE ./misc/
```

```bash
mv template.c standalone.c
```

#### 変更前

```makefile
# Makefile
all : flash

TARGET:=template

TARGET_MCU?=CH32V003
include ../../ch32fun/ch32fun.mk

flash : cv_flash
clean : cv_clean
```

#### 変更後

```makefile
# Makefile
all : flash

# プロジェクト名を指定
TARGET:=standalone

TARGET_MCU?=CH32V003

# minichlinkのパスを指定
MINICHLINK:=$(shell dirname $(shell which minichlink))

# コピーしたch32funのコアライブラリの場所を指定
include ./ch32fun/ch32fun.mk

flash : cv_flash
clean : cv_clean
```

#### 変更前

```makefile
cv_flash : $(TARGET).bin
	make -C $(MINICHLINK) all
	$(FLASH_COMMAND)

```

#### 変更後

```Makefile
cv_flash : $(TARGET).bin
#	make -C $(MINICHLINK) all
	$(FLASH_COMMAND)
```

## 4.4. ビルド、書き込み、デバッグプリント

```
make
```

```
minichlink -T
```

```
make monitor
```

```
make build cv_flash monitor
```

## 4.5. ch32funの最低限のコード

```c
#include "ch32fun.h"
#include <stdio.h>

int main()
{
  SystemInit();

  printf("init\n");

  while (1)
  {
    printf("loop\n");
    Delay_Ms(1000);
  }
}
```

## 4.6. extralibsを使う

```c
#include "ch32v003_GPIO_branchless.h"
```

## 4.7. VS Codeで補完を効かせるには

## 4.8. PlatformIOを使う

## 4.9. レジスタ操作の基礎

```c
// 3bit目だけ1をセット
GPIOA->OUTDR |= 0x1 << 3;

// 3bit目だけ0をセット
GPIOA->OUTDR &= ~(1 << 3);
```

```c
// 12-15bitを0にリセット
GPIOC->CFGLR &= ~(0xf << (4 * 3));
// 12-15bitに設定を適用
// GPIO_Speed_50MHz: 0x0011
// GPIO_CNF_OUT_OD: 0x0100
GPIOC->CFGLR |= (GPIO_Speed_50MHz | GPIO_CNF_OUT_OD) << (4 * 3);
```

## 4.10. rv003usbとは

> cnlohr/rv003usb: CH32V003 RISC-V Pure Software USB Controller<br/>https://github.com/cnlohr/rv003usb
