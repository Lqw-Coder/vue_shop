<template>
  <div class="login_container">
    <div class="container_box">
      <!-- 头像框 -->
      <div class="avator_box">
        <img src="../assets/logo.png" alt />
      </div>
      <!-- 表单输入框 -->
      <el-form
        label-width="0"
        class="login_form"
        :model="loginForm"
        :rules="loginRules"
        ref="numberValidateForm"
      >
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input prefix-icon="iconfont icon-user" v-model="loginForm.username"></el-input>
        </el-form-item>
        <!-- 密码框 -->
        <el-form-item prop="password">
          <el-input
            prefix-icon="iconfont icon-3702mima"
            v-model="loginForm.password"
            type="password"
          ></el-input>
        </el-form-item>
        <!-- 按钮框 -->
        <el-form-item class="btns">
          <el-button type="primary" @click="login('numberValidateForm')">登陆</el-button>
          <el-button type="info" @click="resetForm('numberValidateForm')">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      loginForm: {
        username: "",
        password: ""
      },
      loginRules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
          { min: 3, max: 10, message: "长度在 3 到 5 个字符", trigger: "blur" }
        ],
        password: [
          { required: true, message: "请输入用户名密码", trigger: "blur" },
          { min: 6, max: 15, message: "长度在 3 到 5 个字符", trigger: "blur" }
        ]
      }
    };
  },
  methods: {
    //点击重置按钮，重置登陆表单
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    login(formName) {
      this.$refs[formName].validate(valid => {
        if (!valid) return;
        this.$http.post("login", this.loginForm).then(ret => {
          const res = ret.data;
          if (res.meta.status != 200) {
            this.$message.error("用户名或密码错误");
          } else {
            this.$message.success("登陆成功");
            window.sessionStorage.setItem("token", res.data.token);
            console.log("登陆成功，实现跳转");
            this.$router.push("/home");
          }
        });
      });
    }
  }
};
</script>
<style lang="less" scoped>
.login_container {
  height: 100%;
  background-color: #2b4b6b;
}
.container_box {
  width: 450px;
  height: 300px;
  position: absolute;
  border-radius: 3px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: white;
  .avator_box {
    height: 130px;
    width: 130px;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    border-radius: 50%;
    background-color: #fff;
    box-shadow: 0 0 10px #ddd;
    padding: 10px;
    border: 1px solid #eee;
    img {
      background-color: #eee;
      border-radius: 50%;
      width: 100%;
      height: 100%;
    }
  }
  .login_form {
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 10px;
    box-sizing: border-box;
    padding: 0 20px;
  }
  .btns {
    display: flex; //设置为弹性模型
    justify-content: flex-end; //从右到左排列
  }
}
</style>