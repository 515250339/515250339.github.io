### 图片上传

```python
# settings中配置
MEDIA_ROOT = os.path.join(BASE_DIR, 'upload')
MEDIA_URL = '/media/'

# 项目url配置
from django.conf.urls.static import static
from django.views.staitc import serve
from django.conf import settings
from django.urls import re_path

urlpatterns = [
    re_path('^(?P<path>.*)/$', serve, {'document_root': settings.MEDIA_ROOT})
]

if settings.DEBUG:
    urlpatterns += static(settting.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```



### 图片接收

views.py

```python
def upload_file(request):
    if request.method == "GET":
        context = {'pic': Picture.objects.all()}
        return render(request, "polls/uploadfile.html", context)
    else:
        info = request.FILES.get("image")
        image_path = "{}/img/{}".format(settings.MEDIA_ROOT, info.name)  # 拼接绝对路径，用来写文件
        Picture.objects.create(pic_url="img/{}".format(info.name))
        with open(image_path, "wb") as f:
            for i in info.chunks():
                f.write(i)
        return HttpResponse("OK")
```

