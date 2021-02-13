## Django+Vue首页展示

### django后端

#### 序列化

```python
# 序列化
class GoodsSerializer(serializers.ModelSerializer):
    
    class Meta:
        model = Goods
        fields = '__all__'

class CateSerializer(serializers.ModelSerializer):

    class Meta:
        model = Cate
        fields = '__all__'
```

#### 视图函数

```python
# view视图函数
from django.core.paginator import Paginator, EmptyPage  # 导入分页器
from rest_framework.mixins import (
    ListModelMixin,
    CreateModelMixin,
    UpdateModelMixin,
    DestroyModelMixin,
    RetrieveModelMixin
)
from rest_framework.generics import GenericAPIView


# 展示所有的商品信息，反序列化添加商品
class GoodsView(ListModelMixin, CreateModelMixin, GenericAPIView):
    queryset = Goods.objects.all()
    serializer_class = GoodsSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
    
    
# 序列化、反序列化分类    
class CateView(ListModelMixin, CreateModelMixin, GenericAPIView):

    queryset = Cate.objects.all()
    serializer_class = CateSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)


# 商品的单个获取，修改、删除
class GoodsDUR(RetrieveModelMixin, DestroyModelMixin, UpdateModelMixin, GenericAPIView):
    queryset = Goods.objects.all()
    serializer_class = GoodsSerializer

    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)
    
# 商品分页    
class GoodsPageApi(APIView):

    def get(self, request, pindex):
        # 1、获取queryset查询集合
        goods = Goods.objects.all()
        # 2、实例化分页
        paginator = Paginator(goods, 2)
        # 3、获取分页
        if pindex == '':
            pindex =1

        try:
            paged = paginator.page(pindex)
            # 4、序列化
            goods_serializer = GoodsSerializer(paged, many=True)
            # 5、返回数据
            return Response({
                'page_list': [i for i in paginator.page_range],
                'data': goods_serializer.data
            })
        except EmptyPage:

            paged = paginator.page('1')
            # 4、序列化
            goods_serializer = GoodsSerializer(paged, many=True)
            # 5、返回数据
            return Response({
                'page_list': [i for i in paginator.page_range],
                'data': goods_serializer.data
            })

```

### urls路由

```python
path('goods_page/<pindex>', views.GoodsPageApi.as_view()),   # 分页 
path('goods_view/', views.GoodsView.as_view()),  # 展示所有商品，以及反序列化添加商品
path('cate_view/', views.CateView.as_view()),  # 分类序列化、反序列化
re_path('goods_dur/(?P<pk>\d+)/$', views.GoodsDUR.as_view()),  # 商品的单个获取、修改、删除
```

## Vue首页展示

#### 首页

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
                <td>操作</td>
            </tr>
            <tr v-for="goods in goods_list" >
                <td>{{goods.id}}</td>
                <td>
                    <!-- 使用router-link标签做路由跳转，name为所要跳转的页面，params为要传的参数 -->
                    <router-link :to="{name: 'Detail', params: {'id': goods.id}}">{{goods.goods_name}}</router-link>
                </td>
                <td><img :src="'http://127.0.0.1:8000' + goods.goods_img" alt="" style="width: 100px; heigth: 100px"></td>
                <td>{{goods.goods_content}}</td>
                <td v-if="goods.cate == '1'" style="color:red">{{goods.goods_price | message}}</td>
                <td v-else style="color:green">{{goods.goods_price | message}}</td>
                <td>
                    <!-- <a href="">修改</a> -->
                    <router-link :to="{name: 'Update', params: {'id': goods.id}}">修改</router-link>
                    <a href="#"  @click.prevent="del_goods(goods.id)" >删除</a>
                </td>
            </tr>
            
            
            
        </table>
        <div>
            <button v-for="p in page_list" @click="get_page($event)" :value="p">{{p}}</button>
        </div>
    </div>
</template>

<script>

import axios from 'axios'  //导入axios
export default {
    data(){
        return{
            p: '1',  // get_goods()方法默认访问的后台的页面
            goods_list: Array(),
            page_list: Array()
        }
    },
    methods: {
        // 定义get方法，用created方法加载，实现自动接收
        // created为钩子函数，为页面加载时，自动执行
        get_goods(){
            var self = this
            axios({
                method: 'get',
                url: 'http://127.0.0.1:8000/myapp/goods_page/' + this.p
            }).then(function(res){
                // console.log(res.data.data)
                self.goods_list = res.data.data
                self.page_list = res.data.page_list
            })+'/'
        },
        // 用商品的id删除商品信息，删除以后自己加载
        del_goods(gid){
            
            
            axios({
                method: 'delete',
                url: 'http://127.0.0.1:8000/myapp/goods_dur/' + gid +'/'
            }).then(function(res){
                window.location.reload()
            })
        },
        // 获取分页码，并重新调用get_goods()方法
        get_page(event){
            this.p = event.target.value
            this.get_goods()
        }
    },
    // 使用created钩子函数自动加载当前页面的信息
    created(){
        this.get_goods()
    }
}
</script>

```

#### 详情页

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
            <tr>
                <td>{{goods.id}}</td>
                <td>{{goods.goods_name}}</td>
                <td><img :src="goods.goods_img" alt=""></td>
                <td>{{goods.goods_content}}</td>
                <td>{{goods.goods_price | message}}</td>
                
            </tr>
        </table>
    </div>
</template>
<script>
import axios from 'axios'
export default {
    data(){
        return{
            uid: this.$route.params.id,
            goods: ''
        }
    },
    created(){
        console.log(this.uid)
        axios({
            method: 'get',
            url: 'http://127.0.0.1:8000/myapp/goods_dur/'+this.uid
        }).then(res=>{
            console.log(res.data)
            this.goods = res.data
        })
    }
}
</script>
```

#### 修改 Update.vue

```vue
<template>
    <div>
        <form action="" @submit.prevent="update_goods">
            <p>商品名称：<input type="text" v-model="name"></p>
            <p>商品价格：<input type="text" v-model="price"></p>
            <p><img :src="goods.goods_img" alt=""></p>
            <p>商品图片：<input type="file" id='img'></p>
            <p>商品详情：<input type="text" v-model="describe"></p>
            <p>商品分类：
                <select name="" id="" @change="get_id($event)">
                    <option v-for="cate in cate_array" :value="cate.id">{{cate.c_name}}</option>
                </select>                           
            </p>
            <p> <input type="submit" value="修改"></p>
        </form>
    </div>
</template>
<script>
import axios from 'axios'
export default {
    data(){
        return{
            uid: this.$route.params.id,  // 获取路由中传递的id
            goods: '',
            name: '',
            price: '',
            describe: '',
            cate_array: [],
            cate: ''
        }
    },
    methods: {
        // 通过事件，获取分类的id
        get_id(event){
            this.cate = event.target.value
        },
        // 获取后台的商品分类
        get_cate(){
            var self = this
            axios({
                method:'get',
                url: 'http://127.0.0.1:8000/myapp/cate_view/'
            }).then(function(res){
                self.cate_array = res.data
            })
        },
        // 使用put方式修改商品，制作成表单数据，并提交到后台
        update_goods(){
            var img = document.getElementById('img')
            let form_data = new FormData()
            form_data.append('goods_name', this.name)
            form_data.append('goods_price', this.price)
            form_data.append('goods_img', img.files[0])
            form_data.append('goods_content', this.describe)
            form_data.append('cate', this.cate)
            axios({
                method: 'put',
                url: 'http://127.0.0.1:8000/myapp/goods_dur/'+ this.uid + '/',
                data: form_data
            }).then(res=>{
                console.log(res.data)
            })
        }
    },
    created(){
        this.get_cate()
        console.log(this.uid)
        // 通过id获取要修改的商品的信息， get方式，获取单条数据
        axios({
            method: 'get',
            url: 'http://127.0.0.1:8000/myapp/goods_dur/'+this.uid
        }).then(res=>{
            console.log(res.data)
            this.goods = res.data
            this.name = res.data.goods_name
            this.price = res.data.goods_price
            this.describe = res.data.goods_content
            this.cate = res.data.cate
        })
    }
}
</script>
```

#### router下index.js添加

```vue
import Vue from 'vue'
import Router from 'vue-router'
import Index from '@/components/Index'
import Detail from '@/components/Detail'
import Update from '@/components/Update'
Vue.use(Router)

export default new Router({
  mode: 'history',  // 去掉路由地址当中的#
  routes: [
    {
      path: '/',
      name: 'Index',
      component: Index
    },
    {
      path: '/detail/:id',
      name: 'Detail',
      component: Detail
    },
    {
      path: '/update/:id',
      name: 'Update',
      component: Update
    } 
  ]
})

```

#### 在src下的main.js中添加filter过滤器

```vue
Vue.filter('message', function(msg){
  return msg + '元'
})
```

#### 组件嵌套

**写Footer.vue**

```vue
<template>
    <div>
        <footer>
            <p>{{copyright}}</p>
        </footer> 
    </div>
</template>

<script>
    export default {
    data () {
        return {
            copyright:"Copyright 2019 Vue Demo"
 
        }
    }
}
</script>
```

**在App.vue中引入组件**

```vue
<template>
  <div id="app">
    
    <router-view/>
    <app-footer></app-footer>  
  </div>
</template>

<script>
import Footer from '@/components/Footer'
export default {
  name: 'App',
  component:{
  	  'app-footer': Footer    
  }
}
</script>
```

