# PHP代码审计

## 一、代码审计环境

### 环境搭建

- https://www.xp.cn/  phpStudy  PHP集成环境,支持Windows与Linux系统
-  http://www.wampserver.com/ wampserver 法国人开发的Apache Web服务器、PHP解释器以及MySQL数据库的整合软件包。
- https://www.apachefriends.org XAMPP（Apache+MySQL+PHP+PERL）是一个功能强大的建站集成软件包。
- https://www.appserv.org/en/  AppServ是 PHP 网页架站工具组合包,作者将一些网络上免费的架站资源重新包装成单一的安装程序,以方便初学者快速完成架站。

### php.ini核心配置详解

- register_globals 全局变量开关，直接把用户GET、POST等方式提交上来的参数注册成全局变量并初始化值为参数对应的值。
- allow_url_include 包含远程文件
- magic_quotes_gqc 魔术引号自动过滤，在变量里的单引号、双引号、反斜杠及空字符的前面加上反斜杠。
- magic_quotes_runtime 魔术引号自动过滤。只对从数据库或者文件中获取的数据进行过滤。在php5.4.0之后移除了这个选项。
- magic_quotes_sybase 魔术引号自动过滤，与magic_quotes_gqc=on处理的对象一致。在php5.4.0之后移除了这个选项。
- safe_mode 安全模式，php5.4.0之后移除了这个选项。
- disable_functions 禁用函数。禁止一些敏感函数的使用。需要禁用dl()函数，否则攻击者可以利用dl()函数加载自定义的PHP扩展突破。
- display_errors和error_reporting 错误显示， 显示PHP脚本内部错误的选项。

## 二、审计辅助与漏洞验证工具

### 编辑器与文本工具

- https://notepad-plus-plus.org/ Notepad++ 支持多种语言高亮显示。
- https://www.ultraedit.com/ ultraedit编辑文本工具，还支持十六进制查看以及编辑。文件比对功能快速找补丁部分修改了哪段代码。
- https://www.zend.com/en/products/studio Zend Studio，除了有强大的PHP开发支持外也支持HTML、js、CSS，但只对PHP语言提供调试支持。

### 代码审计工具

- Seay源代码审计系统
- Fortify SCA
- RIPS

### 漏洞验证辅助

- Burpsuite
- 浏览器扩展

```
- Firefox

HackerBar
FireBug
Live HTTP Headers
Modify
Tamper Data

- chrome

Http Headers
EditThisCookie
Modheader
```

- 超级加解密转换工具
- SQLMAP

### 正则调试工具

- 灵者正则调试

### SQL执行监控工具

- MySQL，general_log记录MySQL历史执行语句。

```
- 直接记录到general_log表

set global general_log =on;
SET GLOBAL log_ouput='table';

- MySQL配置，在[mysqld]配置中加入如下代码：

general_log=ON
general_log_file={日志路劲}/query.log
```

- SQL server 事件跟踪

## 三、通用代码审计思路

- 1、根据敏感关键字回溯参数传递过程
- 2、查找可控变量，正向追踪变量传递过程
- 3、寻找敏感功能点，通读功能点代码
- 4、通读全文代码

### 敏感关键字回溯参数

- SQL注入：select、insert结合from、where关键字，关注HTTP头里面的HTTP_CLIENT_IP和HTTP_X_FORWORDFOR等获取IP地址直接写入SQL语句中，没过滤的情况。

### 通读全文代码

通读全文代码要了解业务逻辑，知道哪部分是关键功能、核心文件是哪些。

- 函数集文件：命名中包含functions或者common等关键字，这些文件里面是公共的函数，提供给其他文件统一调用。
- 配置文件：通常命名中包括config关键字，包含有web程序运行时的功能性配置以及数据库配置信息。
- 安全过滤文件：这类文件主要是对参数进行过滤，命名中有filter、safe、check等关键字，这类文件主要是对参数进行过滤，比较常见针对SQL注入和XSS、文件路径、执行的系统命令参数使用addslashes()函数进行过滤。
- index文件：index是一个程序的入口文件，核心文件里的index有不同的实现方式。每个不同的函数都有特定含义。

### 根据功能点定向审计

- 文件上传功能：文章编辑、资料编辑、头像上传、附件上传、常见于任意文件上传。还经常发生SQL注入漏洞，没有过滤文件名但又需要把文件名保存到数据库中。
- 文件管理功能：程序把文件名或者文件路劲直接在参数中传递，容易出现任意文件操作的漏洞。如任意文件读取、任意文件下载、跳转目录
- 登录认证功能：认证方式大多数基于cookie和session，把当前登录的用户帐户等认证信息放到cookie中。不会在退出浏览器或者session超时就退出帐户。如果没有salt一类的加密，容易导致任意用户登录漏洞。
- 找回密码功能：验证码大多是4位，没有限制验证错误次数和有效时间，容易出现爆破漏洞。

## 四、漏洞挖掘

### SQL注入漏洞

- 登录页面注入：HTTP头里面的client-ip和x-forward-for，一般用来记录登录的IP地址。
- 普通注入：白盒审计关键字select from、update、insert、delete、mysql_connect、mysql_query、mysql_fetch_row。
- 编码注入
- - 宽字节注入 GBK数据编码转换的注入问题。  关键字：SET NAMES 、character_set_client=gbk、mysql_set_charset('gbk')
- - 二次urldecode注入，urldecode使用不当引起的，挖掘二次urldecode注入漏洞关键字:urldecode、rawurldecode

#### 解决方案

1、设置编码 GBK，然后使用mysql_real_escape_string()

2、pdo方式

```
<?php
    $dbh = new PDO("mysql:host=localhost;dbname=demo","user","pass");
    $dbh->setAttribute(PDO::ATTR_EMULATE_PREPARES,false);
    $dbh->exec("set names 'utf8'");
    $sql="select * from test where name = ? and password =?";
    $stmt = $dbh->prepare($sql);
    $exeres = $stmt->execute(array($name,$pass));
?>
```

3、magic_quotes_gpc 魔术引号

4、过滤函数 addslashes、mysql_real_escape_string函数

5、intval字符转换

### XSS

- 文章发表
- 评论回复
- 留言
- 资料设置
- 图片音乐
- 文字格式设置
- 标签过滤不严
- 用户昵称
- 用户签名
- 注册
- 修改资料

### 反射性XSS

### CSRF

### 文件包含

### 文件读取

### 文件上传

### 文件删除

### 代码执行

### 命令执行

### 变量覆盖

### 逻辑处理

### 会话认证

## 五、二次漏洞

## 六、代码审计技巧

## 七、安全编程规范

## 八、应用安全体系建设



## PHP代码审计书籍

- [Online-CodeBook](https://phpaudit.books.virzz.com/) - PHP代码审计
- [Online-CodeBook](https://wizardforcel.gitbooks.io/php-common-vulnerability/content/) - 论PHP常见的漏洞
- 代码审计:企业级Web代码安全架构
- PHP Web安全开发实战

## 远程执行函数

- `passthru()` 允许执行一个外部程序并回显输出，类似于 exec()。
- `exec()`  允许执行一个外部程序（如 UNIX Shell 或 CMD 命令等）。
- `system()`  允许执行一个外部程序并回显输出，类似于 passthru()。
- `shell_exec()`通过 Shell 执行命令，并将执行结果作为字符串返回。
- `assert()` 如果按照默认值来，在程序的运行过程中调用assert()来进行判断表达式，遇到false时程序也是会继续执行的，跟eval()类似，不过eval($code_str)只是执行符合php编码规范的$code_str。assert的用法却更详细一点。

## 输入位置

### 显性输入

```
$_SERVER
$_GET
$_POST
$_COOKIE
$_REQUEST
$_FILES
$_ENV
```

### 隐式输入

用户的输入可能先进入了数据库,然后程序从数据库再取得这个输入送入某些危险的函数执行。

### 变量覆盖

```
extract()
parse_str()
$$
```

寻找利用环境

- 表前缀
- Session
- 系统配置变量`$_config[]`
- 未初始化变量带入的 sql 语句



## 

## 审计技巧

- [利用PHP应用程序中的远程文件包含（RFI）并绕过远程URL包含限制](https://www.cnblogs.com/17bdw/p/10987338.html)

## 代码审计案例

- https://github.com/CHYbeta/Code-Audit-Challenges 代码审计练习题

- https://github.com/Xyntax/1000php  1000个PHP代码审计案例(2016.7以前乌云公开漏洞)

