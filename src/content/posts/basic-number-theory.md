---
title: "基础数论复习"
published: 2023-08-29
description: '旧文搬运'
image: ''
tags: ['Number Theory']
category: 'Algorithm'
draft: false 
lang: ''
---

数论，是一门研究整数的纯数学的分支，而整数的基本元素是素数(也称质数)，所以数论的本质是对素数性质的研究。

## 素数

一些基本概念就不写了.

注意两点:

1. **0、1、负整数**既不是素数也不是合数.
2. $\pi(n)$ 表示不大于 $n$ 的素数个数, 有如下近似关系 $\pi(n)\sim\dfrac{n}{\ln{n}}$ .

### 素数判定

如何判定正整数 $n$ 是否是素数?

很容易想到用每一个小于 $n$ 且大于 $1$ 的数来判断它是否是 $n$ 的约数, 若都不是, 则 $n$ 为素数, 否则为合数.

这个暴力算法是 $\mathcal{O}(n)$ 的, 但是我们考虑到, 若数 $p$ 为 $n$ 的约数, 则 $\frac{n}{p}$ 也是 $n$ 的约数, 所以我们只需用 $2\sim \sqrt{n}$ 的数来判定即可, 于是优化一下我们得到一个 $\mathcal{O}(\sqrt{n})$ 的算法.

```cpp
bool isPrime(int n) {
  if (n == 1) return false;
  if (n == 2) return true;
  for (int i = 2; i * i <= n; ++i)
    if (n % i == 0) return false;
  return true;
}
```

### 整数惟一分解定理

> 若整数 $n\ge2$ , 那么 $n$ 一定可以惟一地表示为若干素数的乘积, 形如
>
> $$n=p_1^{r_1}p_2^{r_2}\cdots p_k^{r_k}(p_i为素数, r_i\ge0)$$ 

```cpp
vector <int> factor(int n) {
  vector <int> ret;
  for (int i = 2; i * i <= n; ++i)
    while (n % i == 0) {
      ret.push_back(i);
      n /= i;
    }
  if (n > 1) ret.push_back(n);
  return ret;
}
```

## 素数筛法

主要是埃氏筛法和欧拉筛法.

### Eratosthenes 筛法

Eratosthenes 筛法可以快速列举出给定范围内的所有素数.

一个合数一定可以写成 $p\cdot x$ 的形式, 其中 $p$ 为素数, $x$ 为整数, 因此对每一个 $p$ , 我们从小到大枚举 $x$ , 筛掉相应的 $p\cdot x$ 即可.

一点小改进: 当 $x<p$ 时, $p\cdot x$ 一定被 $x$ 的素因子筛过一遍, 所以我们枚举 $x$ 时从 $x=p$ 开始枚举.

时间复杂度: $\mathcal{O}(\frac{n}{2}+\frac{n}{3}+\frac{n}{5}+\cdots)=\mathcal{O}(n\log{\log{n}})$

```cpp
bool isprime[N];

void Eratos(int n) {
  memset(isprime, 1, sizeof isprime);
  isprime[0] = isprime[1] = 0;
  for (int i = 2; i * i <= n; ++i)
    if (isprime[i]) {
      for (int j = i * i; j <= n; j += i)
        isprime[j] = 0;
    }
}
```

#### 例题

[「POJ 2689」Prime Distance](http://poj.org/problem?id=2689)

由于这里 $L,R$ 的范围很大, 暴力筛完之后枚举求解肯定会 T , 但是 $R-L\le 10^6$ 的范围却是在我们的承受范围之内的.

于是我们考虑用 $1\sim n$ 这一小范围的素数筛掉 $L\sim R$ 之间的合数.

假设数 $k\in[L, R]$ , 其中 $k$ 的最小质因数为 $p$ , 满足 $p>n$ , 则 $k=p\cdot x$ , 若 $x<p$ , 则 $k$ 一定存在小于 $p$ 的质因数, 矛盾, 故而 $x\ge p$ , 也就是说, 会被漏筛的数 $k$ 满足 $k\ge p^2$ , 为了保证算法的正确性, 我们要让这样的数落在 $[L, R]$ 外, 也就是让 $p^2>R$ , 于是我们令 $n=\sqrt{R}$ 即可.

#### 筛法优化质因数分解

利用埃氏筛法可以快速实现素因数分解, 只要在筛的时候记录下每个数的最小素因数即可.

```cpp
bool isprime[N];
int minfac[N];

void Eratos(int n) {
  memset(isprime, 1, sizeof isprime);
  isprime[0] = isprime[1] = 0;
  for (int i = 1; i <= n; ++i) minfac[i] = i;
  for (int i = 2; i * i <= n; ++i)
    if (isprime[i]) {
      for (int j = i * i; j <= n; j += i) {
        isprime[j] = 0;
        if (minfac[j] == j) minfac[j] = i;
      }
    }
}

vector <int> factor(int n) {
  vector <int> ret;
  while (n > 1) {
    ret.push_back(minfac[n]);
    n /= minfac[n];
  }
  return ret;
}
```

### Euler 筛法

在埃氏筛中, 30 这个数被 $p=2,3,5$ 分别筛了三次, 造成了重复, 如果我们能让每个数只被它最小的素因数筛, 就能做到 $\mathcal{O}(n)$ 的时间复杂度.

在 Euler 筛法中, 我们从 $2\sim n$ 枚举每一个数 $i$ , 若 $i$ 是素数, 则加入素数表中, 然后我们再从小到大枚举前面的素数 $prime[j]$ , 筛掉 $i\cdot prime[j]$ , 当 $prime[j]$ 是 $i$ 的一个素因数时, 那么后面再继续枚举到的素数一定不是所筛的数的最小素因数, 毕竟后面的素数一定大于当前的 $prime[j]$ , 所以这时可以终止筛选.

时间复杂度: $\mathcal{O}(n)$

```cpp
int prime[N];
bool isprime[N];

void Euler(int n) {
  memset(isprime, 1, sizeof isprime);
  isprime[0] = isprime[1] = 0;
  for (int i = 2; i <= n; ++i) {
    if (isprime[i]) prime[++prime[0]] = i;
    for (int j = 1; j <= prime[0] && prime[j] * i <= n; ++j) {
      isprime[i * prime[j]] = 0;
      if (i % prime[j] == 0) break;
    }
  }
}
```

## 约数

### 整数惟一分解定理的推论

$$N=p_1^{r_1}p_2^{r_2}\cdots p_k^{r_k}$$

1. $N$ 的正约数的个数为 $(r_1+1)(r_2+1)\cdots(r_k+1)=\prod_{i=1}^k(r_i+1)$
2. $N$ 的约数个数上界为 $\sqrt{N}$
3. $N$ 的所有正约数的和为 $(1+p_1+p_1^2+\cdots+p_1^{r_1})\cdots(1+p_k+\cdots+p_k^{r_k})=\prod_{i=1}^k(\sum_{j=0}^{r_i}p_i^j)$
4. 时间复杂度: $\mathcal{O}(\sqrt{N})$

#### 例题

[「Luoggu P1463」反素数](https://www.luogu.com.cn/problem/P1463)

这道题只需要知道 $1\sim 2\cdot 10^9$ 内前 12 个素数的积就已经超出 $N$ 的范围, 所以打个表然后直接爆搜即可.

(亏我想了半天要怎么转化, 完全没考虑只有 12 个素数会被用到)

## 最大公约数

左转 [gcd & lcm 复习笔记](../gcd-and-lcm/)

### 高精度 $\gcd{(a, b)}$

更相减损术适用于大整数下求 $\gcd(a, b)$

1. $a<b$ 时, $\gcd(a, b)=\gcd(b, a)$
2. $a=b$ 时, $\gcd(a, b)=a$
3. $a, b$ 同为偶数时, $\gcd(a, b)=2\gcd(\frac{a}{2}, \frac{b}{2})$
4. $a$ 为偶数, $b$ 为奇数时, $\gcd(a, b)=\gcd(\frac{a}{2}, b)$
5. $a$ 为奇数, $b$ 为偶数时, $\gcd(a, b)=\gcd(a, \frac{b}{2})$
6. $a, b$ 同为奇数时, $\gcd(a, b)=\gcd(b, a-b)$

## 容斥原理

原理很简单, 记住一个公式即可

$$|\cap A_i|=\sum_{1\le i\le m}|A_i|-\sum_{1\le i< j\le m}|A_i\cap A_j|+\sum_{1\le i<j<k\le m}|A_i\cap A_j\cap A_k|-\cdots+(-1)^{m+1}\sum|A_1\cap A_2\cap\cdots\cap A_m|$$

### 错排问题

1. 利用容斥原理求解

   令集合 $S$ 表示 $1,2, \cdots, n$ 的全排列, 则 $|S|=n!$

   设子集 $S_i$ 表示 $i$ 排在第 $i$ 位的排列, 则 $|S_i|=(n-1)!$

   同理, 设 $S_{i_1}\cap S_{i_2}\cap\cdots\cap S_{i_k}$ 表示 $i_1, i_2, \cdots, i_k$ 这 $k$ 个数排在相应位置的排列数, 则 $|S_{i_1}\cap S_{i_2}\cap\cdots\cap S_{i_k}|=(n-k)!$

   每个元素都不排在自己位置的排列数为 $D_n=|\overline{S_1}\cap\overline{S_2}\cap\cdots\cap\overline{S_n}|=|S|-|S_1\cup S_2\cup\cdots\cup S_n|$

   由容斥原理可得

   $$D_n=n!(1-\frac{1}{1!}+\frac{1}{2!}-\frac{1}{3!}+\cdots\pm\frac{1}{n!})$$

2. 递推公式

   推导过程并不难想, 懒得写了, 直接上公式

   $$D_n=(n-1)(D_{n-1}+D_{n-2})$$

## 欧拉函数

1. 若 $\gcd{(a, b)}=1$ , 则称 $a, b$ 互质(互素), 记作 $a\bot b$ 
2. 欧拉函数 $\varphi(n)$ 定义为 $[1, n]$ 中与 $n$ 互质的数的个数.
3. 若 $n$ 为素数, 则 $\varphi(n)=n-1$

### 容斥原理求欧拉函数

根据整数惟一分解定理, 将 $N$ 分解为

$$N=p_1^{r_1}p_2^{r_2}\cdots p_k^{r_k}$$

设 $[1, n]$ 中 $p_i$ 的倍数的集合为 $A_i$ , 则 $|A_i|=\lfloor \frac{N}{p_i}\rfloor$

对于 $p_i\not=p_j$ , 有 $|A_i\cap A_j|=\lfloor\frac{N}{p_ip_j}\rfloor$

根据容斥原理, 

$$\begin{aligned}\varphi(N)&=|A|-|\overline{A_1}\cup\overline{A_2}\cup\cdots\cup\overline{A_k}| \\\ &=N-(\frac{N}{p_1}+\frac{N}{p_2}+\cdots+\frac{N}{p_k})+(\frac{N}{p_1p_2}+\cdots+\frac{N}{p_1p_k})-\cdots\pm\frac{N}{p_1p_2\cdots p_k} \\\ &=N(1-\frac{1}{p_1})(1-\frac{1}{p_2})\cdots(1-\frac{1}{p_k})\end{aligned}$$

### 素因数分解求欧拉函数的算法

时间复杂度: $\mathcal{O}(\sqrt{n})$

```cpp
int euler_phi(int n) {
  int ret = n;
  for (int i = 2; i * i <= n; ++i)
    if (n % i == 0) {
      ret = ret / i * (i - 1);//先除后乘可以一定程度上避免爆 int
      while (n % i == 0) n /= i;
    }
  if (n > 1) ret = ret / n * (n - 1);
  return ret;
}
```

### 埃氏筛法预处理欧拉函数值

时间复杂度: $\mathcal{O}(n\log{\log{n}})$

```cpp
int phi[N];

void euler_phi(int n) {
  for (int i = 2; i <= n; ++i) phi[i] = i;
  for (int i = 2; i * i <= n; ++i)
    if (phi[i] == i) {
      phi[i] = i - 1;
      for (int j = i; j <= n; j += i)//注意这里 j 的起点是 i 而不是 i * i
        phi[j] = phi[j] / i * (i - 1);
    }
}
```

### 欧拉函数的性质

若 $a\bot b$ , 则 $\varphi(a)\varphi(b)=\varphi(ab)$ , 即欧拉函数具有积性函数的性质.

上式分别将 $a, b$ 用整数惟一分解定理表示, 然后带入欧拉函数计算式即可得到证明.

#### 例题

[「POJ 3090」Visible Lattice Points](http://poj.org/problem?id=3090)

设某点坐标为 $(x_0, y_0)$ , 若该点能被看见, 则这个点一定是直线 $y=\frac{y_0}{x_0}x$ 上距离原点最近的一个整数点.

我们设一点 $(x_1, y_1)$ 满足 $\frac{y_1}{x_1}=\frac{y_0}{x_0}, x_1>x_0$ , 则该点显然不可见, 从这个式子中我们可以知道, $x_1, y_1$ 必然存在一个公约数, 而 $x_0, y_0$ 不存在公约数, 即 $x_0, y_0$ 互质, 因此可见点就是图中所有满足 $x_0, y_0$ 互质的点再加上 $(1, 0), (0, 1), (1, 1)$ .

固定 $y=y_0$ , 当 $x<y_0$ 时, 满足条件的点的数量就是 $\varphi(y_0)$ , 当 $x>y_0$ 时, 将点 $(x, y_0)$ 以 $y=x$ 对称过去得到的点就是固定 $x=x_i$ 时 $y<x_i$ 的情况, 根据坐标轴关于 $y=x$ 对称的性质, $x<y_0$ 和 $x>y_0$ 两种情况下的答案数相等, 所以我们只需考虑 $x<y_0$ 的情况, 然后乘 2 即可.

最终的答案为: $3+2\cdot\sum_{i=2}^N \varphi(i)$

[「Luogu P2303」Longge 的问题](https://www.luogu.com.cn/problem/P2303)

考虑满足 $\gcd(i, n)=d$ 的 $i$ 的个数.

由 $\gcd(i, n)=d$ 可以推出 $\gcd(\frac{i}{d}, \frac{n}{d})=1$, 即 $\frac{i}{d}, \frac{n}{d}$ 互质, 又 $\frac{i}{d}<\frac{n}{d}$ , 所以这样的 $\frac{i}{d}$ 的个数为 $\varphi(\frac{n}{d})$ , 也就是 $i$ 的个数为 $\varphi(\frac{n}{d})$

所以答案为 $\sum_{d|n}d\varphi(\frac{n}{d})$

## 同余

### 剩余系

1. **剩余系**指模正整数 $n$ 的余数所组成的集合.
2. **完全剩余系**: 如果一个剩余系中包含了这个正整数 $n$ 所有可能的余数 ($0, 1, \cdots, n-1$) , 那么就称它为模 $n$ 的一个完全剩余系, 记作 $Z_n$ .
   **简化剩余系**:完全剩余系中与 $n$ 互质的数的集合, 记作 $Z_n^*$ .
3. **同余等价类**: $Z_n$ 里的每一个元素代表所有模 $n$ 意义下与它同余的整数, 我们把满足同余关系的所有整数看作一个同余等价类.
4. 在 $Z_n$ 中的四则运算结果全部都要在模 $n$ 意义下.

### 模运算

1. 如果 $a\equiv b\pmod{m}$ 且有 $c\equiv d\pmod{m}$ , 那么:

   $$a+c\equiv b+d\pmod{m}$$

   $$a-c\equiv b-d\pmod{m}$$

   $$a\times c\equiv b\times d\pmod{m}$$

2. $$(a+b)\bmod m=((a\bmod m)+(b\bmod m))\bmod m$$

   $$(a-b)\bmod m=((a\bmod m)-(b\bmod m)+m)\bmod m$$

   $$(a\times b)\bmod m=((a\bmod m)\times(b\bmod m))\bmod m$$

### 快速幂

右转 [快速幂复习笔记](../quickpow/)

### 费马小定理

>  若 $p$ 为素数, 且 $a\bot p$ , 则 $a^{p-1}\equiv 1\pmod{p}$ .

因为 $p$ 为素数, 则 $2, 3, \cdots, p-1$ 与 $p$ 互质, 且不存在两个数模 $p$ 同余.

又 $a\bot p$ , 所以 $a, 2a, 3a, \cdots, (p-1)a$ 与 $p$ 互质, 且不存在两个数模 $p$ 同余.

因此 $a\cdot 2a\cdot 3a\cdots(p-1)a=(p-1)!a^{p-1}$ 与 $p$ 互质, 且 $(p-1)!a^{p-1}\equiv (p-1)!\pmod{p}$

两边同时除以 $(p-1)!$ 得到 $a^{p-1}\equiv 1\pmod{p}$

1. 费马小定理的另一种形式: 若 $p$ 为素数, 则对任意正整数 $a$ 有 $a^p\equiv a\pmod{p}$.
2. 应用: $p$ 是素数, $a, p$ 互质, 则 $a^b\bmod{p}= a^{b\bmod (p-1)}\bmod{p}$ .

#### 费马小定理判断素数

1. 多次选取 $a$ 来判断 $p$ 是否满足费马小定理, 正确的概率随 $a$ 选取的次数的增加而增加.
2. 时间复杂度: 设选取 $k$ 次 $a$ , 每次用快速幂处理的时间为 $\mathcal{O}(\log{p})$ , 则总时间复杂度为 $\mathcal{O}(k\log{p})$ .
3. 由于存在 Carmichael 数, 因此上述算法存在缺陷, 即存在合数满足费马小定理.
4. $10^4$ 以内的 Carmichael 数有: 561, 1105, 1729, 2465, 2821, 6601, 8911.

## 欧拉定理

> (在 $p$ 不是素数的情况下, 可以使用欧拉定理)
>
> 对于和 $m$ 互素的 $a$ , 有: $a^{\varphi(m)}\equiv1\pmod{m}$ .

证明方法和费马小定理差不多.

引理: 若 $a, m$ 互质, 满足 $a^x\equiv 1\pmod{m}$ 的最小正整数 $x_0$ 是 $\varphi(m)$ 的约数.

## 欧拉定理的推论

> $a, m(m>1)$ 互素, 则有 $a^b\bmod{m}=a^{b\bmod\varphi(m)}\bmod m$ .

#### 例题

[「POJ 3696」The Luckiest number](http://poj.org/problem?id=3696)

设 $n$ 个 8 连在一起的正整数为 $A_n$ , 则 $A_n=10A_{n-1}+8$ , 容易求得 $A_n=\frac{8}{9}(10^n-1)$ .

若 $L|A_n$ , 即 $L|\frac{8}{9}(10^n-1)$, 可以得到 $9L|8(10^n-1)$ , 令 $d=\gcd(9L, 8)$ , 则有 $\frac{9L}{d}|(10^n-1)$

于是 $10^n\equiv 1\pmod{\frac{9L}{d}}$

根据欧拉定理的引理, 只需求出 $\varphi(\frac{9L}{d})$ , 然后枚举它的约数, 一一验证后取最小的一个数即可.

时间复杂度: $\mathcal{O}(\sqrt{L}\log{L})$

## 扩展欧几里得算法

### 裴蜀定理 ( Bézout 定理 )

> 对任何整数 $a, b$ , 关于未知数 $x, y$ 的线性不定方程 ( 称为裴蜀等式 ) $ax+by=c$ 有整数解当且仅当 $c$ 为 $\gcd{(a, b)}$ 的倍数.
>
> 裴蜀等式有解时必有无穷多个解.

推论: $a, b$ 互素等价于 $ax+by=1$ 有整数解.

#### 例题

[「BZOJ 1441」Min](http://www.lydsy.com/JudgeOnline/problem.php?id=1441)

假设 $S$ 已知, 于是有方程 $A_1x_1+A_2x_2+\cdots+A_nx_n=S$

根据裴蜀定理, $S$ 必然是 $\gcd{(A_1, A_2, \cdots, A_n)}$ 的倍数, 取 $S_{min}=\gcd{(A_1, A_2, \cdots, A_n)}$ 即可.

### 扩展欧几里得算法

见 [gcd & lcm 复习笔记](http://www.hlwer.com/index.php/archives/27/)

根据裴蜀定理, $ax+by=\gcd{(a, b)}$ 有无穷组解, 而扩展欧几里得算法求出来的只是其中一组特解 $(x_0, y_0)$ .

下面考虑求 $ax+by=c$ 的一般解.

首先设 $c=t\gcd(a, b)$ ( $t$ 为整数 ) , 则使用扩展欧几里得算法求出 $ax+by=\gcd{(a, b)}$ 的特解 $(x_0, y_0)$ 后, 容易得到 $ax+by=c$ 特解为 $(tx_0, ty_0)$ .

由于之后考虑的是 $ax+by=c$ , 所以后面用 $(x_0, y_0)$ 代替 $(tx_0, ty_0)$ .

将方程的所有解按照 $x$ 的大小排序, 则特解后的下一组解 $(x_1, y_1)$ 可以表示为 $(x_0+d_1, y_0+d_2)$ , 其中 $d_1$ 为满足条件的最小正整数.

带入方程后得到 $ad_1+bd_2=0$ , 变形后有 $-\dfrac{d_1}{d_2}=\dfrac{b}{a}=\dfrac{\frac{b}{\gcd{(a, b)}}}{\frac{a}{\gcd{(a, b)}}}$ .

因此方程的一般解可以表示为 $\left(x_0+k\dfrac{b}{\gcd{(a, b)}}, y_0-k\dfrac{a}{\gcd{(a, b)}}\right)(k\in \mathbf{Z})$ .

为什么这里要将 $\dfrac{b}{a}$ 再变形为 $\dfrac{\frac{b}{\gcd{(a, b)}}}{\frac{a}{\gcd{(a, b)}}}$ 呢?

因为 $\frac{b}{a}$ 不是最简分数, 这一步变形是化简 $\frac{b}{a}$ , 直接用 $(x_0+kb, y_0-ka)$ 来表示特解的话会漏掉一些情况, 如 $(x_0+\frac{b}{\gcd{(a, b)}}, y_0-\frac{a}{\gcd{(a, b)}})$ 就不能被 $(x_0+kb, y_0-ka)$ 表示.

## 逆元

### 模 $n$ 意义下乘法的逆

如果在 $Z_n$ 中两元素 $a, b$ 满足 $a\cdot b=1$ , 那么我们就说 $a, b$ 互为模 $n$ 意义下乘法的逆元, 记作 $a=b^{-1}, b=a^{-1}$ .

### 逆元

当 $a, m$ 互素的时候, 若 $ax\equiv 1\pmod{m}$ , 则称 $x$ 为 $a$ 关于模 $m$ 的逆元, 记作 $a^{-1}$ .

在 $[0, m)$ 的范围内, 逆元是唯一的.

### 求解逆元

求解逆元等价于解方程 $ax+my=1$

直接使用扩展欧几里得算法即可.

#### 欧拉定理求逆元

因为 $a, m$ 互素, 根据欧拉定理, 有 $a\cdot a^{\varphi(m)-1}\equiv 1\pmod{m}$ , 因此 $a^{\varphi(m)-1}$ 为所求逆元.

#### 线性求逆元: 递推法

递推法求解 $1\sim n$ 的逆元.

假设当前求解的是 $i$ 关于模 $p$ 的逆元.

根据带余除法, $p=iq+r$ , 两边分别对 $p$ 取模得到 $iq+r\equiv 0\pmod{p}$ 

若 $i$ 不为 $p$ 的倍数, 则 $r$ 不为 0 , 因此 $r^{-1}$ 存在.

在上式两边乘 $i^{-1}r^{-1}$ 得到 $qr^{-1}+i^{-1}\equiv 0\pmod{p}$ , 即

$$r^{-1}\equiv -qr^{-1}=-r^{-1}\left\lfloor \dfrac{p}{i}\right\rfloor\pmod{p}$$

为了防止答案为负数, 使用 $r^{-1}(p-\left\lfloor \frac{p}{i}\right\rfloor)$ 代替 $-r^{-1}\left\lfloor \frac{p}{i}\right\rfloor$ .

时间复杂度: $\mathcal{O}(n)$

```cpp
inv[1] = 1;
for (int i = 2; i <= n; ++i)
  inv[i] = inv[p % i] * (p - p / i) % p;
```

#### 线性求逆元: 倒推法

先直接求解 $n!$ 的逆元, 然后利用 $((n-1)!)^{-1}\equiv(n!)^{-1}\cdot n\pmod{p}$ 倒推求得 $1!\sim (n-1)!$ 的逆元.

接着再利用 $n^{-1}\equiv (n!)^{-1}\cdot (n-1)!\pmod{p}$ 求解 $1\sim n$ 的逆元.

### 例题

[「POJ 1845」Sumdiv](http://poj.org/problem?id=1845)

首先利用整数惟一分解定理将 $A$ 表示为 $A=p_1^{r_1}p_2^{r_2}\cdots p_n^{r_n}$ .

则 $A^B$ 约数和为

$$(1+p_1+p_1^2+\cdots +p_1^{B\cdot r_1})(1+p_2+p_2^2+\cdots+p_2^{B\cdot r_2})\cdots(1+p_n+p_n^2+\cdots+p_n^{B\cdot r_n})$$

可以发现, 每一个括号里都是一个等比数列求和, 以 $p_i$ 为例, 它的和可以写成 $\dfrac{p_i^{B\cdot r_i+1}-1}{p_i-1}$ , 然后我们可以利用快速幂求解分子 $p_i^{B\cdot r_i+1}\bmod{9901}$ , 这时若 $p_i-1>(p_i^{B\cdot r_i+1}-1)\bmod{9901}$ , 我们会得到 $0$ , 这显然是错的, 而模数 $9901$ 正好为素数, 因此考虑用 $p_i-1$ 的逆元与分子的结果相乘来代替除法.

当 $p_i-1$ 为 $9901$ 的倍数时不存在逆元, 这时我们可以发现 $p_i\equiv 1\pmod{9901}$ , 因此 $1, p_i, p_i^2, \cdots, p_i^{B\cdot r_i+1}\equiv 1\pmod{9901}$ , 所以 $1+p_i+p_i^2+\cdots+p_i^{B\cdot r_i}\equiv B\cdot r_i+1\pmod{9901}$ .

## 线性同余方程

形如 $ax\equiv c\pmod{m}$ 的方程，称为线性同余方程。

1. 朴素算法
   将 $x=1, 2, \cdots, m-1$ 一次带进去验证即可，这个朴素算法的复杂度取决于 $m$ 。
2. 扩展欧几里得算法
   可以将方程看作 $ax+my=c$ ，然后利用扩展欧几里得方法求解。
   根据裴蜀定理，当且仅当 $\gcd{(a, m)}|c$ 时方程有解。
   令 $d=\gcd(a, m)$ ，则方程一共有 $d$ 个解，我们用扩展欧几里得算法求得特解 $x_0$ ，则方程的通解 $x_i=x_0+i\cdot \dfrac{m}{d}(i=0, 1, \cdots, d-1)$ 。
3. 欧拉定理
   令 $d=\gcd{(a, m)}$ ，若 $d \nmid c$ 则方程无解。
   然后令 $a'=\dfrac{a}{d}, c'=\dfrac{c}{d}, m'=\dfrac{m}{d}$ ，则 $\gcd{(a', m')}=1$ ，原方程变为 $a'x\equiv c'\pmod{m'}$ ，由欧拉定理，$a'^{\varphi(m')}\equiv 1\pmod{m'}$ ，在方程两边分别乘上上式，得到 $a'x\equiv c'a'^{\varphi(m')}\pmod{m'}$ ，即 $x\equiv c'a'^{\varphi(m')-1}\pmod{m'}$ ，于是我们得到原方程的特解 $x_0=dc'a'^{\varphi(m')-1}$ ，因此原方程的解为 $x_i=x_0+i\cdot m'(i=1, 2, \cdots,d-1)$ 。

#### 例题

**线性组合**

> 对应整数数列 $A_1, A_2, \cdots, A_n$ 是否存在 $X_1, X_2, \cdots, X_n$ ， 使得 $A_1X_1+A_2X_2+\cdots+A_nX_n=C$ ，其中 $\gcd{(A_1, A_2, \cdots, A_n)}\mid C(n\ge2)$ 。如有，请找出一组整数解 $(x_1, x_2, x_3, x_4)$ 满足 $12x_1+24x_2+18x_3+15x_4=3$ .

题中的方程是 n 元形式的扩展欧几里得方程，我们尝试将 n 元逐步降为二元得到最终解。

设 $d=\gcd{(A_1, A_2, \cdots, A_n)}$ ，令 $C'=\dfrac{C}{d},A_i'=\dfrac{A_i}{d}(i=1, 2, \cdots, n)$ ，首先将方程变形为 $A_1'X_1+A_2'X_2+\cdots+A_n'X_n=C'$ ，此时满足 $\gcd{(A_1', A_2', \cdots, A_n', C')}=1$ 。

然后我们考虑求解方程 $A_1'X_1+\gcd(A_2', A_3', \cdots, A_n')Y_1=C'$ ，利用扩展欧几里得方法求得特解 $X_1,Y_1$ ，由于题中仅要求得到一组解，因此我们只取特解即可，然后再以同样的方法求解 $A_2'X_2+A_3'X_3+\cdots+A_n'X_n=\gcd(A_2', A_3', \cdots, A_n')Y_1$ 便可以实现逐渐降元了。

### 线性同余方程组（模互素）

考虑形如 $x\equiv a_i\pmod{m_i}$ 的若干个线性同余方程联合得到的方程组，例如
$$
\begin{cases}
x &\equiv& 2\pmod{3} \\
x &\equiv& 3\pmod{5} \\
x &\equiv& 5\pmod{7}
\end{cases}
$$
一种可行的解法是令 $x=3y+2$ ，然后将方程 $(2)$ 改写成 $y\equiv 2\pmod{5}$ ，接着再令 $y=5z+2$ 代入方程 $(3)$ 得到 $z\equiv 4\pmod{7}$ ，再令 $z=7k+4$ ，最后我们可以得到解 $x\equiv68\pmod{105}$ 。

#### 中国剩余定理

>  若线性同余方程组 $x\equiv a_i\pmod{m_i}(i=1, \cdots, n)$ 的模数 $m_i$ 两两互素，则 $x$ 在 $\bmod{M}(M=\prod_{i=1}^n m_i)$ 下有唯一解。

令 $M_i=\dfrac{M}{m_i}$ ，有 $\gcd{(M_i, m_i)}=1$ ，因此 $M_i$ 在 $\bmod{m_i}$ 下的逆元存在，设其为 $t_i$ ，于是有 $a_iM_it_i\equiv a_i\pmod{m_i},a_iM_it_i\equiv0\pmod{m_j}(j\not= i)$ 。

可以得到解 $x\equiv\sum_{i=1}^na_iM_it_i\pmod{M}$ 。 

```cpp
//China Remainer Theorem
int CRT(const int a[], const int m[], int n) {
  int M = 1, ret = 0;
  for (int i = 1; i <= n; ++i) M *= m[i];
  for (int i = 1; i <= n; ++i) {
    int ti = getinv(m[i], a[i]), Mi = M / m[i];
    ret = (ret + a[i] * ti * Mi % M) % M;
  }
  return ret;
}

//利用 exgcd 求逆元
int getinv(int a, int b) {
  int x, y;
  exgcd(a, b, x, y);
  return x;
}
```

### 线性同余方程组（模不互素）

对于模不互素的线性同余方程组
$$
\begin{cases}
x &\equiv a_1\pmod{m_1} \\
x &\equiv a_2\pmod{m_2} \\
&\cdots \\
x &\equiv a_n\pmod{m_n}
\end{cases}
$$

考虑求解方程数量为 2 的情形，当方程数量大于 2 时迭代求解。

$$
\begin{cases}
x &\equiv a_1\pmod{m_1} \\
x &\equiv a_2\pmod{m_2}
\end{cases}
$$
设 $y=x-a_1$ ，则方程组变为
$$
\begin{cases}
y &\equiv 0 &\pmod{m_1} \\
y &\equiv a_2-a_1 &\pmod{m_2}
\end{cases}
$$
设 $d=\gcd{(m_1, m_2)}$ ，根据裴蜀定理，若 $d \nmid (a_2-a_1)$ ，则方程组无解，否则令 $y'=\dfrac{y}{d},a'=\dfrac{a_2-a_1}{d}, m_i'=\dfrac{m_1}{d}(i=1,2)$ ，有
$$
\begin{cases}
y' &\equiv{0} &\pmod{m_1'} \\
y' &\equiv{a'} &\pmod{m_2'}
\end{cases}
$$
此时 $\gcd{(m_1', m_2')}=1$ ，令 $y'=km_1'$ ，再由欧拉定理可得 $k\equiv{a'm_1'^{\varphi(m_2')-1}}\pmod{m_2'}$ ，于是 $y\equiv{da'm_1'^{\varphi{(m_2')}}}\pmod{dm_1'm_2'}$ ，最后代回去可以得到 $x\equiv{(a_2-a_1)m_1'^{\varphi{(m_2')}}+a_1}\pmod{dm_1'm_2'}$ 。

#### 例题

[「POJ 2891」Strange Way to Express Integers](http://poj.org/problem?id=2891)

算法和上述一致。

假设我们已经通过迭代求解得到前 $k-1$ 个方程的一个解 $x_0$ ，令 $M=\prod_{i=1}^{k-1}m_i$ ，则 $x=x_0+i\cdot M$ 是前 $k-1$ 个方程的通解。

考虑第 $k$ 个方程，我们可以将其改写为 $x_0+t\cdot M\equiv{a_k}\pmod{m_k}$ ，即 $t\cdot M\equiv{a_k-x_0}\pmod{m_k}$ ，接下来找到一个合适的 $t$ 即可，也就是求解一个同余方程了，利用欧拉定理求解即可。

## 高次同余方程

### 第一类高次同余方程

形如 $a^x\equiv{b}\pmod{p}$ 的方程称为第一类高次同余方程，其中 $\gcd(a, p)=1$ 。

#### BSGS

解决这一类方程主要采用 BSGS (Baby Step Giant Step) 算法。

将 $x$ 写成 $x=kt-m(0\le m\le t-1)$ 的形式，则原方程可变形为 $(a^t)^k\equiv{b\cdot a^m\pmod{p}}$ 。

假设 $t$ 已知，那么我们只需枚举 $k$ ，然后计算相应的 $b\cdot a^m$ ，判断是否满足变形后的等式即可。

因为在枚举 $k$ 的过程中 $m$ 的取值也在变化，但 $m$ 的取值范围是不变的，因此我们可以预处理所有可能的 $b\cdot a^m$ 值，用一个 Hash 表储存。

由欧拉定理可知，有 $a^{\varphi(p)}\equiv{1}\pmod{p}$ ，而 $\varphi(p)<p$ ，因此从 1~p ，模 $p$ 意义下 $a$ 的幂至少完成了一次循环，所以最小整数解 $x<p$ ，进而我们可以从 $0$ 到 $\dfrac{p}{t}$ 枚举 $k$ ，而 $m\in[0,t-1]$ ，因此时间复杂度为 $\mathcal{O}(\dfrac{p}{t}+t)$ ，当 $t=\sqrt{p}$ 时复杂度最小。

时间复杂度：$\mathcal{O}(2\sqrt{p})$

```cpp
int BSGS(int a, int b, int p) {
  b %= p;
  if (a % p == 0) return b ? 1 : -1;
  map <int, int> hash;
  int t = sqrt(p) + 1;
  for (int i = 0; i < t; ++i) {
    int val = (long long)b * power(a, i, p) % p;//快速幂
    hash[val] = i;
  }
  a = power(a, t, p);
  for (int i = 0; i <= t; ++i) {
    int val = power(a, i, p);
    int j = hash.find(val) == hash.end() ? -1 : hash[val];
    if (j >= 0 && i * t - j >= 0) return i * t - j;
  }
  return -1;
}
```

### 第二类高次同余方程

形如 $x^k\equiv{b}\pmod{p}$ 的方程称为第二类高次同余方程。

首先引入**阶**和**原根**的概念，这一部分和抽象代数相关，我目前还不会 QAQ 。

### 阶

满足同余式 $a^n\equiv{1}\pmod{p}$ 的最小整数 $n$ 称为 $a$ 模 $p$ 的阶，记作 $\delta_p(a)$ 。

由欧拉定理可知，当 $\gcd(a, p)=1$ 时，有 $a^{\varphi(p)}\equiv{1}\pmod{p}$ ，因此阶的存在性得证。

**性质 1** ：$a,a^2,\cdots,a^{\delta_p(a)}$ 模 $m$ 两两不同余。

**性质 2** ：若 $a^n\equiv{1}\pmod{p}$ ，则 $n \mid \delta_p(a)$ 。

​	**推论** ：若 $a^m\equiv{a^n}\pmod{p}$ ，则 $m\equiv{n}\pmod{\delta_p(a)}$ 。

**性质 3** ：设 $p\in\mathbb{N}^*,\, a,b\in\mathbb{Z},\gcd(a,p)=\gcd(b,p)=1$ ，则 $\delta_p(ab)=\delta_p(a)\delta_p(b)$ 的充要条件是 $\gcd(\delta_p(a),\delta_p(b))=1$ 。

**性质 4** ：设 $k\in\mathbb{N},p\in\mathbb{N}^*,a\in\mathbb{Z},\gcd(a,p)=1$ ，则 $\delta_p(a^k)=\dfrac{\delta_p(a)}{\gcd(\delta_p(a),k)}$ 。

### 原根

设 $p\in\mathbb{N}^*,a\in\mathbb{Z}$ ，若 $\gcd(a,p)=1$ ，且 $\delta_p(a)=\varphi(p)$ ，则称 $a$ 为模 $m$ 的原根。

以下基于 $p$ 为质数的情况进行讨论。

如何判断 $a$ 是否是 $p$ 的原根？

由费马小定理，方程 $a^x\equiv{1}\pmod{p}$ 的一个解为 $x=p-1$ ，因此最小整数解 $x_{min}\mid (p-1)$ ，而满足 $x_{min}=p-1$ 的整数 $a$ 是 $p$ 的原根，所以我们逐个枚举 $p-1$ 的约数 $k$ ，如果只有当 $k=p-1$ 时满足 $a^k\equiv{1}\pmod{p}$ ，则 $a$ 是 $p$ 的原根。

下面考虑求解 $x^k\equiv{b}\pmod{p}$ 。

由**阶的性质 1** ，存在唯一的正整数 $m$ ，满足 $a^m\equiv{b}\pmod{p},0\le m\le p-1$ ，其中 $a$ 是 $p$ 的原根，$m$ 可以由第一类高次同余方程求出。

同样，存在唯一正整数 $y$ ，使得 $x\equiv{a^y}\pmod{p}$ ，于是上述方程变为 $a^{ky}\equiv{a^m}\pmod{p}$ 。

由**阶的性质 2** 的**推论**，$ky\equiv{m}\pmod{p-1}$ ，于是我们求解这个线性同余方程得到 $y$ 便可以得到 $x$ 。
