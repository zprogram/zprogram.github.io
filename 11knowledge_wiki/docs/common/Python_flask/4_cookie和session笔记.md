## cookie：
1.`cookie`出现的原因：在网站中，http请求是无状态的。也就是说即使第一次和服务器连接后并且登录成功后，第二次请求服务器依然不能知道当前请求是哪个用户。cookie的出现就是为了解决这个问题，第一次登录后服务器返回一些数据（cookie）给浏览器，然后浏览器保存在本地，当该用户发送第二次请求的时候，就会自动的把上次请求存储的cookie数据自动的携带给服务器，服务器通过浏览器携带的数据就能判断当前用户是哪个了。
2.如果服务器返回了`cookie`给浏览器，那么浏览器下次再请求相同的服务器的时候，就会自动的把`cookie`发送给浏览器，这个过程，用户根本不需要管。
3.`cookie`是保存在浏览器中的，相对的是浏览器。

## session：
1.`session`介绍：session和cookie的作用有点类似，都是为了存储用户相关的信息。不同的是，cookie是存储在本地浏览器，而session存储在服务器。存储在服务器的数据会更加的安全，不容易被窃取。但存储在服务器也有一定的弊端，就是会占用服务器的资源，但现在服务器已经发展至今，一些session信息还是绰绰有余的。
2.使用`session`的好处：
```
    * 敏感数据不是直接发送回给浏览器，而是发送回一个`session_id`，服务器将`session_id`和敏感数据做一个映射存储在`session`(在服务器上面)中，更加安全。
    * `session`可以设置过期时间，也从另外一方面，保证了用户的账号安全。
```
### flask中的session工作机制：
1.flask中的session机制是：把敏感数据经过加密后放入`session`中，然后再把`session`存放到`cookie`中，下次请求的时候，再从浏览器发送过来的`cookie`中读取`session`，然后再从`session`中读取敏感数据，并进行解密，获取最终的用户数据。
2.flask的这种`session`机制，可以节省服务器的开销，因为把所有的信息都存储到了客户端（浏览器）。
3.安全是相对的，把`session`放到`cookie`中，经过加密，也是比较安全的，这点大家放心使用就可以了。

### 操作session：
1.session的操作方式：
```
    * 使用`session`需要从`flask`中导入`session`，以后所有和`sessoin`相关的操作都是通过这个变量来的。
    * 使用`session`需要设置`SECRET_KEY`，用来作为加密用的。并且这个`SECRET_KEY`如果每次服务器启动后都变化的话，那么之前的`session`就不能再通过当前这个`SECRET_KEY`进行解密了。
    * 操作`session`的时候，跟操作字典是一样的。
    * 添加`session`：`session['username']`。
    * 删除：`session.pop('username')`或者`del session['username']`。
    * 清除所有`session`：`session.clear()`
    * 获取`session`：`session.get('username')`
```
2.设置session的过期时间：
```
    * 如果没有指定session的过期时间，那么默认是浏览器关闭后就自动结束
    * 如果设置了session的permanent属性为True，那么过期时间是31天。
    * 可以通过给`app.config`设置`PERMANENT_SESSION_LIFETIME`来更改过期时间，这个值的数据类型是`datetime.timedelay`类型。
```

**代码：**

```
#encoding: utf-8

from flask import Flask,session
import os
from datetime import timedelta

app = Flask(__name__)
app.config['SECRET_KEY'] = os.urandom(24)
app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(days=7)


# 添加数据到session中
# 操作session的时候，跟操作字典是一样的
# SECRET_KEY

@app.route('/')
def hello_world():
    session['username'] = 'admin'
    # 如果没有指定session的过期时间，那么默认是浏览器关闭后就自动结束
    # 如果设置了session的permanent属性为True，那么过期时间是31天。
    session.permanent = True
    return 'Hello World!'

@app.route('/get/')
def get():
    # sesssion['username']
    # session.get('username')
    print session.get('username')
    return 'sucess'

@app.route('/delete/')
def delete():
    print session.get('username')
    session.pop('username')
    print session.get('username')
    return 'success'

@app.route('/clear/')
def clear():
    print session.get('username')
    # 删除session中的所有数据
    session.clear()
    print session.get('username')
    return 'success'



if __name__ == '__main__':
    app.run(debug=True)

```

## GET请求和POST请求：
1.GET请求：
```
    * 使用场景：如果只对服务器获取数据，并没有对服务器产生任何影响，那么这时候使用get请求。
    * 传参：get请求传参是放在url中，并且是通过`?`的形式来指定key和value的。
```
2.POST请求：
```
    * 使用场景：如果要对服务器产生影响，那么使用post请求。
    * 传参：post请求传参不是放在url中，是通过`form data`的形式发送给服务器的。
```

### GET和POST请求获取参数：

1. get请求是通过`flask.request.args`来获取。
2. post请求是通过`flask.request.form`来获取。
3. post请求在模板中要注意几点：
    * input标签中，要写name来标识这个value的key，方便后台获取。
    * 在写form表单的时候，要指定`method='post'`，并且要指定`action='/login/'`。
4. from示例代码：
```
        <form action="{{ url_for('login') }}" method="post">
            <table>
                <tbody>
                    <tr>
                        <td>用户名：</td>
                        <td><input type="text" placeholder="请输入用户名" name="username"></td>
                    </tr>
                    <tr>
                        <td>密码：</td>
                        <td><input type="text" placeholder="请输入密码" name="password"></td>
                    </tr>
                    <tr>
                        <td></td>
                        <td><input type="submit" value="登录"></td>
                    </tr>
                </tbody>
            </table>
        </form>
```

**flask源码**

```
#encoding: utf-8

from flask import Flask,render_template,request

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/search/')
def search():
    # arguments
    #  GET参数获取
    q = request.args.get('q')
    return u'用户提交的查询参数是：%s' % q


# 默认的视图函数，只能采用get请求
# 如果你想采用post请求，那么要写明
@app.route('/login/',methods=['GET','POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    else:
        # 获取POST的值
        username = request.form.get('username')
        password = request.form.get('password')
        print 'username:',username
        print 'password:',password
        return 'post request'



if __name__ == '__main__':
    app.run(debug=True)
```

## 保存全局变量的g属性：

g：global
1.g对象是专门用来保存用户的数据的。
2.g对象在一次请求中的所有的代码的地方，都是可以使用的。

**示例代码：**

g_demo.py
```

#encoding: utf-8

from flask import Flask,g,render_template,request
from utils import login_log
app = Flask(__name__)


@app.route('/')
def hello_world():
    return 'index'



@app.route('/login/',methods=['GET','POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    else:
        username = request.form.get('username')
        password = request.form.get('password')
        if username == 'admin' and password == '111111':
            # 就认为当前这个用户的用户名和密码正确
            # 全局变量
            g.username = 'admin'
            g.ip = 'xx'
            login_log()
            return u'恭喜！登录成功！'
        else:
            return u'您的用户名或密码错误！'


if __name__ == '__main__':
    app.run(debug=True)

```
utils.py

```
# 可以直接使用g.username
def login_log():
    print u'当前登录用户是：%s' % g.username

```

## 钩子函数（HOOK）：
1.before_request：
```
    * 这个装饰器变量是在请求之前执行
    * 是在视图函数执行之前执行的
    * 这个函数只是一个装饰器，他可以把需要设置为钩子函数的代码放到视图函数执行之前来执行
```

**示例代码**

```
# before_requst：在请求之前执行的
# before_request是在视图函数执行之前执行的
# before_request这个函数只是一个装饰器，他可以把需要设置为钩子函数的代码放到视图函数执行之前来执行

@app.before_request
def my_before_request():
    # user_id = session.get('user_id')
    # user = User.query.filter(User.id==user_id).first()
    # g.user = user
    print 'hello world'
    if session.get('username'):
        g.username = session.get('username',username='admin')
```



2.context_processor：
```
    * 上下文处理器应该返回一个字典。字典中的`key`会被模板中当成变量来渲染。
        * 上下文处理器中返回的字典，在所有页面中都是可用的。
        * **被这个装饰器修饰的钩子函数，必须要返回一个字典，即使为空也要返回。**
```
**示例代码：**
```
# 上下文处理器应该返回一个字典。字典中的`key`会被模板中当成变量来渲染。
@app.context_processor
def my_context_processor():
    # username = session.get('username')
    # if username:
    #     return {'username':username}
    return {'username':'111111'}
```