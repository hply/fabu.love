<template>
  <div class="user">
    <div class="user-left">

      <div class="user-logo"></div>
      <div class="user-divide"></div>
      <div class="user-name">fabu.love</div>

      <div class="user-item" style="margin-top: 24.87vh" @click="clickSubItem('api')">
        <span class="icon-ic_api_s"></span>
        <p>API文档</p>
      </div>
      <div class="user-item" @click="clickSubItem('help')">
        <span class="icon-ic_help_s"></span>
        <p>Help</p>
      </div>
      <div class="user-item" @click="clickSubItem('github')">
        <span class="icon-ic_github_s"></span>
        <p>Github</p>
      </div>
      <div class="user-item" @click="clickSubItem('about')">
        <span class="icon-ic_about_s"></span>
        <p>About</p>
      </div>
    </div>
    <div class="user-right">
      <div class="user-login-panel">
        <div class="user-login-title" v-if="showType==='login'">
          <p> 登&nbsp;&nbsp;&nbsp;&nbsp;录 <span>/Login</span></p>
        </div>
        <div class="user-login-title" v-if="showType==='register'">
          <p> 注&nbsp;&nbsp;&nbsp;&nbsp;册 <span>/Register</span></p>
        </div>
        <div class="user-login-title" v-if="showType==='forget'">
          <p> 找回密码 <span>/Request Password</span></p>
        </div>

        <div v-bind:class="[showType==='login' ? 'user-login-form' : 'user-register-form']">
          <el-input
            type="text"
            class="user-login-form-input"
            v-model="username"
            placeholder="请输入用户名"
          >
            <template #prepend>
              <el-button icon="User" tabindex="-1"/>
            </template>
          </el-input>

          <el-input
            v-if="showType==='register'"
            class="user-login-form-input"
            placeholder="请输入邮箱"
            v-model="email"
            type="text"
          >
            <template #prepend>
              <el-button icon="MessageBox" tabindex="-1"/>
            </template>
          </el-input>

          <el-input
            class="user-login-form-input"
            placeholder="请输入密码"
            v-model="password"
            type="password"
          >
            <template #prepend>
              <el-button icon="Lock" tabindex="-1"/>
            </template>
          </el-input>

          <el-button
            @click="onSubmit"
            v-bind:class="[showType==='login' ? 'user-login-form-btn' : 'user-register-form-btn']"
            type="primary"
            :loading="showLoading"
          >
            {{ showType === 'login' ? '立即登录' : '立即注册' }}
          </el-button>

          <div class="user-login-form-label" v-if="showType==='login'">
            <p>没有账号？<span @click="onRegister">立即注册</span></p>
          </div>
          <div class="user-login-form-label" v-if="showType==='register'">
            <p>已有账号？<span @click="onRegister">现在登录</span></p>
          </div>
          <div class="user-login-form-label user-login-form-bottom">
            <p @click="onForget">忘记密码</p>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script type="text/ecmascript-6">
import { useRouter } from "vue-router";
import * as LoginApi from '../../api/moudle/loginApi'
import TokenMgr from '../../mgr/TokenMgr'
import { saveUserInfo } from '@/mgr/userMgr.js'
import utils from '../../common/js/utils'
import { Search, User } from '@element-plus/icons-vue'

export default {
  computed: {},
  data() {
    return {
      username: '',
      password: '',
      isLogin: false,
      errorInfo: '',
      showType: 'login',
      email: '',
      showLoading: false
    }
  },
  created() {
    this.$nextTick(() => {
    })
    this.router = useRouter()
  },
  components: {
    Search,
    User
  },
  methods: {
    onSubmit() {
      if ( this.showType === 'login' ) {
        this.login()
      } else if ( this.showType === 'register' ) {
        this.register()
      } else {
        this.requestPassword()
      }
    },
    onRegister() {
      this.showType = this.showType === 'register' ? 'login' : 'register'
    },
    onForget() {
      this.$prompt('请输入邮箱,我们会发送新密码到您到邮箱', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        inputPattern: /[\w!#$%&'*+/=?^_`{|}~-]+(?:\.[\w!#$%&'*+/=?^_`{|}~-]+)*@(?:[\w](?:[\w-]*[\w])?\.)+[\w](?:[\w-]*[\w])?/,
        inputErrorMessage: '邮箱格式不正确'
      }).then(({ value }) => {
        this.email = value
        this.requestPassword()
      }).catch((e) => {
        this.$message({
          type: 'info',
          message: e
        })
      })
    },
    login() {
      if ( this.username.length === 0 ) {
        this.errorInfo = '* 用户名不能为空'
        return
      }
      if ( this.password.length === 0 ) {
        this.errorInfo = '* 密码不能为空'
        return
      }
      this.errorInfo = ''
      let body = {
        'username': this.username,
        'password': this.password
      }
      this.showLoading = true
      LoginApi.login(body)
        .then(response => {
          console.log(response)
          this.showLoading = false
          // 存储token
          TokenMgr.add(this.axios.baseURL, response.data.token)
          let user = {
            'userName': this.username,
            'userId': response.data._id,
            'teamArr': response.data.teams,
            'email': response.data.email,
            'apiToken': response.data.apiToken
          }
          // 保存用户信息
          saveUserInfo(user)
          this.router.push('/')
        }, reject => {
          this.showLoading = false
        })
    },
    register() {
      if ( this.username.length === 0 ) {
        this.$message.error('用户名不能为空')
        return
      }
      if ( this.email.length === 0 ) {
        this.$message.error('邮箱不能为空')
        return
      }
      if ( !utils.valideEmail(this.email) ) {
        this.$message.error('邮箱格式错误')
        return
      }
      if ( this.password.length === 0 ) {
        this.$message.error('密码不能为空')
        return
      }
      let body = {
        'username': this.username,
        'password': this.password,
        'email': this.email
      }
      this.showLoading = true
      LoginApi.register(body)
        .then(response => {
          this.showLoading = false
          console.log(response)
          this.$message({
            message: '恭喜你，注册成功',
            type: 'success'
          })
          setTimeout(() => {
            this.onRegister()
          }, 500)
        }, reject => {
          this.showLoading = false
        })
    },
    requestPassword() {
      if ( this.email.length === 0 ) {
        this.$message.error('邮箱不能为空')
        return
      }
      LoginApi.resetPassword({ email: this.email })
        .then(response => {
          console.log(response)
          this.$message({
            message: '密码已重置,新密码已通过邮件发送至您的邮箱.',
            type: 'success'
          })
        }, reject => {
          console.log(reject)
          this.$message.error(reject)
        })
    },
    clickSubItem(data) {

      if ( data === 'api' ) {
        let href = `${ this.axios.defaults.baseURL }api/swagger`
        window.open(href, '_blank')
      }
      if ( data === 'help' ) {
        const domain = this.axios.defaults.baseURL
          .replace('https://', '')
          .replace('http://', '')
          .replace('www.', '');

        let href = `http://wiki.${ domain }`
        console.log(href)
        window.open(href, '_blank')
      }
      if ( data === 'github' ) {
        let href = 'https://github.com/rock-app/fabu.love'
        window.open(href, '_blank')
      }
      if ( data.index === 'About' ) {
      }
    }

  }
}

</script>

<style lang="scss">
@use "../../common/scss/base" as *;

.user {
  width: 100%;
  height: 100%;
  position: absolute;
  display: flex;
  display: -webkit-flex;
  background-image: url("../../common/assets/bg-samll.png");
  background-size: 100% 100%;
  font-family: PingFangSC, serif;
  overflow: hidden;
}

.user-left {
  width: 40%;
  height: 100%;
  background: $mainColor;
  opacity: 0.9;
  display: flex;
  max-width: 600px;
  min-width: 200px;
  display: -webkit-flex;
  flex-direction: column;
  align-items: center;
}

.user-right {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.user-logo {
  width: 210px;
  height: 50px;
  margin-top: 17.6vh;
  background-size: 210px 40px;
  background-repeat: no-repeat;
  background-image: url("../../common/assets/logo_w.png");
}

.user-divide {
  width: 168px;
  height: 3px;
  margin-top: 16px;
  background: white;
}

.user-name {
  font-size: 1.5em;
  color: white;
  margin-top: 24px;
  font-family: STYuanti-SC, serif;
}

.user-item {
  display: flex;
  display: -webkit-flex;
  align-items: center;
  margin-top: 24px;
}

.user-item :hover {
  cursor: pointer;
}

.user-item img {
  width: 24px;
  height: 24px;
}

.user-item p {
  color: white;
  font-size: 1em;
  margin-left: 24px;
  width: 60px;
}

.user-login-panel {
  overflow: hidden;
  height: 528px;
  width: 408px;
  background-color: white;
  position: absolute;
  border-radius: 8px;
}

.user-login-title {
  height: 72px;
  width: 100%;
  font-size: 20px;
  line-height: 24px;
  padding-left: 48px;
  box-shadow: 0 1px 0 0 rgba(0, 0, 0, 0.1)
}

.user-login-title p {
  color: rgba(53, 64, 82, 1);
  height: 100%;
  line-height: 72px;
}

.user-login-title p span {
  color: rgba(155, 155, 155, 1);
}

.user-login-form, .user-register-form {
  width: 100%;
  text-align: center;
}

.user-login-form .user-login-form-input {
  margin-top: 48px;
}

.user-register-form .user-login-form-input {
  margin-top: 24px;
}

.user-login-form-input {
  width: 312px;
  height: 48px;
}

.user-login-form-input input {
  height: 48px;
}

.user .el-input__prefix {
  padding-left: 18px;
}

.user .el-input__prefix i {
  line-height: 48px;
}

.user .el-input--prefix .el-input__inner {
  padding-left: 50px;
  line-height: 48px;
}

.user-register-form-btn, .user-login-form-btn, .user-login-form-btn:hover {
  width: 312px;
  height: 48px;
  border-radius: 24px;
  background: rgba(100, 119, 242, 1);
  border: rgba(100, 119, 242, 1);
  color: white;
  font-size: 14px;
  cursor: pointer;
}

.user-login-form-btn {
  margin-top: 48px;
}

.user-register-form-btn {
  margin-top: 24px;
}

.user-login-form-label {
  margin-top: 48px;
  cursor: pointer;
}

.user-login-form-bottom {
  float: right;
  margin-right: 48px;
  font-size: 14px;
  color: rgba(155, 155, 155, 1);
  margin-bottom: 24px;
  cursor: pointer;
}

.user-footer-label {
  display: block;
  width: 100%;
  position: fixed;
  text-align: center;
  bottom: 24px;
}

.user-footer-label p {
  size: 1rem;
  color: white;
  margin: 0 auto;
}

@media screen and (max-width: 768px) {
  .user-left {
    display: none;
    position: relative;
  }
}

@media screen and (max-height: 650px) {
  .user-footer-label {
    display: none;
  }
}

</style>
