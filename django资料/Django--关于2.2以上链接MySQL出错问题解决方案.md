### django2.2以上版本链接pymysql会报以下错误

##### 因为pymysql目前最高版本为0.9.3,所以链接时会报错

```python
django.core.exceptions.ImproperlyConfigured: mysqlclient 1.3.13 or newer is required; you have 0.9.3.
```

**找到django安装包下："D:\Program Files\Python36\lib\site-packages\django\db\backends\mysql\base.py"第36行和第37行，注释掉这两行，便可解决此问题**

```python
# if version < (1, 3, 13):
#     raise ImproperlyConfigured('mysqlclient 1.3.13 or newer is required; you have %s.' % Database.__version__)
```

