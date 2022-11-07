<template>
  <div id="login-content">
    <div id='login'>
      <div class="title">
        <div class="div-pic"></div>
        <div class="div-title">专线管理</div>

      </div>

      <!--<div class="sec_left">
				<div class="logo"></div>
				<div class="logo_text"><b>智能专线管理</b></div>
			</div>-->

      <div class="form" v-loading='loading'>
        <div class="name">
          <i class="icon iconfont icon-user"></i>
          <span style="text-indent: .5em;">|</span>
          <input type="text" placeholder="请输入用户名" v-model="userinfo.userName" maxlength="20" @keyup.13='submitFun(userinfo.userName,userinfo.passWord)' />

        </div>
        <div class="password">
          <i class="icon iconfont icon-mima"></i>
          <span style="text-indent: .5em;">|</span>
          <input type="password" placeholder="请输入密码" v-model="userinfo.passWord" maxlength="20" @keyup.13='submitFun(userinfo.userName,userinfo.passWord)' />
        </div>
        <div class="btn" @click="submitFun(userinfo.userName,userinfo.passWord)">登 录</div>
        <!--<div class="logo_div" @click="ssoClickFUn">	</div>-->
        <div v-if="showMsg" class="showMsg">
          <span style="color:#65d4ff;">已绑定iMC：{{bindIMC}}</span>
          <span @click="toSetIMC" style="color:#03edff;cursor:pointer;">iMC配置向导</span>
        </div>
      </div>
    </div>
  </div>

</template>

<script>
	var CryptoJS = require("crypto-js");
	var key =    CryptoJS.enc.Base64.parse('c2tsaGRmbHNqZnNkZ2RlZw==');
	var iv =    CryptoJS.enc.Base64.parse('Y2Zic2RmZ3NkZnhjY3ZkMQ==');
	function encrypt (msg, key, iv) {
				return  CryptoJS.AES.encrypt(CryptoJS.enc.Utf8.parse(msg),  key, {
						iv: iv,
						padding: CryptoJS.pad.Pkcs7,
						mode: CryptoJS.mode.CBC
				}).toString();
		}
			
	
export default {
  name: 'login',
  data() {
    return {
      showMsg: false,
      loginMsg: '',
      bindIMC: '',
      loading: false,
      userinfo: {
        userName: '',
        passWord: ''
      }
    }
  },
  methods: {
    toSetIMC: function () {
      this.$router.push('/imcBoard');
    },
    getiMCInfo: function () {
      this.$http.get(this.$apiPrifixUri + "/lhm/paramconfig/queryImcParams", { "noNeedLogin": true }).then(res => {
        if (res.body.imcIp != "") {
          this.bindIMC = res.body.imcIp;
        } else {
          this.bindIMC = "无";
        }
      })
    },
    ssoClickFUn: function () {
      //window.location.href='http://10.10.242.14:8000/login?scope=read&response_type=code&user_oauth_approval=true&client_id=portal&redirect_uri=http://10.10.240.119:8080/portal/ssoLogin&state=123&client_uri=http://10.10.242.14:8888/portal';
      //this.$apiPrifixUri
      //window.location.href=this.$ssoFontUri + '/item'+ '?scope=read&response_type=code&user_oauth_approval=true&client_id=portal&redirect_uri='+this.$redUri+'&state=123&client_uri='+this.$cliUri;
      window.location.href = "http://10.88.30.54:8090/item?scope=read&response_type=code&user_oauth_approval=true&client_id=iPLM&redirect_uri=http://10.88.30.126:8866/portal/lhm/sso/ssoLogin&state=123&client_uri=http://10.88.30.126:8888/login";
    },
    _init() {
      /**
       * 加上回车登陆
       */
      let me = this;
      window.addEventListener('keypress', function (event) {
        if (event.keyCode == 13) {
          //this.hasRegister=true;
          this.submitFun(me.userinfo.userName, me.userinfo.passWord);
        }
      }.bind(this))
    },
    submitFun(username, pwd,type) {
      let url = ''
      if(type){
        url = '/lhm/paramconfig/ssologin'
      }else{
        url = '/lhm/paramconfig/login'
      }
      this.loading = true;
      this.$http.post(this.$apiPrifixUri + url, {
        "param": JSON.stringify({
          "imc_username": username,
          "imc_password":   encrypt(pwd, key, iv)
        })
      }).then((res) => {
		let reg = /^(?=.*\d)(?=.*[a-zA-Z])(?=.*[!_\-.@#$%^&*])[a-zA-Z!_\-.@#$%^&*0-9]{6,16}$/;
		sessionStorage.setItem("isNormalPwd",reg.test(pwd)?1:0)
        this.loading = false;
        //var resData = res.body.message;

        if (res.body.code == '1') {
          if ((this.$router.currentRoute.query.userName) && (this.$router.currentRoute.query.pswd)) {
          
          }else if(type){
          
          }else{
            this.$message({
              type: 'success',
              message: res.body.message,
              showClose: true
            })
          }
          
          this.userinfo.userName = '';
          this.userinfo.passWord = '';
          this.$cookie.set('X-ACCESS-TOKEN', res.body.sessionId);
          this.$cookie.set('userName', res.body.userName, Infinity);
          this.$cookie.set('passWord', res.body.passWord, Infinity);
          if (res.body.privilege) {
            let privilege = res.body.privilege.map(item => (
              item.description
            ))
            this.$cookie.set('privilege', JSON.stringify(privilege), Infinity);
          } else {
            this.$cookie.set('privilege', JSON.stringify(["告警", "专线资费", "资费统计", "报表", "设备导入", "设备删除", "专线添加", "专线导入", "专线导出", "专线删除", "参数配置", "拓扑查看", "拓扑修改"]));
          }
          //this.$router.go(0);
          // this.$router.repalce(0);
          this.$router.push('/ds');
          // this.$router.replace('/ds');
        } else if (res.body.code == '0') {
          this.$message.warning(res.body.message);
          if(!type){
            this.showMsg = true;
          this.loginMsg = "已绑定iMC"
          }
          
        }
      })
    },
    validateLogin: function (accessToken) {
      this.$http({
        method: 'POST',
        url: 'http://10.88.30.54:8078/sso/oauth/token?'
          + "access_token=" + accessToken,
        headers: { 'Content-type': 'application/x-www-form-urlencoded' }
      }).then((tokenResponse) => {
        console.log(tokenResponse);
        if (tokenResponse.data.code == "299") {
          this.$cookie.set('X-ACCESS-TOKEN', tokenResponse.data.data.access_token);
          this.$cookie.set('userName', 'admin');
          this.$cookie.set('passWord', encrypt('admin', key, iv));
          this.$router.push('/ds');
          this.$message({
            type: 'success',
            message: tokenResponse.data.msg,
            showClose: true
          })
        }
      });
    }
  },
  mounted() {
    this.getiMCInfo();
    //			this._init();
    var access_token = this.$route.query.access_token;
    console.log('rout', access_token);
    console.log('u', this.$route);
    if (typeof (access_token) != "undefined" && access_token != null && access_token.trim() != "") {
      this.validateLogin(access_token);
      this.hasRegister = true;
    }
    if ((this.$router.currentRoute.query.userName) && (this.$router.currentRoute.query.pswd)) {
      this.submitFun(this.$router.currentRoute.query.userName, this.$router.currentRoute.query.pswd)
    }
    if (window.opener && this.$router.currentRoute.query.user) {
      this.submitFun(this.$router.currentRoute.query.user,null,1)
    }
  }

}
</script>

<style scoped="scoped">
#login-content {
  background: url(../../assets/img/l.png) no-repeat;
  background-size: cover;
  width: 100%;
  height: 100%;
  z-index: 2;
}
#login {
  width: 4.74rem;
  height: 4.15rem;
  background: rgba(2 59 95 0.5);
  box-shadow: inset 0 0 1.2rem rgba(3, 199, 248, 0.3);
  position: absolute;
  top: 50%;
  left: 70%;
  transform: translate(-50%, -50%);
}
.logo_div {
  position: absolute;
  top: 3.4rem;
  right: 0.5rem;
  width: 0.4rem;
  height: 0.4rem;
  background: url(../../assets/img/logoa.png);
  background-size: contain;
  cursor: pointer;
}
.title {
  color: #fff;
  font-size: 20px;
  text-align: left;
  height: 0.7rem;
  line-height: 0.7rem;
  border-bottom: 1px solid;
  border-image: linear-gradient(to right, #131c2f, #fff, #131c2f) 15 15;
}
.title .div-pic {
  float: left;
  margin: 0.2rem 0 0.2rem 0.2rem;
  width: 0.3rem;
  height: 0.3rem;
  background: url(../../assets/img/logo.png) no-repeat center;
  background-size: cover;
}
.title .div-title {
  float: left;
  line-height: 0.7rem;
  padding-left: 0.2rem;
}
.sec_left {
  position: absolute;
  top: 50%;
  left: -75%;
  transform: translate(-50%, -50%);
  width: 6.8rem;
  height: 6.8rem;
  background: url(../../assets/img/logo_b.png) no-repeat;
  background-size: contain;
}

.sec_left .logo {
  width: 0.67rem;
  height: 0.67rem;
  background: url(../../assets/img/logoa.png) no-repeat;
  background-size: contain;
  position: absolute;
  left: 3.09rem;
  top: 2.5rem;
  animation: move 5s infinite linear;
}

@keyframes move {
  from {
    transform: rotate(0);
  }
  to {
    transform: rotate(360deg);
  }
}

.sec_left .logo_text {
  font-size: 0.4rem;
  color: white;
  text-align: center;
  position: absolute;
  left: 2.17rem;
  top: 3.55rem;
}

.sec_left .logo_text b {
  font-weight: normal;
  color: #00f4ff;
}

.form {
  color: #fff;
  font-size: 20px;
}

.form .name {
  width: 70%;
  height: 0.47rem;
  background: rgba(255, 255, 255, 0.06);
  border-radius: 0.47rem;
  margin: 0.4rem auto 0;

  /*display: flex;
		justify-content: space-around;
		align-items: center;*/
}
.form .name i {
  float: left;
  width: 0.4rem;
  height: 0.4rem;
  text-align: center;
  line-height: 0.4rem;
  font-size: 0.3rem;
  margin: 0.03rem 0 0 0.2rem;
  color: #37a0f3;
}
.form .name span {
  width: 0.4rem;
  line-height: 0.4rem;
  text-align: left;
  color: white;
  float: left;
  color: #37a0f3;
}

.form .name input {
  float: left;
  background: transparent;
  outline: none;
  border: none;
  width: 2.2rem;
  height: 0.4rem;
  color: white;
  font-size: 0.16rem;
  ::-webkit-input-placeholdercolor: rgba(255, 255, 255, 0.2);
}

.form .password {
  width: 70%;
  height: 0.47rem;
  background: rgba(255, 255, 255, 0.06);
  border-radius: 0.47rem;
  margin: 0.2rem auto 0;
}
.form .password i {
  float: left;
  width: 0.4rem;
  height: 0.4rem;
  color: #37a0f3;
  text-align: center;
  line-height: 0.4rem;
  font-size: 0.2rem;
  margin: 0.03rem 0 0 0.2rem;
}
.form .password span {
  width: 0.4rem;
  line-height: 0.4rem;
  text-align: left;
  color: white;
  float: left;
  color: #37a0f3;
}

.form .password input {
  float: left;
  background: transparent;
  outline: none;
  width: 2.2rem;
  height: 0.4rem;
  font-size: 0.16rem;
  border: none;
  color: white;
  ::-webkit-input-placeholdercolor: #878888;
}

.form .btn {
  background: #03a5b2;
  color: white;
  height: 0.5rem;
  width: 70%;
  line-height: 0.5rem;
  text-align: center;
  margin: 0.4rem auto 0;
  border-radius: 0.5rem;
  cursor: pointer;
}
.showMsg {
  width: 70%;
  height: 0.5rem;
  margin: 0.2rem auto 0;
  display: flex;
  justify-content: space-between;
}

.showMsg span {
  font-size: 0.14rem;
}
</style>
