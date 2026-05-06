# @yto/ulp-sdk-general-js 统一登录文档

## 对接联系人

* 前端对接人：周冰冰 01698009
* 后端对接人：张强 01714308（申请 clientId、修改回调域名）

---

## 一、安装

> ⚠️ axios 版本必须升级到 1.1.2 及以上，0.18 版本在 error 时会包装数据结构导致登录包拿不到数据

```bash
# 1. 升级 axios
npm uninstall axios
npm install -S axios@^1.7.5

# 2. 切换到内部 npm 源并安装登录包
npm config set registry https://npm.yto.net.cn
npm install -S @yto/ulp-sdk-general-js
```

---

## 二、接入前准备

1. 联系张强（01714308）申请项目的 `clientId`
2. 提供测试环境域名（扫码回调地址），本地开发时需在 hosts 文件中添加域名拦截：
   

```
   127.0.0.1 smartoa-new-inter.yto56test.com
   ```

3. 提供生产环境域名（发布前联系张强修改）

### 本地开发代理配置（webpack）

```javascript
// webpack.config.js
devServer: {
    proxy: {
        '/api': {
            target: 'http://10.130.134.45:8282',
            pathRewrite: {
                '^/api': ''
            }
        }
    },
    host: 'yto-oauth2-inter.yto56test.com', // hosts 文件中配置的测试域名
    port: '8080',
}
```

---

## 三、接入步骤

### 第一步：创建 axios 实例（src/http/index.js 或 src/api/http.js）

```javascript
import axios from 'axios'

const http = {}
const axiosProxy = axios.create({
    baseURL: '/api',
    timeout: 1000 * 30,
    headers: {
        'Content-type': 'application/json',
        Authorization: ''
    }
})

function successHandler({
    type,
    config,
    result,
    resolve,
    reject,
    url,
    payload,
    default_value
}) {
    const response = result.data || {}
    // 依据项目的 code 码自行调整（示例为 1300）
    if (response.code === 1300) {
        let result_proxy = response.data || response.data === false ?
            response.data :
            default_value === undefined ? {} : default_value
        resolve(result_proxy)
    } else if (response.errMsg) {
        reject(response)
    } else {
        reject(response)
    }
}

http.post = function(url, payload, config = {}, default_value) {
    return new Promise(function(resolve, reject) {
        axiosProxy
            .post(url, payload, config)
            .then(
                function(result) {
                    successHandler({
                        type: 'post',
                        config,
                        result,
                        resolve,
                        reject,
                        url,
                        payload,
                        default_value
                    })
                },
                function(err) {
                    reject(err)
                }
            )
            .catch(function(err) {
                console.log('错误', err)
            })
    })
}

http.get = function(url, config = {}, default_value) {
    return new Promise(function(resolve, reject) {
        axiosProxy
            .get(url, config)
            .then(
                function(result) {
                    successHandler({
                        type: 'get',
                        config,
                        result,
                        resolve,
                        reject,
                        url,
                        payload: {},
                        default_value
                    })
                },
                function(err) {
                    reject(err)
                }
            )
            .catch(function(err) {
                console.log('错误', err)
            })
    })
}

export default {
    install: function(Vue) {
        Vue.prototype.$http = http
    }
}
export {
    http,
    axiosProxy
}
```

### 第二步：在 main.js 中调用 loginAction 初始化登录

```javascript
import {
    loginAction,
    loginInfo
} from '@yto/ulp-sdk-general-js'
import {
    axiosProxy
} from '@/api/http.js'

loginAction({
    clientId: 'XXX', // 联系张强（01714308）申请
    env: 'uat', // 生产环境传 'prod'，其他传 'uat'
    http: axiosProxy // 传入 axios 实例，登录包会自动注入拦截器
}).then(
    function() {
        // 登录成功后，在此发起依赖的初始化接口（菜单、权限、字典等）
        const ps = []
        Promise.all(ps).then(function() {
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
    function() {}
)
```

---

## 四、导出内容说明

```js
import {
    loginAction, // 登录初始化，返回 Promise
    loginInfo, // loginInfo.clientId / .access_token / .userInfo
    login, // 手动触发登录跳转
    logout, // 退出登录，可传 redirect_uri 指定退出后跳转地址
    buildOtherSysMenuUrl, // 构建 oauth2 项目授权链接，返回 Promise
    buildShareUrl, // 构建一次性免登分享地址，返回 Promise
    buildTGCSysMenuUrl // 构建 YTO-TGC 项目登录地址，返回 Promise
} from '@yto/ulp-sdk-general-js'
```

### loginInfo 属性说明

| 属性 | 类型 | 说明 |
|------|------|------|
| clientId | String | 当前系统的 clientId |
| access_token | String | 当前系统的 token |
| token | String | 使用 token+source 登录时的 token |
| source | String | 使用 token+source 登录时的 source |
| userInfo | Object | 用户信息对象 |
| source_client_id | String | 上游系统的 clientId，扫码登录时为空 |
| top_login_client_id | String | iframe 嵌套场景下的顶层登录 clientId |

### userInfo 对象返回格式参考

```js
{
    "authorities": [{
        "authority": "ROLE_USER"
    }],
    "attributes": {
        "belongOrgCode": "240907",
        "orgName": "辽宁省区",
        "belongOrgType": "REGION_MANAGE",
        "belongOrgName": "辽宁省区",
        "mobile": "15779315688",
        "userName": "张三",
        "userCode": "02466920",
        "userClientId": "ai-portal-web",
        "orgType": "REGION_MANAGE",
        "orgCode": "240907"
    },
    "userCode": "02466920",
    "userName": "张三",
    "orgCode": "240907",
    "orgType": "REGION_MANAGE",
    "orgName": "辽宁省区",
    "belongOrgCode": "240907",
    "belongOrgType": "REGION_MANAGE",
    "belongOrgName": "辽宁省区",
    "mobile": "15779315688",
    "userClientId": "ai-portal-web",
    "name": "02466920"
}
```

---

## 五、loginAction 参数完整说明

| 参数 | 类型 | 默认值 | 必须 | 说明 |
|------|------|--------|------|------|
| clientId | String | - | ✅ | 联系张强申请的 clientId |
| env | String | uat | ✅ | 生产传 `prod` ，其他传 `uat` |
| http | Object | - | ✅ | axios 实例，登录包自动在拦截器中注入 Authorization 和 client-id |
| stopLink | Boolean | false | - | 401 时不自动跳转扫码页，调试阶段使用 |
| loginBySourceAndToken | Boolean | false | - | 开启 url 中 source+token 参数登录 |
| interception401 | Function→Promise | - | - | 拦截 401 跳转，返回 `resolve({ source, token })` 重新初始化 |
| parentClientId | Array | [] | - | 微应用场景，子应用借助主应用 clientId 登录 |
| httpType | String | axios | - | 目前只支持 axios，其他请联系 01698009 |
| httpDataPlace | String | - | - | 自定义响应数据层级路径，多层用 `.` 连接 |
| httpErrorCode | Number/String | 401 | - | 触发跳转扫码页的响应状态码（严格类型比对） |
| httpErrorCodePlace | String | response.status | - | 状态码在响应中的位置 |
| httpOthers | Array | - | - | 多 axios 实例时，格式： `[{type:'axios', http: Instance}]` |
| excludePage | Array | ['/'] | - | 免登页面路径，如 `['/', '/index']` （nginx 有前缀时需带上） |
| heartbeatInterval | Number | 60000 | - | 心跳间隔 ms，最小 10000 |
| cookiePath | String | - | - | nginx 同端口不同路径部署多项目时填对应路径 |
| customLoginErrorHandler | Function | - | - | 自定义登录异常处理，用于跳转自定义登录页 |

---

## 六、导出方法说明

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| loginAction | Object | Promise | 初始化登录 |
| login | - | - | 手动触发登录跳转 |
| logout | redirect_uri?: String | - | 退出登录，默认跳回扫码页 |
| buildOtherSysMenuUrl | url: String, open_type?: Number | Promise→newUrl | 构建 oauth2 系统授权链接；open_type=1 为 iframe 内嵌，默认 2 |
| buildShareUrl | url: String | Promise→newUrl | 构建一次性免登地址，返回 `url?onceCode=xxx&user_code_sign=xxx` |
| buildTGCSysMenuUrl | url: String | Promise→newUrl | 构建 YTO-TGC 系统登录地址，返回 `url?YTO-TGC=xxx` |

---

## 七、其他场景

### 钉钉内嵌打开（安卓端首次登录无法返回问题）

```bash
npm install -S dingtalk-jsapi
```

```js
// main.js
import * as dd from 'dingtalk-jsapi'
window.dd = dd // 登录包内部会调用 window.dd.biz.navigation.close()
```

### Cookie 缓存键名说明

登录成功后会写入以下 cookie（clientIdName 为 clientId 的值）：

| Cookie 键名 | 说明 |
|-------------|------|
| `clientIdName_cookie_access_token` | access_token |
| `clientIdName_cookie_params_string` | 其他登录参数 |
| `clientIdName_cookie_user_code_md5` | 工号的 MD5 加密值 |

登录初始化成功后，页面会自动插入一个 iframe 元素用于心跳推送。

### 八、Vue3 对应的 vue-router 路由创建注意事项

1. 由于登录包在初始化登录后会去除登录参数，以及会通过 `history.replaceState` 重置浏览器的 url 到用户指定的地址（而不是只到项目首页）
2. `vue-router` 提供的 `createWebHistory` 方法不能在初始化登录之前执行，否则它会记录登录初始化之前的 url（包含登录参数以及 pathname 为首页的路径），导致在登录后初始化项目时，跳转的路由不是登录包（改变 `history.replaceState`）设置的路径
3. 需要在初始化登录之后执行 `createWebHistory` 方法，这样它记录的 url 才是用户想要跳转的页面路径

#### Vue3 创建路由参考示例

**main.js**

```js
import {
    createApp
} from 'vue';
import router from '@/router/index.js'

const app = createApp(App)
loginAction({
    clientId: 'XXX', // 联系张强（01714308）申请项目的clientId
    env: 'uat', // 生产环境传 'prod' 其他传 'uat'
    http: axiosProxy,
}).then(
    function() {
        app.use(router) // 在登录初始化成功后执行 注册路由
        app.mount('#app') // 初始化项目
    },
    function() {}
)
```

**router/index.js**

```js
import {
    createRouter,
    createWebHistory
} from 'vue-router'

// 路由列表
let router = {
    routes: [{
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
            redirect: {
                name: 'index'
            }
        }
    ]
}

export default {
    install: function(app, options) {
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
export {
    router
}
```
