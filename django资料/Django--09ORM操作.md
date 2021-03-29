### Django模型之ORM操作

#### ORM介绍
- **什么是ORM**
  ORM 全拼Object-Relation Mapping.

  中文意为 对象-关系映射.

  在MVC/MVT设计模式中的Model模块中都包括ORM

- **ORM优势**

  - 只需要面向对象编程, 不需要面向数据库编写代码.

    对数据库的操作都转化成对类属性和方法的操作.
    不用编写各种数据库的sql语句.

  - 实现了数据模型与数据库的解耦, 屏蔽了不同数据库操作上的差异.

    不在关注用的是mysql、oracle...等.
    通过简单的配置就可以轻松更换数据库, 而不需要修改代码.

- **ORM劣势**
  相比较直接使用SQL语句操作数据库,有性能损失.
  根据对象的操作转换成SQL语句,根据查询的结果转化成对象, 在映射过程中有性能损失.

- **ORM和数据库关系：**
  在Django中model是你数据的单一、明确的信息来源。它包含了你存储的数据的重要字段和行为。通常，一个模型（model）映射到一个数据库表.

  基本情况：

  每个模型都是一个Python类，它是django.db.models.Model的子类。

  模型的每个属性都代表一个数据库字段。



#### ORM操作

**增加操作**

```python
# 通过python manage.py shell 进入到shell下
# 进入shell环境以后，首先导入模型
from polls.models import * 			# 导入全部模型
from django.utils import timezone   # 导入时间模块
# 创建方法一：
q = Question(question_text="什么地方的菜最有特色？", pub_date=timezone.now())
q.save()

# 关联创建，用问题关联创建选项
q.choice_set.create(choice_text="湖南")


# 创建方法二：
q = Question()          # 创建实例对象
q.question_text = "什么地方的菜最有特色？"
q.pub_date = timeaone.now()
q.save()

# 创建方法三：
Question.objects.create(question_text="什么地方的菜最有特色？", 
                        pub_date=timezone.now())


# 批量创建，可以提高性能，减少对数据库的访问写入次数
bulk_create()

# 批量添加，需要传入的参数是一个列表
Question.objects.bulk_create(
    [
        Question(question_text="什么地方的菜最有特色？", pub_date=timezone.now()),
        Question(question_text="什么地方的景色最美？", pub_date=timezone.now())，
    ]
)
```

#### 修改操作

```python
# 修改方法1：
Question.objects.filter(pk=1).update(question_text = "什么地方最好玩？")  


```

#### 删除操作

```python
# 删除：(先查询到某个queryset对象，然后用删除命令)
q = Question.objects.get(id=1)
q.delete()
```

#### 查询操作

**必会的方法**

```python
# 1、 all():                 查询所有结果
question_list = Question.objects.all()  # 返回一个queryset集合

# 2、 filter(**kwargs):      它包含了与所给筛选条件相匹配的对象
question_list = Question.objects.filter(pk=1)  # 返回一个queryset集合,如果没有查询到，返回一个空集合,不会报错

# 3、 get(**kwargs):         返回与所给筛选条件相匹配的对象，返回结果有且只有一个，如果符合筛选条件的							  对象超过一个或者没有都会抛出错误。
question = Question.objects.get(pk=1)  # 返回一个queryset对象，并且只会得到一个数据，如果没有查询到，会报DoesNotExist的错误

# 4、 exclude(**kwargs):     它包含了与所给筛选条件不匹配的对象
question = Question.objects.exclude(id__in=11, 22, 33])  # 筛选id除了11，22，33外的，其它的数据

# 5、 values(*field):        返回一个ValueQuerySet——一个特殊的QuerySet，运行后得到的并不是一系列                              model的实例化对象，而是一个可迭代的字典序列
question = Question.objects.values()
# 返回结果：[{"id": 1, "question_name": "xxxxxxx"}, {"id": 2, "question_name": "xxxxxxx"}, ...]

# 6、 values_list(*field):   它与values()非常相似，它返回的是一个元组序列，values返回的是一个字典序							   列
question = Question.objects.values_list()
# 返回结果：[(1, "xxxxxxx"), (2, "xxxxxxx"), ....]

# 7、 order_by(*field):      对查询结果排序
user_list = User.objects.order_by("-id")  # “-” 按id降序排列
user_list = User.objects.order_by()  # 按id升序排列（默认）

# 8、 reverse():             对查询结果反向排序，请注意reverse()通常只能在具有已定义顺序的QuerySet								上调用(在model类的Meta中指定ordering或调用order_by()方法)。
user = User.objects.all().reverse()  # 把查询的结果进行反转

# 9、 distinct():            从返回结果中剔除重复纪录(如果你查询跨越多个表，可能在计算QuerySet时得到							 重复的结果。此时可以使用distinct()，注意只有在PostgreSQL中支持按字段							  去重。)
Question.objects.all().distinct()  # 把结果中重复的记录剔除

# 10、 count():              返回数据库中匹配查询(QuerySet)的对象数量。
user_count = User.objects.count()  # 返回user表中的用户数量

# 11、 first():              返回第一条记录
User.objects.first()

# 12、 last():               返回最后一条记录
User.objects.last()    

# 13、 exists():             如果QuerySet包含数据，就返回True，否则返回False
user = User.objects.filter(pk=1).exists()  # 返回True 或者False
if user:
    print("OK")
```



### 查询条件

**在 ORM 层面，这些查询条件都是使用 field + __ + condition 的方式来使用**

```python
# 精确的 等于,如果提供一个None,SQL解析为Null
article = Article.objects.get(id__exact=14)
article = Article.objects.get(id__exact=None)
'''
对应sql
select ... from article where id=14;
select ... from article where id IS NULL;
'''
# iexact 使用like查询
article = Article.objects.filter(title__iexact='hello world')
'''
等价于 select ... from article where title like 'hello world'
'''
# 包含:contains,区分大小写
articles = Article.objects.filter(title__contains='hello')
'''
等价于select ... where title like binary '%hello%';
'''
# icontains 忽略大小写
articles = Article.objects.filter(title__icontains='hello')
'''
等价于 select ... where title like '%hello%';
'''
# in 提取那些给定的field的值是否在给定的容器中。容器可以为list、tuple或者任何一个可以迭代的对
象，# 包括QuerySet对象
articles = Article.objects.filter(id__in=[1,2,3])
'''
等价于 select ... where id in (1,3,4)
'''
# 当然也可以传递一个QuerySet对象进去。示例代码如下：
inner_qs = Article.objects.filter(title__contains='hello')
categories = Category.objects.filter(article__in=inner_qs)
'''
等价于:以上代码的意思是获取那些文章标题包含hello的所有分类。
select ...from category where article.id in (select id from article where title
like '%hello%');
'''
# gt 大于
articles = Article.objects.filter(id__gt=4)
'''
等价于 select ... where id > 4;
'''
# gte 大于等于
# lt 小于
# lte 小于等于
# startswidth 开始,大小写敏感
articles = Article.objects.filter(title__startswith='hello')
'''
等价于: select ... where title like 'hello%'
'''
# istartswidth 大小写不敏感
# endswidth 以**结尾,大小写敏感
articles = Article.objects.filter(title__endswith='world')
'''
等价于:select ... where title like '%world';
'''
# iendswidht 以**结尾,忽略大小写
# range 判断某个field的值是否在给定的区间中, 两个范围之间
from django.utils.timezone import make_aware
from datetime import datetime
start_date = make_aware(datetime(year=2018,month=1,day=1))
end_date = make_aware(datetime(year=2018,month=3,day=29,hour=16))
articles = Article.objects.filter(pub_date__range=(start_date,end_date))
# isnull
articles = Article.objects.filter(pub_date__isnull=False)
# regex和iregex： 正则
articles = Article.objects.filter(title__regex=r'^hello')
'''
等价:select ... where title regexp binary '^hello';
'''
'''
以上代码的意思是提取所有发布时间在2018/1/1到2018/12/12之间的文章。
将翻译成以下的SQL语句：
select ... from article where pub_time between '2018-01-01' and '2018-12-12'。
需要注意的是，以上提取数据，不会包含最后一个值。也就是不会包含2018/12/12的文章。
而且另外一个重点，因为我们在settings.py中指定了USE_TZ=True，并且设置了
TIME_ZONE='Asia/Shanghai'，因此我们在提取数据的时候要使用django.utils.timezone.make_aware
先将datetime.datetime从navie时间转换为aware时间。make_aware会将指定的时间转换为TIME_ZONE中
指定的时区的时间。
'''
```



### 根据关联的表查

**假如现在有两个 ORM 模型，一个是 Article ，一个是 Category 。代码如下：**

```python
class Category(models.Model):
  """文章分类表"""
  name = models.CharField(max_length=100)
class Article(models.Model):
  """文章表"""
  title = models.CharField(max_length=100,null=True)
  category = models.ForeignKey("Category",on_delete=models.CASCADE)
```

**比如想要获取文章标题中包含"hello"的所有的分类。那么可以通过以下代码来实现：**

```python
categories = Category.object.filter(article__title__contains("hello")) 
```



### 聚合函数

**聚合函数是通过 aggregate 方法来实现的。**

- **Avg ：求平均值。比如想要获取所有图书的价格平均值。那么可以使用以下代码实现**

  ```python
  from django.db.models import Avg
  result = Book.objects.aggregate(Avg('price'))
  print(result)
  ```

  以上的打印结果是：

  ```
  {"price__avg":23.0} 
  ```

  其中 price__avg 的结构是根据 field__avg 规则构成的。如果想要修改默认的名字，那么可以将 Avg 赋值
  给一个关键字参数。示例代码如下：

  ```python
  from django.db.models import Avg
  result = Book.objects.aggregate(my_avg=Avg('price'))
  print(result)
  ```

  那么以上的结果打印为：

  ```python
  {"my_avg":23} 
  ```

- **Count ：获取指定的对象的个数。示例代码如下：**

  ```python
  from django.db.models import Count
  result = Book.objects.aggregate(book_num=Count('id'))
  ```

  以上的 result 将返回 Book 表中总共有多少本图书。  Count 类中，还有另外一个参数叫做 distinct ，默
  认是等于 False ，如果是等于 True ，那么将去掉那些重复的值。比如要获取作者表中所有的不重复的邮箱
  总共有多少个，那么可以通过以下代码来实现：

  ```python
  from djang.db.models import Count
  result = Author.objects.aggregate(count=Count('email',distinct=True))
  ```

- **Max 和 Min ：获取指定对象的最大值和最小值。比如想要获取 Author 表中，最大的年龄和最小的年龄分别**
  **是多少。那么可以通过以下代码来实现：**

  ```python
  from django.db.models import Max,Min
  result = Author.objects.aggregate(Max('age'),Min('age'))
  ```

  如果最大的年龄是88,最小的年龄是18。那么以上的result将为：

  ```python
  {"age__max":88,"age__min":18} 
  ```

- **Sum ：求指定对象的总和。比如要求图书的销售总额。那么可以使用以下代码实现：**

  ```python
  from djang.db.models import Sum
  result =
  Book.objects.annotate(total=Sum("bookstore__price")).values("name","total")
  ```

  以上的代码 annotate 的意思是给 Book 表在查询的时候添加一个字段叫做 total ，这个字段的数据来源是
  从 BookStore 模型的 price 的总和而来。 values 方法是只提取 name 和 total 两个字段的值。

  更多的聚合函数请参考官方文档：https://docs.djangoproject.com/en/2.0/ref/models/querysets/#aggregation-functions



### aggregate和annotate的区别：

- aggregate ：返回使用聚合函数后的字段和值。
- annotate ：在原来模型字段的基础之上添加一个使用了聚合函数的字段，并且在使用聚合函数的时候，会
  使用当前这个模型的主键进行分组（group by）。 比如以上 Sum 的例子，如果使用的是 annotate ，那么将
  在每条图书的数据上都添加一个字段叫做 total ，计算这本书的销售总额。 而如果使用的是 aggregate ，
  那么将求所有图书的销售总额。

```python
from django.db import models
class Author(models.Model):
  """作者模型"""
  name = models.CharField(max_length=100)
  age = models.IntegerField()
  email = models.EmailField()
  class Meta:
    db_table = 'author'
class Publisher(models.Model):
  """出版社模型"""
  name = models.CharField(max_length=300)
  class Meta:
    db_table = 'publisher'
class Book(models.Model):
  """图书模型"""
  name = models.CharField(max_length=300)
  pages = models.IntegerField()
  price = models.FloatField()
  rating = models.FloatField()
  author = models.ForeignKey(Author,on_delete=models.CASCADE)
  publisher = models.ForeignKey(Publisher, on_delete=models.CASCADE)
  class Meta:
    db_table = 'book'
class BookOrder(models.Model):
  """图书订单模型"""
  book = models.ForeignKey("Book",on_delete=models.CASCADE)
  price = models.FloatField()
  class Meta:
    db_table = 'book_order'
```



### F表达式和Q表达式：

#### F表达式：

F表达式 是用来优化 ORM 操作数据库的。比如我们要将公司所有员工的薪水都增加1000元，如果按照正常的流
程，应该是先从数据库中提取所有的员工工资到Python内存中，然后使用Python代码在员工工资的基础之上增加
1000元，最后再保存到数据库中。这里面涉及的流程就是，首先从数据库中提取数据到Python内存中，然后在
Python内存中做完运算，之后再保存到数据库中。示例代码如下：

```python
employees = Employee.objects.all()
for employee in employees:
  employee.salary += 1000
  employee.save()
```

而我们的 F表达式 就可以优化这个流程，他可以不需要先把数据从数据库中提取出来，计算完成后再保存回去，他可以直接执行 SQL语句 ，就将员工的工资增加1000元。示例代码如下：

```python
from djang.db.models import F
Employee.object.update(salary=F("salary")+1000)
# 直接把sql传输到数据库!!!!!
```

F表达式 并不会马上从数据库中获取数据，而是在生成 SQL 语句的时候，动态的获取传给 F表达式 的值。
比如如果想要获取作者中， name 和 email 相同的作者数据。如果不使用 F表达式 ，那么需要使用以下代码来完
成：

```python
authors = Author.objects.all()
for author in authors:
	if author.name == author.email:
   		print(author)
```

如果使用 F表达式 ，那么一行代码就可以搞定。示例代码如下：

```python
from django.db.models import F
authors = Author.objects.filter(name=F("email"))
# where  email = name
```

#### Q表达式：

如果想要实现所有价格高于100元，并且评分达到9.0以上评分的图书。那么可以通过以下代码来实现：

```python
books = Book.objects.filter(price__gte=100,rating__gte=9) 
```

以上这个案例是一个并集查询，可以简单的通过传递多个条件进去来实现。 但是如果想要实现一些复杂的查询语
句，比如要查询所有价格低于10元，或者是评分低于9分的图书。那就没有办法通过传递多个条件进去实现了。这
时候就需要使用 Q表达式 来实现了。示例代码如下：

```python
from django.db.models import Q
books = Book.objects.filter(Q(price__lte=10) | Q(rating__lte=9))
```

以上是进行或运算，当然还可以进行其他的运算，比如有 & 和 ~（非） 等。一些用 Q 表达式的例子如下：

```python
from django.db.models import Q
# 获取id等于3的图书
books = Book.objects.filter(Q(id=3))
# 获取id等于3，或者名字中包含文字"记"的图书
books = Book.objects.filter(Q(id=3)|Q(name__contains("记")))
# 获取价格大于100，并且书名中包含"记"的图书
books = Book.objects.filter(Q(price__gte=100)&Q(name__contains("记")))
# 获取书名包含“记”，但是id不等于3的图书
books = Book.objects.filter(Q(name__contains='记') & ~Q(id=3))
```



### 日期

```python
# data
针对某些date或者datetime类型的字段。可以指定date的范围。并且这个时间过滤，还可以使用链式调用。示
例代码如下：
articles = Article.objects.filter(pub_date__date=date(2018,3,29))
'''
以上代码的意思是查找时间为2018/3/29这一天发表的所有文章。
将翻译成以下的sql语句：
select ... WHERE DATE(CONVERT_TZ(`front_article`.`pub_date`, 'UTC',
'Asia/Shanghai')) = 2018-03-29
注意，因为默认情况下MySQL的表中是没有存储时区相关的信息的。因此我们需要下载一些时区表的文件，然后添
加到Mysql的配置路径中。如果你用的是windows操作系统。那么在
http://dev.mysql.com/downloads/timezones.html下载timezone_2018d_posix.zip - POSIX
standard。然后将下载下来的所有文件拷贝到C:\ProgramData\MySQL\MySQL Server 5.7\Data\mysql
中，如果提示文件名重复，那么选择覆盖即可。
如果用的是linux或者mac系统，那么在命令行中执行以下命令：mysql_tzinfo_to_sql
/usr/share/zoneinfo | mysql -D mysql -u root -p，然后输入密码，从系统中加载时区文件更新到
mysql中。
'''
# year 根据年份进行查找
articles = Article.objects.filter(pub_date__year=2018)
articles = Article.objects.filter(pub_date__year__gte=2017)
'''
等价于:
select ... where pub_date between '2018-01-01' and '2018-12-31';
select ... where pub_date >= '2017-01-01';
'''
# month 同year,根据月份查
# day  同year,根据日期查
# week_day Django 1.11新增的查找方式。同year，根据星期几进行查找。1表示星期天，7表示星期六，2-6代表的是星期一到星期五。
# time 根据时间查
articles = Article.objects.filter(pub_date__time=datetime.time(12,12,12));
# 以上的代码是获取每一天中12点12分12秒发表的所有文章。
```

更多的关于时间的过滤，请参考Django官方文档：
https://docs.djangoproject.com/en/2.0/ref/models/querysets/#range。

