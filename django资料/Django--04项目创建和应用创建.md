### 创建第一个django项目

### 例用命令创建项目

##### 1、进入虚拟环境

```python
workon my_env
```

##### 2、cd到要创建项目的目录下，创建django项目

进入到E:\django_code下，创建django项目

![cmd切换目录](.\django相关截图\cmd切换目录.png)

##### 3、使用命令，创建django项目 django-admin startproject 项目名

![使用命令创建django项目](.\django相关截图\使用命令创建django项目.png)

**创建项目以后，生成以下文件**

![django项目目录结构](.\django相关截图\django项目目录结构.png)

**修改settings文件以下行：**

```python
ALLOWED_HOSTS = ["*"]  # 允许的主机后面加 * 或者加对应的IP
LANGUAGE_CODE = 'zh-hans'  # 语言编码改为zh-hans
TIME_ZONE = 'Asia/Shanghai'  # 时区必变  Asia/Shanghai
USE_TZ = False  # 改为False 
```



##### 4、创建app  cd进入项目目录，例如：cd mysite

![使用命令创建app](.\django相关截图\使用命令创建app.png)

**app创建完成以后，修改settings文件以下内容**

![settings安装app](.\django相关截图\settings安装app.png)

**修改urls内容（作为跳转使用，方便以后管理）**

​        在我们的项目中，不可能只有一个`app`，如果把所有的`app`的`views`中的视图都放在`urls.py`中进行映射，肯定会让代码显得非常乱。因此`django`给我们提供了一个方法，可以在`app`内部包含自己的`url`匹配规则，而在项目的`urls.py`中再统一包含这个`app`的`urls`。使用这个技术需要借助`include`函数。示例代码如下：

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', include('polls.urls')),  # 添加行备注，以便于区分
]
```





### 使用pycharm创建项目

**第一步，file----->New Project**

![pycharm创建项目1](.\django相关截图\pycharm创建项目1.png)

**第二步, 选择django创建的位置**

![pycharm创建项目2](D:\积云教育\django资料\django相关截图\pycharm创建项目2.png)

**第三步**

![pycharm创建项目3](.\django相关截图\pycharm创建项目3.png)

**第四步**

![pycharm创建项目4](.\django相关截图\pycharm创建项目4.png)

**第五步**

![pycharm创建项目5](.\django相关截图\pycharm创建项目5.png)

**第六步**

![pycharm创建项目6](.\django相关截图\pycharm创建项目6.png)

**第七步**

![pycharm创建项目7](.\django相关截图\pycharm创建项目7.png)

**第八步**

![pycharm创建项目完成](.\django相关截图\pycharm创建项目完成.png)

