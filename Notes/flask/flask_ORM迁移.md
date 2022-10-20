## alembic教程：

`alembic`是`sqlalchemy`的作者开发的。用来做`OMR`模型与数据库的迁移与映射。`alembic`使用方式跟`git`有点了类似，表现在两个方面。

1. 第一个，`alembic`的所有命令都是以`alembic`开头
2. 第二，`alembic`的迁移文件也是通过版本进行控制的。首先，通过`pip install alembic`进行安装。

以下将解释`alembic`的用法。

### 一、操作步骤

#### 1. 初始化仓库：

初始化`alembic`仓库。在终端中，`cd`到你的项目目录中，然后执行命令`alembic init alembic`，创建一个名叫`alembic`的仓库。

#### 2. 创建模型（ORM）类：

比如这里创建一个`models.py`模块，然后在里面定义你的模型类，示例代码如下：

```python
from sqlalchemy import Column,Integer,String,create_engine,Text
from sqlalchemy.orm import sessionmaker
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
class User(Base):
    __tablename__ = 'user'

    id = Column(Integer,primary_key=True)
    username = Column(String(20),nullable=False)
    password = Column(String(100),nullable=False)

class Article(Base):
    __tablename__ = 'article'

    id = Column(Integer,primary_key=True)
    title = Column(String(100),nullable=False)
    content = Column(Text, nullable=False)
```

#### 3. 修改配置文件：

- 在`alembic.ini`中设置数据库的连接，`sqlalchemy.url = driver://user:pass@localhost/dbname`，比如以`mysql`数据库为例，则配置后的代码为：

```python
sqlalchemy.url = mysql+mysqldb://root:root@localhost/alembic_demo?charset=utf8
```

- 为了使用模型类更新数据库，需要在`env.py`文件中设置`target_metadata`，默认为`target_metadata=None`。使用`sys`模块把当前项目的路径导入到`path`中：

```python
import os
import sys
sys.path.append(os.path.dirname(os.path.abspath(__file__)) + "/../")
from models import Base
... #省略代码
target_metadata = Base.metadata # 设置创建模型的元类
... #省略代码
```

#### 4. 生成迁移文件

使用命令`alembic revision --autogenerate -m "message"`可以将当前模型中的状态生成迁移文件。

#### 5. 更新数据库

使用`alembic upgrade head`将刚刚生成的迁移文件，真正映射到数据库中。同理，如果要降级，那么使用`alembic downgrade head`。

#### 6. 重复

如果以后修改了代码，则重复4~5的步骤。

### 二、命令和参数解释：

- init：创建一个`alembic`仓库。
- revision：创建一个新的版本文件。
- –autogenerate：自动将当前模型的修改，生成迁移脚本。
- -m：本次迁移做了哪些修改，用户可以指定这个参数，方便回顾。
- upgrade：将指定版本的迁移文件映射到数据库中，会执行版本文件中的`upgrade`函数。如果有多个迁移脚本没有被映射到数据库中，那么会执行多个迁移脚本。
- [head]：代表最新的迁移脚本的版本号。
- downgrade：会执行指定版本的迁移文件中的`downgrade`函数。
- heads：展示head指向的脚本文件版本号。
- history：列出所有的迁移版本及其信息。
- current：展示当前数据库中的版本号。

另外，在你第一次执行`upgrade`的时候，就会在数据库中创建一个名叫`alembic_version`表，这个表只会有一条数据，记录当前数据库映射的是哪个版本的迁移文件。

### 三、经典错误：

| 错误描述                                                     | 原因                                                         | 解决办法                                                     |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `FAILED: Target database is not up to date.`                 | 主要是`heads`和`current`不相同。`current`落后于`heads`的版本。 | 将current移动到head上。alembic upgrade head                  |
| `FAILED: Can't locate revision identified by '77525ee61b5b'` | 数据库中存的版本号不在迁移脚本文件中                         | 删除数据库的`alembic_version`表中的数据，重新执行`alembic upgrade head` |

## Flask-Migrate插件

在实际的开发环境中，经常会发生数据库修改的行为。一般我们修改数据库不会直接手动的去修改，而是去修改`ORM`对应的模型，然后再把模型映射到数据库中。这时候如果有一个工具能专门做这种事情，就显得非常有用了，而`flask-migrate`就是做这个事情的。`flask-migrate`是基于`Alembic`进行的一个封装，并集成到`Flask`中，而所有的迁移操作其实都是`Alembic`做的，他能跟踪模型的变化，并将变化映射到数据库中。

使用`Flask-Migrate`需要安装，命令如下：

```python
pip install flask-migrate
```

### 一、基本用法：

要让`Flask-Migrate`能够管理`app`中的数据库，需要使用`Migrate(app,db)`来绑定`app`和数据库。假如现在有以下`app`文件：

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from constants import DB_URI
from flask_migrate import Migrate

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = DB_URI
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = True
db = SQLAlchemy(app)
# 绑定app和数据库
migrate = Migrate(app,db)

class User(db.Model):
    id = db.Column(db.Integer,primary_key=True)
    username = db.Column(db.String(20))

    addresses = db.relationship('Address',backref='user')

class Address(db.Model):
    id = db.Column(db.Integer,primary_key=True)
    email_address = db.Column(db.String(50))
    user_id = db.Column(db.Integer,db.ForeignKey('user.id'))

db.create_all()

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run()
```

之后，就可以在命令行中映射`ORM`了。首先需要初始化一个迁移文件夹：

```
flask db init
```

然后再把当前的模型添加到迁移文件中：

```python
flask db migrate
```

最后再把迁移文件中对应的数据库操作，真正的映射到数据库中：

```python
flask db upgrade
```

### 二、项目重构：

现在是所有代码都写在一个文件中，这样会导致文件会越来越乱。所以进行一下项目重构，设置为以下的目录结构：
![migrate.png](https://cdn.jsdelivr.net/gh/leonsoul/TyporaFile@master/uPic/rKAhcte9Fcf6DXAeQATfaV.png)

以下对各个文件的作用进行解释。

#### 1. constants.py文件：

常量文件，用来存放数据库配置。

```python
# constants.py
HOSTNAME = '127.0.0.1'
PORT = '3306'
DATABASE = 'xt_flask_migrate'
USERNAME = 'root'
PASSWORD = 'root'
DB_URI = 'mysql+mysqldb://{}:{}@{}:{}/{}'.format(USERNAME,PASSWORD,HOSTNAME,PORT,DATABASE)
```

#### 2. ext.py文件：

把`db`变量放到一个单独的文件，而不是放在主`app`文件。这样做的目的是为了在大型项目中如果`db`被多个模型文件引用的话，会造成`from your_app import db`这样的方式，但是往往也在`your_app.py`中也会引入模型文件定义的类，这就造成了循环引用。所以最好的办法是把它放在不依赖其他模块的独立文件中。

```python
# ext.py
from flask_sqlalchemy import SQLAlchemy
db = SQLAlchemy()
```

#### 3. models.py文件：

模型文件，用来存放所有的模型，并且注意，因为这里使用的是`flask-script`的方式进行模型和表的映射，因此不需要使用`db.create_all()`的方式创建数据库。

```python
# models.py
from ext import db
class User(db.Model): 
    id = db.Column(db.Integer,primary_key=True) 
    username = db.Column(db.String(50)) 
    addresses = db.relationship('Address',backref='user') 

    def __init__(self,username): 
        self.username = username

class Address(db.Model): 
    id = db.Column(db.Integer,primary_key=True) 
    email_address = db.Column(db.String(50)) 
    user_id = db.Column(db.Integer,db.ForeignKey('user.id')) 

    def __init__(self,email_address): 
        self.email_address = email_address
```

#### 4. app.py文件：

这个是主`app`文件，运行文件。并且因为`db`被放到另外一个文件中，所以使用`db.init_app(app)`的方式来绑定数据库。

```python
# app.py
from flask import Flask
from ext import db

app = Flask(__name__)
db.init_app(app)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run()
```

### 三、迁移命令：

通过以上项目重构后，后续我们就只要通过以下三个命令即可完成迁移操作了。

1. 运行命令来初始化迁移文件：

    ```python
    flask db init
    ```

2. 运行命令来将模型的映射添加到文件中：

    ```python
    flask db migrate
    ```

3. 添加将映射文件真正的映射到数据库中：

    ```python
    flask db upgrade
    ```