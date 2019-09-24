## **BurpSuite**

`Burp Suite`是一个Web应用程序集成攻击平台，它包含了一系列`burp`工具，这些工具之间有大量接口可以互相通信，这样设计的目的是为了促进和提高整个攻击的效率。平台中所有工具共享同一`robust`框架，以便统一处理HTTP请求，持久性，认证，上游代理，日志记录，报警和可扩展性。 `Burp Suite`允许攻击者结合手工和自动技术去枚举、分析、攻击`Web`应用程序。这些不同的`burp`工具通过协同工作，有效的分享信息，支持以某种工具中的信息为基础供另一种工具使用的方式发起攻击。

`Proxy` 提供一个直观、友好的用户界面，他的代理服务器包含非常详细的拦截规则，并能准确分析`HTTP` 消息的结构与内容。

`Spider` 爬行蜘蛛工具，可以用来抓取目标网站，以显示网站的内容，基本结构，和其他功能。

`Scanner` `Web` 应用程序的安全漏洞进行自动发现工具。它被设计用于渗透测试，并密切与您现有的技术和方法，以适应执行手动和半自动化的`Web` 应用程序渗透测试。

`Repeater` 可让您手动重新发送单个`HTTP` 请求

`Intruder` 是`burp` 套件的优势,他提供一组特别有用的功能。它可以自动实施各种定制攻击，包括资源枚举、数据提取、模糊测试等常见漏洞等。在各种有效的扫描工具中，它能够以最细化、最简单的方式访问它生产的请求与响应，允许组合利用个人智能与该工具的控制优点。

`Sequencer` 对会话令牌，会话标识符或其他出于安全原因需要随机产生的键值的可预测性进行分析。

`Decoder` 转化成规范的形式编码数据，或转化成各种形式编码和散列的原始数据。它能够智能识别多种编码格式，使用启发式技术。

`Comparer` 是一个简单的工具，执行比较数据之间的任何两个项目（一个可视化的“差异”）。在攻击一个Web 应用程序的情况下，这一要求通常会出现当你想快速识别两个应用程序的响应之间的差异（例如，入侵者攻击的过程中收到的两种反应之间之间，或登录失败的反应使用有效的和无效的用户名）之间，或两个应用程序请求（例如，确定不同的行为引起不同的请求参数）。

`Burp   Target` 组件主要包含站点地图、目标域2部分组成,他们帮助渗透测试人员更好地了解目标应用的整体状况、当前的工作涉及哪些目标域、分析可能存在的攻击面等信息。



### 利用BurpSuite到SQLMap批量测试SQL注入

 https://www.exploit-db.com/docs/english/45428-bulk-sql-injection-using-burp-to-sqlmap.pdf

#### 1）场景

测试大门户的SQL注入漏洞

#### 2）问题难点

SQLMap只能对URL单一测试

#### 3）解决问题的方法

通过Python脚本把Burp的HTTP请求提取出来交给SQLMap批量测试，提升找大门户网站SQL注入点的效率。

#### 4）方法细节

文章里提到的方法是提取出来Brup对HTTP的请求，但只能是手动对网站访问后Burp的历史纪录。如果是GET请求倒还好，如果是POST而且是伪静态类型，参数就检测不出来了。在burp的target选项卡也有扫描URL的，可以用Burp自带的爬虫方法爬行一遍之后再提取出URI进行测试。

#### 5）总结

除此方法外，也可以使用burp的插件实现批量测试的效果。
```
渗透神器合体：在BurpSuite中集成Sqlmap
https://www.freebuf.com/sectool/45239.html
```



### 测试弱口令

#### 1）场景

测试管理员后台登录密码

#### 2）问题难点

生成序列类穷举密码需要测试数量过多，且有许多小概率密码。

#### 3）解决问题的方法

```
cupp：使用口令生成器生成习惯类密码
cewl：通过爬取网站的时候，根据爬取内容的关键字生成一份字典，通过这种方式生成的字典可以作为cupp生成字典的补充。
在线生成密码框架：https://github.com/boy-hack/pythonwebhack
```

#### 4）方法细节

1、使用BurpSuite的intruder 模块 
```
sniper：

使用一组Payload集合，依次替换Payload位置上（一次攻击只能使用一个Payload位置）

被§标志的文本（而没有被§标志的文本将不受影响），对服务器端进行请求。

通常用于测试请求参数是否存在漏洞。

battering ram：

使用单一的Payload集合，依次替换Payload位置上被§标志的文本（而没有被§标志的文本将不受影响），对服务器端进行请求

与sniper模式的区别在于，如果有多个参数且都为Payload位置标志时。

使用的Payload值是相同的，

而sniper模式只能使用一个Payload位置标志。

pitchfork：

可以使用多组Payload集合，在每一个不同的Payload标志位置上（最多20个），遍历所有的Payload。

举例来说：

如果有两个Payload标志位置

第一个Payload值为A、B，
第二个Payload值为C、D，

则发起攻击时，将共发起两次攻击。

第一次使用的Payload分别为A、C，
第二次使用的Payload分别为B、D。

cluster bomb：

可以使用多组Payload集合，在每一个不同的Payload标志位置上（最多20个），依次遍历所有的Payload。

它与pitchfork模式的主要区别在于，执行的Payload数据Payload组的乘积。

举例来说，如果有两个Payload标志位置，第一个Payload值为A和B，第二个Payload值为C和D，则发起攻击时，将共发起四次攻击。

第一次使用的Payload分别为A、C
第二次使用的Payload分别为A、D
第三次使用的Payload分别为B、C
第四次使用的Payload分别为B、D

```
## Burp扩展插件

- https://github.com/laconicwolf/burp-extensions 插件编写教程示例

- https://github.com/c0ny1/chunked-coding-converter 分块传输绕WAF
- https://github.com/1N3/IntruderPayloads BurpsuitePayload