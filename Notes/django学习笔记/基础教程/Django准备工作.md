## 虚拟环境

#### 为什么需要虚拟环境：

到目前位置，我们所有的第三方包安装都是直接通过`pip install xx`的方式进行安装的，这样安装会将那个包安装到你的系统级的`Python`环境中。但是这样有一个问题，就是如果你现在用`Django 1.10.x`写了个网站，然后你的领导跟你说，之前有一个旧项目是用`Django 0.9`开发的，让你来维护，但是`Django 1.10`不再兼容`Django 0.9`的一些语法了。这时候就会碰到一个问题，我如何在我的电脑中同时拥有`Django 1.10`和`Django 0.9`两套环境呢？这时候我们就可以通过虚拟环境来解决这个问题。

#### 虚拟环境原理介绍：

虚拟环境相当于一个抽屉，在这个抽屉中安装的任何软件包都不会影响到其他抽屉。并且在项目中，我可以指定这个项目的虚拟环境来配合我的项目。比如我们现在有一个项目是基于`Django 1.10.x`版本，又有一个项目是基于`Django 0.9.x`的版本，那么这时候就可以创建两个虚拟环境，在这两个虚拟环境中分别安装`Django 1.10.x`和`Django 0.9.x`来适配我们的项目。

#### 安装`virtualenv`：

`virtualenv`是用来创建虚拟环境的软件工具，我们可以通过`pip`或者`pip3`来安装：

```
pip install virtualenv
pip3 install virtualenv
```

#### 创建虚拟环境：

创建虚拟环境非常简单，通过以下命令就可以创建了：

```
virtualenv [虚拟环境的名字]
```

如果你当前的`Python3/Scripts`的查找路径在`Python2/Scripts`的前面，那么将会使用`python3`作为这个虚拟环境的解释器。如果`python2/Scripts`在`python3/Scripts`前面，那么将会使用`Python2`来作为这个虚拟环境的解释器。

#### 进入环境：

虚拟环境创建好了以后，那么可以进入到这个虚拟环境中，然后安装一些第三方包，进入虚拟环境在不同的操作系统中有不同的方式，一般分为两种，第一种是`Windows`，第二种是`*nix`：

1. `windows`进入虚拟环境：进入到虚拟环境的`Scripts`文件夹中，然后执行`activate`。
2. `*nix`进入虚拟环境：`source /path/to/virtualenv/bin/activate`
    一旦你进入到了这个虚拟环境中，你安装包，卸载包都是在这个虚拟环境中，不会影响到外面的环境。

#### 退出虚拟环境：

退出虚拟环境很简单，通过一个命令就可以完成：`deactivate`。

#### 创建虚拟环境的时候指定`Python`解释器：

在电脑的环境变量中，一般是不会去更改一些环境变量的顺序的。也就是说比如你的`Python2/Scripts`在`Python3/Scripts`的前面，那么你不会经常去更改他们的位置。但是这时候我确实是想在创建虚拟环境的时候用`Python3`这个版本，这时候可以通过`-p`参数来指定具体的`Python`解释器：

```python
virtualenv -p C:\Python36\python.exe [virutalenv name]
```

------

### virtualenvwrapper：

`virtualenvwrapper`这个软件包可以让我们管理虚拟环境变得更加简单。不用再跑到某个目录下通过`virtualenv`来创建虚拟环境，并且激活的时候也要跑到具体的目录下去激活。

#### 安装`virtualenvwrapper`：

1. *nix：`pip install virtualenvwrapper`。
2. windows：`pip install virtualenvwrapper-win`。

#### `virtualenvwrapper`基本使用：

1. 创建虚拟环境：

```python
mkvirtualenv my_env
```

那么会在你当前用户下创建一个`Env`的文件夹，然后将这个虚拟环境安装到这个目录下。
如果你电脑中安装了`python2`和`python3`，并且两个版本中都安装了`virtualenvwrapper`，那么将会使用环境变量中第一个出现的`Python`版本来作为这个虚拟环境的`Python`解释器。

1. 切换到某个虚拟环境：

```python
workon my_env
```

1. 退出当前虚拟环境：

```python
deactivate
```

1. 删除某个虚拟环境：

```python
rmvirtualenv my_env
```

1. 列出所有虚拟环境：

```python
lsvirtualenv
```

1. 进入到虚拟环境所在的目录：

```python
cdvirtualenv
```

#### 修改`mkvirtualenv`的默认路径：

在`我的电脑->右键->属性->高级系统设置->环境变量->系统变量`中添加一个参数`WORKON_HOME`，将这个参数的值设置为你需要的路径。

#### 创建虚拟环境的时候指定`Python`版本：

在使用`mkvirtualenv`的时候，可以指定`--python`的参数来指定具体的`python`路径：

```
mkvirtualenv --python==C:\Python36\python.exe hy_env
```

## 准备工作

在学习`Django`之前，需要做好以下准备工作：

1. 确保已经安装`Python 3.8`以上的版本，教学以`Python 3.8`版本进行讲解。
2. 安装`virtualenvwrapper`，这个是用来创建虚拟环境的包，使用虚拟环境可以让我们的包管理更加的方便，也为以后项目上线需要安装哪些包做好了准备工作。安装方式在不同的操作系统有区别。以下解释下：

- windows：`pip instal virtualenvwrapper-win`。
- linux/mac：`pip install virtualenvwrapper`。

1. 虚拟环境相关操作：

- 创建虚拟环境：`mkvirtualenv --python='[python3.6文件所在路径]' [虚拟环境名字] `。比如`mkvirtualenv --python='C:\Python38\python.exe' django-env`。
- 进入到虚拟环境：`workon [虚拟环境名称]`。比如`workon django-env`。
- 退出虚拟环境：`deactivate`。

1. 首先进入到虚拟环境`workon django-env`，然后通过`pip install django==3.2.3`安装`django`，教学以`Django 3.2`版本为例进行讲解。
2. 安装`pycharm profession 2021版`或者`Sublime Text 3`等任意一款你喜欢的编辑器。（推荐使用`pycharm`，如果由于电脑性能原因，可以退而求其次使用`Sublime Text`）。**如果使用`pycharm`，切记一定要下载profession（专业版），community（社区版）不能用于网页开发**。至于破解和正版，大家到网上搜下就知道啦。
3. 安装最新版`MySQL`，`windows`版的`MySQL`的下载地址是：`https://dev.mysql.com/downloads/windows/installer/8.0.html`。如果你用的是其他操作系统，那么可以来到这个界面选择具体的`MySQL`来进行下载：`https://dev.mysql.com/downloads/mysql/`。
4. 安装`pymysql`，这个库是`Python`来操作数据库的。没有他，`django`就不能操作数据库。安装方式也比较简单，`pip install pymysql`就可以啦。

## Django介绍：

Django，发音为[`dʒæŋɡəʊ]，Django诞生于2003年秋天，2005年发布正式版本，由Simon和Andrian开发。当时两位作者的老板和记者要他们几天甚至几个小时之内增加新的功能。两人不得已开发了Django这套框架以实现快速开发目的，因此Django生来就是为了节省开发者时间的。Django发展至今，被许许多多国内外的开发者使用，已经成为web开发者的首选框架。因此，如果你是用python来做网站，没有理由不学好Django。
选读：[Python+Django如何支撑了7 亿月活用户的Instagram？](https://www.infoq.cn/article/instagram-pycon-2017)

### Django版本和Python版本：

![django和python版本.png](https://cdn.jsdelivr.net/gh/leonsoul/TyporaFile@master/uPic/gbFCi28a7VHcA82Pg5uvQi.png)

### web服务器和应用服务器以及web应用框架：

- **web服务器**：负责处理http请求，响应静态文件，常见的有`Apache`，`Nginx`以及微软的`IIS`.
- **应用服务器**：负责处理逻辑的服务器。比如`php`、`python`的代码，是不能直接通过`nginx`这种**web服务器**来处理的，只能通过**应用服务器**来处理，常见的应用服务器有`uwsgi`、`tomcat`等。
- **web应用框架**：一般使用某种语言，封装了常用的`web`功能的框架就是**web应用框架**，`flask`、`Django`以及Java中的`SSH(Structs2+Spring3+Hibernate3)`框架都是web应用框架。

### Django和MVC：

Django是一个遵循`MVC`设计模式的框架，`MVC`是`Model`、`View`、`Controller`的三个单词的简写。分别代表`模型`、`视图`、`控制器`。以下图片说明这三者之间的关系：

![MVC.png](https://cdn.jsdelivr.net/gh/leonsoul/TyporaFile@master/uPic/Bq9HbeuxFTPFsM39QR7AjL-20221020151619350.png)

而`Django`其实也是一个`MTV`的设计模式。`MTV`是`Model`、`Template`、`View`三个单词的简写。分别代表`模型`、`模版`、`视图`。以下图片说明这三者之间的关系：

![MTV.png](https://cdn.jsdelivr.net/gh/leonsoul/TyporaFile@master/uPic/NEn5iGeZu5Qi4Ak5LqqGsa-20221020151748683.png)

### 更多：

1. `Django`的官网：https://www.djangoproject.com/

## URL组成部分详解：

`URL`是`Uniform Resource Locator`的简写，统一资源定位符。

一个`URL`由以下几部分组成：

```python
scheme://host:port/path/?query-string=xxx#anchor
```

- **scheme**：代表的是访问的协议，一般为`http`或者`https`以及`ftp`等。
- **host**：主机名，域名，比如`www.baidu.com`。
- **port**：端口号。当你访问一个网站的时候，浏览器默认使用80端口。
- **path**：查找路径。比如：`www.jianshu.com/trending/now`，后面的`trending/now`就是`path`。
- **query-string**：查询字符串，比如：`www.baidu.com/s?wd=python`，后面的`wd=python`就是查询字符串。
- **anchor**：锚点，后台一般不用管，前端用来做页面定位的。

注意：`URL`中的所有字符都是`ASCII`字符集，如果出现非`ASCII`字符，比如中文，浏览器会进行编码再进行传输