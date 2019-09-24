## 1、学习目标

1. 熟悉Flask相关知识。
2. 熟悉web开发流程。
3. 能独立开发Flask项目。

## 2、环境配置

### Python虚拟环境安装

1. 因为python的框架更新迭代太快了，有时候需要在电脑上存在一个框架的多个版本，这时候虚拟环境就可以解决这个问题。
2. 通过以下命令安装虚拟环境：
> pip install virtualenv
3. 开辟新的虚拟环境：
> virtualenv [virtualenv-name]
4. 激活虚拟环境：
    * [类Linux]：source [虚拟环境的目录]/bin/activate
```
# 进入Python虚拟环境
cd C:\ProjectCode\PythonScript\python2venv\Scripts\
```
    * [Windows]：直接进入到虚拟环境的目录，然后执行activate
       
    * 退出虚拟环境：deactivate

### pip安装Flask
1. 执行activate脚本，进入指定的虚拟环境。
2. 在该虚拟环境中，执行以下命令安装：pip install flask
3. 验证flask是否安装成功：
```
    * 进入python命令行。
    >>> import flask
    >>> print flask.__version__
```

## 3、基础知识

### url 基础知识
1. 如果使用的是http协议，那么浏览器就会使用80端口去请求这个服务器的资源。
2. 如果使用的是https协议，那么浏览器会使用443端口去请求这个服务器的资源。

一个URL由以下几部分组成：

```
    scheme://host:port/path/?query-string=xxx#anchor
```
scheme：代表的是访问的协议，一般为http或者https以及ftp等。
host：主机名，域名，比如www.baidu.com。
port：端口号。当你访问一个网站的时候，浏览器默认使用80端口。
path：查找路径。比如：www.jianshu.com/trending/now，后面的trending/now就是path。
query-string：查询字符串，比如：www.baidu.com/s?wd=python，后面的wd=python就是查询字符串。
anchor：锚点，后台一般不用管，前端用来做页面定位的。

### WEB服务器和应用服务器以及web应用框架

WEB服务器：负责处理HTTP请求，响应静态文件，常见的有Apache，Nginx以及微软的IIS.

应用服务器：负责处理逻辑的服务器。比如PHP、Python的代码，是不能直接通过Nginx这种WEB服务器来处理的，只能通过应用服务器来处理，常见的应用服务器有uwsgi、Tomcat等。

WEB应用框架：一般使用某种语言，封装了常用的WEB功能的框架就是WEB应用框架，Flask、Django以及Java中的SSH(Structs2+Spring3+Hibernate3)框架都是WEB应用框架。

## 4、第一个flask程序

1. Pychrm要添加flask的虚拟环境。添加虚拟环境的时候，选择创建豪的虚拟环境中python执行文件。

```
C:\ProjectCode\PythonScript\python2venv\bin\python.exe
```


2. flask程序代码的详细解释：


01app_hello

```
# 从flask这个框架中导入Flask这个类
from flask import Flask

# 初始化一个Flask对象
# Flaks()
# 需要传递一个参数__name__
# 1. 方便flask框架去寻找资源
# 2. 方便flask插件比如Flask-Sqlalchemy出现错误的时候，好去寻找问题所在的位置
app = Flask(__name__)

# @app.route是一个装饰器
# @开头，并且在函数的上面，说明是装饰器
# 这个装饰器的作用，是做一个url与视图函数的映射
# 127.0.0.1:5000/   ->  去请求hello_world这个函数，然后将结果返回给浏览器
@app.route('/')
def hello_world():
    return 'Hello World!'

# 如果当前这个文件是作为入口程序运行，那么就执行app.run()
if __name__ == '__main__':
    # app.run()
    # 启动一个应用服务器，来接受用户的请求
    # while True:
    #   listen()
    app.run()
```

### 设置debug模式
1. 在app.run()中传入一个关键字参数debug,app.run(debug=True)，就设置当前项目为debug模式。
2. debug模式的两大功能：
    * 当程序出现问题的时候，可以在页面中看到错误信息和出错的位置。
    * 只要修改了项目中的`python`文件，程序会自动加载，不需要手动重新启动服务器。

02Debug_Mod

```
#encoding: utf-8

from flask import Flask
import config

app = Flask(__name__)
app.config.from_object(config)

@app.route('/')
def hello_world():
    return '我是知了课堂'

if __name__ == '__main__':
    app.run(DEBUG = True)
```

### 使用配置文件
1. 新建一个`config.py`文件
2. 在主app文件中导入这个文件，并且配置到`app`中，示例代码如下：
```
import config
app.config.from_object(config)
```
3. 还有许多的其他参数，都是放在这个配置文件中，比如`SECRET_KEY`和`SQLALCHEMY`这些配置，都是在这个文件中。

### url传参数

1. 参数的作用：可以在相同的URL，但是指定不同的参数，来加载不同的数据。
2. 在flask中如何使用参数：
    ```
    @app.route('/article/<id>')
    def article(id):
        return u'您请求的参数是：%s' % id
    ```
    * 参数需要放在两个尖括号中。
    * 视图函数中需要放和url中的参数同名的参数。

03url_params

```
#encoding: utf-8

from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

@app.route('/article/<id>')
def article(id):
    return u'您请求的参数是：%s' % id

if __name__ == '__main__':
    app.run(debug=True)
```



### 反转URL
1. 什么叫做反转URL：从视图函数到url的转换叫做反转url
2. 反转url的用处：
    * 在页面重定向的时候，会使用url反转。
    * 在模板中，也会使用url反转。


04url_reverse

```
#encoding: utf-8

from flask import Flask,url_for

app = Flask(__name__)

@app.route('/')
def index():
    # 作用于登录、注册的跳转
    print url_for('my_list')
    print url_for('article',id='123')
    return 'Hello World!'

@app.route('/list/')
def my_list():
    return 'list'

@app.route('/article/<id>/')
def article(id):
    return u'您请求的id是：%s' % id

if __name__ == '__main__':
    app.run(debug=True)
```

### 页面跳转和重定向
1. 用处：在用户访问一些需要登录的页面的时候，如果用户没有登录，那么可以让她重定向到登录页面。
2. 代码实现：
    ```
    from flask import redirect,url
    redirect(url_for('login'))
    ```

05redirect

```
#encoding: utf-8

from flask import Flask,redirect,url_for

app = Flask(__name__)

@app.route('/')
def index():
    login_url = url_for('login')
    return redirect(login_url)
    return u'这是首页'

@app.route('/login/')
def login():
    return u'这是登录页面'

@app.route('/question/<is_login>/')
def question(is_login):
    if is_login == '1':
        return u'这是发布问答页面'
    else:
        return redirect(url_for('login'))

if __name__ == '__main__':
    app.run(debug=True)
```

## 5、参考

《Flask Web开发——基于Python的Web应用开发实践》
《Python Flask系列（1）——基础》
https://study.163.com/course/courseMain.htm?courseId=1004091002#/courseDetail?tab=1