### Django模型之Meta常用选项

Django模型类的Meta是一个内部类，它用于定义一些Django模型类的行为特性。而可用的选项大致包含以下几类

### abstract

这个属性是定义当前的模型是不是一个抽象类。所谓抽象类是不会对应数据库表的。一般我们用它来归纳一些公共属性字段，然后继承它的子类可以继承这些字段。

Options.abstract
如果abstract = True 这个model就是一个抽象类

```python
from django.db import models


class BaseModel(models.Model):
    """大多数模型的父类模型"""
    create_time = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
    update_time = models.DateTimeField(auto_now=True, verbose_name='修改时间')
    is_delete = models.BooleanField(default=False, verbose_name='删除标记')

    class Meta:
        abstract = True  # 抽象模型类
```



### db_table

db_table是指定自定义数据库表明的。Django有一套默认的按照一定规则生成数据模型对应的数据库表明。
Options.db_table
定义该model在数据库中的表名称
　　db_table = 'Students'
如果你想使用自定义的表名，可以通过以下该属性
　　table_name = 'my_owner_table'

```python
from django.db import models


class AreaInfo(models.Model):
    """
    省市级联
    """
    area_name = models.CharField(max_length=20)
    pid = models.ForeignKey("self", on_delete=models.CASCADE, null=True, blank=True)
    
    class Meta:
        db_table = "areainfo"   # 创建数据库时，生成的表名
```



### verbose_name

verbose_name的意思很简单，就是给你的模型类起一个更可读的名字一般定义为中文，我们：
verbose_name = "学校"

```python
from django.db import models


class AreaInfo(models.Model):
    """
    省市级联
    """
    area_name = models.CharField(max_length=20)
    pid = models.ForeignKey("self", on_delete=models.CASCADE, null=True, blank=True)
    
    class Meta:
        db_table = "areainfo"   # 创建数据库时，生成的表名
        verbose_name = "省市县"
```



### verbose_name_plural

这个选项是指定，模型的复数形式是什么，比如：
verbose_name_plural = "学校"
如果不指定Django会自动在模型名称后加一个’s’

```python
from django.db import models


class AreaInfo(models.Model):
    """
    省市级联
    """
    area_name = models.CharField(max_length=20)
    pid = models.ForeignKey("self", on_delete=models.CASCADE, null=True, blank=True)
    
    class Meta:
        db_table = "areainfo"   # 创建数据库时，生成的表名
        verbose_name = "省市县"
        verbose_name_plural = verbose_name  # 在django的admin中，显示的名称
```

