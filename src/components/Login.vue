<template>
  <div class="main hold-transition login-page">

    <div class="login-box">
      <div class="login-logo">
        <a href="javascript:void(0)"><b>侦码设备</b>管理后台</a>
      </div>
      <!-- /.login-logo -->
      <div class="login-box-body">
        <form>
          <div class="form-group has-feedback" style="margin-top: 30px">
            <input class="form-control" placeholder="账号" v-model="account.loginId" maxlength="16">
            <span class="glyphicon glyphicon-user form-control-feedback"></span>
          </div>
          <div class="form-group has-feedback">
            <input type="password" class="form-control" placeholder="密码" v-model="account.password" maxlength="16">
            <span class="glyphicon glyphicon-lock form-control-feedback"></span>
          </div>
          <el-row>
            <el-col :span="14">
              <input class="form-control" placeholder="验证码" v-model="account.checkcode" maxlength="4"
                     @keyup.13="login()">
            </el-col>
            <el-col :span="8" :offset="1">
              <img alt="验证码" style="cursor:pointer;" title="看不清可单击图片刷新" id="img"
                   v-on:click="refreshImg()" v-bind:src="imgUrl">
            </el-col>
          </el-row>
          <div class="row" style="margin-top: 40px">
            <div class="col-xs-8">
              <!--<div class="checkbox icheck">-->
              <!--<label>-->
              <!--<input type="checkbox"> Remember Me-->
              <!--</label>-->
              <!--</div>-->
            </div>
            <!-- /.col -->
            <div class="col-xs-4">
              <el-button type="primary" class="btn btn-primary btn-block btn-flat" @click="login()"
                         :loading="logining">登录
              </el-button>
            </div>
            <!-- /.col -->
          </div>
        </form>

        <!--<a href="#">忘记密码</a><br>-->
        <!--<a href="register.html" class="text-center">Register a new membership</a>-->

      </div>
      <!-- /.login-box-body -->
    </div>
  </div>
</template>

<script>
  import md5 from 'js-md5';
  import axios from "axios";

  export default {
    data() {
      return {
        logining: false,
        account: {
          loginId: '',
          password: '',
          checkcode: ''
        },
        imgUrl: ''
      }
    },
    methods: {
      login() {
        if (!this.account.loginId || !this.account.password || !this.account.checkcode) {
          this.$message.error('请输入账号、密码或验证码');
          return;
        }
        this.logining = true;
        this.$post("account/login", {
          loginId: this.account.loginId, password: md5(this.account.password), checkcode: this.account.checkcode
        }, "登录成功")
          .then((data) => {
            this.logining = false;
            if (data) {
              sessionStorage.setItem("user", JSON.stringify(data.data));
              this.$router.push("/deviceList");
            }
          });
      },
      getUrl() {
        setTimeout(() => {
          this.imgUrl = axios.defaults.baseURL + "/account/getCheckImage?" + Math.random();
        }, 500);
      },
      //更新验证码
      refreshImg() {
        this.imgUrl = axios.defaults.baseURL + "/account/getCheckImage?" + Math.random();
      }
    },
    mounted() {
      sessionStorage.removeItem("active");
      this.getUrl();
//      let url = axios.defaults.baseURL.replace("http://", "https://");
    }
  }
</script>
<style>
  .main {
    height: 100%;
    width: 100%;
    position: absolute;
  }
</style>
