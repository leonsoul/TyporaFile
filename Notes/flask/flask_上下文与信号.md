## Flask上下文

`Flask`项目中有两个上下文，一个是应用上下文（app），另外一个是请求上下文（request）。请求上下文`request`和应用上下文`current_app`都是一个全局变量。所有请求都共享的。`Flask`有特殊的机制可以保证每次请求的数据都是隔离的，即A请求所产生的数据不会影响到B请求。所以可以直接导入`request`对象，也不会被一些脏数据影响了，并且不需要在每个函数中使用request的时候传入`request`对象。这两个上下文具体的实现方式和原理可以没必要详细了解。只要了解这两个上下文的四个属性就可以了：

- `request`：请求上下文上的对象。这个对象一般用来保存一些请求的变量。比如`method`、`args`、`form`等。
- `session`：请求上下文上的对象。这个对象一般用来保存一些会话信息。
- `current\_app`：返回当前的app。
- `g`：应用上下文上的对象。处理请求时用作临时存储的对象。

### 常用的钩子函数

- before_first_request：处理第一次请求之前执行。例如以下代码：

    ```python
      @app.before_first_request
      def first_request():
          print 'first time request'
    ```

- before_request：在每次请求之前执行。通常可以用这个装饰器来给视图函数增加一些变量。例如以下代码：

    ```python
     @app.before_request
     def before_request():
         if not hasattr(g,'user'):
             setattr(g,'user','xxxx')
    ```

- teardown_appcontext：不管是否有异常，注册的函数都会在每次请求之后执行。

    ```python
     @app.teardown_appcontext
     def teardown(exc=None):
         if exc is None:
             db.session.commit()
         else:
             db.session.rollback()
         db.session.remove()
    ```

- template_filter：在使用

    ```python
    Jinja2
    ```

    模板的时候自定义过滤器。比如可以增加一个

    ```python
    upper
    ```

    的过滤器（当然Jinja2已经存在这个过滤器，本示例只是为了演示作用）：

    ```python
      @app.template_filter
      def upper_filter(s):
          return s.upper()
    ```

- context_processor：上下文处理器。返回的字典中的键可以在模板上下文中使用。例如：

    ```python
      @app.context_processor
      return {'current_user':'xxx'}
    ```

- errorhandler：errorhandler接收状态码，可以自定义返回这种状态码的响应的处理方法。例如：

    ```python
      @app.errorhandler(404)
      def page_not_found(error):
          return 'This page does not exist',404
    ```

## Flask信号：

### 一、安装：

`flask`中的信号使用的是一个第三方插件，叫做`blinker`。通过`pip list`看一下，如果没有安装，通过以下命令即可安装`blinker`：

```python
pip install blinker
```

### 二、内置信号：

`flask`内置集中常用的信号：

1. `flask.template_rendered`：模版渲染完毕后发送，示例如下：

    ```python
      from flask import template_rendered
      def log_template_renders(sender,template,context,*args):
          print 'sender:',sender
          print 'template:',template
          print 'context:',context
    
      template_rendered.connect(log_template_renders,app)
    ```

2. `flask.request_started`：请求开始之前，在到达视图函数之前发送，订阅者可以调用`request`之类的标准全局代理访问请求。示例如下：

    ```
      def log_request_started(sender,**extra):
          print 'sender:',sender
          print 'extra:',extra
      request_started.connect(log_request_started,app)
    ```

3. `flask.request_finished`：请求结束时，在响应发送给客户端之前发送，可以传递`response`，示例代码如下：

    ```
      def log_request_finished(sender,response,*args):
          print 'response:',response
      request_finished.connect(log_request_finished,app)
    ```

4. `flask.got_request_exception`：在请求过程中抛出异常时发送，异常本身会通过`exception`传递到订阅的函数。示例代码如下：

    ```
      def log_exception_finished(sender,exception,*args):
          print 'sender:',sender
          print type(exception)
      got_request_exception.connect(log_exception_finished,app)
    ```

5. `flask.request_tearing_down`：请求被销毁的时候发送，即使在请求过程中发生异常，也会发送，示例代码如下：

    ```
      def log_request_tearing_down(sender,**kwargs):
          print 'coming...'
      request_tearing_down.connect(log_request_tearing_down,app)
    ```

6. `flask.appcontext_tearing_down`：在应用上下文销毁的时候发送，它总是会被调用，即使发生异常。示例代码如下：

    ```
      def log_appcontext_tearing_down(sender,**kwargs):
          print 'coming...'
      appcontext_tearing_down.connect(log_appcontext_tearing_down,app)
    ```

### 三、自定义信号：

自定义信号分为3步，第一是定义一个信号，第二是监听一个信号，第三是发送一个信号。以下将对这三步进行讲解：

1. 定义信号：定义信号需要使用到`blinker`这个包的`Namespace`类来创建一个命名空间。比如定义一个在访问了某个视图函数的时候的信号。示例代码如下：

    ```
      from blinker import Namespace
    
      mysignal = Namespace()
      visit_signal = mysignal.signal('visit-signal')
    ```

2. 监听信号：监听信号使用`singal`对象的`connect`方法，在这个方法中需要传递一个函数，用来接收以后监听到这个信号该做的事情。示例代码如下：

    ```
      def visit_func(sender,username):
        print(sender)
        print(username)
    
      mysignal.connect(visit_func)
    ```

3. 发送信号：发送信号使用`singal`对象的`send`方法，这个方法可以传递一些其他参数过去。示例代码如下：

    ```
      mysignal.send(username='zhiliao')
    ```