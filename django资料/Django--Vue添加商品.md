### Vue添加商品

#### 一、序列化

```python
from rest_framework import serializers
from .models import *


class GoodsSerializer(serializers.ModelSerializer):
    """
    商品序列化
    """
    class Meta:
        model = Goods
        fields = '__all__'


class CateSerializer(serializers.ModelSerializer):
    """
    分类序列化
    """
    class Meta:
        model = Cate
        fields = '__all__'
```

#### 二、view.py视图

```python
from rest_framework.mixins import (
    ListModelMixin,
    CreateModelMixin    
)
from rest_framework.generics import GenericAPIView
from .serializer import GoodsSerializer, CateSerializer

class GoodsApiView(ListModelMixin, CreateModelMixin,  GenericAPIView):
    # queryset和serializer_class为固定变量
    # queryset为查询集
    # serializer_class为指定序列化的类
    queryset = Goods.objects.all()
    serializer_class = GoodsSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)  # self.list为get方式接口

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)  # self.create为反序列化添加数据
```

#### 三、urls.py

```python
path('goods_api/', views.GoodsApiView.as_view(), name='goods_api'),  # 商品接口
```

#### 四、vue页面

```vue
<template>
    <div>
        <div>
            <form action="" @submit.prevent="post_submit">
                <p>商品名称：<input type="text" v-model="goods_name"></p>
                <p>商品价格：<input type="text" v-model="goods_price"></p>
                <p>商品图片：<input type="file" id='image'></p>
                <p>商品详情：<input type="text" v-model="goods_content"></p>
                <p>
                    <--在select中用@change调用get_id方法，来获取option的value-->
                    <select v-model="cate" id="" @change="get_id($event)">
                        <!-- <option value="0">请点击下拉菜单</option> -->
                        <option  v-for="cate in cate_list" :value="cate.id" >{{cate.c_name}}</option>
                    </select>
                </p>
                <p><input type="submit" value="添加商品"></p>

            </form>
        </div>

    </div>
</template>

<script>
import axios from 'axios'
export default {
    data(){
        return{
            goods_name: '',
            goods_price: '',
            goods_content: '',
            cate: '0',
            cate_list: []
        }
    },
    methods:{
        // 通过事件获取option的value
        get_id(event){
            this.cate = event.target.value  //获取option对应的value值
            
        },
        post_submit(){
            var img = document.getElementById('image')  // 通过id获取图片
            let form_data = new FormData()
            form_data.append('goods_name', this.goods_name)
            form_data.append('goods_price', this.goods_price)
            form_data.append('goods_content', this.goods_content)
            form_data.append('goods_img', img.files[0])  // 通过files属性索引0，来提取文件内容
            form_data.append('cate', this.cate)
            axios({
                method: 'post',
                url: 'http://127.0.0.1:8000/myapp/goods_api/',
                data: form_data
            }).then(function(res){
                console.log(res)
            })
        },
        // 使用get方式，获取分类的所有数据
        get_cate(){
            var self = this
            axios({
                method: 'get',
                url: 'http://127.0.0.1:8000/myapp/cate_api/'
            }).then(function(res){
                self.cate_list = res.data
            })
        }
        
    },
    // 打开页面自动调用get_cate方法
    created(){
        this.get_cate()
    }
}
</script>

```



