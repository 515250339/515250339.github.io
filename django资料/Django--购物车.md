### 购物车

#### settings配置

```python
# 缓存redis设置
# 配置Redis为Django缓存
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/2",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    }
}

# 将session缓存在Redis中
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
SESSION_CACHE_ALIAS = "default"

# session 设置
SESSION_COOKIE_AGE = 60 * 60 * 12  # 12小时
SESSION_SAVE_EVERY_REQUEST = True
SESSION_EXPIRE_AT_BROWSER_CLOSE = True  # 关闭浏览器，则COOKIE失效
```

##### 购物车中redis的操作（哈希类型）

```python
# 导入django-redis
from django_redis import get_redis_connection

# 连接redis
conn = get_redis_connection('default')

# 获取单个值（数量）
cart_num = conn.hget(key, field)

# 设置单个
conn.hset(key, field, value)

# 获取key下面的总数量
cart_sum = conn.hlen(key)

# 获取key下的所有的键值对
conn.hgetall(key)

# 删除某一条商品
conn.hdel(key, field)
```

#### django后台views代码

```python
class AddCart(View):
    """
    添加购物车
    """
    def post(self, request):
        """
        <QueryDict: {'user_id': ['10'], 'goods_id': ['5'], 'cart_num': ['1']}>
        """
        print(request.POST)
        user_id = request.POST.get('user_id')
        goods_id = request.POST.get('goods_id')
        cart_num = int(request.POST.get('cart_num'))

        try:
            numbers = int(cart_num)
        except Exception as e:
            return HttpResponse(json.dumps({'msg': '数量值不正确', 'code': 404}))

        # 连接redis
        conn = get_redis_connection('default')

        # 获取redis中商品的数量
        cart_count = conn.hget(user_id, goods_id)
        print(cart_count)
        if cart_count:
            numbers += int(cart_count)
        conn.hset(user_id, goods_id, numbers)
        cart_sum = conn.hlen(user_id)
        return HttpResponse(json.dumps({'msg': "OK", 'code': 200, 'cart_sum': cart_sum}))


class ShowCartView(View):
    """
    获取购物车内的内容
    """
    def get(self, request, user_id):
        conn = get_redis_connection('default')
        cart_all = conn.hgetall(user_id)
        print(cart_all)
        goods_list = []
        for k, v in cart_all.items():
            goods_id = k.decode('utf-8')
            goods_num = v.decode('utf-8')
            goods = Goods.objects.get(pk=goods_id)
            goods_list.append({
                'goods_name': goods.goods_name,
                'goods_price': str(goods.goods_price),
                'goods_code': goods.pk,
                'goods_number': goods_num,
                'goods_sum': float(goods.goods_price) * int(goods_num)
            })
        goods_sums = 0
        for i in goods_list:
            goods_sums += i.get('goods_sum')
        return HttpResponse(json.dumps({'msg': "OK", 'code': 200, 'goods_list': goods_list, 'goods_sums': goods_sums}))


class JianView(View):
    """
    前端减数量接口
    """
    def post(self, request):
        print(request.POST)
        user_id = request.POST.get('user_id')
        goods_id = request.POST.get('goods_id')
        cart_num = int(request.POST.get('cart_num'))

        try:
            numbers = int(cart_num)
        except Exception as e:
            return HttpResponse(json.dumps({'msg': '数量值不正确', 'code': 404}))
        # 连接redis
        conn = get_redis_connection('default')

        # 获取redis中商品的数量
        cart_count = conn.hget(user_id, goods_id)
        print(cart_count)

        if cart_count:
            cart_count = int(cart_count)
            if cart_count > 1:
                cart_count -= 1

            conn.hset(user_id, goods_id, cart_count)
            cart_sum = conn.hlen(user_id)
            return HttpResponse(json.dumps({'msg': "OK", 'code': 200, 'cart_sum': cart_sum}))
        else:
            return HttpResponse(json.dumps({'msg': "商品不存在", 'code': 200}))


class JiaView(View):
    """
    前端加数量接口
    """
    def post(self, request):
        print(request.POST)
        user_id = request.POST.get('user_id')
        goods_id = request.POST.get('goods_id')

        # 连接redis
        conn = get_redis_connection('default')

        # 获取redis中商品的数量
        cart_count = conn.hget(user_id, goods_id)
        print(cart_count)

        if cart_count:
            cart_count = int(cart_count)
            cart_count += 1

            conn.hset(user_id, goods_id, cart_count)
            cart_sum = conn.hlen(user_id)
            return HttpResponse(json.dumps({'msg': "OK", 'code': 200, 'cart_sum': cart_sum}))
        else:
            return HttpResponse(json.dumps({'msg': "商品不存在", 'code': 200}))


```

#### 前端vue代码, 展示页面

```vue
<template>
    <div>
        <table>
            <tr>
                <td>编号</td>
                <td>名称</td>
                <td>价格</td>
                <td>参数</td>
                <td>详情</td>
                <td>图片</td>
                <td>分类</td>
                <td>操作</td>
            </tr>
            <tr v-for="goods in goods_array">
                <td>{{goods.id}}</td>
                <td>
                    <router-link :to="{name: 'Detail', params: {'pk': goods.id}}">{{goods.goods_name}}</router-link>
                </td>
                <td>{{goods.goods_price}}</td>
                <td>{{goods.parameter}}</td>
                <td v-html="goods.goods_content"></td>
                <td><img :src="'http://127.0.0.1:8000' + goods.goods_img" alt=""></td>
                <td v-for="c in cate_array" v-if="c.id == goods.cate">{{c.cate_name}}</td>
                <td>
                    <a href="#" @click.prevent="del_goods(goods.id)">删除</a>
                    <router-link :to="{name: 'UpdateGoods', params: {'pk': goods.id}}">修改</router-link>
                    <a href="#" @click.prevent="add_cart(goods.id)">加入购物车</a>
                    <router-link :to="{name: 'Cart'}">去购物车({{cart_sum}})</router-link>
                </td>
            </tr>
        </table>
        <p>
            <button @click.prevent="up_num">上一页</button>
            <button v-for="p in page_array" @click.prevent="get_page_num(p)">{{p}}</button>
            <button @click.prevent="down_num">下一页</button>
        </p>
    </div>
</template>

<script>
import axios from 'axios'
export default {
    data() {
        return {
           goods_array: Array(),  // List() []
           cate_array: [],
           page: 1 ,
           page_array: [],
           current_page: '',
           sum_page: '',
           user_id: sessionStorage.getItem('user_id'),  // 通过session获取用户id
           cart_sum: ''
        }
    },
    methods: {
        get_goods() {  //跳转到当前页，并根据页码，获取当前页的商品信息
            axios({
                url: 'http://127.0.0.1:8000/mdadmin/page_api/' + this.page,
                method: 'get'
            }).then(res=>{
                console.log(res.data)
                this.goods_array = res.data.goods
                this.page_array = res.data.page_list
                this.current_page = res.data.number
                this.sum_page = res.data.page_nums
            })
        },
        get_cate() {  // 获取分类
            axios({
                url: 'http://127.0.0.1:8000/mdadmin/cate_api/',
                method: 'get'
            }).then(res=>{
                console.log(res.data)
                this.cate_array = res.data
            })
        },
        del_goods(gid) { // 删除商品
            axios({
                url: 'http://127.0.0.1:8000/mdadmin/goods_detail/' + gid + '/',
                method: 'delete'
            }).then(res=>{
                console.log(res.data)
                window.location.reload()
            })
        },
        get_page_num(page_num) { // 获取页码，重新给page赋值，并调用get_goods()
            this.page = page_num
            this.get_goods()
        },
        up_num(){ // 上一页
            if(this.current_page>1 ) {
                this.page -= 1
                this.get_goods()
            }else if(this.current_page == 1){
                this.page = 1
                this.get_goods()
            }
        },
        down_num() {  // 下一页
            if(this.current_page < this.sum_page) {
                this.page += 1
                this.get_goods()
            }else if(this.current_page == this.sum_page){
                this.page = this.sum_page
                this.get_goods()
            }
        },
        add_cart(goods_id) {
            let form_data = new FormData()
            form_data.append('user_id', this.user_id)
            form_data.append('goods_id', goods_id)
            form_data.append('cart_num', 1)
            axios({
                url: 'http://127.0.0.1:8000/mdadmin/add_cart/',
                method: 'post',
                data: form_data
            }).then(res=>{
                console.log(res.data)
                this.cart_sum = res.data.cart_sum
            })
        }

    },
    created() {
        this.get_goods()
        this.get_cate()
        
        if(sessionStorage.getItem('name')){
            
        }else{
            window.location.href = '/login'
        }
    }
}
</script>
```

#### 购物车页面

```vue
<template>
    <div>
        <table>
            <tr>
                <td>商品编号</td>
                <td>名称</td>
                <td>价格</td>
                <td>数量</td>
                <td>总价</td>
            </tr>
            <tr v-for="goods in goods_array">
                <td>{{goods.goods_code}}</td>
                <td>{{goods.goods_name}}</td>
                <td>{{goods.goods_price | msg}}</td>
                <td>
                    <button @click.prevent="jian(goods.goods_code)">-</button>
                    <input type="text" :value="goods.goods_number" style="width: 20px" id="input1">
                    <button @click.prevent="jia(goods.goods_code)">+</button>
                </td>
                <td>{{goods.goods_sum | msg}}</td>
            </tr>
            <tr>
                <td></td>
                <td></td>
                <td></td>
                <td>总价：</td>
                <td>{{goods_sums | msg}}</td>
                
            </tr>
        </table>
    </div>
</template>
<script>
import axios from 'axios'
export default {
    data(){
        return {
            user_id: sessionStorage.getItem('user_id'),
            goods_array: [],
            goods_sums: ''
        }
    },
    methods: {
        get_goods() {
            axios({
                url: 'http://127.0.0.1:8000/mdadmin/show_cart/' + this.user_id,
                method: 'get'
            }).then(res=>{
                console.log(res.data)
                this.goods_array = res.data.goods_list
                this.goods_sums = res.data.goods_sums
            })
        },
        jian(goods_id) {  // 减数量
            let form_data = new FormData()
            form_data.append('user_id', this.user_id)
            form_data.append('goods_id', goods_id)
            
            axios({
                url: 'http://127.0.0.1:8000/mdadmin/jian/',
                method: 'post',
                data: form_data
            }).then(res=>{
                if(res.data.code == 200){
                    this.get_goods()
                }
            })
        },
        jia(goods_id) {  // 加数量
            let form_data = new FormData()
            form_data.append('user_id', this.user_id)
            form_data.append('goods_id', goods_id)
            
            axios({
                url: 'http://127.0.0.1:8000/mdadmin/jia/',
                method: 'post',
                data: form_data
            }).then(res=>{
                if(res.data.code == 200){
                    this.get_goods()
                }
            })
        }
    },

    created() {
        if(sessionStorage.getItem('user_id')){
            this.get_goods()
        }else{
            window.location.href = '/login'
        }
    }
}
</script>
```



