---
title: python 中常见的 import 模块的方法(总结)
date: 2016-01-18 21:23:40
categories:
- Python
tag:
- import
- Python
---
<h2 id="intro">前言</h2>在用python进行编程时，经常会使用第三方模块包。这种包我们可以通过 python setup install  进行安装后，通过 import XXX或from XXX import yyy 进行导入。不过如果是自己遍写的依赖包，又不想安装到 python 的相应目录，可以放到本目录里进行import进行调用；为了更清晰的理清程序之间的关系，例如我们会把这种包放到lib目录再调用。
<!-- more -->
##  同级目录下的调用

　　程序结构如下：

``` bash
-- src
    |-- mod1.py
    |-- test1.py
```
　　若在程序test1.py中导入模块mod1, 则直接使用

``` bash
import mod1
或
from mod1 import *;y
```

##  调用子目录下的模块

　　程序结构如下：

``` bash
-- src
    |-- mod1.py
    |-- lib
    |    |-- mod2.py
    |-- test1.py
```
　　这时看到test1.py和lib目录（即mod2.py的父级目录），如果想在程序test1.py中导入模块mod2.py ，可以在lib件夹中建立空文件__init__.py文件(也可以在该文件中自定义输出模块接口)，然后使用：

``` bash
from lib.mod2 import *
或
import lib.mod2.
```

## 调用上级目录下的文件
　　程序结构如下：

``` bash
-- src
    |-- mod1.py
    |-- lib
    |    |-- mod2.py
    |-- sub
    |    |-- test2.py
```
　　这里想要实现test2.py调用mod1.py和mod2.py ，做法是我们先跳到src目录下面，直接可以调用mod1，然后在lib上当下建一个空文件__init__.py ，就可以像第二步调用子目录下的模块一样，通过import lib.mod2进行调用了。具体代码如下：

``` bash
import sys
sys.path.append("..")
import mod1
import lib.mod2
```
　　这些细碎的知识很琐碎，有时候到很后面的时候，会突然发现，前面有些只是都会忘了。因此就把这些记下来，方便查阅。
