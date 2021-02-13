### 解决跨域问题------django解决方案

**1、安装django-core-headers**

```python
pip install django-cors-headers
```

**2、配置settings文件**

- **加入到INSTALLED_APPS下面**

  ```python
   INSTALLED_APPS = [
      # 第三方框架
      'corsheaders',  # 解决跨域问题
  
  ]
  ```

- **配置MIDDLEWARE中间件，放在SessionMiddleware和CommonMiddleware中间。**

  ```python
  MIDDLEWARE = [
      'django.contrib.sessions.middleware.SessionMiddleware',
      'corsheaders.middleware.CorsMiddleware',  # 解决跨域问题，必须放在这个位置，加载顺序
      'django.middleware.common.CommonMiddleware',
      
  ]
  ```

- **配置允许跨域访问的域名**

  ```python
  CORS_ORIGIN_ALLOW_ALL = True   # 解决跨域，配置允许跨域访问的域名，为True时，允许所有的域名
  ```

  **注意：这三项都是在django项目下的seetings中配置的**



### 解决跨域问题------VUE解决方案

```vue
proxyTable: {
    '/api': {  //使用"/api"来代替"http://f.apiplus.c" 
    target: 'http://127.0.0.1:8000/', //源地址 
    changeOrigin: true, //改变源 
    pathRewrite: { 
      '^/api': '' //路径重写 
      } 
  } 
}
```





### 安装axios

**1、安装axios**

```vue
cnpm install --save axios
```

**2、配制axios,在src文件下的mian.js中配制**

```python
import axios from 'axios'
Vue.prototype.axios = axios
```

**3、axios使用**

```python
axios完整写法：

this.axios({
  method: 'post',
  url: '/user/12345',
  data: {
    firstName: 'Fred',
    lastName: 'Flintstone'
  }
}).then((res)=>{
  	console.log(res)
}).catch((error)=>{
  	console.log(error)
 });

post请求

this.axios.post('',{}).then((res)=>{}).catch((error)=>{})

get请求

<script>
    import Vue from 'vue'
    import axios from 'axios'
    export default{
      name: 'card',
      mounted:function () {
        //vue页面加载时自动执行
        this.send()
      },
      data:{
        url_array: []
        
      },
      methods:{
        
        send(){
          var self = this
              axios({
                  method:'get',
                  url:'http://127.0.0.1:8000/myapp/api_type/'
              }).then(function(res){
                  console.log(res.data.li_list);
                  self.url_array = res.data.li_list
                  console.log(self.url_array)
              });
          }        
      }
    }
    

</script>
```





