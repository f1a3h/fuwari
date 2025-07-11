---
title: "GCD && LCM 复习笔记"
published: 2023-09-15
description: 'Greatest Common Divisor && Least Common Multiple'
image: ''
tags: [Number Theory]
category: 'Algorithm'
draft: false 
lang: ''
---

## 最大公约数

GCD ，即 Greatest Common Divisor .

一组整数的最大公约数就是指这组整数的最大的公共约数 .

~~怎么全是废话~~

那么要如何求解一组整数的 gcd 捏？

我们先考虑两个数 $a,b$ 的情况 .

令 $a=bq+r(q,r\in\mathbb{N})$ ，则 $a \bmod b=r\in[0,b)$ ，假设 $d=\gcd(a,b)$ ，有 $d\mid a,~d\mid b$ ，因此 $d\mid r$，于是我们发现 $\gcd(b,r)=d=\gcd(a,b)$ .

综上，$\gcd (a,b)=\gcd(b,a\bmod b)$ ，于是我们可以利用这个性质不断递归求解直到 $b=0$ ，这时候的 a 就是 gcd 了.

```cpp
int gcd(int a, int b) {
    return b ? gcd(b, a % b) : a;
}
```

当整数的数量大于 2 时，我们求解前两个数的 gcd 与后一个数的 gcd ，便得到前 3 个数的 gcd ，依次进行下去，最后得到的便是这 n 个数的 gcd .

对了，这个算法就是欧几里得算法 .

## 最小公倍数

LCM ，即 Least Common Multiple .

一组整数的公倍数，是指同时是这组数中每一个数的倍数的数 . 0 是任意一组整数的公倍数 .

一组整数的最小公倍数，是指所有正的公倍数里面，最小的一个数 .

~~语言贫瘠，直接蒯 oi-wiki 上的了~~

同样也是先考虑两个数 $a,b$ 的情况 .

将 $a,b$ 写成素数的乘积的形式 .

$$a=p_1^{k_{a_1}}p_2^{k_{a_2}}\cdots p_n^{k_{a_{n}}}$$

$$b=p_1^{k_{b_1}}p_2^{k_{b_2}}\cdots p_n^{k_{b_{n}}}$$

根据定义可知 ，

$$\operatorname{lcm}(a,b)=p_1^{\min(k_{a_1},k_{b_1})}p_2^{\min(k_{a_2},k_{b_2})}\cdots p_n^{\min(k_{a_n},k_{b_{n}})}$$

$$\gcd(a,b)=p_1^{\max(k_{a_1},k_{b_1})}p_2^{\max(k_{a_2},k_{b_2})}\cdots p_n^{\max(k_{a_n},k_{b_{n}})}$$

而 $\max(a,b)+\min(a, b)=a+b$ ，因此 $\gcd(a,b)\times\operatorname{lcm}(a,b)=a\times b$ .

所以 $\operatorname{lcm}(a,b)=\dfrac{a\times b}{\gcd(a,b)}$ .

对于多于 2 的整数数目的情况，处理方法和 gcd 是一样的 .

## 扩展欧几里得算法

扩展欧几里得算法（Extended Euclidean algorithm, EXGCD）常用来求解 $ax+by=\gcd(a,b)$ 的一组可行解的情况 .

我们设 $ax_1+by_1=\gcd(a,b)$ ，$bx_2+(a\bmod b)y_2=\gcd(b,a\bmod b)$ .

下面我们来看看 $x_1,y_1$ 和 $x_2,y_2$ 之间的关系 .

将 $a\bmod b=a-\lfloor\frac{a}{b}\rfloor\times b$ 和 $\gcd(a,b)=\gcd(b,a\bmod b)$ 带入第二个方程，得到

$$bx_2+ay_2-\lfloor\frac{a}{b}\rfloor\times b\times y_2=\gcd(a,b)$$

整理得到

$$ay_2+b(x_2-\lfloor\frac{a}{b}\rfloor y_2)=\gcd(a, b)$$

与第一个方程比对可以发现 $x_1=y_2,y_1=x_2-\lfloor\frac{a}{b}\rfloor y_2$ .

因此我们可以不断递归求解 $\gcd(a,b)$ 直到 $b=0$ ，显然此时的 $x_n=1$ ，然后考虑 $x_{n-1},y_{n-1}$ 的情况，此时的 $b=\gcd(a,b)$ ，而 $a\not= 0$ ，所以 $x_{n-1}=y_n=0$ ，综上，我们得到边界情况 $x_n=1,y_n=0$ 同时求得了 $\gcd(a,b)$ .

最后我们在不断回溯时修改 $x,y$ 的值即可得到 $ax+by=\gcd(a, b)$ 的一组可行解 .

```cpp
int exgcd(int a, int b, int &x, int &y) {
    if (!b) {
        x = 1, y = 0;
        return a;
    }
    int ret = exgcd(b, a % b, x, y);
    int tmp = x;
    x = y, y = tmp - a / b * y;
    return ret;
}
```