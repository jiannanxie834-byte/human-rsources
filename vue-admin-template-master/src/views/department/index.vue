<template>
  <div class="container">
    <div class="app-container">
      <!-- 展示树形结构 -->
      <el-tree
        :expand-on-click-node="false"
        default-expand-all
        :data="depts"
        :props="defaultProps"
      >
        <!-- 节点结构 -->
        <template v-slot="{ data }">
          <el-row
            style="width: 100%; heigth: 40px"
            type="flex"
            justify="space-between"
            align="middle"
          >
            <el-col>{{ data.name }}</el-col>
            <el-col :span="4">
              <span class="tree-manager">{{ data.managerName }}</span>
              <el-dropdown @command="operateDept($event, data.id)">
                <!-- 显示区域 -->
                <span class="el-dropdown-link">
                  操作<i class="el-icon-arrow-down el-icon--right"></i>
                </span>
                <!-- 下拉选项 -->
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item command="add">添加子部门</el-dropdown-item>
                  <el-dropdown-item command="edit">编辑部门</el-dropdown-item>
                  <el-dropdown-item command="del">删除</el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </el-col>
          </el-row>
        </template>
      </el-tree>
    </div>
    <!-- 放置弹层组件 -->
    <!-- .sync修饰，表示可以接收子组件的事件 update：showDialog，值 => showDialog -->
    <!-- ref 可以获取 don 实例对象， ref 也可以获取自定义组件的实例对象 -->
    <add-dept
      ref="addDept"
      @updateDepartment="getDepartment"
      :current-node-id="currentNodeId"
      :show-dialog.sync="showDialog"
    ></add-dept>
  </div>
</template>
<script>
import { getDepartment, delDepartment } from '@/api/department'
import { transListToTreeData } from '@/utils'
import AddDept from './component/add-dept.vue'
export default {
  name: 'Department',
  components: { AddDept },
  data () {
    return {
      currentNodeId: null, // 存储当前点击的id
      showDialog: false, // 控制弹层显示与隐藏
      depts: [], // 数据属性
      defaultProps: {
        label: 'name', // 显示的字段的名字
        children: 'children' // 读取子节点的字段名
      }
    }
  },
  created() {
    // 调用获取数据的接口
    this.getDepartment()
  },
  methods: {
    // 封装好一个方法
    async getDepartment() {
      const result = await getDepartment()
      this.depts = transListToTreeData(result, 0)
    },
    operateDept(type, id) {
    // 添加子部门
    if(type === 'add'){
      // 添加子部门
    this.showDialog = true // 显示弹层
    this.currentNodeId = id
    } else if (type === 'edit') {
      // 编辑部门的场景
      this.showDialog = true
      this.currentNodeId = id
      // 要在子组件获取数据 父组件调用子组件方法来获取数据
      this.$nextTick(() => {
        this.$refs.addDept.getDepartmentDetail() // this.$refs.AddDept等同于子组件的this
      })
    } else {
      // 删除部门
      this.$confirm('您确认要删除该部门吗').then(async () => {
        await delDepartment(id)
        this.$message.success('删除部门成功')
        this.getDepartment()
      })
    }
    }
  }
}
</script>
<style scoped>
.app-container {
  padding: 30px 100px;
  font-size: 14px;
}
.tree-manager {
  width: 75px;
  display: inline-block;
  margin: 10px;
}
</style>
