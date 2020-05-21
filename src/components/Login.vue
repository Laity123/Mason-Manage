<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt />
      </div>
      <!-- 表单区域 -->
      <el-form
        ref="LoginFormRef"
        :model="LoginForm"
        :rules="LoginFormRules"
        label-width="0px"
        class="login_form"
      >
        <el-form-item prop="username">
          <el-input v-model="LoginForm.username" prefix-icon="el-icon-sunny"></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model="LoginForm.password" prefix-icon="el-icon-lock" type="password"></el-input>
        </el-form-item>
        <el-form-item class="login_btn">
          <el-button type="primary" @click="loginForm">登录</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  name: "",
  data() {
    return {
      //登录表单的数据绑定对象
      LoginForm: {
        username: "Admin",
        password: "123456"
      },
      // 表单的验证规则对象
      LoginFormRules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
          { min: 3, max: 10, message: "用户名为3-10个字符", trigger: "blur" }
        ],
        password: [
          { required: true, message: "请输入密码", trigger: "blur" },
          { min: 6, max: 15, message: "密码为6-15个字符", trigger: "blur" }
        ]
      }
    };
  },
  methods: {
    // 重置表单
    resetLoginForm() {
      this.$refs.LoginFormRef.resetFields();
    },
    //表单预验证
    loginForm() {
      this.$refs.LoginFormRef.validate(async valid => {
        if (!valid) return;
        const { data: res } = await this.$http.post("login", this.LoginForm);
        if (res.meta.status !== 200) {
          this.$message.error("登录失败!");
        } else {
          this.$message.success("登录成功!");
        }
        // 将登陆成功后的token，保存在客户端的sessionStorage中 [sessionStorage 是会话期间的存储机制localStorage是持久化存储机制]
        window.sessionStorage.setItem('token',res.data.token);
        this.$router.push('/home');
      });
    }
  }
};
</script>

<style lang='less' scoped>
.login_container {
  background-color: #383452;
  height: 100%;
}
.login_box {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 450px;
  height: 300px;
  background-color: #fff;
  border: 1px solid #241315;
  border-radius: 3px;
  box-shadow: 0 0 10px #535b54;
  .avatar_box {
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    padding: 5px;
    width: 100px;
    height: 100px;
    border: 1px solid #241315;
    border-radius: 50%;
    background-color: rgba(255, 255, 255, .5);
    box-shadow: 0 0 5px #aaa;
    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
    }
  }
}
.login_form {
  box-sizing: border-box;
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0 20px;
  .login_btn {
    display: flex;
    justify-content: flex-end;
  }
}
</style>
