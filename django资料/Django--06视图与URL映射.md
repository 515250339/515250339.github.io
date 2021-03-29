### 视图与URL映射

#### 视图

​        视图一般都写在`app`的`views.py`中。并且视图的第一个参数永远都是`request`（一个HttpRequest）对象。这个对象存储了请求过来的所有信息，包括携带的参数以及一些头部信息等。在视图中，一般是完成逻辑相关的操作。比如这个请求是添加一篇博客，那么可以通过request来接收到这些数据，然后存储到数据库中，最后再把执行的结果返回给浏览器。视图函数的返回结果必须是`HttpResponseBase`对象或者子类的对象。示例代码如下：

```python
from django.http import HttpResponse


def index(request):
    """
    @ author: XXX
    @ date: 2019-8-27
    跳转到首页
    """
    return HttpResponse("这里是首页")
```

**注：author、date多人开发，有利于代码管理**

#### 带参数的视图

```python
from django.http import HttpResponse


def book_info(request, book_id):
    """
    @ author: XXX
    @ date: 2019-8-27
    通过bookid获取书藉的信息
    """
    text = "您输入的书籍的id是：{}" .format(book_id)
    return HttpResponse(text)
```



#### URL映射

​        视图写完后，要与URL进行映射，也即用户在浏览器中输入什么`url`的时候可以请求到这个视图函数。在用户输入了某个`url`，请求到我们的网站的时候，`django`会从项目的`urls.py`文件中寻找对应的视图。在`urls.py`文件中有一个`urlpatterns`变量，以后`django`就会从这个变量中读取所有的匹配规则。匹配规则需要使用`django.urls.path`函数进行包裹，这个函数会根据传入的参数返回`URLPattern`或者是`URLResolver`的对象。示例代码如下：

```python
from django.urls import path
from . import views
app_name = "polls"
urlpatterns = [
    path("", views.index, name="polls"),  # 跳转到首页
]
```

**注：urlpatterns的列表中不只是一个url跳转路径，添加行注释，便于管理url的跳转定向。**

#### 带参数的URL

​        有时候，`url`中包含了一些参数需要动态调整。比如简书某篇文章的详情页的url，是`https://www.jianshu.com/p/a5aab9c4978e`后面的`a5aab9c4978e`就是这篇文章的`id`，那么简书的文章详情页面的url就可以写成`https://www.jianshu.com/p/<id>`，其中id就是文章的id。那么如何在`django`中实现这种需求呢。这时候我们可以在`path`函数中，使用尖括号的形式来定义一个参数。比如我现在想要获取一本书籍的详细信息，那么应该在`url`中指定这个参数。示例代码如下：

```python
from django.urls import path
from . import views
app_name = "polls"
urlpatterns = [
    path("", views.index, name="polls"),  # 跳转到首页
    path("book_info/<book_id>", views.book_info, name="book_info"),  # 通过ID获取书籍信息
]
```

