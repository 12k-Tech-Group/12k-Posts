---
title: double二进制布局
date: 2020-08-26 14:20
author: mq
---

# double二进制布局

## IEEE754

```
+-+-----------+----------------------------------------------------+
|s| exp(11bit)|                     mantissa(52bit)                |
+-+-----------+----------------------------------------------------+
```

* exp存的移码（真实值+1023) (非规格化数指数位为0，真实指数为-1023)
* mantissa二进制小数去掉最高位的非零：1.xxxxxxxx(2) 只保留 x 的部分
* s是符号位

* 真实值 = ${s} \cdot 1 . {mantissa} \cdot 2 ^ { exp - 1023 }$

### e.g. 1.5

```
+-+-----------+----------------------------------------------------+
|0|01111111111|1000000000000000000000000000000000000000000000000000|
+-+-----------+----------------------------------------------------+
|+|   1023    |                  二进制的0.5 (省略首位1)           |
+-+-----------+----------------------------------------------------+
```

### e.g. 最大奇数 (加减乘除不丢精度最大整数范围)
```
+-+-----------+----------------------------------------------------+
|0|10000110011|1111111111111111111111111111111111111111111111111111|
+-+-----------+----------------------------------------------------+
|+| 真实值52  |                 52个1                              |
+-+-----------+----------------------------------------------------+
```
这个数就是 (0b1.<52个1>) * 2^52 = 0b<53个1>

### e.g. 最大值
```
+-+-----------+----------------------------------------------------+
|0|01111111111|1111111111111111111111111111111111111111111111111111|
+-+-----------+----------------------------------------------------+
|+| 真实值1023|                 52个1                              |
+-+-----------+----------------------------------------------------+
```
这个数就是 (0b1.<52个1>) * 2^1023

### e.g. More on the “bias”

* In IEEE 754 floating point numbers, the exponent is biased in the engineering sense of the
word – the value stored is offset from the actual value by the exponent bias.
* Biasing is done because exponents have to be signed values in order to be able to
represent both tiny and huge values, but two's complement, the usual representation for
signed values, would make comparison harder.
* To solve this problem the exponent is biased before being stored, by adjusting its value to
put it within an unsigned range suitable for comparison.
* By arranging the fields so that the sign bit is in the most significant bit position, the biased
exponent in the middle, then the mantissa in the least significant bits, the resulting value
will be ordered properly, whether it's interpreted as a floating point or integer value. This
allows high speed comparisons of floating point numbers using fixed point hardware.
* When interpreting the floating-point number, the bias is subtracted to retrieve the actual
exponent.
* For a single-precision number, an exponent in the range −126 .. +127 is biased by adding
127 to get a value in the range 1 .. 254 (0 and 255 have special meanings).
* For a double-precision number, an exponent in the range −1022 .. +1023 is biased by adding
1023 to get a value in the range 1 .. 2046 (0 and 2047 have special meanings).

### e.g. INF
```
+-+-----------+----------------------------------------------------+
|0|11111111111|0000000000000000000000000000000000000000000000000000|
+-+-----------+----------------------------------------------------+
|+| 全1       |                 全0                                |
+-+-----------+----------------------------------------------------+
```

### e.g. NAN
```
+-+-----------+----------------------------------------------------+
|0|11111111111|XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX|
+-+-----------+----------------------------------------------------+
|+| 全1       |                 非0                                |
+-+-----------+----------------------------------------------------+
```
因此非零的部分可以用来存数据，取值区间是1 ~ 2^52-1

### e.g. 最小规格化正数
```
+-+-----------+----------------------------------------------------+
|0|00000000001|0000000000000000000000000000000000000000000000000000|
+-+-----------+----------------------------------------------------+
|+| 表示-1022 |                 全0                                |
+-+-----------+----------------------------------------------------+
```
表示 1 * 2^-1022

### e.g. 非规格化
```
+-+-----------+----------------------------------------------------+
|0|00000000000|XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX|
+-+-----------+----------------------------------------------------+
|+| 全0       |                 非0                                |
+-+-----------+----------------------------------------------------+
```
指数位全0，真实指数为-1023

表示值为 (0b1.XXXXXXX...) * 2^-1023

非规格化数小于任何规格化数(正的，负的相反)
