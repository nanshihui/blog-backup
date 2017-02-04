---
title: The Spy in the Sandbox :Practical Cache Attacks in JavaScript and their Implications
date: 2016-05-27 21:46:36
categories:
- 论文阅读
tag:
- CCS
---
<h2 id="intro">前言</h2>仅记录The Spy in the Sandbox: Practical Cache Attacks in JavaScript and their Implications 读后短评
### 结构
- **概要** 论文概要；
- **介绍:** 论文主要内容；
- **结论:** 论文结论；

<!-- more -->
## 背景资料

* 单位及作者 ：Yossef Oren Vasileios P. Kemerlis Simha Sethumadhavan Angelos D. Keromytis, Columbia University
* 年份 ：2015
* 类型：CCS

## 概要
　　在浏览器上提出一个微架构的侧信道攻击。相较于之前的侧信道攻击，攻击者不需要在受害者电脑上安装任何软件。攻击者只需要在恶意网站上嵌入恶意代码就行。这使我们的攻击模型高度可扩展，在今天的网络非常有针对性和实用性。因为目前大多数用于访问因特网的桌面浏览器都受到这种侧信道威胁。该攻击是最后一级缓存的扩展，允许远程攻击者以恢复属于其他进程的信息、用户、甚至是与受害人的Web浏览器运行在同一台物理主机上的虚拟机。
　　此外本文描述了该攻击的基本原理，并评估其性能特点。并展示在一个共同的环境中它如何被用来危及用户隐私，让攻击者窥视受害者的浏览隐私。抵御这侧信道攻击是可能的，但所需要的对策在友好使用浏览器上要付出不切实际的成本。（but the required countermeasures can exact an impractical cost on benign uses of the browser.）
## 原文摘要
　　We present a micro-architectural side-channel attack that runs entirely in the browser. In contrast to previous work in this genre, our attack does not require the attacker to install software on the victim's machine; to facilitate the attack,the victim needs only to browse to an untrusted webpage that contains attacker-controlled content. This makes our attack model highly scalable, and extremely relevant and practical to today'sWeb, as most desktop browsers currently used to access the Internet are a_ected by such side channel threats. Our attack, which is an extension to the last-level cache attacks of Liu et al. , allows a remote adversary to recover information belonging to other processes, users, and even virtual machines running on the same physical host with the victim web browser.
　　We describe the fundamentals behind our attack, and evaluate its performance characteristics. In addition, we show how it can be used to compromise user privacy in a common setting, letting an attacker spy after a victim that uses private browsing. Defending against this side channel is possible, but the required countermeasures can exact an impractical cost on benign uses of the browser..

## 介绍
　　文章中的模型是建立在受害者需要访问攻击者的网站。作者展示攻击者在现实环境中如何攻击和提取受害者机器上有意义的信息。并在此环境下将重点从加密密钥的恢复转向到对用户行为的跟踪。攻击的前提是受害者使用搭载新特尔处理器的计算器，并且受害者使用支持html5的浏览器访问网页。基于JavaScript的缓存攻击，让攻击者访问跟踪到受害者的最后一级缓存。由于这个单一缓存被所有CPU核心共享，这种访问信息可以提供攻击者有关用户和系统的详细知识。
　　作者的工作：（1）提出创建最后一级缓存的非规范驱逐集的方法。该不需要大页面系统的支持，可以立即被应用到更广泛的各种系统。（2）证明只用JavaScript代码对最后一级缓存攻击。在同一台机器上和虚拟机客户端和主机之间运行不同的程序，使用隐蔽信道的方法评估它的性能，媲美原生代码的方法。（3）展示基于高速缓存的攻击是如何跟踪用户的行为。具体来说，提出了一个简单的基于分类的攻击，让网页恶意窥视用户的浏览活动，检测用户使用常见网站的准确度超过80％。它甚至可以窥视一个完全不同浏览器的隐私浏览会话。
## 结论
　　演示了一个微架构如何进行侧信道缓存攻击，这已经是公认的非常有效的攻击方法，可以通过不受信任的网页启动。用缓存的攻击取代传统密码分析，展示了使用该方法如何成功跟踪用户行为。侧信道攻击的潜在范围已扩大，这意味着系统的其他类的设计必须考虑到旁道对策

## 注解
　　最后一级缓存攻击，攻击的角度新颖，而且发动的成本低，巧妙的绕过了js语言不能直接访问内存信息的特性。根据CPU架构的特点，有针对的攻击。随着html5的支持度广泛，会有更多可能可利用的API可以利用。