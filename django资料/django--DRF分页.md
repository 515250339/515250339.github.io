### DRF分页

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
```

#### 二、views.py视图

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from django.core.paginator import Paginator
from .serializer import GoodsSerializer
from django.core.paginator import Paginator, EmptyPage, InvalidPage

class GoodsPageApi(APIView):
    """
    分页
    """
    def get(self, request, pindex):
        # 1、获取商品信息
        goods_list = Goods.objects.all()
        # 2、实例化分页器
        paginat = Paginator(goods_list, 2)
        # 3、 获取分页
        paged = paginat.page(pindex)
        try:  # 判断是否有下一页
            paged.has_next()
            down_page = paged.next_page_number()  # 获取一下页的页码
        except EmptyPage:  # 如果下一页为空的话，
            down_page = paged.paginator.num_pages  # 获取最后一页的页码
            paged = paginat.page(down_page)

        try:
            paged.has_previous()  # 判断是否有上一页
            up_page = paged.previous_page_number()  # 获取上页的页码
        except InvalidPage:  # 如果没有上一页
            up_page = 1  # 返回第一页
            paged = paginat.page(up_page)

        # 4、 做序列化
        page_serializer = GoodsSerializer(paged, many=True)

        # 5、 返回数据
        return Response({
            'code': 200,
            'data': page_serializer.data,
            'page_list': [i for i in paginat.page_range],
            'has_previous': paged.has_previous(),  # 判断是否有上一页
            'has_next': paged.has_next(),  # 判断是否有下一页
            'previous_page_number': up_page,  # 返回上一页的页码
            'next_page_number': down_page,  # 返回下一页的页码
            'page_number': paged.number  # 返回当前页页码
        })

```

#### 三、urls.py

```python
path('goods_page/<pindex>', views.GoodsPageApi.as_view()),  # 商品分页
```

#### 四、vue分页展示

```vue
<template>
    <div>
        <table>
            <tr>
                <td>商品编码</td>
                <td>商品名称</td>
                <td>商品图片</td>
                <td>商品详情</td>
                <td>商品价格</td>
            </tr>
            <tr v-for="goods in goods_list">
                <td>{{goods.id}}</td>
                <td>{{goods.goods_name}}</td>
                <td><img :src="'http://127.0.0.1:8000' + goods.goods_img" alt="" style="width:100px; heigth:100px"></td>
                <td>{{goods.goods_content}}</td>
                <td>{{goods.goods_price}}</td>
            </tr>
        </table>
        <div>
            <button v-for="page in page_list" @click="get_num($event)" :value="page">{{page}}</button>
        </div>
    </div>
</template>

<script>
import axios from 'axios'
export default {
    data(){
        return{
            goods_list: [],
            page_list: [],
            p: '1'
        }
    },
    methods:{
        get_goods(){
            var self = this
            axios({
                method: 'get',
                url: 'http://127.0.0.1:8000/myapp/goods_page/'+ this.p
            }).then(function(res){
                console.log(res.data)
                self.goods_list = res.data.data
                self.page_list = res.data.page_list
            })
        },
        get_num(event){
            this.p = event.target.value  // 通过事件获取button的value
            console.log(this.p)
            this.get_goods()  // 点击按钮的时候，重新调用get_goods方法
        }
    },
    created(){  // 页面加载的时候，自动执行下面方法
        this.get_goods()
    }
}
</script>

```

