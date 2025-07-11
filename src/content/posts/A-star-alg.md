---
title: "A* 算法总结"
published: 2023-08-18
description: ''
image: ''
tags: ['Graph Theory']
category: 'Algorithm'
draft: false 
lang: ''
---

## Introduction

考虑一个经典问题，给出一张图，一个起点 S ，一个终点 G ，求代价最小的路径。

当每条边的权值相同时，显然使用经典的 BFS 算法就可以了，但是当每条边权值不同时普通的 BFS 效率就很一般了。

能不能优化？将 BFS 队列中的结点记为 frontiers ，在 frontiers 中，每个结点都存储下扩展到它时的花费 cost ，然后将队列换成优先队列，每次优先取出 cost 最小的结点，若这个结点是 G 则完成，否则用它扩展其他结点并加入 frontiers 。这样一来，貌似是满足了最优性，但是由于一个结点可能离 S 很近，但是却离 G 很远，我们会发现一个结点会被加入队列多次，产生了大量的冗余状态。

上面这个算法称为 UCS (Uniform Cost Search) ，它仅仅只考虑了结点到 S 的花费来评估一个结点，我们希望能再结合考虑结点到 G 的花费来优化时间和空间。

具体而言，设 $g(x)$ 表示 S 到结点 x 的 cost ，启发式函数 $h(x)$ 表示结点 x 到 G 的**估计** cost ，将 $f(x)=g(x)+h(x)$ 作为从 S 经过 x 到 G 的预估 cost 。我们每次扩展到一个结点 x 时便可以得到**实际**的 $g(x)$ ，这时我们计算出它的 $f(x)$ 合并上 x 作为一个状态丢进优先队列，每次优先取出 $f(x)$ 最小的状态，若这个状态的结点是 G ，则找到最优路径。

这便是 A\* 算法的流程，它的难点在于设计一个合理、优秀的启发式函数 $h(x)$ 。

## Constraints

一个合理的启发式函数需要满足两个约束条件：Admissibility（不高估结点到 G 的花费）和 Consistency（不高估 S 到 G 经过该结点的最优路径的花费）

### Admissibility

设 $h^*(x)$ 表示 x 到 G 实际花费，若满足：

$$
\forall n, 0\le h(n)\le h^*(n)
$$

则 A\* 算法一定能找到最优解。

#### Proof

设 G 可以由可达结点 A\' 和 B\' 扩展得到，其中 A\' 位于最优路径上，B\' 不位于最优路径，设 A 和 B 分别表示 G 由 A\' 和 B\' 扩展得到存入 frontiers 中的状态。

记 n 为 A\' 的祖先结点，有 $h^{\ast}(n)-h^{\ast}(A)=g(A)-g(n)$ ，于是

$$f(n)=g(n)+h(n) \le g(n)+h^{\ast}(n)=g(A)+h(A) = f(A) < f(B)$$

也就是说，最优路径上的状态一定会在 B 之前被扩展到，从而 A\* 算法一定能找出一条最优路径。

### Consistency

仅仅满足 Admissibility 会发现 A\* 算法与 UCS 并没有多少区别，同样会把一个结点扩展多次，于是还需要满足 Consistency 约束：

$$
\forall A, C \ h(A)-h(C)\le cost(A, C)
$$

这样一来，A\* 算法便不会加入重复结点。

#### Proof

设 C 由 A 扩展而来，则有

$$f(C)=g(C)+h(C)=g(A)+cost(A, C)+h(C) \le g(A)+h(A)=f(A)$$

也就是说，一条路径上的结点对应的 $f(n)$ 是单调不增的。

假设 A 和 B 都可以扩展到 C ，我们一开始用 A 扩展了 C ，于是可以得到 $f(C)\le f(A)<f(B)$ ，又考虑到 $g(B)<g(C)$ ，即便用 C 去扩展 B 也不会让 $f(B)$ 变小，更何况 B 已经在 frontiers 中了我们不会再次加入 B ，也就不可能用 B 来重复扩展 C 了，从而 A\* 算法不会加入重复结点。

## Dominance

对于两个满足 constraints 的启发式函数 $h_a$ 和 $h_b$ ，若 $\forall n, h_a(n)\ge h_b(n)$ ，则说明 $h_a$ 比 $h_b$ 更接近实际函数 $h^*$ ，也就更优秀。

## Example

[P1379 八数码难题](https://www.luogu.com.cn/problem/P1379)

将每个状态看作一个结点，$h(x)$ 定义为当前状态中每个数字到目标位置的曼哈顿距离（记得排除数字 0 的），然后跑 A\* 即可。

OI-wiki 上将 $h(x)$ 定义为当前状态中位置不在目标位置上的数字的数量，这个定义并不能保证找到的是最优路径，在 Luogu 上提交会发现如果 dx 和 dy 数组与标答不同会有一个点过不去。

## References

- [A Star Algorithm 总结与实现 | Cheng Wei's Blog (scm_mos.gitlab.io)](https://scm_mos.gitlab.io/motion-planner/a-star/)
- [A 星寻路算法介绍 - 莫水千流 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zhoug2020/articles/3468167.html)
- [A\* - OI Wiki (oi-wiki.org)](https://oi-wiki.org/search/astar)
- [cs188-fa22-note02](https://inst.eecs.berkeley.edu/~cs188/fa22/assets/notes/cs188-fa22-note02.pdf)
