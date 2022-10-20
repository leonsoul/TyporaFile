## Flask-WTF表单验证

`Flask-WTF`是简化了`WTForms`操作的一个第三方库。`WTForms`表单的两个主要功能是验证用户提交数据的合法性以及渲染模板。当然还包括一些其他的功能：`CSRF保护`，文件上传等。安装`Flask-WTF`默认也会安装`WTForms`，因此使用以下命令来安装`Flask-WTF`:

```
pip install flask-wtf
```

### 一、表单验证：

安装完`Flask-WTF`后。来看下第一个功能，就是用表单来做数据验证，现在有一个`forms.py`文件，然后在里面创建一个`RegistForm`的注册验证表单：

```python
class RegistForm(Form):
    name = StringField(validators=[length(min=4,max=25)])
    email = StringField(validators=[email()])
    password = StringField(validators=[DataRequired(),length(min=6,max=10),EqualTo('confirm')])
    confirm = StringField()
```

在这个里面指定了需要上传的参数，并且指定了验证器，比如`name`的长度应该在`4-25`之间。`email`必须要满足邮箱的格式。`password`长度必须在`6-10`之间，并且应该和`confirm`相等才能通过验证。

写完表单后，接下来就是`regist.html`文件：

```python
<form action="/regist/" method="POST">
    <table>
        <tr>
            <td>用户名：</td>
            <td><input type="text" name="name"></td>
        </tr>
        <tr>
            <td>邮箱：</td>
            <td><input type="email" name="email"></td>
        </tr>
        <tr>
            <td>密码：</td>
            <td><input type="password" name="password"></td>
        </tr>
        <tr>
            <td>确认密码：</td>
            <td><input type="password" name="confirm"></td>
        </tr>
        <tr>
            <td></td>
            <td><input type="submit" value="提交"></td>
        </tr>
    </table>
</form>
```

再来看视图函数`regist`：

```python
@app.route('/regist/',methods=['POST','GET'])
def regist():
    form = RegistForm(request.form)
    if request.method == 'POST' and form.validate():
        user = User(name=form.name.data,email=form.email.data,password=form.password.data)
        db.session.add(user)
        db.session.commit()
        return u'注册成功!'
    return render_template('regist.html')
```

`RegistForm`传递的是`request.form`进去进行初始化，并且判断`form.validate`会返回用户提交的数据是否满足表单的验证。

### 二、渲染模板：

`form`还可以渲染模板，让你少写了一丢丢的代码，比如重写以上例子，`RegistForm`表单代码如下：

```python
class RegistForm(Form):
    name = StringField(u'用户名：',validators=[length(min=4,max=25)])
    email = StringField(u'邮箱：'validators=[email()])
    password = StringField(u'密码：',validators=[DataRequired(),length(min=6,max=10),EqualTo('confirm')])
    confirm = StringField(u'确认密码：')
```

以上增加了第一个位置参数，用来在html文件中，做标签提示作用。

在`app`中的视图函数中，修改为如下：

```python
@app.route('/regist/',methods=['POST','GET'])
def regist():
    form = RegistForm(request.form)
    if request.method == 'POST' and form.validate():
        user = User(name=form.name.data,email=form.email.data,password=form.password.data)
        db.session.add(user)
        db.session.commit()
        return u'注册成功!'
    return render_template('regist.html',form=form)
```

以上唯一的不同是在渲染模板的时候传入了`form`表单参数进去，这样在模板中就可以使用表单`form`变量了。

接下来看下`regist.html`文件：

```python
<form action="/regist/" method="POST">
    <table>
        <tr>
            <td>{{ form.name.label }}</td>
            <td>{{ form.name() }}</td>
        </tr>
        <tr>
            <td>{{ form.email.label }}</td>
            <td>{{ form.email() }}</td>
        </tr>
        <tr>
            <td>{{ form.password.label }}</td>
            <td>{{ form.password() }}</td>
        </tr>
        <tr>
            <td>{{ form.confirm.label }}</td>
            <td>{{ form.confirm() }}</td>
        </tr>
        <tr>
            <td></td>
            <td><input type="submit" value="提交"></td>
        </tr>
    </table>
</form>
```

## Flask-WTF常用字段和验证器

### 一、Field常用参数：

在使用`Field`的时候，经常需要传递一些参数进去，以下将对一些常用的参数进行解释：

- label（第一个参数）：`Field`的label的文本。
- validators：验证器。
- id：`Field`的id属性，默认不写为该属性名。
- default：默认值。
- widget：指定的`html`控件。

### 二、常用Field：

- BooleanField：布尔类型的Field，渲染出去是`checkbox`。

- FileField：文件上传Field。

    ```python
      # forms.py
      from flask_wtf.file import FileField,FileAllowed,FileRequired
      class UploadForm(FlaskForm):
      	avatar = FileField(u'头像：',validators=[FileRequired(),FileAllowed([])])
    
      # app.py
      @app.route('/profile/',methods=('POST','GET'))
      def profile():
      	form = ProfileForm()
      	if form.validate_on_submit():
          	filename = secure_filename(form.avatar.data.filename)
          	form.avatar.data.save(os.path.join(app.config['UPLOAD_FOLDER'],filename))
      		return u'上传成功'
    
      	return render_template('profile.html',form=form)
    ```

- FloatField：浮点数类型的Field，但是渲染出去的时候是`text`的input。

- IntegerField：整形的Field。同FloatField。

- RadioField：`radio`类型的`input`。表单例子如下：

    ```python
    # form.py
    class RegistrationForm(FlaskForm):
        gender = wtforms.RadioField(u'性别：',validators=[DataRequired()])
    ```

    模板文件代码如下：

    ```html
      <tr>
          <td>
              {{ form.gender.label }}
          </td>
          <td>
              {% for gender in form.gender %}
                  {{ gender.label }}
                  {{ gender }}
              {% endfor %}
          </td>
      </tr>
    ```

    `app.py`文件的代码如下，给`gender`添加了`choices`：

    ```python
      @app.route('/register/',methods=['POST','GET'])
      def register():
          form = RegistrationForm()
          form.gender.choices = [('1',u'男'),('2',u'女')]
          if form.validate_on_submit():
              return u'success'
    
          return render_template('register.html',form=form)
    ```

- SelectField：类似于`RadioField`。看以下示例：

    ```python
      # forms.py
      class ProfileForm(FlaskForm):
          language = wtforms.SelectField('Programming Language',choices=[('cpp','C++'),('py','python'),('text','Plain Text')],validators=[DataRequired()])
    ```

    再来看`app.py`文件：

    ```python
      @app.route('/profile/',methods=('POST','GET'))
      def profile():
          form = ProfileForm()
          if form.validate_on_submit():
              print form.language.data
              return u'上传成功'
          return render_template('profile.html',form=form)
    ```

    模板文件为：

    ```html
      <form action="/profile/" method="POST">
          {{ form.csrf_token }}
          {{ form.language.label }}
          {{ form.language() }}
          <input type="submit">
      </form>
    ```

- StringField：渲染到模板中的类型为`<input type='text'>`，并且是最基本的文本验证。

- PasswordField：渲染出来的是一个`password`的`input`标签。

- TextAreaField：渲染出来的是一个`textarea`。

### 三、常用的验证器：

数据发送过来，经过表单验证，因此需要验证器来进行验证，以下对一些常用的内置验证器进行讲解：

- Email：验证上传的数据是否为邮箱。
- EqualTo：验证上传的数据是否和另外一个字段相等，常用的就是密码和确认密码两个字段是否相等。
- InputRequired：原始数据的需要验证。如果不是特殊情况，应该使用`InputRequired`。
- Length：长度限制，有min和max两个值进行限制。
- NumberRange：数字的区间，有min和max两个值限制，如果处在这两个数字之间则满足。
- Regexp：自定义正则表达式。
- URL：必须要是`URL`的形式。
- UUID：验证`UUID`。

### 四、自定义验证字段：

使用`validate_fieldname(self,field)`可以对某个字段进行更加详细的验证，如下：

```python
class ProfileForm(FlaskForm):
    name = wtforms.StringField('name',[validators.InputRequired()])
    def validate_name(self,field):
        if len(field.data) > 5:
            raise wtforms.ValidationError(u'超过5个字符')
```

## CSRF保护：

在flask的表单中，默认是开启了`csrf`保护功能的，如果你想关闭表单的`csrf`保护，可以在初始化表单的时候传递`csrf_enabled=False`进去来关闭`csrf`保护。如果你想关闭这种默认的行为。如果你想在没有表单存在的请求视图函数中也添加`csrf`保护，可以开启全局的`csrf`保护功能：

```python
csrf = CsrfProtect()
csrf.init_app(app)
```

或者是针对某一个视图函数，使用`csrf.protect`装饰器来开启`csrf`保护功能。并且如果已经开启了全局的`csrf`保护，想要关闭某个视图函数的`csrf`保护功能，可以使用`csrf.exempt`装饰器来取消本视图函数的保护功能。

### AJAX的CSRF保护：

在`AJAX`中要使用`csrf`保护，则必须手动的添加`X-CSRFToken`到`Header`中。但是`CSRF`从哪里来，还是需要通过模板给渲染，而`Flask`比较推荐的方式是在`meta`标签中渲染`csrf`，如下：

```html
<meta name="csrf-token" content="{{ csrf_token() }}">
```

如果要发送`AJAX`请求，则在发送之前要添加`CSRF`,代码如下（使用了jQuery）：

```html
var csrftoken = $('meta[name=csrf-token]').attr('content')
$.ajaxSetup({
    beforeSend: function(xhr, settings) {
        if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type) && !this.crossDomain) {
            xhr.setRequestHeader("X-CSRFToken", csrftoken)
        }
    }
})
```