---
title: Scan-T简介
date: 2016-01-21 09:25:37
categories:
- Web安全
tag:
- 爬虫
- Zmap
- Nmap
- Python
---
<h2 id="intro">前言</h2>看着自己编写的代码没有说明的确不方便别人开发，便写了部分说明在这边。简单的记录该项目的相关说明。
### 关于Scan-T项目的说明
- **文件分布:**详细文件分类
- **功能使用:**大致的功能描述
- **效果展示:** 初步的界面效果
- **TODO:** 进一步需要做的事情
- **其他:** 补充的地方
<!-- more -->

## 简介
　　Scan-T是一个网络空间指纹扫描工具，用于记录主机上的端口信息。以分布的方式，方便数据的抓取以及呈现，未来将会添加统计以及总结这些数据潜在的意义（暂时思路比较模糊）。有愿意指点我的，可以提供一点思路，谢谢。

## 整体架构
　　Scan-T的整体架构为nginx+Django+uwsgi,主体语言为python,抓取信息时,使用线程,进程,协程相互配合方式,检索数据使用ES配合redis,提高检索效率.

### 项目地址
　　[https://github.com/nanshihui/Scan-T](https://github.com/nanshihui/Scan-T)
## 效果展示


### 首页
![PS-PNG-main](//nanshihui.github.io/public/main.png)
### 检索结果
![PS-PNG-result](//nanshihui.github.io/public/result.png)
### 后台
![PS-PNG-work](//nanshihui.github.io/public/work.png)
### 统计分析页面
![PS-PNG-analyse](//nanshihui.github.io/public/analyse.png)
### 主机位置明细页面
![PS-PNG-analyse](//nanshihui.github.io/public/mapshow.png)
<!--### 移动端-->
<!--<div>-->
<!--<img src="http://nanshihui.github.io/public/phone.png" width = "268" height = "480" alt="图片名称" align="center" />-->
<!--</div>-->
## 整体思路
　　整个项目是一个分布式的形式。如果在单台机子上，就是一个WEB项目，如果在多台机上面，就是一个主从的运行模式，由主机从用户的输入以及定时产生的任务产生数据，自己在处理数据的同时，如果从机有响应任务请求。主机会将一部分的任务发送给从机。待从机执行完任务后，将结果反馈给主机。主机再将结果返回给用户以及数据库里。由于所有的消息都是异步的，所以在实现的时候，大部分都是异步，以任务的形式分发，执行。

## 安装方法


### Prepare the environment
``` bash
apt-get update 
apt-get install -y nmap
apt-get install -y libjson-c-dev libjson-c2  libjson0 libjson0-dev
apt-get install -y redis-server zmap libffi-dev libssl-dev python-pip libmysqlclient-dev
apt-get install -y wget unzip
apt-get install mysql-server
```
### Get the code
``` bash
wget https://github.com/nanshihui/Scan-T/archive/master.zip
unzip master.zip 
cd Scan-T-master/

```
### Install package
``` bash
pip install MySQL-python
pip install BeautifulSoup==3.2.1
pip install beautifulsoup4==4.4.1
pip install Django==1.9
pip install python-nmap==0.5.0.post1
pip install DBUtils==1.1
pip install paramiko==1.16.0
pip install ruamel.ordereddict==0.4.9
pip install scapy==2.3.2
pip install scapy-http==1.7
pip install objgraph==2.0.1
pip install pycrypto==2.6.1
pip install dozer
pip install faulthandler
pip install apscheduler

```
### Set the mysql config
* create table


``` bash
mysql -uroot -p -e 'create database datap'


```

* change password

change you sql password both in spidermanage/spidertool/config.py
toolforspider/spidermanage/spidermanage/settings.py

* import the data structure

``` bash
mysql -uroot -p datap -e 'source /toolforspider-master/spidermanage/sqldata/Dump20160331.sql'


```

* add your username

add your username and password in usertable
### Add css file
* PATH:/toolforspider-master/spidermanage/common_static/nmaptool/css/bootstrap/ 

create floder named lib and download css below 
``` bash
http://cine9deabril.com.br/fn/jquery/jquery-ui-1.10.2.custom/css/custom-theme/jquery-ui-1.10.2.custom.css
https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.6.3/css/font-awesome.css


```
### Add logfile

``` bash

mkdir spidermanage/lib/
touch spidermanage/lib/__init__.py


```
add the following code to spidermanage/lib/logger.py 

``` bash

#!/usr/bin/python
# encoding=utf-8


import logging

def initLog(logfile, level=2, verbose=False):
    '''
    日志记录函数，日志等级默认为2，即INFO级别的日志
    verbose - 是否屏幕输出，默认False
    '''
    logLevel = logging.INFO
    if level == 0:
        logLevel = logging.NOTSET
    elif level == 1:
        logLevel = logging.DEBUG
    elif level == 2:
        logLevel = logging.INFO
    elif level == 3:
        logLevel = logging.WARNING
    elif level == 4:
        logLevel = logging.ERROR
    elif level == 5:
        logLevel = logging.CRITICAL
    logger = logging.getLogger('pocdect')
    console = logging.FileHandler(logfile)
    formatter = logging.Formatter('[%(asctime)s]%(filename)s-%(process)d-%(thread)d-%(lineno)d-%(levelname)8s-"%(message)s"','%Y-%m-%d %a %H:%M:%S')
    console.setFormatter(formatter)
    logger.addHandler(console)
    logger.setLevel(logLevel)

    if verbose:
        console = logging.StreamHandler()
        console.setLevel(logLevel)  # always debug mode to screen
        formatter = logging.Formatter('%(name)-8s: %(levelname)-8s %(message)s')
        console.setFormatter(formatter)
        logger.addHandler(console)

    return logger



```
### Start
``` bash
python spidermanage/manage.py runserver 0.0.0.0:80 --insecure


```





## 使用方法
  软件不需要用户手动的创建任务便会在后台随机产生任务,并在互联网上抓取,也可以直接创建任务抓取信息.
　　暂时先这样，未完待续。。。