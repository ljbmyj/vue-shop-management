<template>
    <div>
        <!-- 面包屑 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片 -->
        <el-card>
            <!-- 添加角色按键 -->
            <el-row>
                <el-col>
                    <el-button type="primary" @click="addSort">添加分类</el-button>
                </el-col>
            </el-row>
            <!-- 树形表格 -->
            <tree-table :data="cateList" :columns="columns" :selection-type="false" :expand-type="false" show-index index-text="#" border>
              <template slot="isok" slot-scope="scope">
                <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen;"></i>
                <i class="el-icon-error" v-else style="color: red;"></i>
              </template>
              <template slot="sort" slot-scope="scope">
                <el-tag v-if="scope.row.cat_level === 0" size="mini">一级</el-tag>
                <el-tag type="success" v-else-if="scope.row.cat_level === 1" size="mini">二级</el-tag>
                <el-tag type="warning" v-else size="mini">三级</el-tag>
              </template>
              <template slot="option">
                <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
              </template>
            </tree-table>
            <!-- 步骤条 -->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
             :current-page="queryInfo.pagenum" :page-size="queryInfo.pagesize" layout="total, prev, pager, next, jumper" :total="total">
            </el-pagination>
        </el-card>
        <!-- 添加分类对话框 -->
        <el-dialog title="添加分类" :visible.sync="addSortDialogVisible" width="50%" @close="addSortDialogClosed">
          <div>
           <el-form :model="addSortForm" :rules="addSortRules" ref="addSortFormRef" label-width="100px">
            <el-form-item label="分类名称" prop="cat_name">
              <el-input v-model="addSortForm.cat_name"></el-input>
            </el-form-item>
            <el-form-item label="父级分类">
              <el-cascader :options="parentCateList" :props="cascaderProps" change-on-select
              v-model="selectedKeys" @change="parentCateChanged" clearable></el-cascader>
            </el-form-item>
           </el-form>
          </div>
          <span slot="footer" class="dialog-footer">
            <el-button @click="addSortDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="makeSureAddSort">确 定</el-button>
          </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    return {
      // 获取商品列表
      cateList: [],
      parentCateList: [],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      // cascader配置
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectedKeys: [],
      // dialog是否可见
      addSortDialogVisible: false,
      // dialog数据
      addSortForm: {
        cat_pid: 0,
        cat_name: '',
        cat_level: 0
      },
      // dialog验证规则
      addSortRules: {
        cate_name: [
          { required: true, message: '请输入分类名称', trigger: ['blur', 'change'] },
          { min: 0, max: 10, message: '分类名长度在10个字符之间', trigger: 'blur' }
        ]
      },
      // 树状columns
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'sort'
        },
        {
          label: '操作',
          type: 'template',
          template: 'option'
        }]
    }
  },

  created () {
    this.getCateList()
  },

  methods: {
    async getCateList () {
      const { data: res } = await this.$axios.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.cateList = res.data.result
      this.total = res.data.total
    },
    // 获取父级分类列表
    async getParentCateList () {
      const { data: res } = await this.$axios.get('categories', { type: 2 })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      console.log(res.data)
      this.parentCateList = res.data
    },
    // 步骤条操作
    handleSizeChange (newPageSize) {
      this.queryInfo.pagesize = newPageSize
      this.getCateList()
    },
    handleCurrentChange (newPageNum) {
      this.queryInfo.pagenum = newPageNum
      this.getCateList()
    },
    // 添加分类操作
    addSort () {
      this.getParentCateList()
      this.addSortDialogVisible = true
    },
    // 父级分类改变操作
    parentCateChanged () {
      console.log(this.selectedKeys)
      if (this.selectedKeys.length > 0) {
        this.addSortForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addSortForm.cat_level = this.selectedKeys.length
      } else {
        this.addSortForm.cat_pid = 0
        this.addSortForm.cat_level = 0
      }
    },
    // 清空dialog
    addSortDialogClosed () {
      this.$refs.addSortFormRef.resetFields()
      this.selectedKeys = []
      this.addSortForm.cat_level = 0
      this.addSortForm.cat_pid = 0
    },
    makeSureAddSort () {
      this.$refs.addSortFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$axios.post('categories', this.addSortForm)
        if (res.meta.status !== 201) return this.$message.error(res.meta.msg)
        this.$message.success(res.meta.msg)
        this.getCateList()
        this.addSortDialogVisible = false
      })
    }
  }
}
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
