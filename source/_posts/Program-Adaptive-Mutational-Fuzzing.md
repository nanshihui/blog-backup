---
title: Program-Adaptive Mutational Fuzzing
date: 2016-05-10 20:41:10
categories:
- 论文阅读
tag:
- S&P
---
<h2 id="intro">前言</h2>仅记录Program-Adaptive Mutational Fuzzing读后短评
### 结构
- **概要** 论文概要；
- **介绍:** 论文主要内容；
- **结论:** 论文结论；

<!-- more -->
## 背景资料

* 单位及作者 ：Sang Kil Cha, Maverick Woo, and David Brumley，Carnegie Mellon University
* 年份 ：2015
* 类型：S&P

## 概要
　　针对黑盒模糊测试所提供的程序和一个种子输入，设计出一个算法尽可能的找出更多的bug，主要是通过给定一个程序和一个种子对，通过白盒元素分析来追踪以检测输入串比特位置之间的依赖。使用这个依赖关系，针对给定的程序和种子对，计算出一个最佳的突变率。最终使用之前3款模糊测试工具，在8款应用使用相同的模糊测试时间测试，比其他模糊测试软件发现的bug数量，平均多发现37.2%。
## 原文摘要
　　We present the design of an algorithm to maximize the number of bugs found for black-box mutational fuzzing given a program and a seed input. The major intuition is to leverage white-box symbolic analysis on an execution trace for a given program-seed pair to detect dependencies among the bit positions of an input, and then use this dependency relation to compute a probabilistically optimal mutation ratio for this program-seed pair. Our result is promising: we found an average of 37.2% more bugs than three previous fuzzers over 8 applications using the same amount of fuzzing time.

## 介绍
　　在文章中介绍了一个名为SYMFUZZ的系统，该系统可以在给定程序和输入种子的情况下，计算出最佳突变率来发现可能的软件崩溃情况。其通过白盒技术分析程序执行情况得出一个有效的突变模糊比率，来增强效果，相较于传统的黑盒突变模糊测试衍生出一个突变率。尽管白盒分析会花去很多时间，但是每个程序种子对，只需要一次预处理步骤。
　　文章认为最佳突变率可以从输入比特之间的依赖关系进行推导。举个例子，当给出一个程序和一个由32位幻数和两个连续的32位整数所组成的96位的种子，假设这个幻数是42424242（16进制），两个整数都为0.且当输入值为42424242，以及第三个字段为负整数的时候，程序崩溃。因此我们需要翻转第三字段最有效的比特位，不去翻转第一字段的比特位。第二段的比特值并不会影响程序，而第一字段和第三字段存在一种依赖关系。
　　软件运行分两步：先翻转输入的种子比特位产生测试用例。再评估使用测试用例是否会使程序崩溃。假设预先知道输入种子的类型以及程序是怎么运行。那么模糊软件选择变动的比特位取决于程序，输入种子，突变率有关。
　　作者的工作：（1）设计出一个数学架构标准化突变模糊测试，对模糊测试比特位之间的关联性进行建模。（2）提出最佳突变率，来尽可能发现更多的bug。（3）结合黑白盒来提高模糊测试的效率。（4）设计出一个突变模糊测试框架。
## 结论
　　设计出一个算法，在给定程序以及输入种子会产生一个最佳突变率。结合黑白盒来查找程序的bug。同时将输入位之间的关联所引起的程序崩溃率进行标准化。并且将突变率最优化进行建模，通过执行追踪评估出最佳突变率。通过本程序的数据集，可以找到的bug数量比BFF（一款模糊测试工具）多39.5%，比zzuf多57.9%.。另外经过同样24小时的测试，使用本算法的AFL-FUZZ找到bug的数量比原始多了18.5%。

## 注解
　　该文侧重于只针对单个输入，对一定长度序列选择最佳的测试用例，针对于协议的漏洞可以收到比较良好的效果。另外比较受欢迎的方式就是白盒测试，做代码审计形成一个语法树，找出存在危险的语句。而对于WEB应用的漏洞，采用这样的方法，就不一定会适合，首先输入的位置较多，其次输入的测试用例不同于本实验的字符串，无法穷举。