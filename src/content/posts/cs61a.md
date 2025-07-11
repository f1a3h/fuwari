---
title: "CS61A 通关感想"
published: 2023-11-05
description: 'CS61A, Summer 2020'
image: ''
tags: [CS61A]
category: 'CSDIY'
draft: false 
lang: ''
---

从 8 月初到现在，历时将近 3 个月，我才终于通关了 [CS61A, Summer 2020](https://inst.eecs.berkeley.edu/~cs61a/su20/) ，因为 8 月时过于怠惰，开学之后又有丰富多彩的课程表（不愧是你啊，HUST），导致战线拉的这么长（碎碎念）。

在 CS61A 这门课中，收获的远远不止是 Python、Scheme、SQL 语法，它通过一个个精心设计的 lab、hw、proj，让我能切身实际的感受到编程的魅力，另外，认真完成这些任务后，我对 CS 领域中的不少理念，如 “abstraction” 等，有了更深刻的理解，这绝对是乖乖跟着 HUST 答辩课程学习所得不到的，更重要的是，CS61A 为我打开了 CS 的第一扇门，尽管它不是我看的第一个国外优质课程。

不仅仅是 CS 本身的乐趣，教授这门课的 John DeNero 教授也是幽默风趣，能传授人生哲理的老师。比如万圣节时 DeNero 教授倾心录制的视频，带着搞怪南瓜帽子的教授着实有趣，但是更值得注意的是他的话：我们不应该为 GPA 所困扰，CS 的学习本身是一件很有趣的事情。

<iframe width="680" height="427" src="https://www.youtube.com/embed/9DyfRMlyxfg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

> Computers and science, how did they meet?
> 
> Computers are cool, and science is neat.
> 
> But then, one day, they must have teamed up on the street.
> 
> And created this course, in which I have a seat. 
> 
> And it's hard, the projects are tough to complete,
> 
> And Scheme seems more like a trick than a treat.
> 
> And the tests, the tests are the very worst part.
> 
> I study, I practise, and I'm really smart.
> 
> And I know all the rules of assignment by heart.
> 
> But the day of the midterm, the TA says start, 
>
> And all of my test taking plans fall apart, 
> 
> Because the problems are even more abstract than art, 
> 
> Sometimes I can't even solve problem one, 
> 
> And why is problem named some stupid pun?
> 
> And that jerk Fibonacci, he's so overdone.
> 
> Computers and science has teamed up and won.
> 
> But hold on..., courses aren't lost, they aren't won,
> 
> And learning to program is suppose to be fun.
> 
> So don't let these tests and these points get the best of you,
> 
> Grades in your major count less than you think they do. 
> 
> I've never heard them come up in an interview.
> 
> And the faculty I know really likes students who might not aced every test, but don't quit
> 
> They focus on learning, improvement, and grit
> 
> And soon they have learned about more than a few things
> 
> And all their hard work made them able to do things
> 
> And doing things, that's what the world cares for.
> 
> What you can do matters more than your score.

也是因为看过了 CS61A ，这更加坚定了我一定要申请到 PhD 的想法，希望大三的时候能有机会报名 UCB 的 BGA 项目，拿一封份量足够的推荐信（做梦也想去四大😭

好了，回归正题，说说 CS61A 究竟讲了些啥。

首先，当然是基础的 Python 语法了，但是和国内教学 C 语言语法不同，CS61A 通过 Python 还讲授了 CS 领域重要的概念——abstraction，另外，通过他的 lab、hw ，你能对递归有一个更深刻的理解。

其次，Python 作为著名的 OOP (Object-Oriented-Programming) 例子，当然要讲讲 OOP 究竟是个啥玩意了。我之前对面向过程、面向对象的理解纯粹来自于廖雪峰的 Java 教程中提到的「建造凳子」的例子，可以说并没有完全理解，但是被问到可以用这个例子糊弄过去（捂脸）。在看过 John DeNero 教授对 OOP 的解释并且做完了相应的编程练习之后，我还是挺有自信应该理解了 OOP 的概念的，相比于面向过程，OOP 考虑的是一个个的 object ，它属于一个 class ，可以继承这个 class 的 attributes ，它作为一个独立的个体也可以拥有属于自己的 attributes ，同时，object 的所谓的「函数」更准确的说法是 methods，它的作用并不是解决某一个问题，而是对这个 object 做某种操作的「方法」，语言的表达能力终究有限，要实际理解 OOP ，还是得看看 DeNero 教授的讲解，然后做完对应的练习，get your hands dirty 永远是学习 CS 亘古不变的真理。

之后是 scheme——lisp 的一种方言，用来学习 functional programming 函数式编程的优秀的样例。函数式编程的思想和另外两种编程有很大的不同，最大的感受是你不得不把每一个步骤考虑清楚才能动手 coding ，不像我写 Python 和 C++ ，总是没想清楚实现细节就开始写代码。还有很重要的一点就是 recursion，在 scheme 的练习中需要大量用到 recursion，这可以说是在逼迫你加深对 recursion、abstraction 的理解与运用了。另外，tail- recursion 也是一个很神奇的东西，这玩意让我明白了当年搞 OI 时听到的“开不开无限栈？”中的无限栈到底是啥，同时思考把一个浪费一堆 frame 的 recursion 通过多设置几个带的变量变成了只有线性大小的 frame 的 tail-recursion 也很有意思。

最后是以 SQL 作为 declarative programming language 的例子讲了 SQL 的基础语法以及 declarative programming language 的概念，这对之后数据库的学习能带来不少帮助。

总而言之，CS61A 一定是一门优秀的 CS 入门课，无论是大一还是大二，抑或是研究生，甚至是博士生，我都相信 CS61A 是值得花时间认真刷完的一门课，毕竟，它可是能点燃对 CS 的热情的一把火啊！（暴论）