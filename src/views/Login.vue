<template>
  <div class="login">
    <div class="login-box">
      <van-nav-bar @click-right="goPage('Home')">
        <template #left>
          <div class="left-text">
            <div class="logo">
              <img class="auto-img" src="../assets/images/home_active.png" alt="">
            </div>
            <div class="logo-title">Milk tea shop</div>
          </div>
        </template>
        <template #right>
          <div class="close-box">先逛一逛</div>
        </template>
      </van-nav-bar>

      <div class="login-form">
        <div class="title">欢迎回来!</div>
        <div class="en-title">Please login to your accounts</div>

        <div class="login-form-box">

          <van-form>
            <van-field
              label="手机号"
              placeholder="手机号"
              v-model="userLoginInfo.phone"
              left-icon="phone-o"
              maxlength="11"
            />
            <van-field
              :type="isPassword ? 'password' : 'text'"
              label="密码"
              placeholder="密码"
              v-model="userLoginInfo.password"
              left-icon="lock"
              :right-icon="isPassword ? 'closed-eye' : 'eye-o'"
              @click-right-icon="viewPassword"
            />
            <div class="clearfix forgot">
              <span class="fr" @click="goPage('Forgot')">忘记密码?</span>
            </div>
            <div class="login-btn">
              <van-button round block native-type="submit" color="#0C34BA" @click="login">登&nbsp;&nbsp;录</van-button>
            </div>
            <div class="login-btn">
              <van-button round block type="default" @click="openRegister">注&nbsp;&nbsp;册</van-button>
            </div>
          </van-form>

        </div>

      </div>
    </div>

    <van-popup class="popup-box" closeable v-model="isRegister" position="bottom">
      <div class="popup-item">
        <div class="register-text">注册</div>
        <van-form>
            <van-field
              label="手机号"
              placeholder="11位手机号"
              v-model="userRegisterInfo.phone"
              left-icon="phone-o"
              maxlength="11"
            />
            <van-field
              label="昵称"
              placeholder="1-10位昵称"
              v-model="userRegisterInfo.nickName"
              left-icon="manager-o"
            />
            <van-field
              :type="isPassword ? 'password' : 'text'"
              label="密码"
              placeholder="6-16位密码且以字母开头"
              v-model="userRegisterInfo.password"
              left-icon="phone-o"
              :right-icon="isPassword ? 'closed-eye' : 'eye-o'"
              @click-right-icon="viewPassword"
            />
            <div class="login-btn">
              <van-button round block color="#0C34BA" native-type="submit" @click="register">注&nbsp;&nbsp;册</van-button>
            </div>
          </van-form>
      </div>
    </van-popup>

  </div>
</template>

<script>
import "../assets/less/login.less"

import {validForm} from '../assets/js/validForm'

import {utils} from '../assets/js/utils'

export default {

  data() {
    return {

      userLoginInfo: {
        phone: '',
        password: ''
      },

      userRegisterInfo: {
        phone: '',
        nickName: '',
        password: ''
      },

      isPassword: true,
      isRegister: false
    };
  },

  methods: {
    // 跳转忘记密码
    goPage(name) {
      this.$router.push({name});
    },

    //查看密码
    viewPassword() {
      this.isPassword = !this.isPassword;
    },

    //弹出注册面板
    openRegister() {
      this.isRegister = true;
    },

    //注册
    register() {
      //获取用户注册信息
      let userInfo = this.userRegisterInfo;
      

      //验证注册表单
      let o = {
        phone: {
          value: userInfo.phone,
          reg: /^1[3-9]\d{9}$/,
          errorMsg: '手机号格式不正确'
        },
        nickName: {
          value: userInfo.nickName,
          reg: /^[\w\u4e00-\u9fa5]{1,10}$/,
          errorMsg: '昵称格式不正确'
        },
        password: {
          value: userInfo.password,
          reg: /^[A-Za-z]\w{5,15}$/,
          errorMsg: '密码格式不正确'
        }
      };

      if (!validForm.valid(o)) {
        //验证不通过
        return;
      }

      //发起请求
      //开启加载提示
      this.$toast.loading({
        message: '加载中...',
        //防止穿透
        forbidClick: true,

        //显示时间, 如果时间为0，则不会自动关闭
        duration: 0,

        loadingType: 'spinner'
      })

      //序列化参数
      //复制对象
      let info = Object.assign({}, userInfo);
      info.appkey = this.appkey;
      let data = utils.queryString(info);
      

      this.axios({
        method: 'POST',
        //请求参数
        url: '/register',
        data
      }).then(result => {
        //关闭加载提示
        this.$toast.clear();
        

        //如果注册成功
        if (result.data.code == 100) {
          this.isRegister = false;
          //清空注册表单的数据
          for (let key in userInfo) {
            userInfo[key] = '';
          }
        } else {
          //如果注册失败,则提示
          this.$notify({
            type: 'warning', 
            message: result.data.msg
          })
        }
      }).catch(err => {
         this.$toast.clear();
        
      })
    },

    //登录
    login() {
      let userInfo = this.userLoginInfo;

      let o = {
        phone: {
          value: userInfo.phone,
          reg: /^1[3-9]\d{9}$/,
          errorMsg: '手机格式不正确'
        },
        password: {
          value: userInfo.password,
          reg: /^[A-Za-z]\w{5,15}$/,
          errorMsg: '密码格式不正确'
        }
      };

      //验证表单
      if (!validForm.valid(o)) {
        return;
      }

      //验证通过
      this.$toast.loading({
        message: '加载中...',
        forbidClick: true,
        duration: 0,
        loadingType: 'spinner'
      })

      //复制userInfo
      let info = Object.assign({}, userInfo);
      info.appkey = this.appkey;

      //参数序列化
      let data = utils.queryString(info);

      this.axios({
        method: 'POST',
        url: '/login',
        data
      }).then(result => {
        this.$toast.clear();
        
        if (result.data.code == 200) {
          localStorage.setItem('CSTK', result.data.token);
          this.$router.push({name: 'Home'});
        } else {
          this.$notify({
            type: 'warning', 
            message: result.data.msg
          })
        }
      }).catch(err => {
        this.$toast.clear();
        

      })
      

    }
  }
};
</script>