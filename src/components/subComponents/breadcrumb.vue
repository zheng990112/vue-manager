<template>
    <div>
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>{{ fistmenu }}</el-breadcrumb-item>
        <el-breadcrumb-item>{{ secmenu }}</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
</template>

<script>
    export default {
        name: 'breadcrumb',
      props: ['fistmenu', 'secmenu']
    }
</script>

<style scoped>

</style>
