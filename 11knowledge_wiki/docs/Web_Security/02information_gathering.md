
# 信息收集

信息收集是属于前期交互阶段所需要了解的问题。

## 前期交互内容

- 签署授权文件：首要要和受测试方签订授权协议。
- 划定范围:指定了一个二级域名作为测试目标，那么其他二级域名在测试范围内。如果测试目标是在其他服务厂商，或者服务器上有其他额外的测试目标是否划入范围内。

- 情报搜集：开放渠道情报、网络踩点、识别目标防护机制（管理、网络、主机、应用）。

## 开放渠道情报

- 数据来源：搜索引擎提供的开放情报、社交网站、地方性门户、电子邮件服务提供商
```
- whois：查询域名注册信息库、服务商、管理员邮箱地址、域名注册日期和过期日期

- Netcraft：查询DNS与IP信息分析，可以获取子站点、地理位置、DNS服务器地址、操作系统类型、服务器运行状况。

- GEOIP：根据IP定位经纬度，通过Google地图获取位置信息。

- IP反查：通过IP地址可以反查指向同一IP的DNS信息（旁站攻击）。www.ip-adress.com/reverse_ip/、http://s.tool.chinaz.com/same

- Google：

> GHDB（GoogleHackingDatabase）http://www.exploit-db.com/google-dorks/
> 列目录查找：Google搜索【intitle:parent directory】
> 查找敏感文件：Google搜索【Site:testfire.net filetype:xls】
> 登录页面：Google搜索【site:test.com login】
> 网络空间搜索引擎：fofa.so、ZoomEye、netcraft等

```
- 关注点：企业情报、个人情报

## 网络踩点

- 创建目标列表：存活主机/版本信息/识别补丁级别/搜索脆弱的Web应用/确定封禁阈值/出错信息/可以攻击的脆弱端口/过时系统/虚拟化平台和虚拟机/存储基础设施

- 深度挖掘目标细节：端口扫描/SNMP探查/区域传送/SMTP反弹攻击/解析DNS与递归DNS服务器

## 信息收集关注的常见端口（服务）

- WEB 80
- MSSQL 1433
- MySQL 3306
- Oracle 1521
- Microsoft SMB 445
- VNC 5900,5901
- Remote Desktop 3389

## 技巧搜集

[Open Source Intelligence Gathering 201](https://www.exploit-db.com/papers/45360/)

[渗透测试之子域名探测指南](https://xz.aliyun.com/t/3478)

