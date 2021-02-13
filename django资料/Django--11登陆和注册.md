## django--登陆和注册

### django模版登陆注册

> 使用的技术以及实现**





### django+vue登陆注册

#### 登陆

- **django后台做api**

  ```python
  from django.http import HttpResponse
  
  import json
  
  
  def api_login(request):
      """
      登陆接口，get和post方式
      """
      if request.method == "GET":
          return HttpResponse(json.dumps({"code": 404}))
      elif request.method == "POST":
          info = request.POST
          print(info)
          # 获取到数据以后，做逻辑判断，成功返回200，失败返回404
          return HttpResponse(json.dumps({"code": 200}))
  ```

- **使用axios做ajax--post提交数据**

  ```vue
  <script>
      import axios from 'axios'
      export default {
          // 获取用户名和密码框输入的内容，用字典的形式返回
          data:function(){
              return {
                  username: '',
                  password: '',
              }            
          },
          methods:{
              
              on_submit(){
                  // 创建一个FormData()实例
                  let form_data = new FormData()
                  // Application/Json -> request.POST(form-data)
                  // url: 127.0.0.1:8000/api/register
                  form_data.append('username', this.username)                
                  form_data.append('password', this.password)
                  // 使用axios用post方式提交数据
                  axios({
                      method: 'post',
                      url: 'http://127.0.0.1:8000/apilogin/api_login/',
                      data: form_data
                  }).then(function(res){
                      // 成功以后，返回的内容
                      console.log(res)
                      // 成功以后，做页面跳转
                  }).catch((error)=>{
                      // 失败以后，返回的内容
                      console.log(error)
                      // 失败以后，停留在当前页面
                  });
              }
          }
          
      }
  </script>
  ```



#### 注册



