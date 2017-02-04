---
title: Run Zmap in docker
date: 2017-01-05 13:47:03
tag:
- Zmap
- Docker
- Centos
categories:
- Web安全
keywords:
---
<h2 id="intro">前言</h2>最近尝试在Docker环境中安装ZMAP，出现一些问题
### 总体
- **情况描述1:**　出现 SIOCGIFADDR: Invalid argument
- **情况描述2:**　无法发包
<!-- more -->
## 问题描述

　　最近因为需要，遂在Docker环境的下Centos，安装ZMAP。安装的时候出现一系列组件没有完全。网上的在Centos里安装ZMAP的方法，已经无法使用。也不能使用
``` bash
　　yum install zmap
```
类似这样的方法，因此就只能自己乖乖的安装基本组件。

## 安装流程
　　相关的安装方法可以参考[ZMAP](https://github.com/zmap/zmap)官方的教程。在安装[ZMAP](https://github.com/zmap/zmap)之前还需要安装[JSON-C](https://github.com/json-c/json-c).相关的参考文档，在对应的git都有相应的教程。
### 需要的环境
* gcc
* libtool>=2.2.6b
* autoconf>=2.64 (autoreconf)
* automake>=1.10.3

　　在遇到这些库没有的时候，在Google直接搜索对应的组件以及版本，找到对应的RPM包进行安装。
　　在安装完JSON-C后，安装ZMAP会出现缺少JSON-C组件，这时候就需要自己添加路径。或者安装一个libjson的库。

## 运行问题
　　在运行ZMAP的时候，会出现如下错误:

``` bash
zmap:could not detect default network interface(e,g.eth0).Try running as root or setting interface using -i flag
```

　　这时候通过ifconfig查找对应interface name，然后在运行ZMAP运行的时候添加-i 参数，并附上interface name就行了。		
　　虽然能运行起来，但是会发现程序无法收发包，通过 tcpdump 抓包发现程序也无法发包，显示错误：
``` bash
SIOCGIFADDR: Invalid argument
```
通过gdb单步调试，确定报错的位置在send-linux.h文件的如下函数
``` bash
if (ioctl(sock, SIOCGIFADDR, &if_ip) < 0) {
		perror("SIOCGIFADDR");
		return EXIT_FAILURE;
	}

```


然后观察前后文，并没有发现```if_ip```被使用的情况
因此就注释了这一段，然后重新把源码编译一下，就能够运行了。

然后将问题反馈给ZMAP作者，并修复了这个问题。详细细节见[https://github.com/zmap/zmap/issues/279#issuecomment-270719911](https://github.com/zmap/zmap/issues/279#issuecomment-270719911)
