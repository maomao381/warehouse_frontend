<template>
  <el-container>
    <el-main>
      <el-form
        :model="registerForm"
        :rules="rules"
        ref="registerForm"
        label-position="left"
        label-width="0px"
        class="demo-ruleForm login-container"
      >
        <h3 class="title">用户注册</h3>
        <el-form-item prop="username">
          <el-input
            type="text"
            v-model="registerForm.username"
            auto-complete="off"
            placeholder="请输入用户名"
          ></el-input>
        </el-form-item>
        <el-form-item prop="password" style="margin-top: 20px">
          <el-input
            type="password"
            v-model="registerForm.password"
            auto-complete="off"
            placeholder="请输入密码"
          ></el-input>
        </el-form-item>
        <el-form-item prop="passConfirm" style="margin-top: 20px">
          <el-input
            type="password"
            v-model="registerForm.passConfirm"
            auto-complete="off"
            placeholder="请再次输入密码"
          ></el-input>
        </el-form-item>
        <el-form-item prop="nickname" style="margin-top: 20px">
          <el-input
            type="text"
            v-model="registerForm.nickname"
            auto-complete="off"
            placeholder="请输入姓名"
          ></el-input>
        </el-form-item>
        <el-form-item prop="gender" style="margin-top: 5px">
          <el-radio-group v-model="registerForm.gender">
            <el-radio label="男"></el-radio>
            <el-radio label="女"></el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item prop="workid" style="margin-top: 20px">
          <el-input
            type="text"
            v-model="registerForm.workid"
            auto-complete="off"
            placeholder="请输入工号"
          ></el-input>
        </el-form-item>
        <el-form-item prop="auth" style="margin-top: 5px">
          <el-radio-group v-model="registerForm.auth">
            <el-radio :label="0">原料管理员</el-radio>
            <el-radio :label="1">财务经理</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item prop="phone" style="margin-top: 20px">
          <el-input
            type="text"
            v-model="registerForm.phone"
            auto-complete="off"
            placeholder="请输入电话"
          ></el-input>
        </el-form-item>
        <el-form-item prop="email" style="margin-top: 20px">
          <el-input
            type="text"
            v-model="registerForm.email"
            auto-complete="off"
            placeholder="请输入邮箱"
          ></el-input>
        </el-form-item>
        <el-form-item style="width:100%;">
          <el-button
            type="primary"
            style="width:100%;"
            @click="handleSubmit"
            :loading="registering"
          >注册</el-button>
        </el-form-item>
        <el-form-item style="width:100%;">
          <el-button type="text" style="width:100%;" @click="toLogin">登录</el-button>
        </el-form-item>
      </el-form>
    </el-main>
  </el-container>
</template>

<script>
export default {
  name: "Register",
  data() {
    let passwordValidity = (rule, value, callback) => {
      // Evan: 这里设定密码的规则，value值是输入框中的值
      // 密码必须含有字母和数字，长度为6-18位
      let myReg = /^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z_]{6,16}$/;
      if (!value) {
        return callback(new Error("请输入密码"));
      } else if (!myReg.test(value)) {
        return callback(new Error("包含字母和数字，且不小于6位"));
      } else {
        return callback();
      }
    };
    let repasswordValidity = (rule, value, callback) => {
      let myReg = /^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z_]{6,16}$/;
      if (!value) {
        return callback(new Error("请再次输入密码以确认"));
      }
      if (value !== this.registerForm.password) {
        return callback(new Error("两次输入的密码不一致"));
      } else if (!myReg.test(value)) {
        return callback(new Error("包含大写字母和数字，且不小于6位"));
      } else {
        callback();
      }
    };
    return {
      registerForm: {
        username: "",
        password: "",
        passConfirm: "",
        auth: 0,
        nickname: "",
        gender: "男",
        workid: ""
      },
      rules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" }
        ],
        password: [
          { required: true, validator: passwordValidity, trigger: "blur" }
        ],
        passConfirm: [
          { required: true, validator: repasswordValidity, trigger: "blur" }
        ],
        nickname: [{required: true, message: "请输入姓名", trigger: "blur"}],
        email: [{required: true, message: "请输入邮箱", trigger: "blur"}],
        phone: [{required: true, message: "请输入电话", trigger: "blur"}]
      },
      registering: false
    };
  },
  methods: {
    toLogin() {
      this.$router.push("/login");
    },
    handleSubmit() {
      this.$refs.registerForm.validate(valid => {
        if (valid) {
          this.registering = true;
          this.$http
            .post(
              "/apis/register",{},
              {
                params: {
                  username: this.registerForm.username,
                  password: this.registerForm.password,
                  auth: this.registerForm.auth,
                  nickname: this.registerForm.nickname,
                  workid: this.registerForm.workid,
                  gender: this.registerForm.gender,
                  phone: this.registerForm.phone.toString(),
                  email: this.registerForm.email
                },
                //headers: {'Content-Type': 'application/json'}
              }
            )
            .then(
              response => {
                if (response.status === 200) {
                  this.$notify({
                    title: "注册成功",
                    message: "跳转到登录界面",
                    type: "success",
                    duration: 2000
                  });
                  window.setTimeout(_ => {
                    this.$router.push("/login");
                  }, 1000 * 3);
                  this.registering = false;
                } else {
                  this.$notify.error({
                    title: "注册失败",
                    message: "网络错误",
                    duration: 200
                  });
                  this.registering = false;
                }
              },
              response => {
                this.$notify.error({
                  title: "注册失败",
                  message: "网络错误",
                  duration: 2000
                });
                this.registering = false;
              }
            );
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    }
  }
};
</script>

<style scoped>
.login-container {
  -webkit-border-radius: 5px;
  border-radius: 5px;
  -moz-border-radius: 5px;
  background-clip: padding-box;
  margin: 180px auto;
  width: 350px;
  padding: 35px 35px 15px 35px;
  background: #fff;
  border: 1px solid #eaeaea;
  box-shadow: 0 0 25px #cac6c6;
}
.title {
  margin: 0 auto 40px auto;
  text-align: center;
  color: #505458;
}
</style>
