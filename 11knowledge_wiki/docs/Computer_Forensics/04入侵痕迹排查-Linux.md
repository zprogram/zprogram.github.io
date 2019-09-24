## 入侵痕迹排查-Linux



### 1、文件排查

敏感目录的文件分析[类/tmp目录，命令目录/usr/bin /usr/sbin等]，按时间排序查看。

```
echo $PATH           # 查看系统环境变量
ls -rtal /bin
ls -rtal /sbin
ls -rtal /usr/bin/
ls -rtal /usr/sbin/
ls -rtal /tmp
ls -rtal /etc/init.d/
```

可疑文件用stat查看创建、修改、访问时间的查看。

#### 按时间搜索文件

Linux按日期查找文件并列出在特定日期修改的文件。

- 在/data/images目录中查找在2007年1月1日到2008年1月1日之间修改的文件：

```

touch --date "2007-01-01" /tmp/start
touch --date "2008-01-01" /tmp/end
find /data/images -type f -newer /tmp/start -not -newer /tmp/end
```

- find命令的-newerXY选项

```
a - 文件的访问时间
B - 文件的创建时间
c - inode状态改变时间
m - 文件的修改时间
t - 引用被直接解释为时间

find /dir/ -type f -newerXY 'yyyy-mm-dd'
find /dir/ -type f -newerXY 'yyyy-mm-dd' -ls

## 查看当前目录2017年9月24日修改的所有文件 
find . -type f -newermt 2017-09-24
## 显示属性
find . -type f -newermt 2017-09-24 -ls
## 时间查找
find /path/to/dir -newermt "yyyy-mm-dd HH:mm:ss" -not -newermt "yyyy-mm-dd HH:mm:ss+1"
## 按日期范围查找
find / -newermt "2019-09-09 00:00:00" -not -newermt "2019-09-10 00:00:00+1"
```

#### 特殊权限文件查看

- 查找777权限的文件

```
find . -name *.jsp -perm 777
```

- 查找隐藏文件

隐藏的文件（以 "."开头的具有隐藏属性的文件）

ls -a可以将目录下的全部文件（包括隐藏文件）显示出来
ls -r将排序结果反向输出，例如：原本文件名由小到大，反向则由大到小
ls -R连同子目录一同显示出来，也就所说该目录下所有文件都会显示出来（显示隐藏文件要加-a参数）
简单的介绍就到这了，我们放码说话。

```
# ls -arR | grep "^\."
```

### 2、账号安全

- 基本知识

```
1、用户信息文件/etc/passwd
root:x:0:0:root:/root:/bin/bash
account:password:UID:GID:GECOS:directory:shell
用户名：密码：用户ID：组ID：用户说明：家目录：登陆之后shell
注意：无密码只允许本机登陆，远程不允许登陆

2、影子文件/etc/shadow
root:$6$oGs1PqhL2p3ZetrE$X7o7bzoouHQVSEmSgsYN5UD4.kMHx6qgbTqwNVC5oOAouXvcjQSt.Ft7ql1WpkopY0UV9ajBwUt1DpYxTCVvI/:16809:0:99999:7:::
用户名：加密密码：密码最后一次修改日期：两次密码的修改时间间隔：密码有效期：密码修改到期到的警告天数：密码过期之后的宽限天数：账号失效时间：保留


who     查看当前登录用户（tty本地登陆  pts远程登录）
w       查看系统信息，想知道某一时刻用户的行为
uptime  查看登陆多久、多少用户，负载
```

- 入侵排查

```
1、查询特权用户特权用户(uid 为0)
[root@localhost ~]# awk -F: '$3==0{print $1}' /etc/passwd
2、查询可以远程登录的帐号信息
[root@localhost ~]# awk '/\$1|\$6/{print $1}' /etc/shadow
3、除root帐号外，其他帐号是否存在sudo权限。如非管理需要，普通帐号应删除sudo权限
[root@localhost ~]# more /etc/sudoers | grep -v "^#\|^$" | grep "ALL=(ALL)"
4、禁用或删除多余及可疑的帐号
    usermod -L user    禁用帐号，帐号无法登录，/etc/shadow第二栏为!开头
    userdel user       删除user用户
    userdel -r user    将删除user用户，并且将/home目录下的user目录一并删除
```

### 3、历史命令

- 基础知识

```
通过.bash_history查看帐号执行过的系统命令
1、root的历史命令

histroy
cat ~/.bash_history

2、打开/home各帐号目录下的.bash_history，查看普通帐号的历史命令

为历史的命令增加登录的IP地址、执行命令时间等信息：
1）保存1万条命令

sed -i 's/^HISTSIZE=1000/HISTSIZE=10000/g' /etc/profile

2）在/etc/profile的文件尾部添加如下行数配置信息：

###### 加固历史命令显示 #########
USER_IP=`who -u am i 2>/dev/null | awk '{print $NF}' | sed -e 's/[()]//g'`
if [ "$USER_IP" = "" ]
then
USER_IP=`hostname`
fi
export HISTTIMEFORMAT="%F %T $USER_IP `whoami` "
shopt -s histappend
export PROMPT_COMMAND="history -a"
###### 加固历史命令显示 #########

3）source /etc/profile让配置生效

生成效果： 1  2018-07-10 19:45:39 192.168.204.1 root source /etc/profile

3、历史操作命令的清除：history -c
但此命令并不会清除保存在文件中的记录，因此需要手动删除.bash_profile文件中的记录。

4、当前会话历史操作命令的清除：history -r
```

- 入侵排查：

```
进入用户目录下
cat .bash_history >> history.txt
```

### 4、网络连接排查

#### 排查端口

使用netstat 网络连接命令，分析可疑端口、IP、PID

```
netstat -antlp|more

查看下pid所对应的进程文件路径，
运行ls -l /proc/$PID/exe或file /proc/$PID/exe（$PID 为对应的pid 号）
```

#### 排查进程

使用ps命令，分析进程

```
ps aux | grep pid  # 排查指定PID
ls /proc/{$pid}/   # 找进程路径
```

### 5、开机启动项

- 基础知识：

系统运行级别示意图：

| 运行级别 | 含义                                                      |
| -------- | --------------------------------------------------------- |
| 0        | 关机                                                      |
| 1        | 单用户模式，可以想象为windows的安全模式，主要用于系统修复 |
| 2        | 不完全的命令行模式，不含NFS服务                           |
| 3        | 完全的命令行模式，就是标准字符界面                        |
| 4        | 系统保留                                                  |
| 5        | 图形模式                                                  |
| 6        | 重启动                                                    |

查看运行级别命令 runlevel

系统默认允许级别

```
vi  /etc/inittab
id=3：initdefault  系统开机后直接进入哪个运行级别
```

开机启动配置文件

```
/etc/rc.local
/etc/rc.d/rc[0~6].d
```

例子:当我们需要开机启动自己的脚本时，只需要将可执行脚本丢在/etc/init.d目录下，然后在/etc/rc.d/rc*.d中建立软链接即可

```
root@localhost ~]# ln -s /etc/init.d/sshd /etc/rc.d/rc3.d/S100ssh
```

此处sshd是具体服务的脚本文件，S100ssh是其软链接，S开头代表加载时自启动；如果是K开头的脚本文件，代表运行级别加载时需要关闭的。

- 入侵排查

```
启动项文件： more /etc/rc.local /etc/rc.d/rc[0~6].d ls -l /etc/rc.d/rc3.d/
```

### 6、计划任务

- 基础知识

```
1、利用crontab创建计划任务

## 列出某个用户cron服务的详细内容，默认编写的crontab文件会保存在 (/var/spool/cron/用户名 例如: /var/spool/cron/root

crontab -l      # 列出某个用户cron服务的详细内容

ls /etc/cron*   # 查看etc目录任务计划相关文件

##  删除每个用户cront任务

crontab -r

2、利用anacron实现异步定时任务调度

anacron 任务被列在 /etc/anacrontab 中，任务可以使用下面的格式（anacron 文件中的注释必须以 # 号开始）安排。

period   delay   job-identifier   command

从上面的格式中：

- period - 这是任务的频率，以天来指定，或者是@daily 、@weekly、@monthly 代表每天、每周、每月一次。你也可以使用数字：1 - 每天、7 - 每周、30- 每月，或者N - 几天。
- delay - 这是在执行一个任务前等待的分钟数。
- job-id - 这是写在日志文件中任务的独特名字。
- command - 这是要执行的命令或 shell 脚本。

## 例子 

每天运行 /home/backup.sh脚本

 vi /etc/anacrontab 
 @daily 10 /bin/bash /home/backup.sh

```

- 入侵排查

```
cat /var/spool/cron/* 
cat /etc/crontab
ls -rtal /etc/cron.d/*
ls -rtal /etc/cron.daily/* 
cat /etc/cron.hourly/* 
cat /etc/cron.monthly/*
cat /etc/cron.weekly/*
cat /etc/anacrontab
cat /var/spool/anacron/*

 more /etc/cron.daily/*  查看目录下所有文件
```

### 7、服务

- 基础知识

```
# 第一种修改方法：

chkconfig [--level 运行级别] [独立服务名] [on|off]
chkconfig –level  2345 httpd on  开启自启动
chkconfig httpd on （默认level是2345）

第二种修改方法：

修改/etc/re.d/rc.local 文件  
加入 /etc/init.d/httpd start

第三种修改方法：

使用ntsysv命令管理自启动，可以管理独立服务和xinetd服务。
```
- 入侵排查

```
1、查询已安装的服务-RPM包安装的服务：

chkconfig  --list  查看服务自启动状态，可以看到所有的RPM包安装的服务
ps aux | grep crond 查看当前服务

系统在3与5级别下的启动项 
中文环境
chkconfig --list | grep "3:启用\|5:启用"
英文环境
chkconfig --list | grep "3:on\|5:on"

2、源码包安装的服务

查看服务安装位置 ，一般是在/user/local/
service httpd start
搜索/etc/rc.d/init.d/  查看是否存在
```

### 8、日志分析

#### 系统日志

日志默认存放位置：/var/log/

查看日志配置情况：more /etc/rsyslog.conf

| 日志文件         | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| /var/log/cron    | 记录了系统定时任务相关的日志                                 |
| /var/log/cups    | 记录打印信息的日志                                           |
| /var/log/dmesg   | 记录了系统在开机时内核自检的信息，也可以使用dmesg命令直接查看内核自检信息 |
| /var/log/mailog  | 记录邮件信息                                                 |
| /var/log/message | 记录系统重要信息的日志。这个日志文件中会记录Linux系统的绝大多数重要信息，如果系统出现问题时，首先要检查的就应该是这个日志文件 |
| /var/log/btmp    | 记录错误登录日志，这个文件是二进制文件，不能直接vi查看，而要使用lastb命令查看 |
| /var/log/lastlog | 记录系统中所有用户最后一次登录时间的日志，这个文件是二进制文件，不能直接vi，而要使用lastlog命令查看 |
| /var/log/wtmp    | 永久记录所有用户的登录、注销信息，同时记录系统的启动、重启、关机事件。同样这个文件也是一个二进制文件，不能直接vi，而需要使用last命令来查看 |
| /var/log/utmp    | 记录当前已经登录的用户信息，这个文件会随着用户的登录和注销不断变化，只记录当前登录用户的信息。同样这个文件不能直接vi，而要使用w,who,users等命令来查询 |
| /var/log/secure  | 记录验证和授权方面的信息，只要涉及账号和密码的程序都会记录，比如SSH登录，su切换用户，sudo授权，甚至添加用户和修改用户密码都会记录在这个日志文件中 |

- /var/log/wtmp

```
last
last -f /var/log/wtmp
```

- /var/log/utmp

```
last -f /var/log/utmp
who
```

- /var/log/lastlog（lastlog）  最后一次登录时间的日志

- /var/log/btmp（lastb）

  

- /vat/log/secure 日志分析技巧（结合shell）：

```

1、定位有多少IP在爆破主机的root帐号：    
grep "Failed password for root" /var/log/secure | awk '{print $11}' | sort | uniq -c | sort -nr | more

定位有哪些IP在爆破：
grep "Failed password" /var/log/secure|grep -E -o "(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)"|uniq -c

爆破用户名字典是什么？
 grep "Failed password" /var/log/secure|perl -e 'while($_=<>){ /for(.*?) from/; print "$1\n";}'|uniq -c|sort -nr

2、登录成功的IP有哪些：   
grep "Accepted " /var/log/secure | awk '{print $11}' | sort | uniq -c | sort -nr | more

登录成功的日期、用户名、IP：
grep "Accepted " /var/log/secure | awk '{print $1,$2,$3,$9,$11}' 

3、增加一个用户kali日志：
Jul 10 00:12:15 localhost useradd[2382]: new group: name=kali, GID=1001
Jul 10 00:12:15 localhost useradd[2382]: new user: name=kali, UID=1001, GID=1001, home=/home/kali
, shell=/bin/bash
Jul 10 00:12:58 localhost passwd: pam_unix(passwd:chauthtok): password changed for kali
#grep "useradd" /var/log/secure 

4、删除用户kali日志：
Jul 10 00:14:17 localhost userdel[2393]: delete user 'kali'
Jul 10 00:14:17 localhost userdel[2393]: removed group 'kali' owned by 'kali'
Jul 10 00:14:17 localhost userdel[2393]: removed shadow group 'kali' owned by 'kali'
# grep "userdel" /var/log/secure

5、su切换用户：
Jul 10 00:38:13 localhost su: pam_unix(su-l:session): session opened for user good by root(uid=0)

sudo授权执行:
sudo -l
Jul 10 00:43:09 localhost sudo:    good : TTY=pts/4 ; PWD=/home/good ; USER=root ; COMMAND=/sbin/shutdown -r now
```

#### apache日志

apache默认生成两个日志文件，access_log、error_log

- access_log

查询访问量前十的IP地址：

```
cat access_log |cut -d ' ' -f 1 |sort |uniq -c | sort -nr | awk '{print $0 }' | head -n 10 |less
```

### 9、查杀工具

- 1) Rootkit查杀

chkrootkit

网址：http://www.chkrootkit.org

```
使用方法：
wget ftp://ftp.pangeia.com.br/pub/seg/pac/chkrootkit.tar.gz
tar zxvf chkrootkit.tar.gz
cd chkrootkit-0.52
make sense
#编译完成没有报错的话执行检查
./chkrootkit
```
rkhunter

网址：http://rkhunter.sourceforge.net

```
使用方法：
Wget https://nchc.dl.sourceforge.net/project/rkhunter/rkhunter/1.4.4/rkhunter-1.4.4.tar.gz
tar -zxvf rkhunter-1.4.4.tar.gz
cd rkhunter-1.4.4
./installer.sh --install
rkhunter -c
```

- 2) 病毒查杀

- Clamav

ClamAV的官方下载地址为：<http://www.clamav.net/download.html>

安装方式一：
```
1、安装zlib：
wget http://nchc.dl.sourceforge.net/project/libpng/zlib/1.2.7/zlib-1.2.7.tar.gz 
tar -zxvf  zlib-1.2.7.tar.gz
cd zlib-1.2.7
#安装一下gcc编译环境： yum install gcc
CFLAGS="-O3 -fPIC" ./configure --prefix= /usr/local/zlib/
make && make install

2、添加用户组clamav和组成员clamav：
groupadd clamav
useradd -g clamav -s /bin/false -c "Clam AntiVirus" clamav

3、安装Clamav
tar –zxvf clamav-0.97.6.tar.gz
cd clamav-0.97.6
./configure --prefix=/opt/clamav --disable-clamav -with-zlib=/usr/local/zlib
make
make install

4、配置Clamav
mkdir /opt/clamav/logs
mkdir /opt/clamav/updata
touch /opt/clamav/logs/freshclam.log
touch /opt/clamav/logs/clamd.log
cd /opt/clamav/logs
chown clamav:clamav clamd.log
chown clamav:clamav freshclam.log

5、ClamAV 使用：
 /opt/clamav/bin/freshclam 升级病毒库
./clamscan –h 查看相应的帮助信息
./clamscan -r /home  扫描所有用户的主目录就使用
./clamscan -r --bell -i /bin  扫描bin目录并且显示有问题的文件的扫描结果
```
安装方式二：
```
# 安装
yum install -y clamav
# 更新病毒库
freshclam
# 扫描方法
clamscan -r /etc --max-dir-recursion=5 -l /root/etcclamav.log
clamscan -r /bin --max-dir-recursion=5 -l /root/binclamav.log
clamscan -r /usr --max-dir-recursion=5 -l /root/usrclamav.log
# 扫描并杀毒
clamscan -r  --remove  /usr/bin/bsd-port
clamscan -r  --remove  /usr/bin/
clamscan -r --remove  /usr/local/zabbix/sbin
# 查看日志发现
cat /root/usrclamav.log |grep FOUND
```
- 3) WebShell查杀

```
河马webshell查杀：
http://www.shellpub.com

深信服Webshell网站后门检测工具：http://edr.sangfor.com.cn/backdoor_detection.html
```


### 10、自动化辅助脚本

```
# # 查看当前登录用户
who > who.txt
# # 显示目前登入系统的用户信息
w > w.txt
# # 显示时间
date > date.txt
# # 查看CPU信息
cat /proc/cpuinfo > cpuinfo.txt
# # 查询系统版本
lsb_release -a > lsb_release.txt
# # 当前系统相关信息(内核版本号、硬件架构、主机名称和操作系统类型等)
uname -a > uname.txt
# # Linux查看当前操作系统版本信息
cat /proc/version >  version.txt
# # 以批处理模式显示进程信息，更新1次后不再更新
top -b -n 1 > top.txt
# # 查看系统负载
uptime > uptime.txt
# # MB显示当前内存的使用
free -m > free_m.txt
# # 文件系统的磁盘空间占用情况
df -lhT > df_lhT.txt
# #  显示分区类型
fdisk -l > fdisk_l.txt
# # 挂载设备情况
mount > mount.txt
# # 显示系统中已存在的环境变量
env > env.txt
# # 查看自定义环境变量
cat ~/.bashrc > bashrc.txt
# # 读取内核信息
cat /proc/meminfo > meminfo.txt

#account check  7

# # 系统用户信息
cat /etc/passwd > etc_passwd.txt
# # 密文信息
cat /etc/shadow > etc_shadow.txt
# # 查看用户文件状态
stat /etc/passwd > etc_passwd_stat.txt
stat /etc/shadow > etc_shadow_stat.txt
# # 查看特权用户
awk -F: '$3==0 {print $1}' /etc/passwd > etc_passwd_special_usr.txt
grep “0” /etc/passwd > etc_passwd_new_user.txt
awk -F: 'length($2)==0 {print $1}' /etc/shadow > etc_shadow_no_password_user.txt

#process check  4
# # 全格式显示所有进程
ps -elf > ps_elf.txt
# # 显示所有进程，包括其他用户 
ps aux > ps_aux.txt
ps -ef | grep inted  >  ps_inted.txt
ls /proc |sort -n|uniq > proc.txt

#file check 11
# # 根据uid、执行权限来查找
find / -uid 0 -perm -4000 > uid0_perm4000.txt
# # 根据文件大小
find / -size +10000k > size10000.txt
find / -name "..." > 3point_name_file.txt
find / -name ".. " > 2point_name_file.txt
find / -name ". " > 1point_name_file.txt
find / -name " " > blankspace_name_file.txt
# # 查看隐藏文件
find / -name ".*" > hide_file.txt
find / -name "*" > all_file.txt
find / -name ".rhosts" > rhosts.txt
find / -name ".forward" > forward.txt
# # 列出当前系统打开文件
lsof > lsof.txt

#integrity check 5
# # 查询指定文件来自于哪个安装包
rpm -qf /bin/ls > rpm_ls.txt
rpm -Vf /bin/ls >> rpm_ls.txt
rpm -qf /bin/netstat > rpm_netstat.txt
rpm -Vf /bin/netstat >> rpm_netstat.txt
rpm -qf /bin/login > rpm_login.txt
rpm -Vf /bin/login >> rpm_login.txt
rpm -qf /bin/find > rpm_find.txt
rpm -Vf /bin/find >> rpm_find.txt
rpm -qf /usr/bin/top > rpm_top.txt
rpm -Vf /usr/bin/top >> rpm_top.txt

#network check 6
# # 查看路由表条目
ip link | grep PROMISC > ip_promisc.txt
# # 显示所有联网文件
lsof -i > lsof_i.txt
# # 显示TCP、UDP传输协议、Socket程序名称
netstat -ntulpa >  netstat_ntulpa.txt
# # 显示正在使用Socket的程序
netstat -anpo > netstat_anpo.txt
# # 显示arp缓冲区的所有条目
arp -a > arp_a.txt
# # 显示全部接口信息
ifconfig -a > ifconfig_a.txt

#schedule check 5
# # 显示root的crontab文件内容
crontab -l -u root > root_crontab.txt
crontab -l -u coremail > coremail_crontab.txt
# # 计划任务
cat /etc/crontab > etc_crontab.txt
# # 列出计划任务的脚本
ls /etc/cron.* -a > etc_cron.txt
# # 查看定时任务
ls /var/spool/cron/ -a > var_spool_cron.txt

#rc check 4
# # 启动项顺序
cat /etc/rc.d/rc.local > rc_local.txt
# # 该目录下存在各个运行级别的脚本文件
ls /etc/rc.d -a > rc_d.txt
ls /etc/rc*.d -a > rcV_d.txt
# # 搜索执行权限4000的普通类型文件
find / -type f -perm 4000 > type_f_perm_4000.txt

#log check 11
# # 日志进程
ps -ef | grep syslog > syslog.txt
# # 列出日志目录
ls -al /var/log > var_log.txt
# # 列出日志目录状态
stat /var/log/wtmp > stat_wtmp.txt
stat /var/run/utmp > stat_utmp.txt
cat /var/run/utmp > utmp.txt
cat /etc/rsyslog.conf > rsyslog_conf.txt
cat /etc/init.d/rsyslog > rsyslog.txt
# # 列出登入系统失败的用户相关信息
lastb > lastb.txt
# # :列出目前与过去登入系统的用户相关信息
last > last.txt
# # Shell历史命令记录文件
cat ~/.bash_history > history.txt
ls -l ~/.bash_history > bash_history.txt

#inetd sheck 1
# # 扩展互联网服务守护进程配置
cat /etc/xinetd.conf > xinetd_config.txt

#kernel check 2
# # 加载的模块信息
lsmod > lsmod.txt
find / -name core -exec ls -l {} \; > core_file.txt

#service check 2
# # 查看开机启动服务
chkconfig --list > chkconfig_lists.txt
# # 查看本地rpc进程
rpcinfo -p > rpcinfo.txt

#files get 5
# # 打包守护进程文件
tar -zcvf xinetd.tar.gz /etc/xinetd.d/*
# # 打包日志文件
tar -zcvf log.tar.gz /var/log/*
# # 打包自启动脚本
tar -zcvf rcd.tar.gz /etc/rc.d/* 
# # 打包计划任务
tar -zcvf cron.tar.gz /etc/cron.*
tar -zcvf at.tar.gz /var/spool/at/* 
# # 清除当前会话命令
history -r
```

### 11、排查技巧

#### 利用/proc恢复删除的文件

 /proc目录是内核提供给我们的查询中心，通过查询该目录下的文件内容，可以获取到有关系统硬件及当前运行进程的信息。

- fd：fd目录包含了进程打开的每一个文件的文件描述符。这些描述符都指向实际文件：

在/proc/{$PID}/fd目录下找到该文件的文件句柄：在/proc/15922/fd目录下找到该文件的文件句柄：

```
root@kali:~# ls -rtal /proc/15922/fd
total 0
dr-xr-xr-x 9 root root  0 Sep  9 09:38 ..
dr-x------ 2 root root  0 Sep 10 04:13 .
lrwx------ 1 root root 64 Sep 10 04:13 4 -> '/root/VulApps/.git/objects/pack/tmp_pack_ETdZdJ (deleted)'
lrwx------ 1 root root 64 Sep 10 04:13 3 -> 'socket:[276743]'
lrwx------ 1 root root 64 Sep 10 04:13 2 -> /dev/pts/1
l-wx------ 1 root root 64 Sep 10 04:13 1 -> 'pipe:[275299]'
lr-x------ 1 root root 64 Sep 10 04:13 0 -> 'pipe:[275298]'

```

尝试恢复该文件，直接复制文件句柄、并修改文件权限即可：

```
root@kali:~# cp /proc/15922/fd/{$文件句柄序号} /opt/zz
root@kali:~# cp /proc/15922/fd/4 /opt/zz
root@kali:~# md5sum /proc/15922/fd/4
81218b19401d4c7775c7b0dca09da692  /proc/15922/fd/4
root@kali:~# md5sum /opt/zz
81218b19401d4c7775c7b0dca09da692  /opt/zz

```



#### RPM文件检查

 系统完整性可以通过rpm自带的-Va来校验检查所有的rpm软件包，查看哪些命令是否被替换了：

```
./rpm -Va > rpm.log
```

如果一切均校验正常将不会产生任何输出，如果有不一致的地方，就会显示出来，输出格式是8位长字符串，每个字符都用以表示文件与RPM数据库中一种属性的比较结果 ，如果是. (点) 则表示测试通过。

```
验证内容中的8个信息的具体内容如下：
        S         文件大小是否改变
        M         文件的类型或文件的权限（rwx）是否被改变
        5         文件MD5校验是否改变（可以看成文件内容是否改变）
        D         设备中，从代码是否改变
        L         文件路径是否改变
        U         文件的属主（所有者）是否改变
        G         文件的属组是否改变
        T         文件的修改时间是否改变
```

- 检查单个文件

```
# 检查文件所属安装包
[root@centos ~]# rpm  -qf /bin/ls 
coreutils-8.4-47.el6.x86_64
# 检查文件的匹配度
[root@centos ~]# rpm  -Vf /bin/ls 
S.5....T.    /bin/ls
```

如果命令被替换了，如果还原回来：

```
文件提取还原案例：
rpm  -qf /bin/ls  查询ls命令属于哪个软件包
mv  /bin/ls /tmp  先把ls转移到tmp目录下，造成ls命令丢失的假象
rpm2cpio /mnt/cdrom/Packages/coreutils-8.4-19.el6.i686.rpm | cpio -idv ./bin/ls 提取rpm包中ls命令到当前目录的/bin/ls下
cp /root/bin/ls  /bin/ 把ls命令复制到/bin/目录 修复文件丢失
```



### 参考：排查实战

https://www.t00ls.net/viewthread.php?tid=48068&extra=&page=1