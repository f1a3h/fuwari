---
title: "CS:APP 第二章学习笔记"
published: 2023-09-12
description: 'Representing and Manipulating Information'
image: ''
tags: [CSAPP]
category: 'Study Notes'
draft: false 
lang: ''
---

本章主要内容为整数、浮点数的存储与逻辑运算。

## Information Storage

1 byte = 8 bits ，为 memory 的最小存储单元的大小。

word size 是主要的 pointer data 的大小，也是 virtual address space 的最大 size 。

C 语言中，应该使用 `signed char` 来得到一个 1 byte 的有符号值。

### Byte Ordering

对于一个十六进制数 `0x01234567` ，在计算机中有两种表示方法：Big endian 和 Little endian 。

Big endian 相对来说更符合习惯，在存储单元中，低位 address 存储高位 byte ，即 `01 23 45 67` 这样依次存储。

Little endian 则是高位 address 存储高位 byte ，这样看来又显得比较符合习惯了，即 `67 45 23 01` 依次存储。

因为我们将寄存器或存储单元中的 bytes 表达出来习惯将 address 小的写在前面，所以 Little endian 会让人感觉很奇怪，但是实际访问某个存储单元时，我们更多的是类似于栈一样从高位地址开始向低位地址依次访问，这样一来，Little endian 依次访问到的 bytes 实际上是从小到大的了。

计算机使用哪种表示方式，既与 CPU 有关又与 OS 有关。

string 的表示与 byte ordering 无关。

### Shift Operations in C

左移都是往低位补 0 ，右移则分为 logical 和 arithmetic ，logical 是高位补 0 ，arithmetic 是根据最高位决定补 0 还是 1 。

对于 signed data ，一般使用 arithmetic 。

对于 unsigned data ，必须使用 logical 。

## Integer Representations

unsigned integer 的表示就是一般的二进制形式。

signed integer 则采用补码 (two\'s-complement encoding)，最高位为符号位，若 word size 为 $\omega$ ，那么最高位表示为 $-2^{\omega-1}\cdot x_{\omega-1}$ ，其余位的表示与 unsigned 相同。

### Convertions between Signed and Unsigned

word size 相同时，bit 位的表示不变，仅仅改变二进制到十进制的转换方式。

当判断表达式的类型是 signed 还是 unsigned 的时，一般出现了 unsigned 就是 unsigned ，并且会将 signed 隐式转换为 unsigned ，这种情况下很容易发生意想不到的错误，所以一般**尽量不使用 unsigned integer** 。

### Expanding the Bit Representation

对于 unsigned integer ，直接在高位补 0 。

对于 signed integer ，则在高位补原来的最高位的 bit ，类似于 arithmetic right shift 。

如果既要增加长度，又要类型转换，则会先增加长度再进行类型转换。

### Truncating Numbers

对于 unsigned integer ，直接把多余的高位去掉，假设只保留低 k 位 bits ，则相当于 $\bmod{\ 2^k}$ 。

对于 signed integer ，先转换为 unsigned 再做 unsigned truncation ，之后再转换回 signed ，其实也相当于对 $2^k$ 取模。

## Integer Arithmetic

### Addition and Negation

unsigned integer 的加法，直接丢弃溢出的那一位，相当于模 $2^{\omega}$ 。

判断 overflow ：看 `s = x + y` 是否有 `s < x` 或 `s < y` 。

unsigned integer 的取反：

$$
-x\mapsto\begin{cases}
    x, &x=0 \\
    2^{\omega}-x, &x>0
\end{cases}
$$

signed integer 的加法，过程同 unsigned ，overflow 有 positive overflow 和 negative overflow ，仍然相当于模 $2^{\omega}$ 。

判断 overflow ：设 `s = x + y` ，发生 positive overflow 当且仅当 `x > 0 && y > 0 && s <= 0` ，发生 negative overflow 当且仅当 `x < 0 && y < 0 && s >= 0` 。

signed integer 的取反：当成 unsigned 来取反即可，有

$$
-x\mapsto\begin{cases}
    TMin_{\omega}, &x=TMin_{\omega} \\
    -x, &x>TMin_{\omega}
\end{cases}
$$

### Multiplication

都是丢高位，相当于模 $2^{\omega}$ 。

如果某个乘数是常数，则会优化为移位运算、加法运算和减法运算的结合。

优化的性能与这些指令的相对速度、机器本身有关。大部分编译器只会在只有少量位移、加法、减法的情况下采取这种优化。

### Dividing by Powers of 2

都会优化为右移运算。

其中 unsigned integer 是直接 logistcal right shift ，相当于向下取整了。

signed integer 则是 arithmetic right shift ，效果也是向下取整，如果希望向上取整（对于负数就是向零取整了），则需要先加上 bias 在右移，其中 bias 定义为 $2^k-1$ ，k 是要右移的位数。

### Summary

unsigned integer 和 signed integer 在处理这些简单的运算的溢出时相当于于模 $2^{\omega}$ ，补码的表示方式使得二者在 bit 表示层面进行运算的形式一致。

## Floating Point

浮点数的表示一直是个难题。

### IEEE floating point

IEEE floating point 将一个浮点数表示为 $V=(-1)^{s}\times M\times 2^{E}$ 的形式。

1. sign bit ，为符号位 $s$
2. $n$-bit fraction field ，为 $M$ ，下文记 fraction field 为 $f=f_{n-1}\cdots f_1 f_0$
3. $k$-bit exponent field ，为 $E$ ，下文记 exponent field 为 unsigned number $e=e_{k-1}\cdots e_1e_0$

根据 $E$ 的不同，浮点数被分为三类：

- Normalized values:
  - exponent field 不全为 0 也不全为 1
  - $M=1.f_{n-1}\cdots f_1 f_0$
  - $E=e-Bias$ ，其中 $Bias=2^{k-1}-1$
- Denormalized values:
  - exponent field 全为 0
  - $M=0.f_{n-1}\cdots f_1 f_0$
  - $E=1-Bias$
- Special values:
  - exponent field 全为 1
  - 当 fraction field 全为 0 时：
    - `s = 0` ，$+\infty$
    - `s = 1` ，$-\infty$
  - 当 fraction field 不全为 0 时，表示 `NaN`

其中，不同位数的浮点数的不同 field 有不同的长度：

- 64 位 ( `double` ) : k = 11, n = 52
- 32 位 ( `float` ) : k = 8, n = 23

如果简单地使用二进制表示 $E$ 和 $M$ 会出现一些问题：

- $E$ 需要是负数才能表示接近 0 的数，占用了 $e$ 的一个位而导致能表示的最大数缩小一半
- 若 $M=0.f_{n-1}\cdots f_1 f_0$ ，则不同的 $E$ 可能会得到同一个值，不方便进行浮点数之间的比较

而 IEEE 采用的表示方法，

- 加入了 $Bias$ 解决了第一个问题
- 特殊情况下让 $M$ 的 leading bit 变为 1 使得两个浮点数本身的大小与其二进制编码看作 unsigned 后的大小一致

### Example Numbers

exponent field 和 fraction field 全为 0 ，表示 $\pm0.0$ 。

exponent field 全为 0 而 fraction field 最低位为 1 其他位为 0 ，是能表示的最接近 0 的数，为 $2^{2-2^k-n}$ 。

exponent field 最高位为 0 其他位全为 1 ，fraction field 全为 0 ，表示 $\pm1.0$ 。

exponent field 最低位为 0 其他位全为 1 ，fraction field 全为 1 ，是能表示的最大的数。

### Rounding

- round-to-even ：「四舍六入五成双」的二进制形式
- rounf-toward-zero
- round-down
- round-up

### Floating-Point Operations

除了一部分特殊值，浮点数的运算定义为先进行精确的运算之后再进行舍入。

浮点数的加法、乘法会发生溢出、舍入，因此不符合结合律、分配律，但是满足交换律。