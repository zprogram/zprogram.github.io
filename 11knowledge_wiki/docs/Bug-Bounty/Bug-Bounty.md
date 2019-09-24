## 挖掘策略

- 大型厂商（微软、oracle、谷歌..）
- 固件提取类，二进制类
- 政企常用框架级别类

## 资格认证

- [OSCE漏洞认证考试](https://www.freebuf.com/news/206041.html)

## 安全会议

- Virus Bulletin

- ZeroNights

## 信息收集

### WEB常规信息收集思路

```
- 子域名爆破

- 第三方信息收集平台：
  zoomeye
  fofa
  dnsenumer

- IP反查域名

端口扫描获取IP:nmap
ns记录:nslookup

- 职场平台  LinkedIn
- 源代码平台 Github
- 云平台信息 AWS 
- 邮箱信息
```
### 开源项目

- 主动信息收集

搜索引擎：Google，百度，360，必应，搜狗，等主流搜索引擎，通过关键字来搜索相对于域名的信息。

```
https://github.com/bit4woo/teemo 域名与邮箱收集工具

https://github.com/ChrisTruncer/EyeWitness EyeWitness：可用于网站截图，以及提供一些服务器头信息，并在可能的情况下识别默认凭据。

https://github.com/jordanpotti/AWSBucketDump AWSBucketDump：AWS S3安全扫描工具，允许你快速枚举AWS S3 buckets以查找有趣或机密的文件。

https://github.com/michenriksen/aquatone AQUATONE：子域名枚举探测工具

https://github.com/BishopFox/spoofcheck spoofcheck：检查域是否可以被欺骗。检查SPF和DMARC记录是否存在允许欺骗的弱配置。

https://github.com/darkoperator/dnsrecon DNS枚举脚本

https://github.com/laramies/theHarvester The Harvester，一个社会工程学工具，通过搜索引擎、PGP服务器以及SHODAN数据库收集用户的email，子域名，主机，雇员名，开放端口和banner信息。

https://github.com/killswitch-GUI/SimplyEmail SimplyEmail：快速而简单的电子邮件侦察工具。

https://github.com/dxa4481/truffleHog truffleHog，在GitHub上发布的项目是否已经不小心泄漏了任何秘密密钥。
```

- 被动情报收集
```
https://github.com/Urinx/browspy 浏览器用户全部信息收集js
https://github.com/dchrastil/ScrapedIn ScrapedIn：用于爬取LinkedIn的工具，不受API数据侦察的限制。
```
- 主机信息收集
```
https://github.com/kalivim/Powershell_fisher 利用powershell收集用户浏览器中保存的密码，桌面办公文件，电脑硬件软件信息。发送到指定邮件
```

### 扩展信息收集

1、子域名生成-扩大字典

2、IP段拓展信息

- https://github.com/wudicainiao/Fofa-Cscan Fofa查C段，Python

3、 第三方平台业务信息收集

- 微信信息收集，公众号，小程序里的接口，子域名
- 其他网络第三方平台，阿里，百度，以及其它等等的业务信息收集内部URL
- 第三方统计平台信息收集，因为第三方统计中可以明确的记录很多的信息域名，还有页面后台
- 自媒体信息收集，阿里大鱼，百家号，微博，抖音，快手，哔哩哔哩，头条号，搜狐，公众号，等等的媒体号信息收集内部系统地址

4、 舆情业务信息监控

- 利用第三方以及自己的脚本来监控其他企业的业务，企业的舆情，可以添加关键字来监控，以及URL监控，这些都是可以监控很关键的信息，比如监控关键字为，xxxx上新xxx产品，一旦这个产品被媒体号写出曝光就能第一时间被监控到。

## 漏洞测试

- https://github.com/wudicainiao/AWVS12-Scan-Agent AWS12添加目标

- https://www.freebuf.com/sectool/176562.html 刷SRC经验之批量化扫描实践
- [基于Web漏洞扫描的URL及网页框架聚类研究](https://github.com/Cryin/Paper/blob/master/基于Web漏洞扫描的URL及网页框架聚类研究.md)
- https://www.freebuf.com/sectool/161664.html 如何利用NSE检测CVE漏洞

## 源码下载

- http://www.qiqucode.com/browse/category/3?pageindex=2
- https://www.a5.net/

## Bug Bounty Reference

- https://github.com/ngalongc/bug-bounty-reference 按bug性质分类的bug悬赏记录列表
- https://github.com/djadmin/awesome-bug-bounty Bug Bounty 内容
- https://medium.com/tag/bug-bounty medium关于Bug Bounty 的内容
- https://github.com/bugbountycore 拿到赏金的二级域

## 漏洞库

漏洞信息来源

- https://sec.jetlib.com/
- https://www.securitylab.ru/vulnerability/
- https://www.vulnerability-lab.com/
- https://www.rapid7.com/db/
- https://exploits.shodan.io/welcome
- https://packetstormsecurity.com/
- https://cve.mitre.org/ 
- https://www.cvedetails.com
- https://www.exploit-db.com/
- http://routerpwn.com/
- https://cn.0day.today/
- https://cxsecurity.com/
- https://www.zerodayinitiative.com/advisories/published/
- https://www.us-cert.gov/ncas/alerts
- https://nvd.nist.gov/
- https://securiteam.com/
- http://oval.mitre.org/
- http://www.security-database.com/
- http://www.securityfocus.com
- http://hyp3rlinx.altervista.org/
- https://seclists.org/fulldisclosure/
- https://seclists.org/bugtraq/
- https://www.seebug.org/
- https://blog.ripstech.com/
- https://sploitus.com/?query=wsman#tools
- https://vulners.com/search?query=type:zdt

##  SRC众测平台

- https://www.hackerone.com/
- https://www.bugcrowd.com/
- https://www.openbugbounty.org/
- https://www.cnvd.org.cn/ 提供证书
- https://www.butian.net/
- https://dvpnet.io/
- https://www.bugbank.cn/
- https://xianzhi.aliyun.com/productitem/index.htm#/home
- https://www.vulbox.com/

## 公开课

- https://live.freebuf.com/detail/9c82bef60253479f3df1c0f2c802cebf 物联网设备漏洞挖掘思路与技巧
- https://forum.bugcrowd.com/c/ask-a-hacker 学习资源

## 基础知识速学

- https://cloud.tencent.com/edu/learning/ 腾讯云课程
- https://www.cniao5.com/auth/login.html 菜鸟窝
- http://boolan.com/ 互联网教育平台


## 聚合导航

- https://www.shentoushi.top/
- https://www.360zhijia.com/
- https://github.com/topics 
- [安全客-代码安全](https://www.anquanke.com/tag/代码安全)
- [安全客-代码审计](https://www.anquanke.com/tag/代码审计)
- [安全客-漏洞分析](https://www.anquanke.com/tag/漏洞分析)

