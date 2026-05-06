---
name: sso-setup
description: |
  为项目接入圆通统一登录（SSO / oauth2）。
  当用户说"接入统一登录"、"配置登录"、"接入 SSO"、"配置 oauth2"、"配置 @yto/ulp-sdk-general-js" 时使用此技能。
  执行前先读取 references/sso-doc.md 获取完整的参数和 API 说明。
---

# 统一登录接入技能（sso-setup）

## 参考文档

完整参数、API 和示例代码见 `references/sso-doc.md`，执行任何步骤前必须先阅读该文件。

---

# 第一步：收集信息

## 1.1 判断用户是否已提供完整信息

首先检查用户的消息中是否已明确提供以下 **全部 4 项**信息：

1. **clientId**
2. **测试环境地址**（格式：`域名:端口`）
3. **接口前缀（axios baseURL）**
4. **后台服务地址（proxy target）**

**决策规则：**

- ✅ **用户已提供全部 4 项** → 直接记录，**跳过确认步骤，立即进入第二步执行**。
- ⚠️ **用户缺少任意 1 项或多项** → 必须执行以下 1.2 的扫描和确认流程，补全缺失项后再继续。

---

## 1.2 缺少信息时：扫描 + 确认（仅在缺失时执行）

对缺失的项，按以下方式扫描项目文件获取预填值，再通过 `ask_user_question` 让用户确认：

1. **clientId**：无法自动获取，必须询问用户。
2. **测试环境地址**：检查 `vue.config.js` / `vite.config.js` / `webpack.config.js` 中的 `devServer.host` 和 `port` 拼合；若 host 为 `0.0.0.0` 或 `localhost` 等非真实域名则视为未配置。
3. **接口前缀（axios baseURL）**：检查 http 封装文件中 `axios.create()` 的 `baseURL`。
4. **后台服务地址（proxy target）**：检查构建配置文件 `proxy` 配置中的 target；target 为空或不存在则视为未配置。

**使用 `ask_user_question` 工具，将所有缺失项一次性发起询问**，已自动扫描到的值作为第一选项（标注"已自动获取，请确认"），同时提供"自定义答案"选项。

**收到用户回答后，将全部 4 项信息记录，后续步骤直接使用，不再重复询问。**

---

## 第二步：检查并升级 axios

读取 `package.json`，检查 axios 版本：

- 如果版本低于 `1.1.2`，告知用户需要执行以下命令升级，**不要自动执行**：
  ```bash
  npm uninstall axios
  npm install -S axios@^1.7.5
  ```
- 同时告知用户需要手动执行安装登录包的命令：
  ```bash
  npm config set registry https://npm.yto.net.cn
  npm install -S @yto/ulp-sdk-general-js
  ```
- 如果版本已满足要求，只需安装登录包，无需重装 axios

---

## 第三步：检查 http 实例文件

找到项目的 axios 实例文件，检查是否已导出 `axiosProxy`（即 `axios.create()` 创建的实例）：

- 如果已有 axios 实例但未具名导出，添加具名导出：
  ```js
  export { axiosProxy }
  ```
- 如果项目没有 axios 封装文件，根据 `references/sso-doc.md` 中"第二步"的示例，在 `src/http/index.js` 创建完整的 http 封装文件，使用第一步收集到的 `baseURL` 和 `proxy target` 填入对应位置
- 注意：登录包只需要 `axiosProxy`（axios 实例本身），不需要封装后的 `http` 对象

---

## 第四步：检查并更新开发服务器配置

使用第一步收集到的测试环境地址（域名:端口），拆分出 host 和 port 后检查构建配置文件：

- 如果 `host` 和 `port` 与用户确认的值一致，跳过此步
- 如果不一致，更新对应配置文件

**Vue CLI 项目（vue.config.js）示例：**

```javascript
module.exports = {
  devServer: {
    host: 'your-project.yto56test.com', // 替换为第零步确认的测试环境地址中的域名部分
    port: 80, // 替换为第零步确认的测试环境地址中的端口部分
    proxy: {
      '/api': {
        target: 'http://10.x.x.x:8080', // 替换为第零步确认的后台服务地址
        // pathRewrite: { '^/api': '' }
      },
    },
  },
}
```

**Vite 项目（vite.config.js）示例：**

```javascript
export default defineConfig({
  server: {
    host: 'your-project.yto56test.com', // 替换为第零步确认的测试环境地址中的域名部分
    port: 80, // 替换为第零步确认的测试环境地址中的端口部分
    proxy: {
      '/api': {
        target: 'http://10.x.x.x:8080', // 替换为第零步确认的后台服务地址
        // rewrite: (path) => path.replace(/^\/api/, '')
      },
    },
  },
})
```

> ⚠️ 修改配置后需要重启开发服务器才能生效。

---

## 第五步：修改 main.js 接入登录

使用第一步收集到的 clientId，在 `main.js` 中找到 `new Vue(...)` 或 `createApp(...)` 的位置，用 `loginAction` 包裹，**不要再次询问用户**。

`env` 字段必须根据项目已有的环境变量动态判断，不能硬编码为固定字符串。检查项目中的环境变量写法（如 `process.env.NODE_ENV`、`process.env.VUE_APP_ENV`、`import.meta.env.MODE` 等），选择合适的方式动态赋值。

**Vue2 项目示例：**

```javascript
import { loginAction, loginInfo } from '@yto/ulp-sdk-general-js'
import { axiosProxy } from '@/http/index.js' // 路径依据实际调整

loginAction({
  clientId: 'YOUR_CLIENT_ID', // 替换为第零步收集到的 clientId
  env: process.env.NODE_ENV === 'production' ? 'prod' : 'uat', // 依据项目实际环境变量调整
  http: axiosProxy,
}).then(
  function () {
    const ps = [] // 可在此添加初始化接口请求（菜单、权限、字典等）
    Promise.all(ps).then(function () {
      new Vue({
        el: '#app',
        router,
        store,
        provide: { userInfo: loginInfo.userInfo },
        render: (h) => h(App),
      })
    })
  },
  function () {},
)
```

**Vue3 项目示例：**

```javascript
import { loginAction, loginInfo } from '@yto/ulp-sdk-general-js'
import { axiosProxy } from '@/http/index.js' // 路径依据实际调整

loginAction({
  clientId: 'YOUR_CLIENT_ID', // 替换为第零步收集到的 clientId
  env: import.meta.env.MODE === 'production' ? 'prod' : 'uat', // 依据项目实际环境变量调整
  http: axiosProxy,
}).then(
  function () {
    const ps = []
    Promise.all(ps).then(function () {
      const app = createApp(App)
      app.use(router)
      app.use(store)
      app.provide('userInfo', loginInfo.userInfo)
      app.mount('#app')
    })
  },
  function () {},
)
```

> ⚠️ 原有的 `new Vue(...)` 或 `app.mount(...)` 必须移入 `loginAction().then()` 内部，确保登录验证通过后才挂载应用。

---

## 第六步：输出接入完成提示

接入完成后，输出以下提示：

```
✅ 统一登录接入完成，请注意以下事项：

1. clientId：已写入 main.js，如需修改联系张强（01714308）

2. 本地开发 hosts 配置（每台电脑都需要单独配置）：
   将测试环境地址中的域名部分映射到本地，例如：
   127.0.0.1 your-project.yto56test.com

3. 重启开发服务器后生效

4. 如需配置免登页面，在 loginAction 中添加：
   excludePage: ['/', '/login', '/other-public-page']

5. 调试阶段如需禁止 401 自动跳转扫码页，在 loginAction 中添加：
   stopLink: true
```

---

## 注意事项

- 不要改变 http 封装文件的整体结构，只确保 `axiosProxy` 被正确导出
- `loginAction` 必须在应用挂载之前调用，且应用挂载必须在 `.then()` 内部
- `env` 必须动态读取环境变量，不能硬编码
- 如果项目使用了多个 axios 实例请求不同后端，使用 `httpOthers` 参数注册额外实例，详见 `references/sso-doc.md`
- 微应用（乾坤）场景需要额外配置 `parentClientId`，详见 `references/sso-doc.md`
