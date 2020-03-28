<template>
    <el-container class="home-container">
      <!-- 头部区域 -->
      <el-header>
        <div>
          <span>电商后台管理</span>
        </div>
        <el-button @click="userLoginOut" type="info">退出</el-button>
      </el-header>
      <!-- 页面主体区域 -->
      <el-container>
        <!-- 侧边部分 -->
          <el-aside>
            <el-menu text-color="#fff" background-color="#747d8c" active-text-color="#70a1ff" unique-opened router
            :default-active="activePath">
              <template v-for="item in menulist">
                <el-submenu :index="item.id.toString()" :key="item.id">
                  <template slot="title" >
                    <i></i>
                    <span>{{item.authName}}</span>
                  </template>
                  <template v-for="subitem in item.children">
                    <el-menu-item :index="subitem.path.toString()" :key="subitem.id" @click="changeActivePath(subitem.path.toString())">
                      <template slot="title">
                        <i class="el-icon-menu"></i>
                        <span>{{subitem.authName}}</span>
                      </template>
                    </el-menu-item>
                  </template>
                </el-submenu>
              </template>
            </el-menu>
          </el-aside>
        <!-- 右侧内容部分 -->
        <el-main>
          <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>
</template>

<script>
export default {
  data () {
    return {
      menulist: [],
      activePath: ''
    }
  },
  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    userLoginOut () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 获取所有菜单
    async getMenuList () {
      const { data: res } = await this.$axios.get('menus')
      if (res.meta.status !== 200) return this.message.error(res.meta.msg)
      this.menulist = res.data
    },
    // 改变激活高亮状况
    changeActivePath (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}

.el-header {
  background-color: #dcdde1;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.el-aside {
  width: 200px;
  background-color: #747d8c;
}

.el-aside > .el-menu {
  border-right: none;
}

.el-main {
  background-color: #f5f6fa;
}
</style>
