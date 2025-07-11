---
title: "谢惠民做题记录"
published: 2022-10-03
description: '记录刷谢惠民时的一些好题'
image: ''
tags: [Analysis]
category: 'Mathematics'
draft: false 
lang: ''
---

:::note[INFO]
本文写于 2022 年大一上学期，谢惠民上册的课后练习题大部分都写了，但是很可惜，由于时间原因没能把我认为的一些好题摘录完。
:::

## 几个常用初等不等式

1. **关于 Bernoulli 不等式的推广**

   > （3）证明：若 $a_i>-1(i=1,2,\cdots,n)$ 且同号，则成立不等式
   > $$\prod_{i=1}^n(1+a_i)\ge1+\sum_{i=1}^na_i$$

   $\color{red}{\textbf{Proof}}$

   构造辅助数列 $\{x_n\}$ 使 $x_n=\prod_{i=1}^n(1+a_i)-(1+\sum_{i=1}^na_i)$

   显然有 $x_1=0$ ，只需证明 $\{x_n\}$ 为单调递增数列即可.

   $$x_{n+1}-x_n=[\prod_{i=1}^n(1+a_i)-1]a_{n+1}\ge0 \Leftrightarrow x_{n+1}\ge x_n$$

   则有 $x_n\ge x_1=0$ ，即 $\prod_{i=1}^n(1+a_i)\ge1+\sum_{i=1}^na_i$

   $\color{blue}{\textbf{Thoughts}}$

   在存在多项求和的情况下构造辅助数列可以消除绝大多数项，从而化繁为简，同时也能够利用数列的单调性.

2. **阶乘 $n!$ 在数学分析以及其他课程中经常出现，以下是几个有关不等式，它们都可以从平均值不等式得到**

   > （2）利用 $(n!)^2=(1\cdot n)[2\cdot (n-1)]\cdots(n\cdot 1)$ 证明：当 $n>1$ 时成立
   > $$n!<\left(\frac{n+2}{\sqrt{6}} \right)^n $$

   $\color{red}{\textbf{Proof}}$

   首先根据提示和不等式中的 $\sqrt{6}$ 都应该想到将待证不等式平方，有

   $$(n!)^2<\left(\frac{n^2+4n+4}{6} \right)^n$$

   然后对它使用放缩魔法.由于将 $(1\cdot n)[2\cdot (n-1)]\cdots(n\cdot 1)$ 一项一项地放缩会有较大的误差，所以这里直接将这 n 项一起放缩，得到

   $$(n!)^2=(1\cdot n)[2\cdot (n-1)]\cdots(n\cdot 1)\le \left(\frac{(1\cdot n)+[2\cdot (n-1)]+\cdots+(n\cdot 1)}{n}\right)^n$$

   下面构造辅助数列求解分子的表达式.

   设 $x_n=(1\cdot n)+[2\cdot (n-1)]+\cdots+(n\cdot 1)$ ，则 $x_1=n$ ，作差得

   $$x_{n+1}-x_n=1+2+\cdots+n+n+1=\frac{n^2+3n+2}{2}$$

   累加后得到

   $$x_{n+1}-x_1=\frac{n^3+6n^2+11n}{6} $$

   于是 $x_n=\frac{n^3+3n^2+8n-12}{6}$ ，代入上面的式子有

   $$(n!)^2\le \left(\frac{(1\cdot n)+[2\cdot (n-1)]+\cdots+(n\cdot 1)}{n}\right)^n=\left(\frac{n^3+3n^2+8n-12}{6n}\right)^n $$

   因为 $\frac{n^3+3n^2+8n-12}{n}-(n^2+4n+4)=4-(n+\frac{12}{n})<4-4\sqrt{3}<0$ ，所以 $n!<\left(\frac{n+2}{\sqrt{6}} \right)^n$ 成立.

   $\color{blue}{\textbf{Thoughts}}$

   这里为什么要将 $n!$ 写成倒序的乘积的形式呢？假如我们将其写成正序的乘积形式，会得到 $(n!)^2\le \left(\frac{1^2+2^2+\cdots+n^2}{n}\right)^n$ ，带入平方和公式有 $\left(\frac{1^2+2^2+\cdots+n^2}{n}\right)^n=\left[\frac{(n+1)(2n+1)}{6}\right]^n$ ，然后我们会发现 $(n+1)(2n+1)\ge(n+2)^2$ ，这不就放多了嘛.

   于是我们想到排序不等式，逆序和 $\le$ 正序和，为了使放缩的精度尽量大，所以我们将 $(n!)^2$ 写成逆序的乘积的形式，从而得到一个比 $(n+2)^2$ 小的式子.

3. **证明下列不等式**

   > (3)
   > $$\frac{|a+b|}{1+|a+b|}\le\frac{|a|}{1+|a|}+\frac{|b|}{1+|b|}$$

   $\color{red}{\textbf{Proof}}$

   首先左边的取的是 $a+b$ 的绝对值，而右边则是分别取 $a$ 和 $b$ 的绝对值，因此应该要用到三点不等式 $|a+b|\le |a|+|b|$ ，但是问题来了，左边的分子和分母都有 $|a+b|$ 该咋办捏？

   这里我们将分子分母同时除以 $|a+b|$ 得到 $\frac{1}{\frac{1}{|a+b|}+1}$，然后再带入三点不等式得到

   $$\frac{|a+b|}{1+|a+b|}=\frac{1}{\frac{1}{|a+b|}+1}\le\frac{1}{\frac{1}{|a|+|b|}+1}=\frac{|a|+|b|}{|a|+|b|+1}$$

   然后将右边的式子通分

   $$\frac{|a|}{1+|a|}+\frac{|b|}{1+|b|}=\frac{|a|+|b|+2|ab|}{|a|+|b|+|ab|+1}$$

   这时，我们发现两个式子长得十分相似，分母相差一个 $|ab|$ ，分子相差 2 个 $|ab|$ ，而左式放缩后的式子是真分数，于是利用糖水不等式放缩得到

   $$\frac{|a|+|b|}{|a|+|b|+1}\le\frac{|a|+|b|+|ab|}{|a|+|b|+|ab|+1}\le\frac{|a|+|b|+2|ab|}{|a|+|b|+|ab|+1}$$

   综上所述，

   $$\frac{|a+b|}{1+|a+b|}\le\frac{|a|+|b|}{|a|+|b|+1}\le\frac{|a|+|b|+2|ab|}{|a|+|b|+|ab|+1}=\frac{|a|}{1+|a|}+\frac{|b|}{1+|b|}$$

4. **试按下列提示，给出 Cauchy 不等式的几个不同证明**

   > (3) 用不等式 $|AB|\le \frac{A^2+B^2}{2}$

   $\color{red}{\textbf{Proof}}$

   先列出 Cauchy 不等式好了.

   $$\sum_{i=1}^n|a_ib_i|\le\sqrt{\sum_{i=1}^na_i^2}\sqrt{\sum_{i=1}^nb_i^2}\left(当且仅当\frac{a_1}{b_1}=\frac{a_2}{b_2}=\cdots=\frac{a_n}{b_n}时取等\right)$$

   当 $\sum_{i=1}^na_i=0$ 或 $\sum_{i=1}^nb_i=0$ 时，不等式显然成立.

   考虑他们不为 0 的情况，不等式即

   $$\frac{\sum_{i=1}^n|a_ib_i|}{\sqrt{\sum_{i=1}^na_i^2}\sqrt{\sum_{i=1}^nb_i^2}}\le 1$$

   对于 $i=k$ ，有

   $$\frac{|a_kb_k|}{\sqrt{\sum_{i=1}^na_i^2}\sqrt{\sum_{i=1}^nb_i^2}}=\frac{|a_k|}{\sqrt{\sum_{i=1}^na_i^2}}\cdot\frac{|b_k|}{\sqrt{\sum_{i=1}^nb_i^2}}\le \frac{1}{2}\cdot\left(\frac{a_k^2}{\sum_{i=1}^na_i^2}+\frac{b_k^2}{\sum_{i=1}^nb_i^2}\right)$$

   那么对于 $i=1,2,\cdots,n$ ，求和有

   $$\frac{\sum_{i=1}^n|a_ib_i|}{\sqrt{\sum_{i=1}^na_i^2}\sqrt{\sum_{i=1}^nb_i^2}}\le\frac{1}{2}\cdot(1+1)=1$$

   不等式得证.

   $\color{blue}{\textbf{Thoughts}}$

   均值不等式的一个好处就是把乘积变成加法的形式，在这个不等式中，如果我们直接对 $a_ib_i$ 使用均值不等式将会得到 $\sum_{i=1}^n|a_ib_i|\le\frac{\sum_{i=1}^na_i^2+\sum_{i=1}^nb_i^2}{2}$ ，然后我们想要证明 $\frac{\sum_{i=1}^na_i^2+\sum_{i=1}^nb_i^2}{2}\le\sqrt{\sum_{i=1}^na_i^2\cdot\sum_{i=1}^nb_i^2}$ ，左边是加法，右边则是根号下乘积的形式，显然是不好处理的，所以我们想啊，要是右边的也是加法的形式就好了，因此我们想对 $\sum_{i=1}^na_i^2\cdot\sum_{i=1}^nb_i^2$ 也使用均值不等式，一个办法就是在一开始把他们移到不等式左边的分母上，然后拆成 $\frac{|a_k|}{\sqrt{\sum_{i=1}^na_i^2}}\cdot\frac{|b_k|}{\sqrt{\sum_{i=1}^nb_i^2}}$ 的形式，再使用均值不等式，会发现求和后的结果刚好就是 1 .

5. **用向前 - 向后数学归纳法证明 Fan Ky 不等式**

   > 设 $0<x_i\le\frac{1}{2},i=1,2,\cdots,n$ ，则
   > $$\frac{\prod_{i=1}^nx_i}{\left(\sum_{i=1}^nx_i\right)^n}\le\frac{\prod_{i=1}^n(1-x_i)}{\left[\sum_{i=1}^n(1-x_i)\right]^n}$$

   $\color{red}{\textbf{Proof}}$

   不等式即

   $$\frac{\prod_{i=1}^nx_i}{\prod_{i=1}^n(1-x_i)}\le\left[\frac{\sum_{i=1}^nx_i}{\sum_{i=1}^n(1-x_i)}\right]^n$$

   考虑 $n=2^k(k\in \mathbb{N_+})$ 的情形.

   当 $k=1$ 时，不等式为 $\frac{x_1x_2}{(1-x_1)(1-x_2)}\le\frac{(x_1+x_2)^2}{[(1-x_2)+(1-x_2)]^2}$

   这里均值不等式不好使，而参数只有两项，因此使用直接作差法，得到

   $$\frac{x_1x_2}{(1-x_1)(1-x_2)}-\frac{(x_1+x_2)^2}{[(1-x_2)+(1-x_2)]^2}=\frac{(x_1-x_2)^2(x_1+x_2-1)}{(1-x_2)(1-x_2)(2-x_1-x_2)^2}\le 0$$

   得证.

   当 $k=2$ 时，

   $$\begin{aligned} 左边 =& \frac{x_1x_2}{(1-x_1)(1-x_2)}\cdot\frac{x_3x_4}{(1-x_3)(1-x_4)} \\ \le& \left(\frac{x_1+x_2}{1-x_1+1-x_2}\cdot\frac{x_3+x_4}{1-x_3+1-x_4}\right)^2 \\ \le& \left[\frac{x_1+x_2+x_3+x_4}{(1-x_1)+(1-x_2)+(1-x_3)+(1-x_4)}\right]^4 \end{aligned}$$

   以此类推，可以证明当 $n=2^k$ 时不等式成立.

   下面进行反向归纳：

   假设不等式在 $n$ 时已然成立，现在我们要推出它在 $n-1$ 时也成立.

   令 $\frac{\sum_{i=1}^{n-1}x_i}{n-1}=A$ ，则 $\frac{\sum_{i=1}^{n-1}(1-x_i)}{n-1}=1-A$ ，于是 $\left[\frac{\sum_{i=1}^{n-1}x_i}{\sum_{i=1}^{n-1}(1-x_i)}\right]^{n-1}=\left(\frac{A}{1-A}\right)^{n-1}$ .

   待证不等式即

   $$\left(\frac{A}{1-A}\right)^{n-1}\ge\frac{\prod_{i=1}^{n-1}x_i}{\prod_{i=1}^{n-1}(1-x_i)}=\frac{\prod_{i=1}^{n}x_i}{\prod_{i=1}^{n}(1-x_i)}\cdot\frac{1-x_n}{x_n}$$

   等价于

   $$\left(\frac{A}{1-A}\right)^{n-1}\cdot\frac{x_n}{1-x_n}\ge\frac{\prod_{i=1}^{n}x_i}{\prod_{i=1}^{n}(1-x_i)}$$

   由于不等式在 $n$ 时成立，显然有

   $$\left(\frac{A}{1-A}\right)^n\ge\frac{\prod_{i=1}^{n}x_i}{\prod_{i=1}^{n}(1-x_i)}$$

   将 $\frac{x_n}{1-x_n}$ 看成 $\frac{A}{1-A}$ ，因而不等式在 $n-1$ 时也成立.

   从而 Fan Ky 不等式得证.

   $\color{blue}{\textbf{Thoughts}}$

   显然我们如果从待证不等式（即 n - 1 的情况）出发，向已知不等式（即 n 的情况）转化，会多出来一个 $\frac{1-x_n}{x_n}$ ，这是很自然的思路.

   这道题的关键在于发现 $\sum_{i=1}^{n-1}x_i$ 与 $\sum_{i=1}^{n-1}(1-x_i)$ 在均值形式下的高度相似性，将它们相除后的式子又与 $\frac{1-x_n}{x_n}$ 高度一致.

## 2.1 数列极限的基本概念

1. **例题 2.1.1**

   > 证明数列 $\{\frac{n^5}{2^n}\}$ 收敛于 $0$ .

   $\color{red}{\textbf{Proof}}$

   这里用定义法.

   使用 $2^n=(1+1)^n=1+\binom{n}{1}+\cdots+\binom{n}{n}$ 来进行适当放大.

   $$\frac{n^5}{2^n}<\frac{n^5}{\binom{n}{6}}=\frac{n^4\cdot 6!}{(n-1)(n-2)(n-3)(n-4)(n-5)}<\frac{6!}{n}<\epsilon$$

   只需取 $N=\left\lfloor\frac{720}{\epsilon}\right\rfloor$ 即可.

   $\color{blue}{\textbf{Thoughts}}$

   主要是要注意 $2^n=\sum_{i=0}^n\binom{n}{i}$ 的变形技巧.

2. **例题 2.1.2**

   > 证明数列 $\{\sqrt[n]{n}\}$ 的极限是 $1$ .

   $\color{red}{\textbf{Proof \  1}}$

   在 $n\ge2$ 时，根据均值不等式有

   $$1\le\sqrt[n]{n}=\sqrt[n]{\sqrt{n}\cdot\sqrt{n}\cdot\underbrace{1\cdot1\cdots\cdot1}_{n-2个1}}\le\frac{2\sqrt{n}+n-2}{n}\le1+\frac{2}{\sqrt{n}}$$

   因此有 $0\le\sqrt[n]{n}-1\le\frac{2}{\sqrt{n}}$

   取 $N=\max\{2,\lceil\frac{4}{\epsilon^2}\rceil\}$ 即可.

   $\color{blue}{\textbf{Thoughts}}$

   这个地方利用了 1 的任意次方等于 1 的性质和一个数乘以 1 还是它本身的性质来干掉指数的 $\frac{1}{n}$ .

   $\color{red}{\textbf{Proof \  2}}$

   构造辅助数列.

   由于 $\sqrt[n]{n}\ge1$ ，只需关心 $\sqrt[n]{n}-1\le\epsilon$ .

   令 $y_n=\sqrt[n]{n}-1$ ，则 $y_n\ge0$ ，且当 $n>1$ 时有

   $$n=(1+y_n)^n\ge \frac{n(n-1)}{2}y_n^2$$

   于是有 $y_n\le\sqrt{\frac{2}{n-1}}<\epsilon$ ，只需取 $N=[\frac{2}{\epsilon^2}]+1$ 即可.

   $\color{blue}{\textbf{Thoughts}}$

   将待证不等式直接构造为辅助数列可以更方便地放缩从而找出一个合适的 N .

3. **2.1.5 练习题 5**

   > 设 $a>1$ ，证明 $\lim\limits_{n\to \infty}\frac{\log_{a}n}{n}=0$ .
   >
   > （可以利用已知极限 $\lim\limits_{n\to\infty}\sqrt[n]{n}=1$ ）

   $\color{red}{\textbf{Proof}}$

   因为 $\frac{\log_an}{n}=\log_a\sqrt[n]{n}$ ，所以 $\lim\limits_{n\to\infty}\frac{\log_an}{n}=\lim\limits_{n\to\infty}\log_{a}\sqrt[n]{n}=0$

## 2.2 收敛数列的基本性质

1. **例题 2.2.2**

   > 若 $\lim\limits_{n\to\infty}\frac{x_n-a}{x_n+a}=0$ ，证明 $\lim\limits_{n\to\infty}x_n=a$ .

   $\color{red}{\textbf{Proof \  1}}$

   定义法.

   由题设， $a\not ={0}$ ，且 $\forall\epsilon>0,\exist N,s.t \  n>N,\left|\frac{x_n-a}{x_n+a}\right|<\epsilon$

   于是有

   $$|x_n-a|<\epsilon|x_n+a|=\epsilon|x_n-a+2a|\le\epsilon(|x_n-a|+2|a|)$$

   即

   $$(1-\epsilon)|x_n-a|<2\epsilon|a|$$

   限定 $\epsilon<1$ 得到 $|x_n-a|<\frac{2\epsilon|a|}{1-\epsilon}$ ，再限定 $\epsilon<\frac{1}{2}$ ，可以得到 $|x_n-a|<4|a|\epsilon$ ，满足 $\lim\limits_{n\to\infty}x_n=a$ 的定义，得证.

   $\color{blue}{\textbf{Thoughts}}$

   这里利用绝对值不等式的时候不能直接放缩，而是将内部凭空捏造出一个待证式来，然后将多出来的东西减去之后再进行放缩.

   $\color{red}{\textbf{Proof \  2}}$

   构造辅助数列.

   令 $y_n=\left|\frac{x_n-a}{x_n+a}\right|$ ，则 $y_n$ 极限已知，为 0 .

   将 $x_n$ 用 $y_n$ 表示，得到

   $$x_n=a\cdot\frac{1+y_n}{1-y_n}$$

   再由极限的四则运算法则，可以得到 $\lim\limits_{n\to\infty}x_n=a$ .

   $\color{blue}{\textbf{Thoughts}}$

   直接将已知式构造辅助数列，可以利用已知极限直接计算出要求的极限.

2. **2.2.4 练习题 2.(4)**

   > 设 $\{ a_n \}$ 为正数列，并且已知它收敛于 $a>0$ ，证明 $\lim\limits_{n\to\infty}\sqrt[n]{a_n}=1$ .

   $\color{red}{\textbf{Proof}}$

   由 $a_n\rightarrow a$ 可知，$\forall \epsilon>0$ , $\exists N$ ,当 $n>N$ 时, 满足 $a-\epsilon<a_n<a+\epsilon$ .

   令 $\epsilon<\frac{a}{2}$ ，则 $\frac{a}{2}<a_n<\frac{3}{2}a$ ，因为 $\lim\limits_{n\to\infty}\sqrt[n]{\frac{a}{2}}=\lim\limits_{n\to\infty}\sqrt[n]{\frac{3}{2}a}$ ，所以 $\lim\limits_{n\to\infty}\sqrt[n]{a_n}=1$ .

3. **2.2.4 练习题 3.(1)**

   > 求 $\lim\limits_{n\to\infty}(1+x)(1+x^2)\cdots(1+x^{2^n})$ ，其中 $|x|<1$ .

   $\color{red}{\textbf{Proof}}$

   $$(1+x)(1+x^2)\cdots(1+x^{2^n})=\frac{(1-x)(1+x)\cdots(1+x^{2^n})}{1-x}=\frac{1-x^{2^{n+1}}}{1-x}$$

   则 $\lim\limits_{n\to\infty}(1+x)(1+x^2)\cdots(1+x^{2^n})=\lim\limits_{n\to\infty}\frac{1-x^{2^{n+1}}}{1-x}=\frac{1}{1-x}$

4. **2.2.4 练习题 3.(5)**

   > $\lim\limits_{n\to\infty}\sum_{k=1}^n\frac{1}{k(k+1)\cdots(k+\nu)}$（其中 $\nu$ 为正整数）.

   $\color{red}{\textbf{Proof}}$

   注意到

   $$\frac{1}{k(k+1)\cdots(k+\nu)}=\frac{1}{\nu}\left[\frac{1}{k(k+1)\cdots(k+\nu-1)}-\frac{1}{(k+1)\cdots(k+v)}\right]$$

   所以

   $$\lim\limits_{n\to\infty}\sum_{k=1}^n\frac{1}{k(k+1)\cdots(k+\nu)}=\lim\limits_{n\to\infty}\frac{1}{\nu}(\frac{1}{\nu!}-\frac{1}{(n+1)\cdots(n+\nu)})=\frac{1}{\nu\cdot\nu!}$$

   $\color{blue}{\textbf{Thoughts}}$

   记住这里的注意到.

## 2.3 单调数列

1. **例题 2.3.6**

   > 求数列 $\{a_n\}$ 的极限，其中$a_n=\frac{1!+2!+\cdots+n!}{n!}$ .

   $\color{red}{\textbf{Proof}}$

   首先证明 $\{a_n\}$ 收敛，研究相继两项之比，有

   $$\frac{a_{n+1}}{a_n}=\frac{1!+2!+\cdots+(n+1)!}{(n+1)(1!+2!+\cdots+n!)}=\frac{1!+2!+\cdots+(n+1)!}{1!+(n+1)+2!(n+1)+\cdots+n!(n+1)}$$

   分子分母从第三项开始对应有分母的项大于分子的项，因此 $\frac{a_{n+1}}{a_n}<1$ ，即 $\{a_n\}$ 为单调递减数列.

   又有

   $$a_n<\frac{(n-2)(n-1)!+(n-1)!+n!}{n!}=\frac{(n-1)(n-1)!+n!}{n!}<\frac{2n!}{n!}=2$$

   所以 $\{a_n\}$ 单调有界，故 $\{a_n\}$ 收敛.

   可以发现

   $$a_{n+1}=1+\frac{a_n}{n+1}$$

   两边令 $n\to\infty$ ，可知 $\lim\limits_{n\to\infty}a_n=1$ .

   当然，这题用 Stolz 定理证会更简单.

2. **2.3.2 练习题 12**

   > 对每个正整数 $n$ ，用 $x_n$ 表示方程 $x+x^2+\cdots+x^n=1$ 在闭区间 $[0,1]$ 中的根.求 $\lim\limits_{n\to\infty}x_n$ .

   $\color{red}{\textbf{Proof}}$

   由题意，

   $$
   x_n+x_n^2+\cdots+x_n^n=1 \quad  (1) \\
   x_{n+1}+x_{n+1}^2+\cdots+x_{n+1}^{n+1}=1
   $$

   通过比较可以发现 $x_n>x_{n+1}$ ，即 $\{x_n\}$ 为单调递减数列，又有 $x_n<1$ ，所以 $x_n$ 收敛.

   $$x_{n+1}+x_{n+1}^2+\cdots+x_{n+1}^{n+1}=x_{n+1}(1+x_{n+1}+\cdots+x_{n+1}^n)=1$$

   即

   $$x_{n+1}+\cdots+x_{n+1}^n=\frac{1}{x_{n+1}}-1\quad(2)$$

   令 $n\to\infty$ ，用 （2）式减去（1）式得

   $$0=\frac{1}{A}-2(\lim\limits_{n\to\infty}x_n=A)$$

   所以 $A=\frac{1}{2}$ .

## 2.4 Cauchy 命题与 Stolz 定理

1. **命题 2.4.1（Cauchy 命题）**

   > 设 $\{x_n\}$ 收敛于 $l$ ，则它的前 $n$ 项的算术平均值（所成的数列）也收敛于 $l$ ，即有
   > $$\lim\limits_{n\to\infty}\dfrac{x_1+x_2+\cdots+x_n}{n}=l$$

   $\color{blue}{\textbf{Thoughts}}$

   因为这个证明相对简单，所以就不列了，但是它的证明很重要，体现了分治的思想.

   **P.S.** Cauchy 命题在数列 $\{x_n\}$ 为**有确定符号**的无穷大量时也是成立的.

2. **命题 2.4.2（$\frac{0}{0}$ 型的 Stolz 定理）**

   > 设 $\{a_n\}$ 和 $\{b_n\}$ 都是无穷小量，其中 $\{a_n\}$ 还是严格单调减少数列，又存在 $\lim\limits_{n\to\infty}\dfrac{b_{n+1}-b_n}{a_{n+1}-a_n}=l$ （其中 $l$ 为有限或 $\pm\infin$），则有 $\lim\limits_{n\to\infin}\dfrac{b_n}{a_n}=l$ .

   $\color{red}{\textbf{Proof}}$

   这里只对有限的 $l$ 作证明，其他情况的证明也是类似的.

   根据条件，$\forall \epsilon>0,\exists N,s.t.$ 当 $n>N$ 时有 $\left|\dfrac{b_n-b_{n+1}}{a_n-a_{n+1}}-l\right|<\epsilon$ .

   因为 $\{a_n\}$ 单调减少，所以有

   $$(l-\epsilon)(a_n-a_{n+1})<b_n-b_{n+1}<(l+\epsilon)(a_n-a_{n+1})$$

   取 $m>n$ 并累加得

   $$(l-\epsilon)(a_n-a_m)<b_n-b_m<(l+\epsilon)(a_n-a_m)$$

   即

   $$\left|\dfrac{b_n-b_m}{a_n-a_m}-l\right|<\epsilon$$

   令 $m\to\infty$ ，并利用 $\lim\limits_{m\to\infin}a_m=\lim\limits_{m\to\infin}b_m=0$ ，便可以得到当 $n>N$ 时，成立

   $$\left|\dfrac{b_n}{a_n}-l\right|\le\epsilon$$

   $\color{blue}{\textbf{Thoughts}}$

   在这里有一个问题，既然后面可以用到 $\lim\limits_{n\to\infin}a_n=\lim\limits_{n\to\infin}b_n=0$ ，那为什么一开始不直接用然后直接由极限的四则运算法则得到 $\lim\limits_{n\to\infin}\dfrac{b_n}{a_n}=l$ 呢？

   答案的关键就在于这个 `m > n` .

   当我们直接对 $\lim\limits_{n\to\infty}=\dfrac{b_{n+1}-b_n}{a_{n+1}-a_n}$ 使用极限的四则运算法则时，会发现分母 $\lim\limits_{n\to\infty}(a_{n+1}-a_n)=0$ ，此时显然是不能用极限的四则运算法则的.

   而若我们取 $m>n$ ，然后令 $m\to\infin$ ，这里 $n$ 并没有 $\to\infin$ ，相当于我们固定了 n ，而 m 才是不确定的那个，此时 $a_n,b_n$ 都是一个确定的数，进而 $\lim\limits_{m\to\infty}(a_n-a_m)=a_n$ 而不是 0 ，也就满足了利用 $\lim\limits_{m\to\infin}a_m=\lim\limits_{m\to\infin}b_m=0$ 的条件限制，从而有了 $\left|\dfrac{b_n}{a_n}-l\right|\le\epsilon$ ，最后通过定义证明了 $\lim\limits_{n\to\infin}\dfrac{b_n}{a_n}=l$ .

   还要注意一点，这个方法必须是通过定义来证明，因为直接利用极限的四则运算法则必然要令 $n\to\infin$ ，这样就无法避免分母为 0 了.

3. **命题 2.4.3 （$\frac{*}{\infin}$）型的 Stolz 定理**

   > 设数列 $\{a_n\}$ 是严格单调增加的无穷大量，又存在 $\lim\limits_{n\to\infty}\dfrac{b_{n+1}-b_n}{a_{n+1}-a_n}=l$（其中 $l$ 为有限或 $\pm\infin$），则有 $\lim\limits_{n\to\infin}\dfrac{b_n}{a_n}=l$ .

   $\color{red}{\textbf{Proof}}$

   此处同样只针对 l 为有限的情况证明.

   根据题意，$\forall\epsilon>0,\exists N,s.t.$ 当 $n>N$ 时有 $\left|\dfrac{b_n-b_{n+1}}{a_n-a_{n+1}}-l\right|<\epsilon$ .

   因为 $\{a_n\}$ 单调增加，所以

   $$(l-\epsilon)(a_{n+1}-a_n)<b_{n+1}-b_n<(l+\epsilon)(a_{n+1}-a_n)$$

   将上式的 n 改成 n - 1 并累加到 N ，得到

   $$(l-\epsilon)(a_n-a_N)<b_{n}-b_N<(l+\epsilon)(a_{n}-a_N)$$

   即

   $$\left|\dfrac{b_n-b_{N}}{a_n-a_{N}}-l\right|<\epsilon$$

   我们又有恒等式

   $$\frac{b_n}{a_n}-l=\left(1-\frac{a_N}{a_n}\right)\cdot\left(\frac{b_n-b_N}{a_n-a_N}-l\right)+\frac{b_N-la_N}{a_n}$$

   其中，由于 $\lim\limits_{n\to\infty}a_n=+\infty$ ，存在 $N_1$ ，使得当 $n>N_1$ 时，成立 $1-\dfrac{a_N}{a_n}<2$ 和 $\left|\dfrac{b_N-la_N}{a_n-a_N}\right|<\epsilon$ .

   因此，当 $n>\max(N, N_1)$ 时，有 $\left|\frac{b_n}{a_n}-l\right|<3\epsilon$ .

   $\color{blue}{\textbf{Thoughts}}$

   这里的证明思路和前一个 $\frac{0}{0}$ 型的类似，但是此处 $\{a_n\}$ 是无穷大量，因此不能利用极限来解决最后一步.

   为了方便放缩和利用 $\{a_n\}$ 的极限，我们仍然需要在已知式中引入固定的数 $a_N$ 和 $b_N$ ，然后这里“强行”将待证式用已知式表示，再借助 $\{a_n\}$ 的极限进行放缩，从而证明.

   这个地方的待证式和已知式看上去好像不能转化，但实际上转化是完全可行的，所谓的 trick 其实也不过是很朴素的思想，当想不到其他巧妙的方法时，这个“强行”转化的方法可能会很好用.

   $$\color{yellow}{\textbf{Attention}}$$

   - 以上三个命题的逆命题都不成立.
   - 若在这三个命题中将 $l$ 改为**不带符号**的无穷大量 $\infin$ ，则结论均不成立.

4. **例题 2.4.3**

   > 设已知 $\lim\limits_{n\to\infty}a_n=a$ ，证明：$\lim\limits_{n\to\infty}\frac{1}{2^n}\sum_{k=0}^n\binom{n}{k}=a$ .

   $\color{red}{\textbf{Proof}}$

   利用 $2^n=(1+1)^n=\sum_{k=0}^n\binom{n}{k}$ ，有

   $$\left|\frac{1}{2^n}\sum_{k=0}^n\binom{n}{k}a_k-a\right|=\left|\frac{1}{2^n}\sum_{k=0}^n\binom{n}{k}(a_k-a)\right|\le\frac{1}{2^n}\sum_{k=0}^n\binom{n}{k}|a_k-a|$$

   由题意，$\forall\epsilon>0,\exists N,s.t.$ 当 $n>N$ 时有 $|a_n-a|<\epsilon$ .

   将上面转化后的式子分拆得到

   $$\frac{1}{2^n}\sum_{k=0}^n\binom{n}{k}|a_k-a|=\frac{1}{2^n}\sum_{k=0}^N\binom{n}{k}|a_k-a|+\frac{1}{2^n}\sum_{k=N+1}^n\binom{n}{k}|a_k-a|$$

   对第二部分的估计是很容易的

   $$\frac{1}{2^n}\sum_{k=N+1}^n\binom{n}{k}|a_k-a|<\frac{\epsilon}{2^n}\sum_{k=N+1}^n\binom{n}{k}<\frac{\epsilon}{2^n}\sum_{k=0}^n\binom{n}{k}=\epsilon$$

   而对于前一项，我们需要用到一个不等式 ——$\binom{n}{k}<n^k$ .

   记 $M=\max|a_k-a|$ ，放缩有

   $$\frac{1}{2^n}\sum_{k=0}^N\binom{n}{k}|a_k-a|<\frac{M(n^0+n^1+\cdots+n^N)}{2^n}$$

   因为 $N$ 固定，当 $n\to\infin$ 时，右边的式子的极限为 0 ，因此存在 $N_1>N$ ，当 $n>N_1$ 时成立 $\frac{1}{2^n}\sum_{k=0}^N\binom{n}{k}|a_k-a|<\epsilon$ .

   合并两式的估计，有当 $n>N_1$ 时成立

   $$\left|\frac{1}{2^n}\sum_{k=0}^n\binom{n}{k}a_k-a\right|<2\epsilon$$

   $\color{blue}{\textbf{Thoughts}}$

   此处的证明用了两个很重要的 trick ：

   - $2^n=(1+1)^n=\sum_{k=0}^n\binom{n}{k}$
   - $\binom{n}{k}<n^k$

   同时在对 $\frac{M(n^0+n^1+\cdots+n^N)}{2^n}$ 的估计中，分子只是一个最大指数确定的 n 的多项式，而分母则是指数趋于 $\infty$ 且底数大于 1 的指数函数，因此整个表达式一定是无穷小量.

5. **2.4.3 练习题 5**

   > 设 $\{a_n\}$ 为正数列，且收敛于 $A$ ，证明：$\lim\limits_{n\to\infty}(a_1a_2\cdots a_n)^{\frac{1}{n}}=A$ .

   $\color{red}{\textbf{Proof}}$

   我们利用夹逼准则来证.

   $$(a_1a_2\cdots a_n)^{\frac{1}{n}}\le\frac{a_1+a_2+\cdots+a_n}{n}$$

   根据 Cauchy 命题，$\lim\limits_{n\to\infty}\frac{a_1+a_2+\cdots+a_n}{n}=\lim\limits_{n\to\infty}a_n=A$ .

   考虑 $\frac{\frac{1}{a_1}+\frac{1}{a_2}+\cdots+\frac{1}{a_n}}{n}$ 的极限.

   由 Stolz 定理可知， $\lim\limits_{n\to\infty}\frac{\frac{1}{a_1}+\frac{1}{a_2}+\cdots+\frac{1}{a_n}}{n}=\lim\limits_{n\to\infty}\frac{1}{a_n}=\frac{1}{A}$ ，因此 $\lim\limits_{n\to\infty}\frac{n}{\frac{1}{a_1}+\frac{1}{a_2}+\cdots+\frac{1}{a_n}}=A$ ,进而

   $$(a_1a_2\cdots a_n)^{\frac{1}{n}}\ge\frac{n}{\frac{1}{a_1}+\frac{1}{a_2}+\cdots+\frac{1}{a_n}}$$

   从而左右两边极限都为 A ，根据夹逼准则有 $\lim\limits_{n\to\infty}(a_1a_2\cdots a_n)^{\frac{1}{n}}=A$ .

   $\color{blue}{\textbf{Thoughts}}$

   右边的式子很好想，但左边的式子却不太好处理.

   首先容易想到的比几何均值要小的只有调和均值，由于调和均值的分子分母都可以写成和的形式，我们想到利用 Stolz 定理，但是问题来了，它既不满足 $\frac{0}{0}$ 的形式也不满足 $\frac{*}{\infty}$ 的形式，该怎么办呢？

   这时我们注意到它的分子满足 $\infin$ 的形式，于是我们可以利用 Stolz 定理求出它的倒数的极限值，也就求出了它的极限值.

6. **2.4.3 练习题 9**

   > 设数列 $\{a_n\}$ 满足条件 $0<a_1<1$ 和 $a_{n+1}=a_n(1-a_n)(n\ge1)$ ，证明：$\lim\limits_{n\to\infty}na_n=1$ .

   $\color{red}{\textbf{Proof}}$

   可以发现 $\{a_n\}$ 是迭代数列，我们只需证明出 $\{a_n\}$ 单调有界，便可以很快求出它的极限值.

   由题意，有 $a_{n+1}-a_n=-a_n^2<0$ ，因此 $\{a_n\}$ 严格单调减少.

   因为 $0<a_1<1$ ，所以 $0<a_2=a_1(1-a_1)<1$ ，于是我们可以归纳地得出 $0<a_n=a_{n-1}(1-a_{n-1})<1$ ，进而得到 $\{a_n\}$ 存在下界 0 ，因此 $\{a_n\}$ 收敛.

   在 $a_{n+1}=a_n(1-a_n)(n\ge1)$ 两边令 $n\to\infin$ ，可以求得 $\lim\limits_{n\to\infty}a_n=0$（等于 1 的情况要排除，此处证明略）.

   又 $na_n=\dfrac{n}{\frac{1}{a_n}}$ ，满足 $\frac{*}{\infty}$ 型 Stolz 定理，于是可以求出 $\lim\limits_{n\to\infty}na_n=\lim\limits_{n\to\infty}\dfrac{1}{\frac{1}{a_n}-\frac{1}{a_{n-1}}}$ .

   将 $a_{n+1}=a_n(1-a_n)(n\ge1)$ 变形为 $\frac{1}{a_n}-\frac{1}{a_{n-1}}=\frac{1}{1-a_n}$ ，因此得到 $\lim\limits_{n\to\infty}(\frac{1}{a_n}-\frac{1}{a_{n-1}})=1$ ，所以 $\lim\limits_{n\to\infty}na_n=1$ .

   $\color{blue}{\textbf{Thoughts}}$

   这是一个需要变形的 Stolz 定理题，关键就在于想到将 $na_n$ 变形为满足 Stolz 定理的形式.

   我们不知道 $\{a_n\}$ 中任何一项的具体值，已知的大小关系也仅有它和 1 与 0 的关系，$na_n$ 简单的表达式也不好想到它能怎样放缩，因此定义法和夹逼准则都不好做，这时就要考虑 Stolz 定理了.

   由于 $\{a_n\}$ 是迭代数列，所以它的极限很好求，也很容易求得相邻两项之间的关系，而 $na_n$ 中含有 n ，这显然是很适合用 Stolz 定理的，我们只需将其适当变形即可.

## 2.5 自然对数的底 $e$ 和 Euler 常数 $\gamma$

更不完了 /dk
