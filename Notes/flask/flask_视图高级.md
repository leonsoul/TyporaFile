### 类视图

​	之前接触的视图都是函数，所以一般简称视图函数。其实视图也可以基于类来实现，类视图的好处是支持继承，但是类视图不能跟函数视图一样，写完类视图还需要通过`app.add_url_rule(url_rule,view_func)`来进行注册。以下将对两种类视图进行讲解：

#### 	标准类视图：

​	标准类视图是继承自`flask.views.View`，并且在子类中必须实现`dispatch_request`方法，这个方法类似于视图函数，也要返回一个基于`Response`或者其子类的对象。以下将用一个例子进行讲解：

```python
from flask.views import View
class PersonalView(View):
    def dispatch_request(self):
        return "测试视图"
# 类视图通过add_url_rule方法和url做映射
app.add_url_rule('/users/',view_func=PersonalView.as_view('personalview'))
```

#### 	基于调度方法的视图

​	`Flask`还为我们提供了另外一种类视图`flask.views.MethodView`，对每个HTTP方法执行不同的函数（映射到对应方法的小写的同名方法上），以下将用一个例子来进行讲解：

```python
class LoginView(views.MethodView):
    # 当客户端通过get方法进行访问的时候执行的函数
    def get(self):
        return render_template("login.html")

    # 当客户端通过post方法进行访问的时候执行的函数
    def post(self):
        email = request.form.get("email")
        password = request.form.get("password")
        if email == 'xx@qq.com' and password == '111111':
            return "登录成功！"
        else:
            return "用户名或密码错误！"

# 通过add_url_rule添加类视图和url的映射，并且在as_view方法中指定该url的名称，方便url_for函数调用
app.add_url_rule('/myuser/',view_func=LoginView.as_view('loginview'))
```

​	如果用类视图，我们怎么使用装饰器呢？比如有时候需要做权限验证的时候，比如看以下例子：

```python
from flask import session
def login_required(func):
    def wrapper(*args,**kwargs):
        if not session.get("user_id"):
            return 'auth failure'
        return func(*args,**kwargs)
    return wrapper
```

装饰器写完后，可以在类视图中定义一个属性叫做`decorators`，然后存储装饰器。以后每次调用这个类视图的时候，就会执行这个装饰器。示例代码如下：

```python
class UserView(views.MethodView):
    decorators = [user_required]
    ...
```

###  蓝图和子域名

#### 	蓝图

  之前我们写的`url`和视图函数都是处在同一个文件，如果项目比较大的话，这显然不是一个合理的结构，而蓝图可以优雅的帮我们实现这种需求。以下看一个使用蓝图的文件的例子：

```python
from flask import Blueprint
bp = Blueprint('user',__name__,url_prefix='/user/')

@bp.route('/')
def index():
    return "用户首页"

@bp.route('profile/')
def profile():
    return "个人简介"
```

  然后我们在主程序中，通过`app.register_blueprint()`方法将这个蓝图注册进url映射中，看下主`app`的实现：

```python
from flask import Flask
import user

app = Flask(__name__)
app.register_blueprint(user.bp)

if __name__ == '__main__':
    app.run(host='0.0.0.0',port=9000)
```

  以后访问`/user/`，`/user/profile/`，都是执行的`user.py`文件中的视图函数，这样就实现了项目的模块化。

  以上是对蓝图的一个简单介绍，但是使用蓝图还有几个需要注意的地方，就是在蓝图如何寻找静态文件、模板文件，`url_for`函数如何反转`url`，以下分别进行解释：

#####  	1.1 寻找静态文件

​	默认不设置任何静态文件路径，`Jinja2`会在项目的`static`文件夹中寻找静态文件。也可以设置其他的路径，在初始化蓝图的时候，`Blueprint`这个构造函数，有一个参数`static_folder`可以指定静态文件的路径，如：

```python
bp = Blueprint('admin',__name__,url_prefix='/admin',static_folder='static')
```

​	`static_folder`可以是相对路径（相对蓝图文件所在的目录），也可以是绝对路径。在配置完蓝图后，还有一个需要注意的地方是如何在模板中引用静态文件。在模板中引用蓝图，应该要使用`蓝图名+.+static`来引用，如下所示：

```python
  <link href="{{ url_for('admin.static',filename='about.css') }}">
```

##### 	1.2 寻找模板文件

​	跟静态文件一样，默认不设置任何模板文件的路径，将会在项目的`templates`中寻找模板文件。也可以设置其他的路径，在构造函数`Blueprint`中有一个`template_folder`参数可以设置模板的路径，如下所示：

```python
bp = Blueprint('admin',__name__,url_prefix='/admin',template_folder='templates')
```

​	模板文件和静态文件有点区别，以上代码写完以后，如果你渲染一个模板`return render_template('admin.html')`，`Flask`默认会去项目根目录下的`templates`文件夹中查找`admin.html`文件，如果找到了就直接返回，如果没有找到，才会去蓝图文件所在的目录下的`templates`文件夹中寻找。

##### 	1.3  url_for生成`url`：

​	用`url_for`生成蓝图的`url`，使用的格式是：`蓝图名称+.+视图函数名称`。比如要获取`admin`这个蓝图下的`index`视图函数的`url`，应该采用以下方式：

```python
url_for('admin.index')
```

其中这个**蓝图名称**是在创建蓝图的时候，传入的第一个参数。`bp = Blueprint('admin',__name__,url_prefix='/admin',template_folder='templates')`

#### 子域名：

​	子域名在许多网站中都用到了，比如一个网站叫做`xxx.com`，那么我们可以定义一个子域名`cms.xxx.com`来作为`cms`管理系统的网址，子域名的实现一般也是通过蓝图来实现，在之前章节中，我们创建蓝图的时候添加了一个`url_prefix=/user`作为url前缀，那样我们就可以通过`/user/`来访问`user`下的url。但使用子域名则不需要。另外，还需要配置`SERVER_NAME`，比如`app.config[SERVER_NAME]='example.com:9000'`。并且在注册蓝图的时候，还需要添加一个`subdomain`的参数，这个参数就是子域名的名称，先来看一下蓝图的实现(admin.py)：

```python
from flask import Blueprint
bp = Blueprint('admin',__name__,subdomain='admin')

@bp.route('/')
def admin():
    return 'Admin Page'
```

这个没有多大区别，接下来看主`app`的实现：

```python
from flask import Flask
import admin

# 配置`SERVER_NAME`
app.config['SERVER_NAME'] = 'example.com:8000'
# 注册蓝图，指定了subdomain
app.register_blueprint(admin.bp)

if __name__ == '__main__':
    app.run(host='0.0.0.0',port=8000,debug=True)
```

​	写完以上两个文件后，还是不能正常的访问`admin.example.com:8000`这个子域名，因为我们没有在`host`文件中添加域名解析，你可以在最后添加一行`127.0.0.1 admin.example.com`，就可以访问到了。另外，子域名不能在`127.0.0.1`上出现，也不能在`localhost`上出现。
