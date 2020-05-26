---
title:  .TDD学习笔记（一）单元测试
date: 2020-4-30 19:07
tags: 技术
author: 邹溪源
categories:
  - 技术
---
# 引子
## 回顾
虽然我很早以前就听说单元测试，也曾经多次在项目中引入单元测试框架和单元测试的实践为代码质量的提升带来了一丝助力。

但这种方式更多的是从软件调试的角度出发，即将单元测试作为一种测试方法可用性的入口，而非从TDD、极限编程、或从"Fail Fast,Fix Fast”这种获得快速反馈的方式来使用单元测试，使得实际过程中单元测试的效果并不明显。

直到去年8月下旬开始参加极客学院的[TDD实战课](https://www.jiker.com/plus/2?_comefrom=jikexueyuan&_code_slot=6&_creative=190)才进一步深入了解基于TDD的单元测试的流程、方法和实践的全过程，当时也间歇性的练习了一点Args等Kata项目，才逐步体会到TDD的妙处。

虽然到目前为止对于TDD的了解依然很浅，但在开发过程中，总是有意无意的“站在调用者的角度思考业务逻辑”，并尽可能的思考如何“编写可测试的代码”，总归是一种进步。

## 我的教训
总结自己学习TDD的一些经验教训：

1. 需求的识别，总是按惯性一次性把整个需求全部提取了。
2. 总是习惯于拿着代码一把梭，没有按照“Arange,Assert,Art”的步骤来规划任务。
3. 步子迈得太大，方法拆得不够细，过程式代码的味道很浓，例如，在练习String Calculate过程中，就有非常明显的问题。当然，这也是许多初学TDD开发者的通病。

![图片](https://uploader.shimo.im/f/idWlhCM560UviNjE.png!thumbnail)

![图片](https://uploader.shimo.im/f/k1eYNKHvgyQAHJL8.png!thumbnail)

4. 没有深刻理解“重构”的意义，只是把通过单元测试当做一个目的。
5. Kata的练习频率依然不高，一周只有两到三次，每次不到一个小时。 
6. 单元测试和方法的命名不太规范，无法让人产生直接的理解。
7. 方法的代码行较多，不符合优质代码的标准。
8. 方法间适当的空行（分段）很重要。
# 什么是TDD
## 定义
TDD的全称是“测试驱动开发”，也是一种旨在提升代码质量的开发实践。这种开发实践的主要步骤是在编写产品代码之前，先编写单元测试代码，然后再由测试代码来决定写什么产品代码，其目的是取得快速反馈，并使用“illustrate the main line”方法来构建程序。

测试驱动开发是戴两顶帽子思考的开发方式：先戴上实现功能的帽子，在测试的辅助下，快速实现其功能；再戴上重构的帽子，在测试的保护下，通过去除冗余的代码，提高代码质量。测试驱动着整个开发过程：首先，驱动代码的设计和功能的实现；其后，驱动代码的再设计和重构。

## 原则
在上述经验教训过程中，有些步骤其实与TDD的三原则相违背，让我们来回顾一下这三个原则：

* 不允许编写任何产品代码，除非允许失败的测试通过。
* 不允许编写多余一个的失败测试，编译成功也是失败。
* 不允许编写多于恰好让测试通过的产品代码。
## 步骤
TDD其实也有一系列完整的操作流程，包括如下五个步骤：

* 添加一个小的测试
* 运行所有测试并且失败
* 做一点修改
* 运行所有测试并且成功
* 重构以消除重复

![图片](https://uploader.shimo.im/f/dRjAx1w2z20tzyt1.png!thumbnail)

[图1（红绿重构）](https://xp123.com/articles/tdd-tcr-commits/)

## 可行性之争
TDD也是一种充满争议的开发实践，许多人都吐槽，这种方式在原本开发代码之余，还得额外花三分之一的时间来编写测试代码。不过，我还是推荐《[代码整洁之道-程序员的自我修养](https://item.jd.com/11977659.html)》这本书中，Robert Bob大叔在第5.1小节说的几句话：

>1、此事已有定论！
>2、争论已经结束！
>3、GOTO是有害的！
>4、TDD确实可行。

他明确指出：

>过去人们对TDD充满争议，就此发表了不少博客和文章，如今争议依旧来袭。所不同的是，以前人们是认真尝试着去批判和理解TDD，而现在只有夸夸奇谈而已。结论很清楚，TDD的确切实可行，而且每个开发人员都要适应和掌握TDD。
## TCR
在《极限中国社区》曾经介绍了一种测试驱动开发过程中的实践模式，这种实践被称为“[TCR](https://xp123.com/articles/tdd-tcr-commits/)”。

在实践过程中，开发者始终保持着测试，成功则提交，失败则回滚到上次的代码这样的循环。在并使用了插件来进行自动化回滚，确保每个方法的开发时间被控制在非常小的时间粒度上。这样保证了开发者能够以非常小的步子，非常快的频率，实现代码的开发过程。

![图片](https://uploader.shimo.im/f/BKMKzoRcX7MbRVwk.png!thumbnail)

[TCR](https://xp123.com/articles/tdd-tcr-commits/)

# Kata
优秀的代码从来不是天生的，而是通过后天不断的练习培养出来的，尤其是要想写出符合面向对象设计的的好代码，更是需要“刻意练习”。

Kata被人称为是唯一的一种[练习TDD](http://www.peterprovost.org/blog/2012/05/02/kata-the-only-way-to-learn-tdd/)的形式。

“Kata”是一种来自日语词汇“形式”的翻译，它描述了武术练习中，通过一种精心编排的动作模式，用来训练自己达到肌肉记忆的水平。

Kata的练习例子是[如此之多](http://codingdojo.org/kata/)，只要你有心，总是能在海量的示例代码中找到最适合自己的一个例子。

例如，我所使用的Roy Osherove设计的例子“String Calculate”就是一个非常不错的示例。（当然，我给出的是一段反例代码。）

```
1、An empty string returns zero
2、A single number returns the value
3、Two numbers, comma delimited, returns the sum
4、Two numbers, newline delimited, returns the sum
5、Three numbers, delimited either way, returns the sum
6、Negative numbers throw an exception
7、Numbers greater than 1000 are ignored
8、A single char delimiter can be defined on the first line (e.g. //# for a ‘#’ as the delimiter)
9、A multi char delimiter can be defined on the first line (e.g. //[###] for ‘###’ as the delimiter)
10、Many single or multi-char delimiters can be defined (each wrapped in square brackets)
```
当然，[FizzBuzz](http://codingkata.net/Katas/Beginner/FizzBuzz)或[The Prime Factors Kata](http://www.butunclebob.com/ArticleS.UncleBob.ThePrimeFactorsKata)，Args也是一个非常不错的示例。重要的并非例子本身，而是通过持续不断的练习，形成自己的肌肉记忆。
在[引文](http://www.peterprovost.org/blog/2012/05/02/kata-the-only-way-to-learn-tdd/)中，作者Peter Provost认为，最好的办法：

>我对人们的建议是连续两周每天早上做一个30分钟的练习。然后再选一个，每天做一次，坚持两周
>我不建议大家在工作中练习Kata，除非他们已经准备好了。你应该通过练习一周或六次，直到你已经决定对TDD的循环非常适应为止。否则，就像参加了一场没有技术水平的比赛。
