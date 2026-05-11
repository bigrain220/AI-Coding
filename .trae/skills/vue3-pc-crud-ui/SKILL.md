---
name: vue3-pc-crud-ui
description: 生成 Vue3 CRUD 页面（列表/新增弹框/编辑弹框）及 @yto/ui-next 组件库规范使用。当用户要求生成/创建 Vue3 页面、CRUD 功能、列表页、弹框、表单时使用。强制 script setup 语法、禁用 yt-select，所有 @yto/ui-next 组件 API 必须查文档不得臆造。
---

# Vue3 CRUD + @yto/ui-next 使用规范

## 绝对规则（违反即错误）

- **必须**使用 `<script setup>` 语法，**禁止** Options API
- 单文件组件顺序：`<template>` → `<script setup>` → `<style lang="scss" scoped>`
- **禁止**使用 `yt-select`，单选下拉用 `yt-items-list`，多选用 `yt-items-list-checkbox`
- **禁止臆造**任何组件的 props / events / slots / 实例方法
- 必须使用 `http` 从 `@/http/index.js` 导入
- 必须使用 `message` 从 `@yto/ui-next/es/message` 导入
- 必须使用 `confirm` 从 `@yto/ui-next/es/alert` 导入
- 组件标签统一 `yt-*`（kebab-case 小写中划线）
- `src/page/` 最后一级文件夹有且只有一个 `index.vue`；弹框等子组件放 `src/page-view/` 对应层级下

---

## 使用 @yto/ui-next 组件的流程

使用任意组件前必须：

1. 访问在线文档：`http://cloud-inter.yto56test.com/yto_ui_next/<组件目录>/index.md`
2. 阅读 `### 属性` `### slot 插槽` `### $emit 事件` `### 实例方法` 四个表格
3. items 字段名（`name`/`id` 等）以文档示例为准
4. 无法访问在线文档时，读 `node_modules` 包源码获取正确用法

**常见搜索规则**：标签 `yt-tree-flat-selector` → 目录 `tree-flat-selector`

---

## 搜索区组件速查

| 场景 | 组件 |
|------|------|
| 文本输入 | `<yt-input>` |
| 数字输入 | `<yt-input-number>` |
| 单选下拉 | `<yt-items-list>` |
| 多选下拉 | `<yt-items-list-checkbox>` |
| 树形选择 | `<yt-tree-flat-selector>` |
| 日期 | `<yt-date type="date">` |
| 日期范围 | `<yt-date-range>` |
| 日期时间范围（占两列） | `<yt-date-range type="datetime" extend>` |

## 表单组件速查

| 场景 | 组件 |
|------|------|
| 文本输入 | `<yt-input>` |
| 单选下拉 | `<yt-items-list>` |
| 多选下拉 | `<yt-items-list-checkbox>` |
| 单选框组 | `<yt-radios :items="" label-key="name" value-key="id">` |
| 开关 | `<yt-form-label>` 包裹 `<yt-switch>` |
| 多行文本 | `<yt-textarea>` |

---

## CRUD 页面规范要点

### 列表页（crud-index）

- 搜索区：`<yt-fixed-top>` → `<yt-search>` + `<yt-search-item>`
- 内容区：`<yt-height-to-body-bottom class="app-container-block">`
- 表格：`<yt-table>`；分页：`<yt-pagination>` 用 `<yt-fixed-bottom>` 包裹吸底
- 弹框子组件通过 `ref` 调用 `start()` 打开，需在子组件中 `defineExpose({ start })`
- `form` 用 `reactive()` 定义；`ths` / `items` / `loading` / `total` / `selectedIds` 用 `ref()`
- `dicData` 用 `reactive({})` 定义字典数据
- `btnQuery()` → 调用 `paginationRef.value.query()` 重置第一页
- `query(page, size)` 执行请求，完成后清空 `selectedIds.value = []`
- `onMounted` 调用 `btnQuery()`
- `yt-table` 自带序号，不需要时设 `number-visible="false"`，勿手动加序号列
- 排序时不能改变分页状态

### 删除规范

```js
import { confirm } from '@yto/ui-next/es/alert'
import { message } from '@yto/ui-next/es/message'

confirm({
  message: '您确认要删除 ' + item.name + ' 吗？',
  asyncRequest: true,
  async: (data, cb) => {
    http.post('/api/delete', { id: item.id }).then(
      () => { cb(true); message({ status: 'success', message: '删除成功' }); query() },
      () => { cb(false) }
    )
  }
})
```

### 新增弹框（crud-add）

- `<yt-modal :close-disabled="loading" @closed="closed">` 
- `defaultForm` 定义初始值，`form` 用 `reactive({ ...defaultForm })` 初始化
- `closed()` 中用 `Object.assign(form, defaultForm)` 重置（不在 `close()` 里重置）
- 提交方法名：`submit`，成功后调用 `emit('success')`
- **必须** `defineExpose({ start })` 暴露给父组件
- 弹框 footer 用 `<template #footer>` 具名插槽

### 编辑弹框（crud-edit）

- `<yt-modal @open="open" @closed="closed">`
- `start(data)` 存入 `itemsData.value`；`open()` 中回填表单
- `closed()` 额外执行 `itemsData.value = {}`
- 提交方法名：`submit`
- `defaultForm` 中包含 `id` 字段
- **必须** `defineExpose({ start })` 暴露给父组件

---

## 详细模板与组件清单

- 完整 Vue3 代码模板（列表/新增/编辑）→ [crud-templates.md](crud-templates.md)
- 完整 @yto/ui-next 组件清单（含文档地址）→ [ui-components.md](ui-components.md)
