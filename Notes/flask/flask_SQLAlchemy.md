## SQLAlchemy介绍

### 一、安装：

`SQLAlchemy`是一个数据库的`ORM`框架，让我们操作数据库的时候不要再用`SQL`语句了，跟直接操作模型一样。安装命令为：`pip install SQLAlchemy`。

### 二、通过`SQLAlchemy`连接数据库：

首先来看一段代码：

```python
from sqlalchemy import create_engine

# 数据库的配置变量
HOSTNAME = '127.0.0.1'
PORT     = '3306'
DATABASE = 'xt_flask'
USERNAME = 'root'
PASSWORD = 'root'
DB_URI = 'mysql+pymysql://{}:{}@{}:{}/{}'.format(USERNAME,PASSWORD,HOSTNAME,PORT,DATABASE)

# 创建数据库引擎
engine = create_engine(DB_URI)

#创建连接
with engine.connect() as con:
    rs = con.execute('SELECT 1')
    print rs.fetchone()
```

首先从`sqlalchemy`中导入`create_engine`，用这个函数来创建引擎，然后用`engine.connect()`来连接数据库。其中一个比较重要的一点是，通过`create_engine`函数的时候，需要传递一个满足某种格式的字符串，对这个字符串的格式来进行解释：

```python
dialect+driver://username:password@host:port/database?charset=utf8
```

`dialect`是数据库的实现，比如`MySQL`、`PostgreSQL`、`SQLite`，并且转换成小写。`driver`是`Python`对应的驱动，如果不指定，会选择默认的驱动，比如MySQL的默认驱动是`MySQLdb`。`username`是连接数据库的用户名，`password`是连接数据库的密码，`host`是连接数据库的域名，`port`是数据库监听的端口号，`database`是连接哪个数据库的名字。

如果以上输出了`1`，说明`SQLAlchemy`能成功连接到数据库。

### 二、用SQLAlchemy执行原生SQL：

我们将上一个例子中的数据库配置选项单独放在一个`constants.py`的文件中，看以下例子：

```python
from sqlalchemy import create_engine
from constants import DB_URI

#连接数据库
engine = create_engine(DB_URI,echo=True)

# 使用with语句连接数据库，如果发生异常会被捕获
with engine.connect() as con:
    # 先删除users表
    con.execute('drop table if exists authors')
    # 创建一个users表，有自增长的id和name
    con.execute('create table authors(id int primary key auto_increment,name varchar(25))')
    # 插入两条数据到表中
    con.execute('insert into persons(name) values("abc")')
    con.execute('insert into persons(name) values("xiaotuo")')
    # 执行查询操作
    results = con.execute('select * from persons')
    # 从查找的结果中遍历
    for result in results:
        print(result)
```

## SQLAlchemy基本使用

### 一、使用SQLAlchemy：

#### 1. 创建`ORM`模型：

要使用`ORM`来操作数据库，首先需要创建一个类来与对应的表进行映射。现在以`User表`来做为例子，它有`自增长的id`、`name`、`fullname`、`password`这些字段，那么对应的类为：

```python
from sqlalchemy import Column,Integer,String
from constants import DB_URI
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base

engine = create_engine(DB_URI,echo=True)

# 所有的类都要继承自`declarative_base`这个函数生成的基类
Base = declarative_base(engine)
class User(Base):
    # 定义表名为users
    __tablename__ = 'users'

    # 将id设置为主键，并且默认是自增长的
    id = Column(Integer,primary_key=True)
    # name字段，字符类型，最大的长度是50个字符
    name = Column(String(50))
    fullname = Column(String(50))
    password = Column(String(100))

    # 让打印出来的数据更好看，可选的
    def __repr__(self):
        return "<User(id='%s',name='%s',fullname='%s',password='%s')>" % (self.id,self.name,self.fullname,self.password)
```

#### 2. 映射到数据库中：

`SQLAlchemy`会自动的设置第一个`Integer`的主键并且没有被标记为外键的字段添加自增长的属性。因此以上例子中`id`自动的变成自增长的。以上创建完和表映射的类后，还没有真正的映射到数据库当中，执行以下代码将类映射到数据库中：

```python
Base.metadata.create_all()
```

#### 3. 添加数据到表中：

在创建完数据表，并且做完和数据库的映射后，接下来让我们添加数据进去：

```python
ed_user = User(name='ed',fullname='Ed Jones',password='edspassword')
# 打印名字
print ed_user.name
> ed
# 打印密码
print ed_user.password
> edspassword
# 打印id
print(ed_user.id)
> None
```

可以看到，name和password都能正常的打印，唯独`id`为`None`，这是因为`id`是一个自增长的主键，还未插入到数据库中，`id`是不存在的。接下来让我们把创建的数据插入到数据库中。和数据库打交道的，是一个叫做`Session`的对象：

```python
from sqlalchemy.orm import sessionmaker
Session = sessionmaker(bind=engine)
# 或者
# Session = sessionmaker()
# Session.configure(bind=engine)
session = Session()
ed_user = User(name='ed',fullname='Ed Jones',password='edspassword')
session.add(ed_user)
```

现在只是把数据添加到`session`中，但是并没有真正的把数据存储到数据库中。如果需要把数据存储到数据库中，还要做一次`commit`操作：

```python
session.commit()
# 打印ed_user的id
print ed_user.id
> 1
```

#### 4. 回滚：

这时候，`ed_user`就已经有id。 说明已经插入到数据库中了。有人肯定有疑问了，为什么添加到`session`中后还要做一次`commit`操作呢，这是因为，在`SQLAlchemy`的`ORM`实现中，在做`commit`操作之前，所有的操作都是在事务中进行的，因此如果你要将事务中的操作真正的映射到数据库中，还需要做`commit`操作。既然用到了事务，这里就并不能避免的提到一个回滚操作了，那么看以下代码展示了如何使用回滚（接着以上示例代码）：

```python
# 修改ed_user的用户名
ed_user.name = 'Edwardo'
# 创建一个新的用户
fake_user = User(name='fakeuser',fullname='Invalid',password='12345')
# 将新创建的fake_user添加到session中
session.add(fake_user)
# 判断`fake_user`是否在`session`中存在
print fake_user in session
> True
# 从数据库中查找name=Edwardo的用户
tmp_user = session.query(User).filter_by(name='Edwardo')
# 打印tmp_user的name
print tmp_user
# 打印出查找到的tmp_user对象，注意这个对象的name属性已经在事务中被修改为Edwardo了。
> <User(name='Edwardo', fullname='Ed Jones', password='edspassword')>
# 刚刚所有的操作都是在事务中进行的，现在来做回滚操作
session.rollback()
# 再打印tmp_user
print tmp_user
> <User(name='ed', fullname='Ed Jones', password='edspassword')>
# 再看fake_user是否还在session中
print fake_user in session
> False
```

#### 5. 查找数据：

接下来看下如何进行查找操作，查找操作是通过`session.query()`方法实现的，这个方法会返回一个`Query`对象，`Query`对象相当于一个数组，装载了查找出来的数据，并且可以进行迭代。具体里面装的什么数据，就要看向`session.query()`方法传的什么参数了，如果只是传一个`ORM`的类名作为参数，那么提取出来的数据就是都是这个类的实例，比如：

```python
for instance in session.query(User).order_by(User.id):
    print instance
# 输出所有的user实例
> <User (id=2,name='ed',fullname='Ed Json',password='12345')>
> <User (id=3,name='be',fullname='Be Engine',password='123456')>
```

如果传递了两个及其两个以上的对象，或者是传递的是`ORM`类的属性，那么查找出来的就是元组，例如：

```python
for instance in session.query(User.name):
    print instance
# 输出所有的查找结果
> ('ed',)
> ('be',)
```

以及：

```python
for instance in session.query(User.name,User.fullname):
    print instance
# 输出所有的查找结果
> ('ed', 'Ed Json')
> ('be', 'Be Engine')
```

或者是：

```python
for instance in session.query(User,User.name).all():
    print instance
# 输出所有的查找结果
> (<User (id=2,name='ed',fullname='Ed Json',password='12345')>, 'Ed Json')
> (<User (id=3,name='be',fullname='Be Engine',password='123456')>, 'Be Engine')
```

另外，还可以对查找的结果（`Query`）做切片操作：

```python
for instance in session.query(User).order_by(User.id)[1:3]
    instance
```

如果想对结果进行过滤，可以使用`filter_by`和`filter`两个方法，这两个方法都是用来做过滤的，区别在于，`filter_by`是传入关键字参数，`filter`是传入条件判断，并且`filter`能够传入的条件更多更灵活，请看以下例子：

```python
# 第一种：使用filter_by过滤：
for name in session.query(User.name).filter_by(fullname='Ed Jones'):
    print name
# 输出结果：
> ('ed',)

# 第二种：使用filter过滤：
for name in session.query(User.name).filter(User.fullname=='Ed Jones'):
    print name
# 输出结果：
> ('ed',)
```

### 二、Column常用参数：

- `default`：默认值。
- `nullable`：是否可空。
- `primary_key`：是否为主键。
- `unique`：是否唯一。
- `autoincrement`：是否自动增长。
- `onupdate`：更新的时候执行的函数。
- `name`：该属性在数据库中的字段映射。

### 三、sqlalchemy常用数据类型：

- `Integer`：整形。
- `Float`：浮点类型。
- `Boolean`：传递`True/False`进去。
- `DECIMAL`：定点类型。
- `enum`：枚举类型。
- `Date`：传递`datetime.date()`进去。
- `DateTime`：传递`datetime.datetime()`进去。
- `Time`：传递`datetime.time()`进去。
- `String`：字符类型，使用时需要指定长度，区别于`Text`类型。
- `Text`：文本类型。
- `LONGTEXT`：长文本类型。

## 查找操作

### 一、query可用参数：

1. 模型对象。指定查找这个模型中所有的对象。
2. 模型中的属性。可以指定只查找某个模型的其中几个属性。
3. 聚合函数。
    - `func.count`：统计行的数量。
    - `func.avg`：求平均值。
    - `func.max`：求最大值。
    - `func.min`：求最小值。
    - `func.sum`：求和。

### 二、过滤条件：

过滤是数据提取的一个很重要的功能，以下对一些常用的过滤条件进行解释，并且这些过滤条件都是只能通过`filter`方法实现的：

1. `equals`：

    ```python
    query.filter(User.name == 'ed')
    ```

2. `not equals`:

    ```python
    query.filter(User.name != 'ed')
    ```

3. `like`：

    ```python
    query.filter(User.name.like('%ed%'))
    ```

4. `in`：

    ```python
    query.filter(User.name.in_(['ed','wendy','jack']))
    # 同时，in也可以作用于一个Query
    query.filter(User.name.in_(session.query(User.name).filter(User.name.like('%ed%'))))
    ```

5. `not in`：

    ```python
    query.filter(~User.name.in_(['ed','wendy','jack']))
    ```

6. `is null`：

    ```python
    query.filter(User.name==None)
    # 或者是
    query.filter(User.name.is_(None))
    ```

7. `is not null`:

    ```python
    query.filter(User.name != None)
    # 或者是
    query.filter(User.name.isnot(None))
    ```

8. `and`：

    ```python
    from sqlalchemy import and_
    query.filter(and_(User.name=='ed',User.fullname=='Ed Jones'))
    # 或者是传递多个参数
    query.filter(User.name=='ed',User.fullname=='Ed Jones')
    # 或者是通过多次filter操作
    query.filter(User.name=='ed').filter(User.fullname=='Ed Jones')
    ```

9. `or`：

    ```python
    from sqlalchemy import or_  query.filter(or_(User.name=='ed',User.name=='wendy'))
    ```

### 三、查找方法：

介绍完过滤条件后，有一些经常用到的查找数据的方法也需要解释一下：

1. `all()`：返回一个`Python`列表（`list`）：

    ```python
    query = session.query(User).filter(User.name.like('%ed%').order_by(User.id)
    # 输出query的类型
    print type(query)
    > <type 'list'>
    # 调用all方法
    query = query.all()
    # 输出query的类型
    print type(query)
    > <class 'sqlalchemy.orm.query.Query'>
    ```

2. `first()`：返回`Query`中的第一个值：

    ```python
    user = session.query(User).first()
    print user
    > <User(name='ed', fullname='Ed Jones', password='f8s7ccs')>
    ```

3. `one()`：查找所有行作为一个结果集，如果结果集中只有一条数据，则会把这条数据提取出来，如果这个结果集少于或者多于一条数据，则会抛出异常。总结一句话：有且只有一条数据的时候才会正常的返回，否则抛出异常：

    ```python
    # 多于一条数据
    user = query.one()
    > Traceback (most recent call last):
    > ...
    > MultipleResultsFound: Multiple rows were found for one()
    # 少于一条数据
    user = query.filter(User.id == 99).one()
    > Traceback (most recent call last):
    > ...
    > NoResultFound: No row was found for one()
    # 只有一条数据
    > query(User).filter_by(name='ed').one()
    ```

4. `one_or_none()`：跟`one()`方法类似，但是在结果集中没有数据的时候也不会抛出异常。

5. `scalar()`：底层调用`one()`方法，并且如果`one()`方法没有抛出异常，会返回查询到的第一列的数据：

    ```python
    session.query(User.name,User.fullname).filter_by(name='ed').scalar()
    ```

### 四、文本SQL：

`SQLAlchemy`还提供了使用**文本SQL**的方式来进行查询，这种方式更加的灵活。而文本SQL要装在一个`text()`方法中，看以下例子：

```python
from sqlalchemy import text
for user in session.query(User).filter(text("id<244")).order_by(text("id")).all():
    print user.name
```

如果过滤条件比如上例中的244存储在变量中，这时候就可以通过传递参数的形式进行构造：

```python
session.query(User).filter(text("id<:value and name=:name")).params(value=224,name='ed').order_by(User.id)
```

在文本SQL中的变量前面使用了`:`来区分，然后使用`params`方法，指定需要传入进去的参数。另外，使用`from_statement`方法可以把过滤的函数和条件函数都给去掉，使用纯文本的SQL:

```python
sesseion.query(User).from_statement(text("select * from users where name=:name")).params(name='ed').all()
```

使用`from_statement`方法一定要注意，`from_statement`返回的是一个`text`里面的查询语句，一定要记得调用`all()`方法来获取所有的值。

### 五、计数（Count）：

`Query`对象有一个非常方便的方法来计算里面装了多少数据：

```python
session.query(User).filter(User.name.like('%ed%')).count()
```

当然，有时候你想明确的计数，比如要统计`users`表中有多少个不同的姓名，那么简单粗暴的采用以上`count`是不行的，因为姓名有可能会重复，但是处于两条不同的数据上，如果在原生数据库中，可以使用`distinct`关键字，那么在`SQLAlchemy`中，可以通过`func.count()`方法来实现：

```python
from sqlalchemy import func
session.query(func.count(User.name),User.name).group_by(User.name).all()
## 输出的结果
> [(1, u'ed'), (1, u'fred'), (1, u'mary'), (1, u'wendy')]
```

另外，如果想实现`select count(*) from users`，可以通过以下方式来实现：

```python
session.query(func.count(*)).select_from(User).scalar()
```

当然，如果指定了要查找的表的字段，可以省略`select_from()`方法：

```python
session.query(func.count(User.id)).scalar()
```

## 表关系：

表之间的关系存在三种：一对一、一对多、多对多。而`SQLAlchemy`中的`ORM`也可以模拟这三种关系。因为一对一其实在`SQLAlchemy`中底层是通过一对多的方式模拟的，所以先来看下一对多的关系：

### 一、外键：

在Mysql中，外键可以让表之间的关系更加紧密。而SQLAlchemy同样也支持外键。通过ForeignKey类来实现，并且可以指定表的外键约束。相关示例代码如下：

```python
class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer,primary_key=True,autoincrement=True)
    title = Column(String(50),nullable=False)
    content = Column(Text,nullable=False)
    uid = Column(Integer,ForeignKey('user.id'))

    def __repr__(self):
        return "<Article(title:%s)>" % self.title

class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True,autoincrement=True)
    username = Column(String(50),nullable=False)
```

外键约束有以下几项：

1. `RESTRICT`：父表数据被删除，会阻止删除。默认就是这一项。
2. `NO ACTION`：在MySQL中，同`RESTRICT`。
3. `CASCADE`：级联删除。
4. `SET NULL`：父表数据被删除，子表数据会设置为NULL。

### 二、一对多：

拿之前的`User`表为例，假如现在要添加一个功能，要保存用户的邮箱帐号，并且邮箱帐号可以有多个，这时候就必须创建一个新的表，用来存储用户的邮箱，然后通过`user.id`来作为外键进行引用，先来看下邮箱表的实现：

```python
from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship

class Address(Base):
    __tablename__ = 'address'
    id = Column(Integer,primary_key=True)
    email_address = Column(String,nullable=False)
    # User表的外键，指定外键的时候，是使用的是数据库表的名称，而不是类名
    user_id = Column(Integer,ForeignKey('users.id'))
    # 在ORM层面绑定两者之间的关系，第一个参数是绑定的表的类名，
    # 第二个参数back_populates是通过User反向访问时的字段名称
    user = relationship('User',back_populates="addresses")

    def __repr__(self):
        return "<Address(email_address='%s')>" % self.email_address

## 重新修改User表，添加了addresses字段，引用了Address表的主键
class User(Base):
    __tablename__ = 'users'
    id = Column(Integer,primary_key=True)
    name = Column(String(50))
    fullname = Column(String(50))
    password = Column(String(100))
    # 在ORM层面绑定和`Address`表的关系
    addresses = relationship("Address",order_by=Address.id,back_populates="user")
```

其中，在`User`表中添加的`addresses`字段，可以通过`User.addresses`来访问和这个user相关的所有address。在`Address`表中的`user`字段，可以通过`Address.user`来访问这个user。达到了双向绑定。表关系已经建立好以后，接下来就应该对其进行操作，先看以下代码：

```python
jack = User(name='jack',fullname='Jack Bean',password='gjffdd')
jack.addresses = [Address(email_address='jack@google.com'),
                Address(email_address='j25@yahoo.com')]
session.add(jack)
session.commit()
```

首先，创建一个用户，然后对这个`jack`用户添加两个邮箱，最后再提交到数据库当中，可以看到这里操作`Address`并没有直接进行保存，而是先添加到用户里面，再保存。

### 三、一对一：

一对一其实就是一对多的特殊情况，从以上的一对多例子中不难发现，一对应的是`User`表，而多对应的是`Address`，也就是说一个`User`对象有多个`Address`。因此要将一对多转换成一对一，只要设置一个`User`对象对应一个`Address`对象即可，看以下示例：

```python
class User(Base):
  __tablename__ = 'users'
  id = Column(Integer,primary_key=True)
  name = Column(String(50))
  fullname = Column(String(50))
  password = Column(String(100))
  # 设置uselist关键字参数为False
  addresses = relationship("Address",back_populates='addresses',uselist=False)
class Address(Base):
  __tablename__ = 'addresses'
  id = Column(Integer,primary_key=True)
  email_address = Column(String(50))
  user_id = Column(Integer,ForeignKey('users.id')
  user = relationship('Address',back_populates='user')
```

从以上例子可以看到，只要在`User`表中的`addresses`字段上添加`uselist=False`就可以达到一对一的效果。设置了一对一的效果后，就不能添加多个邮箱到`user.addresses`字段了，只能添加一个：

```
user.addresses = Address(email_address='ed@google.com')
```

### 四、多对多：

多对多需要一个中间表来作为连接，同理在`sqlalchemy`中的`orm`也需要一个中间表。假如现在有一个`Teacher`和一个`Classes`表，即老师和班级，一个老师可以教多个班级，一个班级有多个老师，是一种典型的多对多的关系，那么通过`sqlalchemy`的`ORM`的实现方式如下：

```python
association_table = Table('teacher_classes',Base.metadata,
	Column('teacher_id',Integer,ForeignKey('teacher.id')),
	Column('classes_id',Integer,ForeignKey('classes.id'))
)

class Teacher(Base):
    __tablename__ = 'teacher'
    id = Column(Integer,primary_key=True)
    tno = Column(String(10))
    name = Column(String(50))
    age = Column(Integer)
    classes = relationship('Classes',secondary=association_table,back_populates='teachers')

class Classes(Base):
    __tablename__ = 'classes'
    id = Column(Integer,primary_key=True)
    cno = Column(String(10))
    name = Column(String(50))
    teachers = relationship('Teacher',secondary=association_table,back_populates='classes')
```

要创建一个多对多的关系表，首先需要一个中间表，通过`Table`来创建一个中间表。上例中第一个参数`teacher_classes`代表的是中间表的表名，第二个参数是`Base`的元类，第三个和第四个参数就是要连接的两个表，其中`Column`第一个参数是表示的是连接表的外键名，第二个参数表示这个外键的类型，第三个参数表示要外键的表名和字段。
创建完中间表以后，还需要在两个表中进行绑定，比如在`Teacher`中有一个`classes`属性，来绑定`Classes`表，并且通过`secondary`参数来连接中间表。同理，`Classes`表连接`Teacher`表也是如此。定义完类后，之后就是添加数据，请看以下示例：

```python
teacher1 = Teacher(tno='t1111',name='xiaotuo',age=10)
teacher2 = Teacher(tno='t2222',name='datuo',age=10)
classes1 = Classes(cno='c1111',name='english')
classes2 = Classes(cno='c2222',name='math')
teacher1.classes = [classes1,classes2]
teacher2.classes = [classes1,classes2]
classes1.teachers = [teacher1,teacher2]
classes2.teachers = [teacher1,teacher2]
session.add(teacher1)
session.add(teacher2)
session.add(classes1)
session.add(classes2)
```

### 五、`ORM`层面的`CASCADE`：

如果将数据库的外键设置为`RESTRICT`，那么在`ORM`层面，删除了父表中的数据，那么从表中的数据将会`NULL`。如果不想要这种情况发生，那么应该将这个值的`nullable=False`。

在`SQLAlchemy`，只要将一个数据添加到`session`中，和他相关联的数据都可以一起存入到数据库中了。这些是怎么设置的呢？其实是通过`relationship`的时候，有一个关键字参数`cascade`可以设置这些属性：

1. `save-update`：默认选项。在添加一条数据的时候，会把其他和他相关联的数据都添加到数据库中。这种行为就是`save-update`属性影响的。
2. `delete`：表示当删除某一个模型中的数据的时候，是否也删掉使用`relationship`和他关联的数据。
3. `delete-orphan`：表示当对一个ORM对象解除了父表中的关联对象的时候，自己便会被删除掉。当然如果父表中的数据被删除，自己也会被删除。这个选项只能用在一对多上，不能用在多对多以及多对一上。并且还需要在子模型中的`relationship`中，增加一个`single_parent=True`的参数。
4. `merge`：默认选项。当在使用`session.merge`，合并一个对象的时候，会将使用了`relationship`相关联的对象也进行`merge`操作。
5. `expunge`：移除操作的时候，会将相关联的对象也进行移除。这个操作只是从session中移除，并不会真正的从数据库中删除。
6. `all`：是对`save-update, merge, refresh-expire, expunge, delete`几种的缩写。

## 查询高级

### 一、排序：

1. `order_by`：可以指定根据这个表中的某个字段进行排序，如果在前面加了一个`-`，代表的是降序排序。

2. 在模型定义的时候指定默认排序：有些时候，不想每次在查询的时候都指定排序的方式，可以在定义模型的时候就指定排序的方式。有以下两种方式：

    - relationship的order_by参数：在指定`relationship`的时候，传递`order_by`参数来指定排序的字段。

    - 在模型定义中，添加以下代码：

        ```python
         __mapper_args__ = {
             "order_by": title
           }
        ```

        即可让文章使用标题来进行排序。

3. 正向排序和反向排序：默认情况是从小到大，从前到后排序的，如果想要反向排序，可以调用排序的字段的`desc`方法。

### 二、limit、offset和切片：

1. `limit`：可以限制每次查询的时候只查询几条数据。
2. `offset`：可以限制查找数据的时候过滤掉前面多少条。
3. 切片：可以对`Query`对象使用切片操作，来获取想要的数据。

### 三、懒加载：

在一对多，或者多对多的时候，如果想要获取多的这一部分的数据的时候，往往能通过一个属性就可以全部获取了。比如有一个作者，想要或者这个作者的所有文章，那么可以通过`user.articles`就可以获取所有的。但有时候我们不想获取所有的数据，比如只想获取这个作者今天发表的文章，那么这时候我们可以给`relationship`传递一个`lazy='dynamic'`，以后通过`user.articles`获取到的就不是一个列表，而是一个`AppendQuery`对象了。这样就可以对这个对象再进行一层过滤和排序等操作。

### 四、查询高级：

#### 1. group_by：

根据某个字段进行分组。比如想要根据性别进行分组，来统计每个分组分别有多少人，那么可以使用以下代码来完成：

```
session.query(User.gender,func.count(User.id)).group_by(User.gender).all()
```

#### 2. having：

`having`是对查找结果进一步过滤。比如只想要看未成年人的数量，那么可以首先对年龄进行分组统计人数，然后再对分组进行`having`过滤。示例代码如下：

```
result = session.query(User.age,func.count(User.id)).group_by(User.age).having(User.age >= 18).all()
```

#### 3. join方法：

`join`查询分为两种，一种是`inner join`，另一种是`outer join`。默认的是`inner join`，如果指定`left join`或者是`right join`则为`outer join`。如果想要查询`User`及其对应的`Address`，则可以通过以下方式来实现：

```python
for u,a in session.query(User,Address).filter(User.id==Address.user_id).all():
  print(u)
  print(a)
## 输出结果：
> <User (id=1,name='ed',fullname='Ed Jason',password='123456')>
> <Address id=4,email=ed@google.com,user_id=1>
```

这是通过普通方式的实现，也可以通过`join`的方式实现，更加简单：

```python
for u,a in session.query(User,Address).join(Address).all():
  print(u)
  print(a)
# 输出结果：
> <User (id=1,name='ed',fullname='Ed Jason',password='123456')>
> <Address id=4,email=ed@google.com,user_id=1>
```

当然，如果采用`outerjoin`，可以获取所有`user`，而不用在乎这个`user`是否有`address`对象，并且`outerjoin`默认为左外查询：

```
for instance in session.query(User,Address).outerjoin(Address).all():
  print(instance)

# 输出结果：
(<User (id=1,name='ed',fullname='Ed Jason',password='123456')>, <Address id=4,email=ed@google.com,user_id=1>)
(<User (id=2,name='xt',fullname='xiaotuo',password='123')>, None)
```

#### 4. 别名：

当多表查询的时候，有时候同一个表要用到多次，这时候用别名就可以方便的解决命名冲突的问题了：

```python
from sqlalchemy.orm import aliased
adalias1 = aliased(Address)
adalias2 = aliased(Address)
for username,email1,email2 in session.query(User.name,adalias1.email_address,adalias2.email_address).join(adalias1).join(adalias2).all():
  print(username,email1,email2)
```

### 5. 子查询：

`sqlalchemy`也支持子查询，比如现在要查找一个用户的用户名以及该用户的邮箱地址数量。要满足这个需求，可以在子查询中找到所有用户的邮箱数（通过group by合并同一用户），然后再将结果放在父查询中进行使用：

```python
from sqlalchemy.sql import func
# 构造子查询
stmt = session.query(Address.user_id.label('user_id'),func.count(*).label('address_count')).group_by(Address.user_id).subquery()

# 将子查询放到父查询中
for u,count in session.query(User,stmt.c.address_count).outerjoin(stmt,User.id==stmt.c.user_id).order_by(User.id):
  print(u,count)
```

从上面我们可以看到，一个查询如果想要变为子查询，则是通过`subquery()`方法实现，变成子查询后，通过`子查询.c`属性来访问查询出来的列。以上方式只能查询某个对象的具体字段，如果要查找整个实体，则需要通过`aliased`方法，示例如下：

```python
stmt = session.query(Address)
adalias = aliased(Address,stmt)
for user,address in session.query(User,stmt).join(stmt,User.addresses):
  print(user,address)
```

## Flask-SQLAlchemy库

另外一个库，叫做`Flask-SQLAlchemy`，`Flask-SQLAlchemy`是对`SQLAlchemy`进行了一个简单的封装，使得我们在`flask`中使用`sqlalchemy`更加的简单。可以通过`pip install flask-sqlalchemy`。使用`Flask-SQLAlchemy`的流程如下：

1. 数据库初始化：数据库初始化不再是通过`create_engine`，请看以下示例：

    ```python
    from flask import Flask
    from flask_sqlalchemy import SQLAlchemy
    from constants import DB_URI
    app = Flask(__name__)
    app.config['SQLALCHEMY_DATABASE_URI'] = DB_URI
    db = SQLAlchemy(app)
    ```

2. `ORM`类：之前都是通过`Base = declarative_base()`来初始化一个基类，然后再继承，在`Flask-SQLAlchemy`中更加简单了（代码依赖以上示例）：

    ```python
    class User(db.Model):
    	id = db.Column(db.Integer,primary_key=True)
    	username = db.Column(db.String(80),unique=True)
    	email = db.Column(db.String(120),unique=True)
    	def __init__(self,username,email):
    		self.username = username
    		self.email = email
    	def __repr__(self):
    		return '<User %s>' % self.username
    ```

3. 映射模型到数据库表：使用`Flask-SQLAlchemy`所有的类都是继承自`db.Model`，并且所有的`Column`和数据类型也都成为`db`的一个属性。但是有个好处是不用写表名了，`Flask-SQLAlchemy`会自动将类名小写化，然后映射成表名。
    写完类模型后，要将模型映射到数据库的表中，使用以下代码创建所有的表：

    ```python
    db.create_all()
    ```

4. 添加数据：这时候就可以在数据库中看到已经生成了一个`user`表了。接下来添加数据到表中：

    ```python
    admin = User('admin','admin@example.com')
    guest = User('guest','guest@example.com')
    db.session.add(admin)
    db.session.add(guest)
    db.session.commit()
    ```

    添加数据和之前的没有区别，只是`session`成为了一个`db`的属性。

5. 查询数据：查询数据不再是之前的`session.query`了，而是将`query`属性放在了`db.Model`上，所以查询就是通过`Model.query`的方式进行查询了：

    ```python
    users = User.query.all()
    # 再如：
    admin = User.query.filter_by(username='admin').first()
    # 或者：
    admin = User.query.filter(User.username='admin').first()
    ```

6. 删除数据：删除数据跟添加数据类似，只不过`session`是`db`的一个属性而已：

    ```python
    db.session.delete(admin)
    db.session.commit()
    ```