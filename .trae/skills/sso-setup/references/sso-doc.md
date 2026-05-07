# 安装

### 1. 特点
1. **通用**：pc端和H5端通用登录，前端多个web项目可以对应一个后台服务
2. **体积小**：(压缩后的)体积大小仅**14kb**，为项目体积减负
3. **灵活易用**：项目可以使用任意第三方的 http 请求，登录和项目使用的技术栈（vue、react、angular 等）无关


### 2. 对接
1. **前端对接人** 周冰冰-01698009
2. **后端对接人** 张强-01714308 <a href="https://ytokeji.yuque.com/staff-zwdr9d/iln5z6/toeoa5xe68dy5h4g" target="_blank">oauth2SDK对接文档</a>

### 3. 安装
<span class="c-red">axios 版本需要升级到 1.1.2 及以上，在 0.18 版本中 axios 在 error 时包装了数据结构导致登录包拿不到数据</span>
```
npm uninstall axios
npm install -S axios@^1.7.5
```

安装登录包
```
npm config set registry https://npm.yto.net.cn
npm install -S @yto/ulp-sdk-general-js
```

### 5. 功能介绍
1. 1 可以生成下游`oauth2`、`YTO-TGC`系统的链接
2. 2 PC、H5（移动端） 项目通用
3. 3 支持免登页面配置、支持一个ip相同端口 通过nginx路径区分部署多个项目
4. 4 支持同一个项目（同一个浏览器）多个窗口打开并保持登录状态同步的机制
5. 5 一个后端服务可以支持多个前端项目( 后端服务可以配置多个前端 clientId )，不同的前端项目单独申请 oauth2 的 clientId
6. 6 支持前端微应用（乾坤）中的登录场景

### 6. 缓存和心跳

1. 1 登录相关的cookie缓存键名有：<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;clientIdName_cookie_access_token（access_token）<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;clientIdName_cookie_params_string（其它登录参数）<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;clientIdName_cookie_user_code_md5（ 工号的 Md5加密 Md5.hashStr(工号).toString() ）
2. 2 在项目初始化成功后项目会插入一个iframe元素用于心跳推送


# 快速上手
### 1.联系张强（01714308）申请项目的 clientId

1. 填写项目的 clientId（自定义）
2. 填写测试环境域名（扫码回调地址）、<span class="c-pink">在本地开发时需要在 host 中添加域名拦截到本地， 例如：127.0.0.1 smartoa-new-inter.yto56test.com </span>
3. 填写生产环境域名（生产扫码回调环境地址可以在发布之前在联系张强修改）

### 2.前端代理配置

```javascript
// webpack.config.js
devServer: {
  // 项目构建代理配置示例（依据项目需要自行调整配置）
  proxy: {
    '/api': {
      target: 'http://10.130.134.45:8282',
      pathRewrite: { '^/api': '' }
    }
  },
  // 添加 hosts文件配置: 127.0.0.1 yto-oauth2-inter.yto56test.com (本地开发时把测试域名拦截到本地)
  // 该域名为 oauth2 配置的 uat 回调地址
  host: 'yto-oauth2-inter.yto56test.com',
  port: '8080',
  // historyApiFallback: {
  //   rewrites: [{ from: /^(\/.*)?$/, to: serverPrefixPath + '/index.html' }]
  // },
}
```

### 3. 创建 axios 实例

<span class="c-red">注：axios 版本升级到 1.1 版本及以上</span>

```javascript
// http.js 示例，这里使用 axios 举例：

import axios from 'axios'
// import { message } from '@yto/ui/lib/message'

const http = {}
const axiosProxy = axios.create({
  // 统一设置接口前缀
  baseURL: '/api',
  timeout: 1000 * 30,
  headers: { 'Content-type': 'application/json', Authorization: '' }
})

function successHandler({ type, config, result, resolve, reject, url, payload, default_value }) {
  // 返回结果处理
  const response = result.data || {}
  // 1300 依据自己项目的 code 码自行调整
  if (response.code === 1300) {
    let result_proxy = response.data || response.data === false ? response.data : default_value === undefined ? {} : default_value
    // 把对象中 null 值，转换成 undefined 以便默认给值
    resolve(result_proxy)
  } else if (response.errMsg) {
    // message({ message: response.errMsg, status: 'warning' })
    reject(response)
  } else {
    reject(response)
  }
}

// post 请求
http.post = function (url, payload, config = {}, default_value) {
  return new Promise(function (resolve, reject) {
    axiosProxy
      .post(url, payload, config)
      .then(
        function (result) {
          successHandler({ type: 'post', config, result, resolve, reject, url, payload, default_value })
        },
        function (err) {
          // message({ title: '请求数据异常', message: err.message, status: 'warning' })
          reject(err)
        }
      )
      .catch(function (err) {
        console.log('错误', err)
      })
  })
}

// get 请求
http.get = function (url, config = {}, default_value) {
  return new Promise(function (resolve, reject) {
    axiosProxy
      .get(url, config)
      .then(
        function (result) {
          successHandler({ type: 'get', config, result, resolve, reject, url, payload: {}, default_value })
        },
        function (err) {
          // message({ title: '请求数据异常', message: err.message, status: 'warning' })
          reject(err)
        }
      )
      .catch(function (err) {
        console.log('错误', err)
      })
  })
}

export default {
  install: function (Vue) {
    Vue.prototype.$http = http
  }
}
export { http, axiosProxy }
```

### 4.调用 loginAction 方法初始化登录

```javascript
// mian.js
import { loginAction, loginInfo } from '@yto/ulp-sdk-general-js'
import { axiosProxy } from '@/api/http.js'
// 配置参数详见loginAction(options:Object) 初始化参数介绍
loginAction({
  clientId: 'XXX', // 联系张强（01714308）申请项目的clientId
  env: 'uat', // 生产环境传 'prod' 其他传 'uat'
  http: axiosProxy
  // cookiePath: '/',   // 如果 nginx 的一个端口通过不同路径部署多个项目时，需要填对应的路径
  // stopLink: true,     // 登录失败时，不自动跳转到登录扫码页面（用于项目调试阶段）
}).then(
  // 登录成功，开始初始化项目
  function () {
    const ps = [] // 这里可以发起项目加载依赖的接口（菜单权限、按钮权限、数据字典 等）
    Promise.all(ps).then(function () {
      new Vue({
        el: '#app',
        components: {
          app: App
        },
        provide: {
          userInfo: loginInfo.userInfo
        },
        template: '<app />'
      })
    })
  },
  function () {}
)
```

### 5. 登录包导出示例

```js
import {
  loginAction, // 登录初始化配置, 建立登录( 返回 Promise )
  loginInfo, // loginInfo.clientId, loginInfo.access_token(token), loginInfo.userInfo(用户信息)
  login, // 登录的方法
  logout, // 退出的方法
  buildOtherSysMenuUrl, // 构建oauth2项目登录地址的方法，返回 Promise
  buildShareUrl, // 构建oauth2项目（可以是自己项目的链接）一次性分享地址的方法，返回 Promise
  buildTGCSysMenuUrl // 构建YTO-TGC项目系统登录地址的方法，返回 Promise
} from '@yto/ulp-sdk-general-js'
```

### 6.loginInfo对象属性说明

| 属性                | 类型     | 说明                                                                                                                                                                                |
| :------------------ | :------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| clientId            | `String` | 当前系统的 clientId                                                                                                                                                                 |
| access_token        | `String` | 当前系统的token                                                                                                                                                                     |
| token               | `String` | 当使用token+source 登录时，记录 token 参数                                                                                                                                          |
| source              | `String` | 当使用token+source 登录时，记录 source 参数                                                                                                                                         |
| terminal            | `String` | 当使用token+source+terminal 登录时，记录 terminal 参数； 一般用于网点管家                                                                                                           |
| userInfo            | `Object` | 用户信息                                                                                                                                                                            |
| source_client_id    | `String` | 如果当前系统是被上游打开的（信息系统导航或者其它系统），记录上游系统的 clientId。<span class="c-pink">通过扫码登录时则为空</span>                                                   |
| top_login_client_id | `String` | 当 B 系统打开 D 系统时， 打开的类型为新窗口打开 D（ulp_open_type 为2时登录解耦），此时 D 系统的 top_login_client_id 一定为自身系统D的 clientId                                      |
|                     |          | 当 B 系统打开 C 系统时， 打开的类型为 iframe 嵌套 C（ ulp_open_type 为1时），此时 C 系统的 top_login_client_id 不一定为 B 系统的 clientId，有可能是打开 B 系统的上游系统的 clientId |

**source_client_id 和 top_login_client_id 字段说明**

<img src="/npm-doc/static/img/field-desc.png" alt="source_client_id 与 top_login_client_id 字段说明" class="max-w-100-p h-auto" />

### 7. 导出方法介绍

| 方法名               | 参数类型                        | 返回值    | 说明                                                                                                                                                                                                                                                                       |
| -------------------- | ------------------------------- | --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| loginAction          | `Object`                        | `Promise` | 登录初始化配置, 建立登录<br/>详见**loginAction(options:Object) 方法参数**                                                                                                                                                                                                  |
| login                | -                               | -         | 登录                                                                                                                                                                                                                                                                       |
| logout               | `redirect_uri:String`           |           | `redirect_uri`: 可选，参数表示退出系统后，跳转的链接地址<br/>默认将到登录扫码页面                                                                                                                                                                                          |
| buildOtherSysMenuUrl | `url:String` `open_type:Number` | `Promise` | 构建oauth2项目登录地址, 该方法返回的授权链接只能用于当前浏览器打开<br/>参数 `url` 为oauth2项目的链接，可自行拼接其他参数<br/>参数 `open_type`：默认 2 ，当您需要<span class="c-red">当前系统iframe内嵌打开</span>返回的地址时设置 1， 否则无需设置 <br/>将返回一个`newUrl` |
| buildShareUrl        | `url:String`                    | `Promise` | 构建oauth2项目一次性免登地址的方法<br/>参数 `url` 为oauth2项目的链接，可自行拼接其他参数<br/>将返回一个`newUrl`：`url?onceCode=xxx&user_code_sign=xxx`<br/>                                                                                                                |
| buildTGCSysMenuUrl   | `url:String`                    | `Promise` | 构建YTO-TGC项目系统登录地址的方法<br/>参数 `url` 为TGC项目系统的链接，可自行拼接其他参数<br/> 将返回一个`newUrl`：`url?YTO-TGC=xxx`                                                                                                                                        |

### 8. loginAction(options:Object) 方法参数

<table style="width:100%; text-align:left;">
  <tr style="vertical-align: top;">
    <th width="80">名称</th>
    <th width="44%">说明</th>
    <th>类型</th>
    <th>默认值</th>
    <th>可选值</th>
    <th width="80">是否必须</th>
  </tr>
  <tr style="vertical-align: top;">
    <td>stopLink</td>
    <td><span class="c-pink">请求状态码为401时，不自动跳转到登录扫码页面（用于项目调试）</span></td>
    <td>Boolean</td>
    <td>false</td>
    <td>true / false</td>
    <td>-</td>
  </tr>
  <tr style="vertical-align: top;">
    <td>clientId</td>
    <td>oauth2 配置的clientId<br/>联系张强（01714308）申请项目的clientId</td>
    <td>String</td>
    <td>-</td>
    <td>-</td>
    <td>是</td>
  </tr>
  <tr style="vertical-align: top;">
    <td>env</td>
    <td>生产环境传 'prod' 其他传 'uat'</td>
    <td>String</td>
    <td>uat</td>
    <td>uat / prod</td>
    <td>是</td>
  </tr>
  <tr style="vertical-align: top;">
    <td>http</td>
    <td>
      项目用的http请求实例对象, 登陆包只是为实例对象注册响应拦截，在headers中添加token请求信息(<b>没有其它多余操作</b>)<br/>
      <span class="c-orange">这里需要的是：登陆包的请求实例的 baseURL 读取了项目请求实例对象的baseURL(baseURL共享)</span><br/>
      <span class="c-pink">在请求拦截器中的headers 中添加 Authorization client-id 属性，用于授权</span><br/>
      <span class="c-pink">在响应拦截其器中比对了code码是否为401，比对成功则跳转到扫码页面</span>
    </td>
    <td>Object</td>
    <td>-</td>
    <td>-</td>
    <td>是</td>
  </tr>
  <tr style="vertical-align: top;">
    <td>loginBySourceAndToken</td>
    <td>是否开启 url 的 <b class="c-pink"> source </b>和 <b class="c-pink"> token </b>参数登录，如果不开启url中source和token参数是不识别的</td>
    <td>Boolean</td>
    <td>false</td>
    <td>true / false</td>
    <td>否</td>
  </tr>
  <tr style="vertical-align: top;">
    <td>interception401</td>
    <td>
      在app内嵌中登录有效期超时接口返回 401 导致跳转到扫码页，<span class="c-pink">此时如果想阻止项目自动返回扫码登录页面</span></br></br>
      需要开发者返回一个 Promise 对象 resolve({ source, token })，或者重新向服务端请求 YTO-TGC 参数后再 resolve({ yto_tgc }) 重新初始化登录</br></br>
      <span class="c-pink">return new Promise((resolve, reject) => { resolve({source, token, yto_tgc }) })</span>
    </td>
    <td>Promise</td>
    <td>-</td>
    <td>-</td>
    <td>否</td>
  </tr>
 <tr style="vertical-align: top;">
    <td>parentClientId</td>
    <td>
    <span class="c-pink">一般用于微应用开发中(主应用与子应用同域名不同端口cookie共享)</span><br/>
    如果在子应用 cookie 中读到 parentClientId 缓存信息，clientId 将被替换成 parentClientId 中的一个<br/>
    主应用A嵌套子应用B的登陆场景，此子应用B的子应用需要借助主应用A的clientId和token建立登录（ 由于 oauth2 的登录机制限制，子应用在没有iframe隔离的情况下不能单独登录，否则浏览器的url将会跳转 ）<br/>
    <span class="c-pink">服务端配置：在子应用的后端服务中需要添加主应用的 clientId</span><br/>
    <img class="max-w-100-p" src="/npm-doc/static/img/yml.png" />
    </td>
    <td>Array</td>
    <td>[]</td>
    <td>-</td>
    <td>否</td>
  </tr>
  <tr style="vertical-align: top;">
    <td>httpType</td>
    <td>项目使用 http 技术，<span class="c-orange">现在登陆包只有 axios 的逻辑，如果使用的请求体不是axios创建的，联系开发人员 01698009 添加对应的的逻辑；</span></td>
    <td>String</td>
    <td>axios</td>
    <td>-</td>
    <td>-</td>
  </tr>
  <tr style="vertical-align: top;">
    <td style="border:none;">httpDataPlace</td>
    <td>
      如果项目对登录相关接口添加了层级包装，登录包会试探性的取值 response.data、response.data.data <br/>
      如果是其他键名的自定义包装层级需要指定<span class="c-pink">httpDataPlace</span> 属性<br/>
      多个嵌套需要用 '.' 链接<br/>
      也就是你在控制台中 network 中看到的层级
    </td>
    <td>String</td>
    <td>-</td>
    <td>-</td>
    <td>-</td>
  </tr>
  <tr>
    <td></td>
    <td colspan="4"><img class="max-w-100-p" src="/npm-doc/static/img/user.png" /></td>
  </tr>
  <tr style="vertical-align: top;">
    <td>httpErrorCode</td>
    <td>
      用于响应拦截器比对, 如果等于该值将跳转到登录扫码页面
      <br/>
      <span class="c-red">注：严格对比httpErrorCode === respone.status，默认为Number类型401</span>
    </td>
    <td>Number、String</td>
    <td>401</td>
    <td>-</td>
    <td>-</td>
  </tr>
  <tr style="vertical-align: top;">
    <td style="border:none;">httpErrorCodePlace</td>
    <td>用于响应拦截器查找状态码的位置<br/>（对象的嵌套位置用 '.' 链接）</td>
    <td>String</td>
    <td>response.status</td>
    <td>-</td>
    <td>-</td>
  </tr>
  <tr >
    <td></td>
    <td colspan="4"><img class="max-w-100-p" src="/npm-doc/static/img/error.png" /></td>
  </tr>
  <tr style="vertical-align: top;">
    <td>httpOthers</td>
    <td>
      如果项目创建了多个请求实例请求其它项目的后台, 可以把这些请求示例添加到登录包中，注册请求和响应拦截</br>
      [{type:'axios', http: Instance, errorCode:'选填', errorCodePlace:'选填'}, ...]</br>
      <span class="c-pink">开发者也可以自己项目其它的请求实例注册请求和响应拦截</span>
    </td>
    <td>Array:</td>
    <td>-</td>
    <td>-</td>
    <td>-</td>
  </tr>
  <tr style="vertical-align: top;">
    <td>excludePage</td>
    <td>免登页面的路径集合</br>
      例如：['/', '/index', '/path/:id/path1']；</br>
      如果nginx有前缀路径需要加上</br>
      <span class="c-pink">这些免登页面的接口需要服务端放开权限</span></td>
    <td>Array</td>
    <td>/</td>
    <td>-</td>
    <td>-</td>
  </tr>
  <tr style="vertical-align: top;">
    <td>heartbeatInterval</td>
    <td>子系统推送心跳的间隔时间,不能小于 1000* 10(10s)</td>
    <td>Number</td>
    <td>1000 * 60</td>
    <td>-</td>
    <td>-</td>
  </tr>
  <tr style="vertical-align: top;">
    <td>customLoginErrorHandler</td>
    <td>自定义登录异常时的自定义处理逻辑，一般用于跳转到自定义登录页面</td>
    <td>-</td>
    <td>-</td>
    <td>-</td>
    <td>-</td>
  </tr>
</table>

### 9. dingding 内打开

安卓端APP 在 dingding中首次登录时 location.replace 导致无法返回的问题解决方案

```js
npm install -S dingtalk-jsapi // 在 package.json 中安装 dd 包

import * as dd from 'dingtalk-jsapi' // 在 main.js 中引入 dd 包
window.dd = dd // 添加全局属性供“登录包”使用（ 登录包会执行 window.dd.biz.navigation.close() ）
```


### vue3 对应的 vue-router 路由创建注意事项

### 注意事项
<p class="c-red">1. 由于登录包在初始化登录后会去除登录参数，以及会通过 history.replaceState 重置浏览器的 url 到用户指定的地址（而不是只到项目首页）</p>
<p class="c-red">2. vue-router 提供的 createWebHistory 方法<b>不能在初始化登录之前执行</b>，否则它会记录登录初始化之前的 url ( 包含登录参数以及 pathname 为首页的路径 )
导致在登录后初始化项目时，跳转的路由不是登录包（ 改变 history.replaceState ） 设置的路径
</p>
<p class="c-red">3. <b>需要在初始化登录之后执行 createWebHistory 方法</b>，这样它记录的 url 才是用户想要跳转的页面路径</p>

# vue3 创建路由参考示例

```javascript
// main.js
  import { createApp } from 'vue';
  import router from '@/router/index.js'

  const app = createApp(App)
  loginAction({
    clientId: 'XXX', // 联系张强（01714308）申请项目的clientId
    env: 'uat', // 生产环境传 'prod' 其他传 'uat'
    http: axiosProxy,
  }).then(
    function () {
      app.use(router)     // 在登录初始化成功后执行 注册路由
      app.mount('#app')   // 初始化项目
    },
    function () {}
  )
```


```javascript
// router/index.js
  import { createRouter, createWebHistory } from 'vue-router'
  // 路由列表
  let router = {
    routes: [
      {
        path: '/',
        name: 'index',
        component: 'xxx'
      },
      {
        path: '/path_a',
        name: 'path_a',
        component: 'xxx'
      },
      {
        path: '/:pathMatch(.*)',
        redirect: { name: 'index' }
      }
    ]
  }

  export default {
    install: function (app, options) {
      // createWebHistory() 不能在登录包初始化之前执行，否则登陆包( history.replaceState )设置的 url 将无效
      router.history = createWebHistory()
      const router_proxy = createRouter(router)
      Object.assign(router, router_proxy) // router 变为路由实例对象

      // 路由拦截等
      // router.beforeEach((to = {}, from = {}) => {
      //   return true
      // })

      app.use(router)
    }
  }
  // 导出路由实例对象
  export { router }
```




# 登陆场景及逻辑概要
1. 项目url: http://xxx-inter.yto56test.com:8080

### 场景一、开发环境登录oauth2项目
1. 本地开发时添加hosts文件配置
```hosts
# 把测试域名拦截到本地
127.0.0.1 		xxx-inter.yto56test.com
```
2. 访问`项目url`，会按照`场景二`逻辑建立登录

### 场景二、访问项目url登录
1. 浏览器地址栏输入`项目url`： http://xxx-inter.yto56test.com:8080/pathname?search#hash (pathname search hash 可选)
2. 如果项目处于`未登录状态`，登录包会请求接口`baseUrl/oauth2/authorization/ulp_oauth2Test?continue_url=`**{当前项目的url}**，服务端收到请求后，会将页面重定向到`统一登录扫码页面`
3. 扫码登录后，将打开 continue_url 并携带新的登录参数`code`、`state`, 用于向自己的后台请求登录（返回 token）

### 场景三、通过信息导航系统，跳转免登
1. 扫码登录[`信息系统导航`](https://yto-sso-apisix.yto56test.com/sso-index/login), 从[`信息系统导航`](https://yto-sso-apisix.yto56test.com/sso-index/login)导航跳转`当前项目地址`建立登录

### 场景四、通过统一登录扫码登录
1. 浏览器地址栏输入[`统一登录地址`](https://yto-sso-apisix.yto56test.com/sso-index/login?clientId=oauth2Test)
2. 页面会重定向到项目地址，并按照`场景二`逻辑建立登录