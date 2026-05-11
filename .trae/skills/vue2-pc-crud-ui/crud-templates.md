# CRUD 完整代码模板

## 一、列表页模板

```vue
<template>
  <div>
    <yt-fixed-top>
      <yt-search class="page-form-layout custom-style" :validator="validator" label-width="120px" @query="btnQuery" ref="formRef">
        <yt-search-item>
          <yt-input v-model="form.username" name="username" label="名称" placeholder="请输入名称" />
        </yt-search-item>
        <yt-search-item>
          <yt-items-list v-model="form.list_radio" name="list_radio" :items="dicData.list_radio" label="单选列表" placeholder="请选择单选列表" />
        </yt-search-item>
        <yt-search-item>
          <yt-button text="查询" :disabled="!validator.valid" :loading="loading" @click="btnQuery" />
          <yt-button text="重置" theme="white" @click="reset" />
        </yt-search-item>
      </yt-search>
    </yt-fixed-top>
    <yt-height-to-body-bottom class="app-container-block" :max-prop="false" min-prop>
      <div class="c-fix p-b-16">
        <yt-button text="新增" @click="addItem" class="m-r-8 f-left" />
        <yt-button text="导出" theme="white" class="m-l-8 f-left f-right-lg" />
      </div>
      <div>
        <yt-table :ths="ths" :items="items" excluded-top-selector=".app-header-layout,.app-menu-tab-layout,.page-form-layout" ref="tableRef">
          <a slot="link" slot-scope="{ item }" href="#" class="t-ellipsis d-inline-block max-w-100-p c-theme a-unstyle">
            {{ item.link }}
            <yt-popover placement="top" auto-line-visible type="hover">
              <div class="p-x-20 p-y-10">{{ item.link }}</div>
            </yt-popover>
          </a>
          <div class="t-ellipsis" slot="popover" slot-scope="{ item, keyName }">
            <span>{{ item[keyName] }}</span>
            <yt-popover placement="top" auto-line-visible type="hover">
              <div class="p-x-20 p-y-10">{{ item[keyName] }}</div>
            </yt-popover>
          </div>
          <div slot="status" slot-scope="{ item, keyName }">
            <yt-tag v-if="item[keyName] === 1" border mode="light" type="success" size="sm">是</yt-tag>
            <yt-tag v-else border mode="light" type="fail" size="sm">否</yt-tag>
          </div>
          <div slot="ctrl" slot-scope="{ item }">
            <yt-button mode="text" size="sm" text="编辑" @click="editItem(item)" />
            <yt-button mode="text" size="sm" theme="red" text="删除" @click="deleteItem(item)" />
          </div>
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

<script>
import addModal from '@/page-view/demo/add.vue'
import editModal from '@/page-view/demo/edit.vue'

export default {
  components: { addModal, editModal },
  data() {
    return {
      validator: { valid: false, status: {} },
      form: {
        username: '',
        list_radio: ''
      },
      ths: [
        { name: '文本点击', key: 'link', width: '260px', fixed: 'left' },
        { name: '文本单行显示效果', key: 'popover', width: '220px' },
        { name: '状态', key: 'status', width: '100px' },
        { name: '对接人', key: 'manager' },
        { name: '创建时间', key: 'createTime', width: '140px' },
        { name: '操作', key: 'ctrl', width: '120px', fixed: 'right' }
      ],
      items: [],
      loading: false,
      total: 0,
      dicData: {
        list_radio: [
          { id: '1', name: '选项一' },
          { id: '2', name: '选项二' }
        ]
      }
    }
  },
  methods: {
    reset() {
      this.$refs.formRef.reset()
    },
    btnQuery() {
      this.$refs.paginationRef.query()
    },
    query(page, size) {
      const pageNum = page || this.$refs.paginationRef.page
      const pageSize = size || this.$refs.paginationRef.pageSize
      const payload = Object.assign({ pageNum, pageSize }, this.form)
      this.loading = true
      this.$http
        .post('/admin/pageList', payload)
        .then(({ list = [], total = 0 }) => {
          this.items = list
          this.total = total
        })
        .finally(() => { this.loading = false })
    },
    addItem() {
      this.$refs.addModalRef.start()
    },
    editItem(item) {
      this.$refs.editModalRef.start(item)
    },
    deleteItem(item) {
      this.$yt.confirm({
        message: '您确认要删除 ' + item.username + ' 吗？',
        asyncRequest: true,
        async: (data, cb) => {
          this.$http.post('/api/delete', { id: item.id }).then(
            () => {
              cb(true)
              this.$yt.message({ status: 'success', message: '删除成功' })
              this.query()
            },
            () => { cb(false) }
          )
        }
      })
    }
  },
  mounted() {
    this.btnQuery()
  }
}
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
          <yt-items-list v-model="form.list_radio" name="list_radio" :items="dicData.list_radio" label="单选列表" placeholder="请选择单选列表" required />
        </div>
        <div class="col p-y-10">
          <yt-radios v-model="form.radio" :items="dicData.list_radio" label="单选框集合" label-key="name" value-key="id" required />
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
    <div slot="footer">
      <yt-button class="m-r-8" theme="white" text="取消" :disabled="loading" @click="close" />
        <yt-button :disabled="!validator.valid" :loading="loading" text="确定" @click="submit" />
    </div>
  </yt-modal>
</template>

<script>
export default {
  props: {
    dicData: { type: Object, default: () => ({}) }
  },
  data() {
    return {
      loading: false,
      validator: { valid: false, status: {} },
      form: {
        roleName: '',
        list_radio: '',
        radio: '',
        switch: true,
        description: ''
      }
    }
  },
  methods: {
    start() {
      this.$refs.modalRef.open()
    },
    closed() {
      this.$refs.formRef.reset()
      this.form = this.$options.data().form
    },
    close() {
      this.$refs.modalRef.close()
    },
    submit() {
      const payload = Object.assign({}, this.form)
      this.loading = true
      this.$http
        .post('/api/add', payload)
        .then(() => {
          this.$yt.message({ status: 'success', message: '新增成功' })
          this.$emit('success')
          this.close()
        })
        .finally(() => { this.loading = false })
    }
  }
}
</script>

<style lang="scss" scoped></style>
```

---

## 三、编辑弹框模板

与新增弹框的差异：
- `@open="open"` 在弹框打开后回填数据
- `start(data)` 接收行数据存入 `itemsData`
- `open()` 中逐字段回填 form
- `closed()` 额外清空 `itemsData = {}`
- 提交方法名为 `submit`

```vue
<template>
  <yt-modal title="编辑" :close-disabled="loading" @open="open" @closed="closed" ref="modalRef">
    <div class="p-y-22">
      <yt-form ref="formRef" :validator="validator" label-width="120px" class="row">
        <div class="col p-y-10">
          <yt-input v-model="form.roleName" name="roleName" label="名称" placeholder="请输入名称" required />
        </div>
        <div class="col p-y-10">
          <yt-items-list v-model="form.list_radio" name="list_radio" :items="dicData.list_radio" label="单选列表" placeholder="请选择单选列表" required />
        </div>
        <div class="col p-y-10">
          <yt-radios v-model="form.radio" :items="dicData.list_radio" label="单选框集合" label-key="name" value-key="id" required />
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
    <div slot="footer">
      <yt-button class="m-r-8" text="取消" :disabled="loading" theme="white" @click="close" />
      <yt-button text="保存" :loading="loading" @click="submit" :disabled="!validator.valid" />
    </div>
  </yt-modal>
</template>

<script>
export default {
  props: {
    dicData: { type: Object, default: () => ({}) }
  },
  data() {
    return {
      loading: false,
      validator: { valid: false, status: {} },
      form: {
        id: '',
        roleName: '',
        list_radio: '',
        radio: '',
        switch: true,
        description: ''
      },
      itemsData: {}
    }
  },
  methods: {
    start(itemsData = {}) {
      this.itemsData = itemsData
      this.$refs.modalRef.open()
    },
    open() {
      Object.keys(this.form).forEach((key) => {
        this.form[key] = this.itemsData[key]
      })
    },
    closed() {
      this.$refs.formRef.reset()
      this.form = this.$options.data().form
      this.itemsData = {}
    },
    close() {
      this.$refs.modalRef.close()
    },
    submit() {
      const payload = Object.assign({}, this.form)
      this.loading = true
      this.$http
        .post('/api/update', payload)
        .then(() => {
          this.$yt.message({ status: 'success', message: '保存成功' })
          this.$emit('success')
          this.close()
        })
        .finally(() => { this.loading = false })
    }
  }
}
</script>

<style lang="scss" scoped></style>
```
