<template>
    <div>
        <el-button @click="userLoginOut" type="info">退出</el-button>
    </div>
</template>

<script>
export default {
  methods: {
    userLoginOut () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script>

<style lang="less" scoped></style>
