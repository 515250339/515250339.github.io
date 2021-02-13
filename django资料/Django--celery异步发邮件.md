### celery介绍

#### 将redis发布订阅模式用做消息队列和rabbitmq的区别：

#### 可靠性

redis：没有相应的机制保证消息的可靠消费，如果发布者发布一条消息，而没有对应的订阅者的话，这条消息将丢失， 不会存在内存中；rabbbitmq： 具有消息消费确认机制，如果发布一条消息，还没有消费者消费该队列，那么这条消息将一直存放在队列中，直到有消费者消费了该条消息，以此可以保证消息的可靠消费。

#### 实时性

redis实时性高，redis作为高效的缓存服务器，所有数据都存在在服务器中，所以它具有更高的实时性消费者负载均衡；rabbitmq队列可以被多个消费者同时监控消费，但是每一条消息只能被消费一次，由于rabbitmq的消费确认机制，因此它能够根据消费者的消费能力而调整它的负载，redis发布订阅模式，一个队列 可以被多个消费者同时订阅，当有消息到达时，会将该消息依次发送给每个订阅者；

#### 持久性

redis：redis的持久化是针对 于整个redis缓存的内容，它有RDB和AOF两个持久化方式，可以将整个redis实例持久化到磁盘，以此来做数据备份，防止异常情况下导致数据丢失。rabbitmq：队列、消息都可以选择性持久化，持久化粒度更小，更灵活；队列监控rabbitmq实现了后台监控平台，可以在该平台上看到所有创建的队列的详细情况，良好的台后管理平台可以方便我们更好的使用；redis没有所谓的监控平台。

总结：

redis：轻量级、低延迟，高并发、低可靠性；

rabbitmq：重量级、高可靠，异步，不保证实时

rabbitmq是一个专门的AMQP协议队列， 它的优势就在于提供可靠的队列服务，并且可做到异步，而redis主要是用于缓存的，redis的发布订阅模块，可用于实现及时性，且可靠性低的功能



#### 名词

task(任务)： 就是一个Python函数

queue(队列)：将需要执行的任务加入到队列中

worker(工人)：在一个新进程中，负责执行队列中的任务

borker(代理人)：负责调度，在布置环境中使用redis



#### 使用

### 安装包

- pip install celery==3.1.25
- pip install  celery-with-redis==3.0
- pip install django-celery==3.3.0
- pip install django-redis==4.10.0
- pip install redis==2.10.6
- pip install itsdangerous == 1.1.0

### 在settings中配置

```python
# INSTALLED_APPS中安装应用
ISNTALLED_APPS = [
    'djcelery',
]

# 配置邮件发送
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.qq.com'  # 如果为163邮箱，设置为smtp.163.com
EMAIL_PORT = 25  # 或者 465/587是设置了 SSL 加密方式
# 发送邮件的邮箱
EMAIL_HOST_USER = '370686999@qq.com'
# 在邮箱中设置的客户端授权密码
EMAIL_HOST_PASSWORD = 'mtidtiqwquhiqzzbgbi'  # 第三方登陆使用的授权密码
EMAIL_USE_TLS = True  # 这里必须是 True，否则发送不成功
# 收件人看到的发件人, 必须是一直且有效的
EMAIL_FROM = '海上明月<370686999@qq.com>'
DEFAULT_FROM_EMAIL = EMAIL_HOST_USER
```

### 在项目下，创建celery_tasks文件夹，在里面创建文件，__init__.py, celery.py，celeryconfig.py以及tasks.py文件

#### celery.py下写入的内容

```python
from celery import Celery
from celery_tasks import celeryconfig # 导入celery配置文件

import os
# 为celery设置环境变量
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mydemo.settings")

## 创建celery app
app = Celery('celery_tasks')

# 从单独的配置模块中加载配置
app.config_from_object(celeryconfig)

# 设置app自动加载任务
app.autodiscover_tasks(['celery_tasks'])
```

#### celeryconfig.py内写入的内容

```python
# 设置结果存储
CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/9'

# 设置代理人broker
BROKER_URL = 'redis://127.0.0.1:6379/8'
```

#### tasks.py写入的内容

```python
from celery_tasks.celery import app as celery_app  # 导入创建好的celery应用
from django.core.mail import send_mail  # 使用django内置函数发送邮件
from django.conf import settings  # 导入django的配置


@celery_app.task
def send_mail_task(email, token):
    # 使用django内置函数发送邮件
    subject = "海上明月"
    message = ""
    sender = settings.EMAIL_FROM
    recipient = [email]
    html_message = "<h1>{}恭喜您成为美多商城会员，请点击以下链接进行激活邮箱：<a href='http://127.0.0.1:8000/mdadmin/active/{}'>\
    http://127.0.0.1:8000/mdadmin/active/{}</a></h1>".format("海上明月", token, token)
    send_mail(subject, message, sender, recipient, html_message=html_message)
```

### views.py

```python
from django.http import HttpResponse
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
from itsdangerous import SignatureExpired
from celery_tasks.tasks import send_mail_task
class SendEmailView(View):
    """
    发送邮件
    """
    def post(self, request):
        print(request.POST)
        email = request.POST.get('email')
        user_info = {"user": 1}
        serialize = Serializer(settings.SECRET_KEY, 1800)
        token = serialize.dumps(user_info).decode()
        send_mail_task.delay(email, token)
        return HttpResponse(json.dumps({'msg': 'OK', 'code': 200}))
```

#### 激活用户

```python
from itsdangerous import SignatureExpired


class ActiveView(View):
    """
    激活用户
    """
    def get(self, request, token):
        serialize = Serializer(settings.SECRET_KEY, 60)
        try:
            user_info = serialize.loads(token)
            print(user_info)
            user_id = user_info["user"]
            print(user_id)
            user = User.objects.get(pk=user_id)
            user.is_active = True
            user.save()
            return HttpResponse(json.dumps({'msg': '激活成功', 'code': 200}, ensure_ascii=False))
        except SignatureExpired:
            return HttpResponse(json.dumps({'msg': '激活链接失效，请重新发送邮件进行激活', 'code': 404}, ensure_ascii=False))

```



#### 启动celery服务进行测试

```python
# 启动celery服务进行测试
# 启动django服务
python manage.py runserver
# 启动celery的worker
celery -A celery_tasks worker -l info -P eventlet
```

