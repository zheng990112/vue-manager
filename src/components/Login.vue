<template>
      <div class="login_container">
        <div class="login_box">
          <!--登陆表单区域-->
          <!--:model="表单数据对象" :rules="表单验证规则"-->
          <el-form :model="loginForm" :rules="loginRules" ref="loginRuleForm" class="login_form" >
            <el-form-item prop="username">
              <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
            </el-form-item>
            <el-form-item prop="password">
              <el-input v-model="loginForm.password" prefix-icon="iconfont icon-3702mima"></el-input>
            </el-form-item>
            <el-form-item class="btns">
              <el-button type="primary" @click="loginForm1">登录</el-button>
              <el-button type="info" @click="resetLoginForm">重置</el-button>
            </el-form-item>
          </el-form>
        </div>
      </div>
</template>

<script>
    export default {
        name: 'Login',
      data () {
          return {
            loginForm: {
              username: 'admin',
              password: '123456'
            },
            loginRules: { // 验证规则
              username: [
                { required: true, message: '请输入登录名称', trigger: 'blur' },
                { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
              ],
              password: [
                { required: true, message: '请输入登录密码', trigger: 'blur' },
                { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
              ]
            }
          }
      },
      methods: {
        //  点击重置登录表单
        resetLoginForm () {
          // console.log(this.$refs)
          this.$refs.loginRuleForm.resetFields()
        },
        //  点击登录
        loginForm1 () {
        //  通过表单对象. validate() 获取最终校验结果
          this.$refs.loginRuleForm.validate(async valid => {
            // console.log(valid)
            // 如果valid为false,不执行后面的操作
            if (!valid) return
            // 发送请求，使用async await简化异步操作
            const { data: res } = await this.$http.post('login', this.loginForm)
            // console.log(res)
            if (res.meta.status !== 200) {
              return this.$message.error(res.meta.msg)
            }
            this.$message({
              message: '登录成功',
              type: 'success'
            })
            // 保存token值
            window.sessionStorage.setItem('token', res.data.token)
            // 通过编程式导航跳转到后台首页
            this.$router.push('/home')
          })
        }
      }
    }
</script>

<style scoped lang="less">
  .login_container {
    background-color: #2b4b6b;
    height: 100%;
    .login_box{
      width: 450px;
      height: 300px;
      background: #fff;
      border-radius: 3px;
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      .login_form{
        position: absolute;
        width: 100%;
        padding: 20px;
        box-sizing: border-box;
        top: 20%;
      }
    }
    .btns{
      display: flex;
      justify-content: center;
    }
    .el-form-item div{
      margin-left: 0 !important;
    }
  }
</style>
