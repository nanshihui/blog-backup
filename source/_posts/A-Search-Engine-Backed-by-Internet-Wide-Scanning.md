---
title: A Search Engine Backed by Internet-Wide Scanning
date: 2016-05-17 19:44:29
categories:
- 论文阅读
tag:
- CCS
---
<h2 id="intro">前言</h2>仅记录A Search Engine Backed by Internet-Wide Scanning读后短评
### 结构
- **概要** 论文概要；
- **介绍:** 论文主要内容；
- **结论:** 论文结论；

<!-- more -->
## 背景资料

* 单位及作者 ：Zakir Durumeric、 David Adrian† Ariana Mirian、 Michael Bailey、 J. Alex Halderman，University of Michigan ,University of Illinois, Urbana Champaign
* 年份 ：2015
* 类型：CCS

## 概要
　　快速的网络扫描给安全研究创建了一个新的途径，从揭露随机数发生器的脆弱性到心脏出血的影响。然而，这种技术仍然需要很大的改进：比如，“哪款嵌入式设备更倾向于CBC加密？”，即使是简单的问题，需要开发一个应用扫描器，手动识别和标记设备，与网络管理员协商，并需要应对一些投诉。在本文中，我们将介绍Censys，他是公共搜索引擎和数据处理设备，其数据来源于持续的Internet范围扫描。旨在帮助研究人员回答与安全有关的问题，Censys在协议banner上支持全文搜索和查询范围很广的字段（例如443.https.cipher）。它可以识别特定的易受攻击的设备和网络，并对广泛的使用模式和趋势生成统计报告。 Censys能在毫秒级的时间返回这些结果，大幅降低了解互联网主机所付出的代价。我们提出了一个搜索引擎的架构并实验对其性能进行评估。我们还探索Censys，如何简单回答最近的研究中提出的问题。
## 原文摘要
　　Fast Internet-wide scanning has opened new avenues for security research, ranging from uncovering widespread vulnerabilities in random number generators to tracking the evolving impact of Heartbleed. However, this technique still requires significant effort: even simple questions, such as, “What models of embedded devices prefer CBC ciphers?”, require developing an application scanner, manually identifying and tagging devices, negotiating with network administrators, and responding to abuse complaints. In this paper, we introduce Censys, a public search engine and data processing facility backed by data collected from ongoing Internet-wide scans. Designed to help researchers answer security-related questions, Censys supports full-text searches on protocol banners and querying a wide range of derived fields (e.g., 443.https.cipher). It can identify specific vulnerable devices and networks and generate statistical reports on broad usage patterns and trends. Censys returns these results in sub-second time, dramatically reducing the effort of understanding the hosts that comprise the Internet. We present the search engine architecture and experimentally evaluate its performance. We also explore Censys’s applications and show how questions asked in recent studies become simple to answer.

## 介绍
　　使用ZMAP可以快速的扫描网络，但是在Internet范围收集有意义的数据仍然是一个专业化和劳动密集的过程。比如回答“哪一部分的https服务器更倾向于使用forward secret key exchange methods”这样的问题，这可能要花费一周的时间。在这种情况下专业人员需要开发高性能的应用扫描器对其他主机的443端口建立https连接并进行监听，测试并修复那些不完全遵循TLS规范的主机，然后又要扫描处理大量的返回数据。在开始此过程之前，安全研究人员必须与他们的权限机构和网络团队谈判进行扫描，还要和他们的上游网络供应商进行协调，以及后来滥用的回应。许多机构（和独立的研究人员）缺乏网络设施或行政支持，执行扫描。由于这些原因，Internet范围扫描仍然是一个小数目的研究群体，这严重地限制了应用这个方法的应用程序。Internet范围的扫描已经在发现安全问题和理解复杂的分布式系统的安全性表现出了极大的潜力。通过移动扫描到云，Censys显着减少调查这些问题所需的工作量，使研究人员能够专注于问更重要的问题，而不是回答他们的机制。此外，Censys允许安全社区增加全球协议的覆盖度，并为理解互联网增加的越来越多的嵌入式设备提供了易于处理的解决方案。同时，它最大限度地减少研究小组多余的扫描以及网络运营商监控所传入的网络流量。
　　作者的工作：（1）提出了一个数据收集架构（2）针对近期的安全研究，给出了POODLE以及追踪工业控
制系统漏洞的影响。（3）实现这个架构。
## 结论
　　在回答有意义的安全研究问题上，对IPv4地址空间，进行主机发现扫描的技术能力仍有差距。在本文中，介绍了Censys，通过持续的Internet范围扫描收集数据来支撑的公共查询引擎和数据处理设备。旨在帮助研究人员回答安全有关的问题，Censys收集IPv4地址空间的数据，支持从扫描结果导出的字段进行查询并生成统计报表。探讨Censys的几个安全应用，展现Censys如何能用来方便地回答最近研究的问题。希望Censys使研究人员能够轻松地回答关于以前需要大量精力在网络上的问题，同时减少了重复劳动和总流量扫描。
## 注解
　　该文侧重于从工程的角度描述了如何搭建一个全局的扫描框架体系，并以其中一个漏洞，针对该漏洞的影响力，生成一个可视化的报告。国内的ZOOMEYE以及俄罗斯的sodan都有些相似。随着以后安全的发展，基于大数据的安全操作会将变得越来越突出。