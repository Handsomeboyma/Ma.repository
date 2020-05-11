### 多环境配置

由于实际开发过程中，需要在本地环境，测试环境，和生产环境切换，避免需要多次切换请求地址和相关配置，在vue-cli3 中的相关配置如下两种方式：

* 第一种 

  1.在根目录新建3个文件 ,后缀为（只有后缀的文件）：<font color='red'>.env.development</font>  和<font      color='red'>`.env.production`，`.env.test`</font> 

  <font color='red'>.env.development</font>   模式用于开发环境 ；

  <font color='red'>.env.production</font> 用于生产环境；

  <font color='red'> .env.test</font>  用于测试环境；

  

  2.在<font color='#F56C6C'>相应</font>的文件夹下写下对应内容

  ```javascript
  // 开发环境 .env.development
  VUE_APP_BASE_API = 'http:// development url' 开发环境地址 
  
  // 线上环境 .env.production 
  VUE_APP_BASE_API = 'http:// production url' 开发环境地址 
  
  // 测试环境 .env.test
  VUE_APP_BASE_API = 'http:// test url' 开发环境地址 
  ```

  当需要用到该变量的时候就用 procsee.env.VUE_APP_BASE_API 拿到使用。

  

  3.修改 packpage.json 文件内容 

  ```javascript
   "scripts": {
      "dev": "vue-cli-service serve",
      "test": "vue-cli-service serve --mode test",
      "build": "vue-cli-service build",
      "build:test": "vue-cli-service build --mode test",
      "lint": "vue-cli-service lint"
    },
  ```

* 第二种方式 

  1.在src文件目录下 新建一个 config 文件夹 ，下面新建一个index.js文件， 根据全局的环境变量来进行判断，然后输出。

  ```javascript
  const modeUrl = {
      // 生产环境
      'production':{
          baseURL:"生产环境-url "，
          authBaseURL:''
      },
      
      // 开发环境 
      'development':{
           baseURL:"开发环境-url "，
          authBaseURL:''
      }
        'production':{
          baseURL:"测试环境-url "，
          authBaseURL:''
      },
  }
  export default modeUrl[process.env.NODE_ENV]
  ```

  2.然后修改packpae.json 文件 和第一种方式一致，然后在引用

  ```javascript
  import config from '../config/index'
  
  config.baseURL
  ```

  3 运行命令 

  ```javascript
   npm run dev // 开发环境
  
  npm run test // 测试环境
  
  npm run build // 正式环境打包
  
  npm run build:test // 测试环境打包
  ```



 ### 2.axios封装

在vue项目中，和后台交互获取数据这块，我们通常使用的是axios库，它是基于promise的http库，可运行在浏览器端和node.js中。他有很多优秀的特性，例如拦截请求和响应、取消请求、转换json、客户端防御XSRF等。所以我们的尤大大也是果断放弃了对其官方库vue-resource的维护，直接推荐我们使用axios库。如果还对axios不了解的，可以移步[axios文档](https://www.kancloud.cn/yunye/axios/234845)。

* 第一步 安装依赖  在终端输入  npm install axios --save 

* 配置axios文件 在/src 下新建 utils 公共文件夹 ，然后新增一个 request.js对aixos 进行一个初始化

  ``` javascript
  import axios from 'axios'
  import config from '../config/index' // 路径配置
  
  // 创建axios 实例 
  const http = axios.create({
      baseURL: config.baseURL
      timeout: 6000 //请求延时时间
  })
  
  // 由于axios 可以配置响应拦截和请求拦截 ，可做如下配置
  
   // 请求拦截器
  http.interceptors.request.use(
    config => {
        //定义config的一些配置
        return config
    }，
      error => {
      //处理异常
      Promise.reject(error)
      
  )
  
  // 响应拦截
  http.interceptors.response.use(
    response =>{
        const res = response.data 
        // 对返回的数据做处理
        
        return res
    },
      error =>{
          // 处理出错的逻辑
          
          return Promise.reject(error)
      }
  )
  export default http
  
  ```

* api 请求配置 在 /src/api 文件下新建分类api.js , http.js ,manage.js  

   1./src/api/manage.js

   ```javascript
import {axios} from '@/utils/request'
 
// 定义需要用到的 api
const api = {
    // 例：
    user:'/api/uesr',
    role:'/api/role'
    ....
}
export default api 

//封装公共请求方法 

// post 
export function postAction(url,parameter){
    return axios({
        url: url ,
        method:' post',
        data:parameter
    })
}

// get
export function getAction(url,paramatet){
    return axios ({
        url:url,
        method:"get",
        data:parameter
    })
}
// 调用 api
export function getUserList(parameter){
    return axios({
        url: api.user,
        method:'get'
        params: parameter
    })
}

   ```

2. src/api / api.js 

```javascript
import {postAction,getAction,getUserList} from '@/api/manage'

// 不同模块打好注释 便于管理
// 角色管理
const addRole = (paramas) => postAciton('/sys/role/add',paramas)
const queryall = (paramas) => getAciton('/sys/role/add',paramas)

// ...管理
...

export { addRole, queryall, ...}
```

3.  在需要的组件下引入 

```javascript
<templata>
<div clas='component'>
<input v-model='value'/>
</div>
</templata>

<script>
import {addRole} from '../api/api'
export default = {
  name:"component",
  data(){
   return{
    value:''
   }
  },
  method:{
      // async await 是ES7 中的处理异步的 使异步看起来更像同步操作
    async add(){
        let paramas = this.value
      let res = await addRole(paramas)
    }
  }
}
</script>
```

axios 的配置 基本如此 ，可能还有没注意到的地方 需要以后慢慢修改。

ps： 此文仅为平时学习作为一个记录使用。

