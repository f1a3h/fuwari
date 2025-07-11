---
title: "NJU「软件分析」学习笔记：Datalog-Based Program Analysis"
published: 2024-01-01
description: 'Datalog-Based Program Analysis'
image: ''
tags: [Program Analysis]
category: 'Study Notes'
draft: false 
lang: ''
---

:::note[INFO]
南京大学「[软件分析](https://tai-e.pascal-lab.net/lectures.html)」课程 Datalog-Based Program Analysis 部分的学习笔记。
:::

## Motivation

- imperative: how to do (~implementation)
  - E.g. C. 需要告诉机器要如何去实现
- declarative: what to do (~specification)
  - E.g. SQL. 仅仅需要指明你想要什么，如何实现则交给程序（DBMS）自行选择

在 program analysis 中，我们指明对于不同的 statement 对应不同的 rule，这就是一种 specification，而在 algorithm 中，我们则指明了每一步具体要怎么做，在实际的代码实现里更是要考虑更多的 implementation details。可以看出，前面所学的 program analysis 即是 declarative 也是 imperative 的。

在 tai-e assignments 中，我们使用 imperative 的方式实现 program analysis，同样的，我们也可以通过 declarative 的方式来实现。(via Datalog)

## Introduction to Datalog

datalog 最早是以 database language 的身份出现的[^1]，现在有了更广泛的应用。

[^1]: David Maier, K. Tuncay Tekle, Michael Kifer, and David S. Warren, *\"Datalog: Concepts, History, and Outlook\"*. Chapter, 2018.

- Datalog = Data + Logic (and, or, not)
  - no side-effects
  - no control flows
  - no functions
  - not turing-complete

### Predicates (Data)

在 datalog 中，一个 predicate (relation) 就是值一系列的 statemetns，本质上其实是 a table of data。而 fact 则是指这个 table 中某个 tuple。

atoms 是 datalog 的 basic elements。

`P(X1, X2, ..., Xn)` 的形式表示一个 relation atom，其中 P 是 predicate 的名字，Xi 则是 arguments， 它的值可以是一个 variable 也可以是 constant。一个 relation atom 的值为 true 当且仅当 P 中存在 `(X1, X2, ..., Xn)` 这个 tuple。

除了 relation atoms，datalog 还有 arithmetic atoms。

### Datalog Rules (Logic)

rule 用来表示 logic inferences，并且表明 facts 是如何被 deduced 的，它的形式为 `H <- B1, B2, ,,,, Bn` ，其中 head 和 body 中的每一项都是 atom，`Bi` 也被称为 subgoal。

一个 datalog program 由 facts 和 rules 组成。更多的语法直接看 slides 吧，懒得写了（

datalog 的 predicates 被分为 EDB 和 IDB 两种：

1. EDB (extensional database)
   - 提前定义好的 predicates
   - relations are immutable
   - 可以看作 input relations
2. IDB (intensional database)
   - 由 rules 推导出的 predicates
   - 可以看作是 output relations

datalog 支持 recursive rules，从而可以从自身进行 deduce。

考虑以下两条 rules：

- `A(x) <- B(y), x > y.`
- `A(x) <- B(y), !C(x, y).`

可以发现它们都有可能使得 A 称为一个 infinite relation，但是这并不是我们使用 datalog 想要得到的结果，所以我们要避免 infinite relation，也就因此引入了 rule safety 的概念。

注意到 EDB 的元素是有限个的，因此对于一个 non-negated realtion atom，它能 deduce 的元素也会是有限的，所以我们只需要保证 head 中的每一个 variable 至少被一个 non-negated atom relation 给限制住了即可。在 datalog 中，只有满足 safe rules 是被允许的。

再考虑这条 rule：

- `A(x) <- B(x), !A(x)`

显然这是一条自相矛盾的 rule，但是它满足了 safe rule 的条件，因此我们需要另外添加限制。观察可以发现这种 contradictory rule 必然也是 recursive 的形式，并且是在 body 中对 head 进行 negate，因此，在 datalog 中，我们规定 recursion and negation of an atom must be separated。

### Execution of Datalog Programs

![execution](./execution.png)

- datalog 使用 datalog engine 根据 EDB 和 rules 进行 deduce 并输出 IDB
- monotonicity: facts 只会从 rules 中产生而不会因为 deduction 而减少
- termination: 通过 rule safety 保证了能被 deduce 的 facts 必然是 finite 的，因此 datalog program 必然是可以终止的

## Pointer Analysis via Datalog

其实只要用 datalog 的语言描述出前面 pointer analysis 的 rules 即可，同样也需要考虑一些细节以及每个 atom 的 arguments 要如何设计。

![pointer analysis](./pa.png)

## Taint Analysis via Datalog

![taint analysis](./ta.png)

使用 datalog 来实现 program analysis 有以下优点和缺点：

- pros
  - 简洁、可读性高
  - 容易实现，写 bug 的概率也小很多
  - off-the-shelf optimized datalog engines 可以让你少考虑一些优化上的细节，make your life easier
- cons
  - 表达能力有限，某些 logics 很难用 datalog 表示甚至根本表示不了
  - datalog engine 作为一个 black box 的存在使得我们很难用更优秀、更有针对性的 algorithm 来实现更优越的性能