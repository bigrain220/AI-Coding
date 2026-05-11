# Vue3 CRUD 完整代码模板

## 一、列表页模板

```vue
<template>
  <div>
    <yt-fixed-top>
      <yt-search class="page-form-layout custom-style" :validator="validator" label-width="120px" @query="btnQuery" ref="formRef">
        <yt-search-item>
          <yt-input v-model="form.enterQuery" name="enterQuery" label="回车查询" placeholder="纯文本框 回车触发查询" />
        </yt-search-item>
        <yt-search-item>
          <yt-input-number v-model="form.counter" name="counter" label="计数器" :min="0" />
        </yt-search-item>
        <yt-search-item>
          <yt-items-list v-model="form.singleList" name="singleList" :items="dicData.singleList" label="单选列表" placeholder="请选择单选列表" />
        </yt-search-item>
        <yt-search-item>
          <yt-items-list-checkbox v-model="form.multiList" name="multiList" :items="dicData.multiList" label="多选列表" placeholder="请选择多选列表" />
        </yt-search-item>
        <yt-search-item>
          <yt-tree-flat-selector v-model="form.treeIds" name="treeIds" :items="dicData.treeList" label="树选择器" placeholder="请选择树节点" />
        </yt-search-item>
        <yt-search-item>
          <yt-date v-model="form.date" name="date" label="日期" type="date" placeholder="请选择日期" />
        </yt-search-item>
        <yt-search-item>
          <yt-date-range v-model="form.dateRange" name="dateRange" label="日期范围" placeholder="请选择日期范围" />
        </yt-search-item>
        <yt-search-item extend>
          <yt-date-range v-model="form.dateTimeRange" name="dateTimeRange" label="日期时分秒范围" type="datetime" placeholder="时分秒宽度占两列" />
        </yt-search-item>
        <yt-search-item>
          <yt-button text="查询" :disabled="!validator.valid" :loading="loading" @click="btnQuery" />
          <yt-button text="重置" :disabled="loading" theme="white" @click="reset" />
        </yt-search-item>
      </yt-search>
    </yt-fixed-top>
    <yt-height-to-body-bottom class="app-container-block" :max-prop="false" min-prop>
      <div class="c-fix p-b-16">
        <yt-button text="新增" @click="addItem" class="m-r-8 f-left" />
        <yt-button text="新增-新开页面" @click="addRoute" class="m-r-8 f-left" />
        <yt-button text="批量删除" theme="red" mode="light" class="m-r-8 f-left" :disabled="selectedIds.length === 0" @click="del_batch" />
        <yt-button text="禁用" theme="red" mode="outline" class="m-r-8 f-left" />
        <yt-button text="启用" theme="green" mode="outline" class="m-r-8 f-left" />
        <yt-button text="更多" theme="white" class="f-left">
          <template #suf>
            <yt-arrow size="5px" class="v-a-middle m-l-5" />
          </template>
          <template #hidden>
            <yt-popover placement="bottom" mode="light" self-click-close self-mouseleave-close>
              <ul style="width: 120px" class="p-y-5">
                <li><yt-button mode="text" theme="white" text="其它-1" class="w-100-p" /></li>
                <li class="b-t-g-50"><yt-button mode="text" theme="white" text="其它-2" class="w-100-p" /></li>
              </ul>
            </yt-popover>
          </template>
        </yt-button>
        <yt-button text="导出" theme="white" class="m-l-8 f-left f-right-lg" />
        <yt-button text="导入" theme="white" class="m-l-8 f-left f-right-lg">
          <template #suf>
            <yt-arrow size="5px" class="v-a-middle m-l-5" />
          </template>
          <template #hidden>
            <yt-popover placement="bottom" mode="light" self-click-close self-mouseleave-close>
              <ul style="width: 120px" class="p-y-5">
                <li><yt-button mode="text" theme="white" text="下载模板" class="w-100-p" /></li>
                <li class="b-t-g-50"><yt-button mode="text" theme="white" text="手动上传" class="w-100-p" /></li>
              </ul>
            </yt-popover>
          </template>
        </yt-button>
      </div>
      <div>
        <yt-table
          :ths="ths"
          :items="items"
          excluded-top-selector=".app-header-layout,.app-menu-tab-layout,.page-form-layout"
          ref="tableRef"
          v-model:checked-value="selectedIds"
          :input-visible="true"
          :multiple-choice="true"
          id-key="id"
        >
          <template #link="{ item }">
            <a href="#" class="t-ellipsis d-inline-block max-w-100-p c-theme a-unstyle">
              {{ item.link }}
              <yt-popover placement="top" auto-line-visible type="hover">
                <div class="p-x-20 p-y-10">{{ item.link }}</div>
              </yt-popover>
            </a>
          </template>
          <template #popover="{ item, keyName }">
            <div class="t-ellipsis">
              <span>{{ item[keyName] }}</span>
              <yt-popover placement="top" auto-line-visible type="hover">
                <div class="p-x-20 p-y-10">{{ item[keyName] }}</div>
              </yt-popover>
            </div>
          </template>
          <template #status="{ item, keyName }">
            <div>
              <yt-tag v-if="item[keyName] === 1" border mode="light" type="success" size="sm">是</yt-tag>
              <yt-tag v-else border mode="light" type="fail" size="sm">否</yt-tag>
            </div>
          </template>
          <template #single="{ item, keyName }">
            <div>{{ getDictionaryName(item[keyName], 'singleList', dicData) }}</div>
          </template>
          <template #ctrl="{ item }">
            <div>
              <yt-button mode="text" size="sm" text="编辑" @click="editItem(item)" />
              <yt-button mode="text" size="sm" theme="red" text="删除" @click="deleteItem(item)" />
            </div>
          </template>
        </yt-table>
        <yt-fixed-bottom class="m-t-16" :watch-val="items" ref="fixedBottomRef">
          <yt-pagination @query="query" :total="total" :disabled="loading" ref="paginationRef" />
        </yt-fixed-bottom>
      </div>
    </yt-height-to-body-bottom>
    <add-modal ref="addModalRef" :dic-data="dicData" @success="btnQuery" />
    <edit-modal ref="editModalRef" :dic-data="dicData" @success="query" />
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import { http } from '@/http/index.js'
import { toRoute } from '@/util/router'
import { getDictionaryName } from '@/util/index'
import { message } from '@yto/ui-next/es/message'
import { confirm } from '@yto/ui-next/es/alert'
import addModal from '@/page-view/demo/add.vue'
import editModal from '@/page-view/demo/edit.vue'

const formRef = ref(null)
const paginationRef = ref(null)
const addModalRef = ref(null)
const editModalRef = ref(null)
const tableRef = ref(null)

const validator = ref({ valid: false, status: {} })

const form = reactive({
  enterQuery: '',
  counter: null,
  singleList: '',
  multiList: [],
  treeIds: [],
  date: '',
  dateRange: ['', ''],
  dateTimeRange: ['', '']
})

const ths = ref([
  { name: '文本点击', key: 'link', width: '260px', fixed: 'left' },
  { name: '文本单行显示效果', key: 'popover', width: '220px' },
  { name: '状态', key: 'status', width: '100px' },
  { name: '对接人', key: 'manager' },
  { name: '带翻译', key: 'single' },
  { name: '创建时间', key: 'createTime', width: '140px' },
  { name: '操作', key: 'ctrl', width: '120px', fixed: 'right' }
])

const selectedIds = ref([])
const items = ref([])
const loading = ref(false)
const total = ref(0)

const dicData = reactive({
  singleList: [
    { id: '1', name: '选项一' },
    { id: '2', name: '选项二' },
    { id: '3', name: '选项三' }
  ],
  multiList: [
    { id: '1', name: '多选项一' },
    { id: '2', name: '多选项二' },
    { id: '3', name: '多选项三' },
    { id: '4', name: '多选项四' }
  ],
  treeList: [
    {
      id: '0',
      name: '一级菜单0',
      children: [
        {
          id: '00',
          name: '二级菜单00',
          children: [
            { id: '000', name: '三级菜单000' },
            { id: '001', name: '三级菜单001' }
          ]
        },
        { id: '01', name: '二级菜单01' }
      ]
    },
    {
      id: '1',
      name: '一级菜单1',
      children: [
        { id: '10', name: '二级菜单10' },
        { id: '11', name: '二级菜单11' }
      ]
    }
  ]
})

const reset = () => { formRef.value.reset() }

const btnQuery = () => { paginationRef.value.query() }

const query = (page, size) => {
  const currentPage = page || paginationRef.value.page
  const pageSize = size || paginationRef.value.pageSize
  const payload = Object.assign({ currentPage, pageSize }, form)
  loading.value = true
  http
    .post('/admin/pageList', payload)
    .then(({ list = [], total: t = 0 }) => {
      items.value = list
      total.value = t
      selectedIds.value = []
    })
    .finally(() => { loading.value = false })
}

const addItem = () => { addModalRef.value.start() }
const editItem = (item) => { editModalRef.value.start(item) }

const deleteItem = (item) => {
  confirm({
    message: '您确认要删除 ' + item.link + ' 吗？',
    asyncRequest: true,
    async: (data, cb) => {
      http.post('/api/delete', { id: item.id }).then(
        () => { cb(true); message({ status: 'success', message: '删除成功' }); query() },
        () => { cb(false) }
      )
    }
  })
}

const del_batch = () => {
  if (selectedIds.value.length === 0) {
    message({ status: 'warning', message: '请选择要删除的数据' })
    return
  }
  confirm({
    message: '您确认要批量删除选中的 ' + selectedIds.value.length + ' 条数据吗？',
    asyncRequest: true,
    async: (data, cb) => {
      http.post('/api/batchDelete', { ids: selectedIds.value }).then(
        () => { cb(true); message({ status: 'success', message: '批量删除成功' }); selectedIds.value = []; query() },
        () => { cb(false) }
      )
    }
  })
}

const addRoute = () => {
  toRoute({ name: 'commonAddIndex', query: { tabName: '新增新开页面' } })
}

onMounted(() => { btnQuery() })
</script>

<style lang="scss" scoped></style>
```

---

## 二、新增弹框模板

```vue
<template>
  <yt-modal title="新增" :close-disabled="loading" @closed="closed" ref="modalRef">
    <div class="p-y-22">
      <yt-form ref="formRef" :validator="validator" label-width="120px" class="row">
        <div class="col p-y-10">
          <yt-input v-model="form.roleName" name="roleName" label="名称" placeholder="请输入名称" required />
        </div>
        <div class="col p-y-10">
          <yt-items-list v-model="form.list_radio" name="list_radio" :items="dicData.singleList" label="单选列表" placeholder="请选择单选列表" required />
        </div>
        <div class="col p-y-10">
          <yt-items-list-checkbox v-model="form.list_multi" name="list_multi" :items="dicData.multiList" label="下拉多选" placeholder="请选择多选项" />
        </div>
        <div class="col p-y-10">
          <yt-radios v-model="form.radio" :items="dicData.singleList" label="单选框集合" label-key="name" value-key="id" required />
        </div>
        <div class="col p-y-10">
          <yt-form-label label="开关" required>
            <yt-switch v-model="form.switch" name="switch" />
          </yt-form-label>
        </div>
        <div class="col p-y-10">
          <yt-textarea v-model="form.description" name="description" label="描述" placeholder="请输入描述" />
        </div>
      </yt-form>
    </div>
    <template #footer>
      <div>
        <yt-button class="m-r-8" theme="white" text="取消" :disabled="loading" @click="close" />
        <yt-button :disabled="!validator.valid" :loading="loading" text="确定" @click="submit" />
      </div>
    </template>
  </yt-modal>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { http } from '@/http/index.js'
import { message } from '@yto/ui-next/es/message'

const props = defineProps({
  dicData: { type: Object, default: () => ({}) }
})

const emit = defineEmits(['success'])

const modalRef = ref(null)
const formRef = ref(null)
const loading = ref(false)
const validator = ref({ valid: false, status: {} })

const defaultForm = {
  roleName: '',
  list_radio: '',
  list_multi: [],
  radio: '',
  switch: true,
  description: ''
}

const form = reactive({ ...defaultForm })

const start = () => { modalRef.value.open() }

const closed = () => {
  formRef.value.reset()
  Object.assign(form, defaultForm)
}

const close = () => { modalRef.value.close() }

const submit = () => {
  const payload = Object.assign({}, form)
  loading.value = true
  http
    .post('/api/add', payload)
    .then(() => {
      message({ status: 'success', message: '新增成功' })
      emit('success')
      close()
    })
    .finally(() => { loading.value = false })
}

defineExpose({ start })
</script>

<style lang="scss" scoped></style>
```

---

## 三、编辑弹框模板

与新增弹框的差异：
- `@open="open"` 在弹框打开后回填数据
- `start(data)` 存入 `itemsData.value`；`open()` 中遍历 `form` 字段回填
- `closed()` 额外执行 `itemsData.value = {}`
- 提交方法名为 `submit`
- `defaultForm` 包含 `id` 字段

```vue
<template>
  <yt-modal title="编辑" :close-disabled="loading" @open="open" @closed="closed" ref="modalRef">
    <div class="p-y-22">
      <yt-form ref="formRef" :validator="validator" label-width="120px" class="row">
        <div class="col p-y-10">
          <yt-input v-model="form.roleName" name="roleName" label="名称" placeholder="请输入名称" required />
        </div>
        <div class="col p-y-10">
          <yt-items-list v-model="form.list_radio" name="list_radio" :items="dicData.singleList" label="单选列表" placeholder="请选择单选列表" required />
        </div>
        <div class="col p-y-10">
          <yt-items-list-checkbox v-model="form.list_multi" name="list_multi" :items="dicData.multiList" label="下拉多选" placeholder="请选择多选项" />
        </div>
        <div class="col p-y-10">
          <yt-radios v-model="form.radio" :items="dicData.singleList" label="单选框集合" label-key="name" value-key="id" required />
        </div>
        <div class="col p-y-10">
          <yt-form-label label="开关" required>
            <yt-switch v-model="form.switch" name="switch" />
          </yt-form-label>
        </div>
        <div class="col p-y-10">
          <yt-textarea v-model="form.description" name="description" label="描述" placeholder="请输入描述" />
        </div>
      </yt-form>
    </div>
    <template #footer>
      <div>
        <yt-button class="m-r-8" text="取消" :disabled="loading" theme="white" @click="close" />
        <yt-button text="保存" :loading="loading" @click="submit" :disabled="!validator.valid" />
      </div>
    </template>
  </yt-modal>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { http } from '@/http/index.js'
import { message } from '@yto/ui-next/es/message'

const props = defineProps({
  dicData: { type: Object, default: () => ({}) }
})

const emit = defineEmits(['success'])

const modalRef = ref(null)
const formRef = ref(null)
const loading = ref(false)
const validator = ref({ valid: false, status: {} })

const defaultForm = {
  id: '',
  roleName: '',
  list_radio: '',
  list_multi: [],
  radio: '',
  switch: true,
  description: ''
}

const form = reactive({ ...defaultForm })
const itemsData = ref({})

const start = (data = {}) => {
  itemsData.value = data
  modalRef.value.open()
}

const open = () => {
  Object.keys(form).forEach((key) => {
    if (itemsData.value[key] !== undefined) {
      form[key] = itemsData.value[key]
    }
  })
}

const closed = () => {
  formRef.value.reset()
  Object.assign(form, defaultForm)
  itemsData.value = {}
}

const close = () => { modalRef.value.close() }

const submit = () => {
  const payload = Object.assign({}, form)
  loading.value = true
  http
    .post('/api/update', payload)
    .then(() => {
      message({ status: 'success', message: '保存成功' })
      emit('success')
      close()
    })
    .finally(() => { loading.value = false })
}

defineExpose({ start })
</script>

<style lang="scss" scoped></style>
```
