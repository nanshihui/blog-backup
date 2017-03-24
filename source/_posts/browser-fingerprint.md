---
title: 浏览器指纹追踪
date: 2017-03-21 18:07:10
tags:
- 浏览器指纹
categories:
- Web安全
---

<h2 id="intro">前言</h2>随着flash逐渐退出浏览器,传统的浏览器指纹开始不适用,为此本文介绍目前最常见的浏览器指纹模型,以及新的浏览器指纹技术,方便需要的人.

### 浏览器指纹技术
- **常见浏览器指纹:** 常见的浏览器指纹技术；
- **Canvas指纹以及WebGL 指纹:** 用过Canvas以及WebGL技术获得指纹信息；
- **新浏览器指纹技术:** 目前前沿的浏览器指纹技术；
- **驻留技术:** 提高cookie的存活率,即使被删也能恢复,甚至是跨浏览器；


<!-- more -->
## 引言
　　最近研究如何在用户匿名的时候,来标识一个用户,于是就发现了浏览器指纹,浏览器指纹是指用户在使用浏览器的时候,脚本会自动收集用户浏览器的配置信息以及系统信息,而这些信息可以用来唯一的标识一个用户.由于flash组件对于浏览器而言始终是一个隐患,明年Chrome浏览器,会逐步消去flash组件,取而代之的是html5技术,在新的技术下,浏览器指纹信息也要相应的更新.

## 常见的浏览器指纹技术
　　相对比较成熟的浏览器指纹模型是AmIUnique模型.其使用具体指纹属性包含如下几种

| 属性| 来源 | 例子  |
| ------------- |:-------------:| -----:|
| User agent | HTTP header | Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.118 Safari/537.36 |
| Accept     | HTTP header      |  text/html,applicationxhtmlxml,applicationxml;q=0.9,image/webp,*/*;q=0.8 |
| Content encoding | HTTP header     |   gzip, deflate, sdch |
|Content language| HTTP header|en-us,en;q=0.5|
|List of plugins|JavaScript|Plugin 1: Chrome PDF Viewer. Plugin 2: Chrome Remote Desktop Viewer. Plugin 3: Native Client.Plugin 4: Shockwave Flash..|
|Cookies enabled|JavaScript|yes|
|Use of local/session storage|JavaScript|yes|
|Timezone|JavaScript|-60 (UTC+1)|
|Screen resolution and color depth|JavaScript|1920x1200x24|
|List of fonts|Flash plugin|Abyssinica SIL,Aharoni CLM,AR PL UMing CN,AR PL UMing HK,AR PL UMing TW...|
|List of HTTP headers|HTTP headers|Referer，X-Forwarded-For，Connection，Accept，Cookie，Accept-Language，Accept-Encoding，User-Agent，Host|
|Platform|JavaScript|Linux x86_64|
|Do Not Track|JavaScript|yes|
|Canvas|JavaScript|vext quiz|
|WebGL Vendor|JavaScript|NVIDIA Corporation|
|WebGL Renderer|JavaScript|GeForce GTX 650 Ti/PCIe/SSE2|
|Use of an ad blocker|JavaScript|no|



　　正如我们所看到的,amiunique使用了多个维度的指纹信息.但是有些属性是属于可以更改的,比如屏幕分辨率.由于目前获得屏幕分辨率的方式是通过js代码完成的.如果用户使用```Ctrll++```,屏幕的分辨率就会不准确了.所以缩放等级可能更加适合用来当做浏览器指纹,或者使用长宽比,因为长宽比不受缩放的影响.
　　此外,availHeight, availWidth, availLeft,availTop,以及screenOrientation这四个属性也有一定的用户,因为其表示浏览器的可用屏幕（不包括系统区域）,比如mac系统的toolbar是不计算的.通过这个可以潜在的标识一些信息.

## Canvas指纹技术和WEBGL指纹技术
　　随着HTML5的更新,新的API可以用来获取指纹信息.
### Canvas指纹
　　HTML5中的canvas元素允许脚本进行2D形状和文本的渲染。通过这种方式,我们可以让一个程序输出打印图案两次，使用不同文字和颜色.来观察他们的区别.

#### 字体探测
　　告诉浏览器渲染同一个字符串（一个字符串包含所有字母）两次。 对于第一次强制浏览器使用它的一个备用字体。 根据设备上安装的操作系统和字体，备用字体不同。 对于第二次浏览器被要求使用常见的Arial字体,通过这种方式,我们可以获得其是哪一种操作系统.

#### 设备和操作系统指纹
　　例如emoji表情的笑脸在不同系统版本显示不一样,如下图所示,因此通过emoji表情可以判断操作系统甚至是版本信息.
![PS-PNG-work](//nanshihui.github.io/public/emoji.png)

#### 硬件操作系统指纹
　　打印字体,观察渲染图像的过程，会有像素点的偏差.虽然此字体具有相同的尺寸,但是在操作系统中,最终图像由于渲染过程的差异,存在像素可见的变化。渲染图像的过程是复杂的,并且依赖硬件和软件（例如GPU，渲染引擎，图形驱动程序，抗锯齿，OS），并且这些层中的任何层中的变化都会使得测试受到影响。有趣的是，测试结果也是相对稳定的，因为在渲染过程中,用户不经常改变图层配置。

### WebGL 指纹
　　通过WEBGL_debug_renderer_info 接口，获得产品名和供应商名
### 其他指纹
　　Platform(UA),Do Not Track & Ad blocker


## 新浏览器指纹技术
### CPU虚拟核心数
　　新的浏览器特性具有```hardwareConcurrency```函数,即使不支持,我们也可以通过侧信道攻击,也能发现,比如可以在增加worker的数量,观察有效载荷的完成时间。当完成时间随着worker的数量级显著增加时，说明达到硬件并发的限制，使得可以预估CPU核心数量。 当然值得注意的是，某些浏览器（例如Safari）会将Web Workers的可用核心数减少一半，那么我们需要将浏览器指纹的worker数量加倍.

### 音频上下文
　　音频上下文借助OS和音频卡中的音频堆栈，为音频信号产生到信号滤波,提供处理功能。具体来说，现有的指纹识别工作使用振荡器节点生成三角波，然后将波馈入产生压缩效果的信号处理模块，该节点可以抑制较大的声音以及放大微小的声音。 然后，经处理的音频信号通过分析节点转换为频域。
　　这种波形在同一台机不同浏览器是不一样的.但是对于跨浏览器,其峰值和它们对应的频率相对稳定.通过映射关系可以将这个作为跨浏览器的特征.其中作为特征的有采样率，最大通道数，输入数，输出数，通道数，通道计数模式和通道解释。
　　此处推荐篇论文[https://nanshihui.github.io/public/crossbrowsertracking_NDSS17.pdf](https://nanshihui.github.io/public/crossbrowsertracking_NDSS17.pdf),其指纹模型目前能够达到99%的识别率.
### 字体检测
　　以往检测字体,是通过falsh达到的,这里使用侧信道攻击,通过测量某个字符串的宽度和高度以确定字体
类型.

### 驻留技术
　　通过各种属性生成的指纹需要长期保存,防止被用户删除,或者可恢复,这就需要驻留技术,驻留技术比较成熟的是evercookie.其存储cookie的思路从以下几个方面

* 标准HTTP Cookie
* Flash本地共享对象
* Silverlight隔离存储
* CSS历史
* 将cookie存储在HTTP ETag中（需要后端服务器）
* 将cookie存储在Web缓存中（需要后端服务器）
* window.name缓存
* Internet Explorer userData存储
* HTML5会话存储
* HTML5本地存储
* HTML5全球存储
* 通过SQLite的HTML5数据库存储
* HTML5画布 - 将Cookie值存储在自动生成的RGB数据中,强制缓存PNG图像（需要后端服务器）
* HTML5 IndexedDB
* Java JNLP 持久化服务
* Java漏洞利用CVE-2013-0422 - 尝试将applet沙箱转储并将cookie数据直接写入用户的硬盘驱动器。
　　以上维度,不是所有浏览器都是支持,但是evercookie提供一个很好的支持.

## 总结
　　对于浏览器指纹,攻与防在不断的转换,目前浏览器指纹也不能绝对的标识一台主机,如果用户切换显卡或者双系统,虚拟机这些因素,那么目前的浏览器指纹就无法唯一标识了.未来随着新的HTML5技术不断更新,新的浏览器技术会提供更多的API.以及通过侧信道技术,在浏览器指纹会有新的突破.