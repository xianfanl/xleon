<template>
  <a-modal
    title="操作"
    style="top: 20px;"
    :width="800"
    v-model="visible"
    @ok="handleSubmit"
  >
    <a-form :form="form">
      <a-form-item style="display:none">
        <a-input v-decorator="['menuId']"/>
      </a-form-item>
      <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="上级权限"
      >
        <a-tree-select
          v-decorator="['parentId', {rules: [{ required: true, message: '请选择上级权限' }]}]"
          :dropdownStyle="{ maxHeight: '400px', overflow: 'auto' }"
          :treeData="permissions"
          placeholder="上级权限"
          treeDefaultExpandAll
        >
        </a-tree-select>
      </a-form-item>

      <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="菜单类型"
      >
        <a-select v-decorator="['menuType', {initialValue:'M',rules: [{ required: true, message: '请选择类型' }]}]" @select="menuTypeChange">
          <a-select-option :value="'M'">目录</a-select-option>
          <a-select-option :value="'C'">菜单</a-select-option>
          <a-select-option :value="'F'">按钮</a-select-option>
        </a-select>
      </a-form-item>

      <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="权限名称"
      >
        <a-input
          v-decorator="['menuName',{rules: [{ required: true, message: '请输入权限名称' }]}]"
          placeholder="起一个名字"/>
      </a-form-item>

      <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="动态菜单唯一键"
      >
        <a-input
          v-decorator="['menuKey',{initialValue:'',rules: [{ required: true, message: '请输入动态菜单唯一键' }]}]"
          placeholder="动态菜单唯一键"/>
      </a-form-item>

      <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        v-if="menuType!='M'"
        label="权限标识"
      >
        <a-input
          v-decorator="['perms',{rules: [{ required: true, message: '请输入权限标识' }]}]"
          placeholder="权限标识"/>
      </a-form-item>

      <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        v-if="menuType==='M'"
        label="布局类型"
      >
        <a-select v-decorator="['menuLay', {initialValue:'PageView',rules: [{ required: true,message: '请选择类型' }]}]">
          <a-select-option :value="'PageView'">基础布局，包含了面包屑，和中间内容区 (slot)</a-select-option>
          <a-select-option :value="'RouterView'">空布局，专门为了二级菜单内容区自定义</a-select-option>
          <a-select-option :value="'BlankLayout'">空白的布局</a-select-option>
          <a-select-option :value="'BasicLayout'">基础页面布局，包含了头部导航，侧边栏和通知栏</a-select-option>
          <a-select-option :value="'UserLayout'">登陆注册页面的通用布局</a-select-option>
        </a-select>
      </a-form-item>

      <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        v-if="menuType==='M'"
        label="图标"
      >
        <a-input v-decorator="['icon',{rules: [{ required: true, message: '请选择图标' }]}]" ref="iconInput" @click="iconselect()" enterButton="选择图标" placeholder="选择图标">
          <a-icon slot="prefix" :type="icon" />
          <a-icon slot="suffix" type="close-circle" @click="emitEmpty"/>
        </a-input>
      </a-form-item>

      <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="显示顺序"
      >
        <a-input-number
          v-decorator="['orderNum',{initialValue:'1',rules: [{ required: true, message: '请输入顺序数字' }]}]"
          placeholder="显示顺序"/>
      </a-form-item>

      <a-form-item
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        label="状态"
      >
        <a-select v-decorator="['visible', {initialValue:'0',rules: [{ required: true, message: '请选择状态' }]}]">
          <a-select-option :value="'0'">显示</a-select-option>
          <a-select-option :value="'1'">隐藏</a-select-option>
        </a-select>
      </a-form-item>

    </a-form>
    <iconSelector-modal ref="modal" @ok="setIcon" :icon="icon"/>
  </a-modal>
</template>
<script>
import { getPermissions, savePerm } from '@/api/system'
import pick from 'lodash.pick'
import IconSelectorModal from './IconSelectorModal.vue'
export default {
  name: 'UserModal',
  components: {
    IconSelectorModal
  },
  data () {
    return {
      description: '列表使用场景：后台管理中的权限管理以及角色管理，可用于基于 RBAC 设计的角色权限控制，颗粒度细到每一个操作类型。',
      visible: false,
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      permissions: [{ key: 0, value: '0', title: '无' }],
      mdl: {},
      icon: 'smile',
      menuType: '',
      form: this.$form.createForm(this)
    }
  },
  beforeCreate () {
  },
  created () {
    this.loadPermissions()
  },
  methods: {
    menuTypeChange (type) {
      this.menuType = type
    },
    emitEmpty () {
      this.$refs.iconInput.focus()
      this.form.setFieldsValue({ 'icon': '' })
    },
    iconselect () {
      this.$refs.modal.show()
    },
    setIcon (icon) {
      this.icon = icon
      this.form.setFieldsValue({ 'icon': icon })
    },
    add (parentId) {
      this.form.resetFields()
      this.edit({ parentId: parentId || '0' })
    },
    edit (record) {
      this.mdl = Object.assign({}, record)
      this.visible = true
      this.menuType = this.mdl.menuType || 'M'
      this.$nextTick(() => {
        this.mdl.icon ? this.icon = this.mdl.icon : this.icon = 'smile'
        this.mdl.parentId += ''
        this.form.setFieldsValue(pick(this.mdl, 'icon', 'menuId', 'parentId', 'menuType', 'visible', 'perms', 'orderNum', 'menuName', 'menuKey', 'menuLay'))
        // this.form.setFieldsValue({ ...record })
      })
    },
    loadPermissions () {
      getPermissions().then(res => {
        this.buildtree(res.rows, this.permissions, 0)
      })
      console.log(this.permissions)
    },
    buildtree (list, arr, parentId) {
      list.forEach(item => {
        if (item.parentId === parentId) {
          var child = {
            key: item.menuId,
            value: item.menuId + '',
            title: item.menuName,
            children: []
          }
          this.buildtree(list, child.children, item.menuId)
          arr.push(child)
        }
      })
    },
    handleSubmit (e) {
      e.preventDefault()
      this.form.validateFields((err, values) => {
        if (!err) {
          console.log('Received values of form: ', values)
          this.confirmLoading = true
          savePerm(values).then(res => {
            if (res.code === 0) {
              this.$message.success('保存成功')
              this.$emit('ok')
              this.visible = false
            } else {
              this.$message.success(res.msg)
            }
          }).catch(() => {
            this.$message.error('系统错误，请稍后再试')
          }).finally(() => {
            this.confirmLoading = false
          })
        }
      })
    }
  },
  watch: {
    /*
      'selectedRows': function (selectedRows) {
        this.needTotalList = this.needTotalList.map(item => {
          return {
            ...item,
            total: selectedRows.reduce( (sum, val) => {
              return sum + val[item.dataIndex]
            }, 0)
          }
        })
      }
      */
  }
}
</script>
