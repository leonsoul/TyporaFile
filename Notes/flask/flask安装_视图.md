## Flask 简介

`flask`是一款非常流行的`Python Web`框架，出生于2010年，作者是`Armin Ronacher`,本来这个项目只是作者在愚人节的一个玩笑，后来由于非常受欢迎，进而成为一个正式的项目。

`flask`自2010年发布第一个版本以来，大受欢迎，深得开发者的喜爱，目前在`Github`上的Star数已经超过`55.5k`了，有超`Django`之趋势。`flask`能如此流行的原因，可以分为以下几点：

- 微框架、简洁、只做他需要做的，给开发者提供了很大的扩展性。

- Flask和相应的插件写得很好，用起来很爽。

## 安装Flask：

    通过`pip install flask`即可安装。

## 第一个flask程序：

    用`pycharm`新建一个`flask`项目，新建项目的截图如下：
    
    ![新建项目](https://www.zlkt.net/media/course/87L8AaNBNKGcAAn7oomKYT.png)

- 开发效率非常高，比如使用`SQLAlchemy`的`ORM`操作数据库可以节省开发者大量书写`sql`的时间。

`Flask`的灵活度非常之高，他不会帮你做太多的决策，一些你都可以按照自己的意愿进行更改。比如：

- 使用`Flask`开发数据库的时候，具体是使用`SQLAlchemy`还是`MongoEngine`，选择权完全掌握在你自己的手中。区别于`Django`，`Django`内置了非常完善和丰富的功能，并且如果你想替换成你自己想要的，要么不支持，要么非常麻烦。

- 把默认的`Jinija2`模板引擎替换成其他模板引擎都是非常容易的。

## 项目配置
  #### 	设置为DEBUG模式

​	默认情况下`flask`不会开启`DEBUG`模式，开启`DEBUG`模式后，flask会在每次保存代码的时候自动的重新载入代码，并且如果代码有错误，会在终端进行提示。

![image.png](https://cdn.jsdelivr.net/gh/leonsoul/TyporaFile@master/uPic/2UZz2JSDySff5o8CPeqPRN.png)

​	如果一切正常，会在终端打印以下信息：

```bash
* Restarting with stat
* Debugger is active!
* Debugger pin code: 294-745-044
* Running on http://0.0.0.0:9000/ (Press CTRL+C to quit)
```

​	需要注意的是，只能在开发环境下开启`DEBUG`模式，因为`DEBUG`模式会带来非常大的安全隐患。

  #### 	配置文件

   `Flask`项目的配置，都是通过`app.config`对象来进行配置的。比如要配置一个项目的`SECRET_KEY`，那么可以使用`app.config['SECRET_KEY'] = "xxx"`来进行设置，在`Flask`项目中，有四种方式进行项目的配置：

1. 直接硬编码：

    ```python
    app = Flask(__name__)
    app.config['SECRET_KEY'] = "xxx"
    ```

2. 因为`app.config`是`flask.config.Config`的实例，而`Config`类是继承自`dict`，因此可以通过`update`方法：

    ```python
    app.config.update(
       DEBUG=True,
       SECRET_KEY='...'
    )
    ```

3. 如果你的配置项特别多，你可以把所有的配置项都放在一个模块中，然后通过加载模块的方式进行配置，假设有一个`settings.py`模块，专门用来存储配置项的，此时你可以通过`app.config.from_object()`方法进行加载，并且该方法既可以接收模块的的字符串名称，也可以模块对象：

    ```python
    # 1. 通过模块字符串
    app.config.from_object('settings')
    # 2. 通过模块对象
    import settings
    app.config.from_object(settings)
    ```

4. 也可以通过另外一个方法加载，该方法就是`app.config.from_pyfile()`，该方法传入一个文件名，通常是以`.py`结尾的文件，但也不限于只使用`.py`后缀的文件：

    ```python
    app.config.from_pyfile('settings.py',silent=True)
    # silent=True表示如果配置文件不存在的时候不抛出异常，默认是为False，会抛出异常。
    ```

`Flask`项目内置了许多的配置项，所有的内置配置项，可以在[这里查看](https://flask.palletsprojects.com/en/2.0.x/config/)

## URL与视图
  #### 	URL与函数的映射
```python
从之前的`helloworld.py`文件中，我们已经看到，一个`URL`要与执行函数进行映射，使用的是`@app.route`装饰器。`@app.route`装饰器中，可以指定`URL`的规则来进行更加详细的映射，比如现在要映射一个文章详情的`URL`，文章详情的`URL`是`/article/id/`，id有可能为1、2、3…,那么可以通过以下方式：
```

```python
@app.route('/article/<id>/')
def article(id):
   return '%s article detail' % id
```

```python
其中`<id>`，尖括号是固定写法，语法为`<variable>`，`variable`默认的数据类型是字符串。如果需要指定类型，则要写成`<converter:variable>`，其中`converter`就是类型名称，可以有以下几种：
```

- string: 默认的数据类型，接受没有任何斜杠`/`的字符串。

- int: 整形

- float: 浮点型。

- path： 和`string`类似，但是可以传递斜杠`/`。

- uuid： `uuid`类型的字符串。

- any：可以指定多种路径，这个通过一个例子来进行说明:

    ```python
    @app.route('/<any(article,blog):url_path>/')
    def item(url_path):
      return url_path
    ```

    以上例子中，`item`这个函数可以接受两个`URL`,一个是`/article/`，另一个是`/blog/`。并且，一定要传`url_path`参数，当然这个`url_path`的名称可以随便。

    如果不想定制子路径来传递参数，也可以通过传统的`?=`的形式来传递参数，例如：`/article?id=xxx`，这种情况下，可以通过`request.args.get('id')`来获取`id`的值。如果是`post`方法，则可以通过`request.form.get('id')`来进行获取。

  ####构造URL（url_for）：

    一般我们通过一个`URL`就可以执行到某一个函数。如果反过来，我们知道一个函数，怎么去获得这个`URL`呢？`url_for`函数就可以帮我们实现这个功能。`url_for()`函数接收两个及以上的参数，他接收**函数名**作为第一个参数，接收对应**URL规则的命名参数**，如果还出现其他的参数，则会添加到`URL`的后面作为**查询参数**。

通过构建`URL`的方式而选择直接在代码中拼`URL`的原因有两点：

1. 将来如果修改了`URL`，但没有修改该`URL`对应的函数名，就不用到处去替换`URL`了。
2. `url_for()`函数会转义一些特殊字符和`unicode`字符串，这些事情`url_for`会自动的帮我们搞定。

下面用一个例子来进行解释：

```python
from flask import Flask,url_for
app = Flask(__name__)

@app.route('/article/<id>/')
def article(id):
    return '%s article detail' % id

@app.route('/')
def index(request):
    print(url_for("article",id=1))
    return "首页"
```

在访问index的时候，会打印出`/article/1/`。

  ####   指定URL末尾的斜杠：

​	有些`URL`的末尾是有斜杠的，有些`URL`末尾是没有斜杠的。这其实是两个不同的`URL`。举个例子：

```python
 @app.route('/article/')
 def articles():
     return '文章列表页'
```

上述例子中，当访问一个结尾不带斜线的`URL`：`/article`，会被重定向到带斜线的`URL`：`/article/`上去。但是当我们在定义`article`的`url`的时候，如果在末尾没有加上斜杠，但是在访问的时候又加上了斜杠，这时候就会抛出一个`404`错误页面了：

```python
@app.route("/article")
def articles(request):
    return "文章列表页面"
```

以上没有在末尾加斜杠，因此在访问`/article/`的时候，就会抛出一个`404`错误。

  #### 指定HTTP方法：

​	在`@app.route()`中可以传入一个关键字参数`methods`来指定本方法支持的`HTTP`方法，默认情况下，只能使用`GET`请求，看以下例子：

```python
@app.route('/login/',methods=['GET','POST'])
def login():
    return 'login'
```

以上装饰器将让`login`的`URL`既能支持`GET`又能支持`POST`。

  #### 页面跳转和重定向：

​	重定向分为永久性重定向和暂时性重定向，在页面上体现的操作就是浏览器会从一个页面自动跳转到另外一个页面。比如用户访问了一个需要权限的页面，但是该用户当前并没有登录，因此我们应该给他重定向到登录页面。

- 永久性重定向：`http`的状态码是`301`，多用于旧网址被废弃了要转到一个新的网址确保用户的访问，最经典的就是京东网站，你输入`www.jingdong.com`的时候，会被重定向到`www.jd.com`，因为`jingdong.com`这个网址已经被废弃了，被改成`jd.com`，所以这种情况下应该用永久重定向。
- 暂时性重定向：`http`的状态码是`302`，表示页面的暂时性跳转。比如访问一个需要权限的网址，如果当前用户没有登录，应该重定向到登录页面，这种情况下，应该用暂时性重定向。

在`flask`中，重定向是通过`flask.redirect(location,code=302)`这个函数来实现的，`location`表示需要重定向到的`URL`，应该配合之前讲的`url_for()`函数来使用，`code`表示采用哪个重定向，默认是`302`也即`暂时性重定向`，可以修改成`301`来实现永久性重定向。

以下来看一个例子，关于在`flask`中怎么使用重定向：

```python
 from flask import Flask,url_for,redirect

 app = Flask(__name__)
 app.debug = True

 @app.route('/login/',methods=['GET','POST'])
 def login():
     return 'login page'

 @app.route('/profile/',methods=['GET','POST'])
 def profile():
     name = request.args.get('name')

     if not name:
     # 如果没有name，说明没有登录，重定向到登录页面
         return redirect(url_for('login'))
     else:
         return name
```
