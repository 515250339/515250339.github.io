### 常用字段以及字段参数



#### 以下参数可用于所有字段，所有都是可选的

- **null参数，用法models.CharField(max_length=20, null=True)，表示字段值最大长度为20，并且可以为空。**

- **blank参数，用法models.CharField(max_length=20, blank=True), 表示字段值最大长度为20，并且表单验证可以为空。一般与null同时使用。**

- **choices参数**

  用法：

  ```python
  from django.db import models
  
  
  
  class Person(models.Model):
      """
      性别: M代表男，F代表女
      """
      SEX_CHOICES = [
          ("M", "Male"),
          ("F", "Female")
      ]
      name = models.CharField(max_length=20)
      gender = models.CharField(max_length=4, choices=SEX_CHOICES)
  ```

  添加方法以及获取方法：

  ```python
  # 创建一个用户对象
  >>>p = Person.objects.create(name="张三", gender="M")
  # 获取性别的代表字母
  >>>p.gender
  'M'
  # 获取选项元组
  >>>p.get_gender_display()
  'Male'
  ```

- **default参数，默认值，比如整数型字段models.IntegerField(default=0)， 给定一个默认值，可以在添加的时候，不需要马上就输入值**

- **primary_key参数，需要重设主键时，才需要添加此参数，如：商城订单，需要用订单号来做主键，可以添加此参数**

- **unique参数，唯一值，添加此参数以后，表示这个字段唯一，如：身份证号，手机号等**

- **verbose_name参数，该字段的可读名称。一般定义为中文。**

#### 常用字段

- **CharField()字段**

  ```python
  # max_length为必填项，主要目的是为了防止恶意操作
  models.CharField(max_length=20)
  ```

  扩展：

  models.CharField  对应的是MySQL的varchar数据类型

  **char 和 varchar的区**别 :

  char和varchar的共同点是存储数据的长度，不能 超过max_length限制，

  不同点是varchar根据数据实际长度存储，char按指定max_length的长度存储数据；前者更节省硬盘空间；

- **TextFiled()：大文本字段，一般超过4000个字符时使用**

- **EmailField()字段**

  ```python
  # 一个带有检查 Email 合法性的 CharField
  models.EmailField(max_length=50)  # 默认长度为254
  ```

- **BooleanField()字段**

  不能为空，可以设置default值，也可以设置null=True, blank=True

- **DateField()， 日期字段**

  auto_now=True,保存时自动设置该字段为当期日期，最后修改日期

  auto_now_add=True, 当该对象第一次被创建是自动设置该字段为现在日期，创建日期。

- **IntegerField()整型**

  不能为空，可以设置default值，也可以设置null=True, blank=True

- **FloatField()浮点型**

  不能为空，可以设置default值，也可以设置null=True, blank=True

- **ImageField()图像字段**

  - height_field、width_field：如果提供这两个参数，则图片将按提供的高度和宽度规格保存。
  -  该字段要求 Python Imaging 库Pillow。
  - 会检查上传的对象是否是一个合法图片。

  ```python
  # settings里面设置
  MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
  MEDIA_URL = '/media/'
  
  # urls.py文件添加文件的访问路径
  from mysite.settins import MEDIA_ROOT   # 导入settings
  from django.views.static import serve
  from django.urls import path, include
   
  urlpatterns = [
      
      url(r'^media/(?P<path>.*)/$', serve, {"document_root": MEDIA_ROOT}), 
  ] 
  
  # models.py中添加字段
  from django.db import models
  
  
  
  class Person(models.Model):
      """
      性别: M代表男，F代表女
      """
      SEX_CHOICES = [
          ("M", "Male"),
          ("F", "Female")
      ]
      name = models.CharField(max_length=20)
      gender = models.CharField(max_length=4, choices=SEX_CHOICES)
      myfile = models.ImageField(upload_to='detail')
  ```
  
- **FileField(upload_to=None)**

  upload_to：一个用于保存上传文件的本地文件系统路径，该路径由 MEDIA_ROOT 中设置

  这个字段不能设置primary_key和unique选项.在数据库中存储类型是varchar，默认最大长度为100



### 外键

- **ForeignKey(其它表，on_delete=models.CASCADE)一对多外键（重点）**

  第一参数是其它的表，也可以是自己。

  ```python
  from django.db import models
  
  
  class AreaInfo(models.Model):
      """
      省市级联
      """
      area_name = models.CharField(max_length=20)
      pid = models.ForeignKey("self", on_delete=models.CASCADE, null=True, blank=True)
  ```

  **注：这里的pid必须默认允许为空，验证允许为空。**

  on_delete当删除外键引用时，是否删除下级。models.CASCADE为联级删除，modesl.DO_NOTHING为不采取任何措施。

  related_name等同于重命名，当在一个表中，要同时关联同一个外键时，需要使用related_name

  ```python
  from django.db import models
  
  
  class AreaInfo(models.Model):
      """
      省市级联
      """
      area_name = models.CharField(max_length=20)
      pid = models.ForeignKey("self", on_delete=models.CASCADE, null=True, blank=True)
  
  
  class Person(models.Model):
      """
      性别
      """
      SEX_CHOICES = [
          ("M", "Male"),
          ("F", "Female")
      ]
      name = models.CharField(max_length=20)
      gender = models.CharField(max_length=4, choices=SEX_CHOICES)
  
  
  class Address(models.Model):
      """
      用户的详细地址
      """
      person = models.ForeignKey(Person, on_delete=models.CASCADE)
      province = models.ForeignKey(AreaInfo, on_delete=models.CASCADE, related_name="province", verbose_name="省")
      town = models.ForeignKey(AreaInfo, on_delete=models.CASCADE, related_name="town", verbose_name="市")
      city = models.ForeignKey(AreaInfo, on_delete=models.CASCADE, related_name="city", verbose_name="县") 
  ```

- **ManyToManyField()多对多关系**

  多对多关系，需要一个位置参数，其它参数与一对多相同。

  举例：

  ```python
  # 服装的尺寸、颜色
  class Size(models.Model):
      s_name = models.CharField(max_length=10, verbose_name='尺寸')
  
      def __str__(self):
          return self.s_name
  
  
  class Color(models.Model):
      color_name = models.CharField(max_length=10, verbose_name='颜色')
  
      def __str__(self):
          return self.color_name
  
  
  class Goods(models.Model):
      g_name = models.CharField(max_length=100, verbose_name='商品名称')
      price = models.DecimalField(max_digits=5, decimal_places=2, verbose_name='现价')
      color = models.ManyToManyField(to="Color")
      # 与尺码建立多对多关系
      size = models.ManyToManyField(to="Size")
  
      def __str__(self):
          return self.g_name
  ```

- **OneToOneField()一对一关系**

  ```python
  # 一对一关系
  # 银行账户(Account)和联系人(Contact)，一个银行账户对应一个联系人，而一个联系人也只对应一个账户
  class Account(models.Model):
      """
      账户
      """
      username = models.CharField(max_length=20, null=True, blank=True, verbose_name="用户名")
      password = models.CharField(max_length=20, null=True, blank=True, verbose_name="密码")
      register_date = models.DateTimeField(auto_now_add=True, null=True, blank=True, verbose_name="注册时间")
  
  
  class Contact(models.Model):
      """
      联系人
      """
      address = models.CharField(max_length=50, null=True, blank=True, verbose_name="联系人地址")
      account = models.OneToOneField(Account, on_delete=models.CASCADE)
  ```
