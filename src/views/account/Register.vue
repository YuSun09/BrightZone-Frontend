<template>
  <div class="login-wrap">
    <div class="ms-login">
      <div class="ms-title">Register Account</div>
      <el-form :model="registerForm" :rules="rules" ref="register" label-width="0px" class="ms-content">
        <el-form-item prop="email">
          <el-input v-model="registerForm.emailOrUserId" placeholder="email or userId">
            <el-button slot="prepend" icon="el-icon-lx-people"></el-button>
          </el-input>
        </el-form-item>
        <div class="register-btn">
          <el-button type="primary" @click="register('register')">Register</el-button>
        </div>
        <a class="passwordRecovery" href="http://localhost:8181/passwordRecovery">forget password?</a>
        <div class="login-btn">
          <el-button type="primary">
            <router-link style="color: white" to="/login">Back to Login</router-link>
          </el-button>
        </div>
      </el-form>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import qs from 'qs'

export default {
  name: "Register",
  data: function () {
    return {
      registerForm: {
        emailOrUserId: '',
      },
      responseResult: {
        success: '',
        errMsg: '',
      },
      rules: {
        emailOrUserId: [
          {required: true, message: 'Please input email or userId', trigger: 'blur'},
        ],
      },
    };
  },
  methods: {
    register(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          axios.post('http://localhost:8080/api/account/register', qs.stringify(this.registerForm),
              {headers: {'Content-Type': 'application/x-www-form-urlencoded'}})
          .then((resp) => {
            console.log(resp)
            this.responseResult.success = resp.data.success
            if (resp.data.success) {
              this.$confirm('Register request send success, do you want to go to login page?', 'Success', {
                confirmButtonText: 'Yes',
                cancelButtonText: 'No',
                type: 'success'
              }).then(() => {  // handle yes
                this.$router.push('/login');
              }).catch(() => {});
            } else {
              this.responseResult.errMsg = resp.data.errMsg
              this.$message({
                showClose: true,
                message: 'Error: ' + this.responseResult.errMsg,
                type: 'error',
                duration: 6000,
              });
            }
          }).catch((err) => {})
        } else {
          this.$message({
            showClose: true,
            message: 'Warning: Please input valid email',
            type: 'warning',
            duration: 7000,
          });
          return false
        }
      })
    }
  },
}
</script>

<style scoped>
@import "../../assets/css/main.css";
@import "../../assets/css/color-dark.css";

.el-form >>> .el-form-item__error {
  color: black;
}

.login-wrap {
  position: relative;
  width: 100%;
  height: 100%;
  background-image: url(../../assets/img/login-bg.jpg);
  background-size: 100%;
}

.ms-title {
  width: 100%;
  line-height: 50px;
  text-align: center;
  font-size: 20px;
  color: black;
  border-bottom: 1px solid #ddd;
}

.ms-login {
  position: absolute;
  left: 50%;
  top: 50%;
  width: 350px;
  margin: -190px 0 0 -175px;
  border-radius: 5px;
  background: rgba(255, 255, 255, 0.3);
  overflow: hidden;
}

.ms-content {
  padding: 30px 30px;
}

.register-btn {
  text-align: center;
}

.register-btn button {
  width: 100%;
  height: 36px;
  margin-bottom: 10px;
}

.login-btn {
  float: right;
  margin-bottom: 10px;
}

.passwordRecovery {
  margin-top: 5px;
  margin-bottom: 10px;
  font-size: 14px;
  color: red;
  float: left;
  text-decoration:underline;
}
</style>
