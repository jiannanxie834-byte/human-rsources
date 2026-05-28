<template>
  <el-dialog :title="showTitle" @close="close" :visible="showDialog">
    <!-- 放置弹层内容 -->
    <el-form ref="addDept" :model="formData" :rules="rules" label-width="120px">
      <el-form-item prop="name" label="部门名称">
        <el-input
          v-model="formData.name"
          placeholder="2-10个字符"
          style="width: 80%"
          size="mini"
        ></el-input>
      </el-form-item>
      <el-form-item prop="code" label="部门编码">
        <el-input
          v-model="formData.code"
          placeholder="2-10个字符"
          style="width: 80%"
          size="mini"
        ></el-input>
      </el-form-item>
      <el-form-item prop="managerId" label="部门负责人">
        <el-select
          v-model="formData.managerId"
          placeholder="请选择负责人"
          style="width: 80%"
          size="mini"
        >
          <!-- 下拉选项，循环，value存储字段，label显示字段 -->
          <el-option
            v-for="item in managerList"
            :key="item.id"
            :label="item.username"
            :value="item.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item prop="introduce" label="部门介绍">
        <el-input
          v-model="formData.introduce"
          placeholder="1-100个字符"
          type="textarea"
          :rows="4"
          style="width: 80%"
          size="mini"
        ></el-input>
      </el-form-item>
      <el-form-item>
        <!-- 按钮部分 -->
        <el-row type="flex" justify="center">
          <el-col :span="12">
            <el-button type="primary" size="mini" @click="btnOK"
              >确定</el-button
            >
            <el-button size="mini" @click="close">取消</el-button>
          </el-col>
        </el-row>
      </el-form-item>
    </el-form>
  </el-dialog>
</template>

<script>
import { getDepartment, getManagerList, addDepartment, getDepartmentDetail, updateDepartment } from '@/api/department'

export default {
  props: {
    showDialog: {
      type: Boolean,
      default: false
    },
    currentNodeId: {
      type: Number,
      default: null 
    }
  },
  data() {
    return {
      formData:{
        code: '', // 部门编码
        introduce:'', // 部门介绍
        managerId: '', // 部门负责人Id
        name: '', // 部门名称
        pid: '' // 父级部门的Id
      },
      rules: {
        code: [{ required: true, message: '部门编码不能为空', trigger: 'blur'}, {
          min: 2, max: 10, message: '部门编码长度为2-10个字符', trigger: 'blur'
        },{
          trigger: 'blur',
          // 自定义校验
          validator: async(rule, value, callback) => {
            // value就是输入的编码
            let result = await getDepartment()
            // result数组中是否存在value
            // 判断是否编辑模式
            if(this.formData.id) {
              // 编辑场景
              result = result.filter(item => item.id != this.formData.id)
            }
            if(result.some(item => item.code === value)) {
              callback(new Error('部门中已经有该编码了'))
            } else {
              callback()
            }
          }
        }],
        introduce:[{ required: true, message: '部门介绍不能为空', trigger: 'blur'}, {
          min: 1, max: 100, message: '部门编码长度为1-100个字符', trigger: 'blur'
        }], 
        managerId: [{ required: true, message: '部门负责人不能为空', trigger: 'blur'}], 
        name: [{ required: true, message: '部门名称不能为空', trigger: 'blur'}, {
          min: 2, max: 10, message: '部门名称长度为2-10个字符', trigger: 'blur'
        },{
          trigger: 'blur',
          // 自定义校验
          validator: async(rule, value, callback) => {
            // value就是输入的编码
            let result = await getDepartment()
            // result数组中是否存在value
            if(this.formData.id) {
              // 编辑场景
              result = result.filter(item => item.id != this.formData.id)
            }
            if(result.some(item => item.name === value)) {
              callback(new Error('部门中已经有该名称了'))
            } else {
              callback()
            }
          }
        }], 
      }
    }
  },
  computed: {
    showTitle() {
      return this.formData.id ? "编辑部门" : "新增部门"
    }
  },
  created() {
    this.getManagerList()
  },
  methods: {
    close() {
      // 修改父组件的值，子传父
      // 只能重置模板中绑定的数据
      this.formData = {
        code: '', 
        introduce:'', 
        managerId: '', 
        name: '', 
        pid: ''
      }
      this.$refs.addDept.resetFields()
      this.$emit('update:showDialog', false)
    },
    async getManagerList() {
      this.managerList = await getManagerList()
    },
    // 点击确定时调用
    btnOK() {
      this.$refs.addDept.validate(async isOK => {
        if(isOK) {
          let msg =  '新增'
          // 通过formData中id
          if(this.formData.id) {
          // 编辑场景
          msg = '编辑'
          await updateDepartment(this.formData)
          } else {
          // 新增场景
          await addDepartment({...this.formData, pid: this.currentNodeId})
          }
          // 通知父组件更新
          this.$emit('updateDepartment')
          // 提示消息
          this.$message.success(`${msg}部门成功`)
          this.close()
        }
      })
    },
    async getDepartmentDetail() {
      this.formData = await getDepartmentDetail(this.currentNodeId)
    }
  }
}
</script>