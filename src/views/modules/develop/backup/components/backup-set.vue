<!--
 * @Description:
 * @Author: gumingchen
 * @Email: 1240235512@qq.com
 * @Date: 2021-02-03 15:48:37
 * @LastEditors: gumingchen
 * @LastEditTime: 2021-05-20 15:59:35
-->
<template>
  <el-dialog
    width="600px"
    title="备份配置"
    v-model="visible"
    :close-on-click-modal="false"
    @closed="dialogClosedHandle">
    <el-form
      :model="form"
      :rules="rules"
      v-loading="loading"
      ref="refForm"
      label-position="top">
      <el-form-item label="类型" prop="id">
        <el-radio-group v-model="form.id" @change="changeHandle">
          <el-radio v-for="item in types" :key="item.id" :label="item.id">{{ item.name }}</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="域名" prop="domain">
        <el-input v-model="form.domain" placeholder="域名" />
      </el-form-item>
      <el-form-item label="前缀" prop="prefix">
        <el-input v-model="form.prefix" placeholder="前缀" />
      </el-form-item>
      <el-form-item label="存储目录" prop="path">
        <el-input v-model="form.path" placeholder="存储目录" />
      </el-form-item>
      <el-form-item label="安装目录" prop="location">
        <el-input v-model="form.location" placeholder="安装目录" />
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="visible = false">取消</el-button>
        <el-button v-repeat type="primary" @click="submit()">确认</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script>
import { defineComponent, nextTick, reactive, ref, toRefs } from 'vue'

import { ElMessage } from 'element-plus'

import { isURL } from '@/utils/regular'

import { editValueApi, getInfoApi } from '@/api/develop/config'

export default defineComponent({
  emits: ['refresh'],
  setup(_props, { emit }) {
    const refForm = ref()
    const data = reactive({
      visible: false,
      loading: false,
      key: 'BACKUP_STORAGE',
      types: [],
      form: {
        id: '',
        domain: '',
        prefix: '',
        path: '',
        location: ''
      },
      currentJsonValue: {} // 当前选中的json value
    })

    const rules = reactive(function() {
      const checkDomain = (_rule, value, callback) => {
        if (value === '' || !isURL(value)) {
          callback(new Error('请输入正确的域名'))
        } else {
          callback()
        }
      }
      return {
        // id: [{ required: true, message: '请选择类型', trigger: 'change' }],
        domain: [{ required: true, validator: checkDomain, trigger: 'blur' }],
        path: [{ required: true, message: '请输入存储目录', trigger: 'blur' }]
      }
    }())

    /**
     * @description: 类型切换 表单赋值
     * @param {*}
     * @return {*}
     * @author: gumingchen
     */
    const changeHandle = (id) => {
      const current = data.types.filter(item => item.id === id)[0]
      data.currentJsonValue = JSON.parse(current.json_value)

      data.form.id = current.id
      for (const key in data.currentJsonValue) {
        data.form[key] = data.currentJsonValue[key]
      }
    }

    /**
     * @description: 初始化
     * @param {*}
     * @return {*}
     * @author: gumingchen
     */
    const init = () => {
      data.visible = true
      data.loading = true
      getInfoApi(data.key).then(r => {
        if (r) {
          data.types = r.data
          const current = data.types.filter(item => item.status === 1)[0]
          changeHandle(current.id)
        }
        nextTick(() => {
          data.loading = false
        })
      })
    }

    /**
     * @description: 表单验证提交
     * @param {*}
     * @return {*}
     * @author: gumingchen
     */
    const submit = () => {
      refForm.value.validate(valid => {
        if (valid) {
          const jsonObj = {}
          for (const key in data.currentJsonValue) {
            jsonObj[key] = data.form[key]
          }
          const jsonStr = JSON.stringify(jsonObj)
          const params = {
            id: data.form.id,
            json_key: data.key,
            json_value: jsonStr
          }
          editValueApi(params).then(r => {
            if (r) {
              data.visible = false
              ElMessage({
                message: '操作成功!',
                type: 'success'
              })
              emit('refresh')
            }
          })
        }
      })
    }

    /**
   * @description: 弹窗关闭动画结束时的回调
   * @param {*}
   * @return {*}
   * @author: gumingchen
   */
    const dialogClosedHandle = () => {
      refForm.value.resetFields()
    }

    return {
      refForm,
      ...toRefs(data),
      rules,
      init,
      changeHandle,
      submit,
      dialogClosedHandle
    }
  }
})
</script>
