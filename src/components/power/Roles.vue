<template>
    <div>
        <!-- 面包屑 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片 -->
        <el-card>
            <!-- 添加角色按键 -->
            <el-row>
                <el-col>
                    <el-button type="primary">添加角色</el-button>
                </el-col>
            </el-row>
            <!-- 角色列表区域 -->
            <el-table :data="rolestree" border stripe>
                <!-- 展开列 -->
                <el-table-column type="expand">
                    <template v-slot="scope">
                        <!-- 渲染一级权限 -->
                        <el-row :class="['elbottom', index === 0 ? 'eltop' : '', 'viewcenter']" v-for="(item, index) in scope.row.children" :key="index">
                            <el-col :span="5">
                                <el-tag closable @close="removeAuthorityById(scope.row, item.id)">{{item.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <el-col :span="19">
                                <!-- 渲染二级权限 -->
                                <el-row :class="[index1 === 0 ? '' : 'eltop', 'viewcenter']" v-for="(item1, index1) in item.children" :key="index1">
                                    <el-col :span="6">
                                        <el-tag type="success" closable @close="removeAuthorityById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="18">
                                        <el-tag type="warning" v-for="item2 in item1.children" :key="item2.id" closable @close="removeAuthorityById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <!-- 索引列 -->
                <el-table-column type="index"></el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作" width="300px">
                    <template v-slot="scope">
                        <!-- 修改角色 -->
                        <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                        <!-- 删除角色 -->
                        <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                        <!-- 分配角色权限 -->
                        <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRolesAuthorityDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <!-- 修改角色权限样式框 -->
        <el-dialog title="分配权限" :visible.sync="setRolesAuthorityDialogVisible" @close="handleDialogClosed" width="50%">
            <el-tree :data="rightstree" :props="rightsTreeProps" node-key="id" :default-checked-keys="checkedKeys" ref="treeRef" show-checkbox default-expand-all></el-tree>
            <span slot="footer">
                <el-button @click="setRolesAuthorityDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="setRolesRights">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    return {
      rolestree: [],
      rightstree: [],
      checkedKeys: [],
      // 角色id
      roleId: '',
      // dialog是否可见
      setRolesAuthorityDialogVisible: false,
      // 树形控件绑定
      rightsTreeProps: {
        children: 'children',
        label: 'authName'
      }
    }
  },
  created () {
    this.getRolesTree()
  },
  methods: {
    async getRolesTree () {
      const { data: res } = await this.$axios.get('roles')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.rolestree = res.data
    },
    // 通过id移除权限
    async removeAuthorityById (role, rightId) {
      const confirmRules = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)
      if (confirmRules !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$axios.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除用户权限失败')
      this.$message.success('已删除')
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRolesAuthorityDialog (role) {
      this.roleId = role.id
      const { data: res } = await this.$axios.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.rightstree = res.data
      // 获取三级节点的id
      this.getLeafKeys(role, this.checkedKeys)
      this.setRolesAuthorityDialogVisible = true
    },
    // 修改用户权限
    async setRolesRights () {
      const keys = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()].join()
      const { data: res } = await this.$axios.post(`roles/${this.roleId}/rights`, { rids: keys })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success('分配权限成功')
      this.getRolesTree()
      this.setRolesAuthorityDialogVisible = false
    },
    // 获取角色的权限id
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 监听显示框的关闭
    handleDialogClosed () {
      // 可选权限清空
      this.checkedKeys = []
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
    margin: 7px;
}

.elbottom {
    border-bottom: 1px solid #eee;
}

.eltop {
    border-top: 1px solid #eee;
}

.viewcenter {
    display: flex;
    align-items: center;
}
</style>
