<template>
  <div :class="isMobile?'userAgent':''">
    <div class="header">
      <div class="wapper">
        <a :href="'http://'+agent.bindUrl"><img :src="agent.loginLogo1" alt=""></a>
        <span class="version">{{version}}</span>
      </div>
    </div>
    <div class="content">
      <div class="wapper">
        <img :src="agent.loginLogo2" alt="" class="banner">
        <div class="form">
          <img :src="agent.loginLogo3" alt="" class="logo">
          <div class="f-init" v-if="page !=='login' && page !=='register' && page !=='mobile' && page !=='message' && page !== 'mobileLogin' && page !== 'bind'">
            <a @click="page='login'" href="javascript:;" class="f-btn">手机号／账号 登录</a>
            <a @click="page='register'" href="javascript:;" class="f-btn">注册</a>
            <div class="other">
              <div class="other-text">
                <span>其他登录方式</span>
              </div>
              <div class="other-icon">
                <a @click="wxLogin" href="javascript:;" class="f-icon" v-if="!isAppMobile"></a>
                <a @click="qqLogin" href="javascript:;" class="f-icon icon-wx" v-if="!isAppMobile"></a>
                <a @click="page='mobile'" href="javascript:;" class="f-icon icon-mobile"></a>
                <br/>
                <span v-if="!isAppMobile">微信</span><span v-if="!isAppMobile">QQ</span><span>手机</span>
              </div>
            </div>
          </div>
          <!-- 账号登录 -->
          <div class="f-login" v-if="page ==='login'">
            <div class="f-input">
              <input class="username" name="username" v-model="model.username" @keyup.enter="submit" type="text" placeholder="手机号／账号"/>
              <input class="username" name="subusername" v-model="model.subusername" @keyup.enter="submit" type="text" placeholder="请输入员工账号" v-if="checked"/>
              <div style="position: relative;">
                <input class="password username" name="password" v-model="model.password" @keyup.enter="submit" type="password" placeholder="请输入密码"/>
                <a class="forget-password" target="_blank" href="http://www.jihui88.com/member/forget_password.html">重设密码</a>
              </div>
              <div style="position: relative;">
                <input type="text" name="randCode" v-model="model.randCode" @keyup.enter="submit" placeholder="填写验证码">
                <img class="veriImg" :src="'/alphveriImg'+verifyPic"  @click="refreshCode"/>
              </div>
            </div>
            <div class="f-checkbox">
              <input type="checkbox" v-model="checked" hidden>
              <label @click="check">
                <span class="f-checkout-select" v-if="!checked"></span>
                <span class="f-checkout-select select" v-if="checked"></span>使用员工账号登录
              </label>
            </div>
            <button type="button" class="submit" @click="submit">登录</button>
          </div>
          <!-- 手机登录、注册 -->
          <div class="f-register" v-if="page ==='register'|| page ==='mobile'">
            <div class="f-input">
              <input :class="page==='register'?'username':''" name="phone" v-model="p.phone" @keyup.enter="registerMobile" type="text" placeholder="手机号"/>
              <span class="forget-password" style="color: red"  v-if="page ==='register'&&validate">该手机号已注册</span>
              <input class="password" name="password" v-model="p.password" @keyup.enter="registerMobile" type="password" placeholder="设置登录密码，不少于6位" v-if="page ==='register'"/>
            </div>
            <button type="button" class="submit" @click="registerMobile"><span v-if="bindType ==='cellphone'">确定</span><span v-else-if="page ==='mobile'">登录</span><span v-else-if="page ==='register'">注册</span></button>
          </div>
          <!-- 手机验证 -->
          <div class="f-message" v-if="page ==='message'||page ==='mobileLogin'||page ==='bind'">
            <div class="tip">
              为了安全，我们会向你的手机<br/>发送短信校验码
            </div>
            <input v-if="page === 'bind'" class="username" name="phone" v-model="p.phone" @keyup.enter="mobileSubmit" type="text" placeholder="请输入手机号" style="width: 228px;"/>
            <div v-if="!isCode">
              <input class="randCode" type="text" name="randCode" v-model="model.randCode" @keyup.enter="mobileSubmit" placeholder="图片验证码">
              <img class="veriImg" :src="'/alphveriImg'+verifyPic"  @click="refreshCode" />
              <span class="refresh-btn" @click="refreshCode" title="看不清？点击换一张"></span>
            </div>
            <div v-if="isCode">
              <input type="text" name="mobileCode" v-model="mobileCode" @keyup.enter="mobileSubmit" placeholder="短信验证码">
              <input class="mobileCode" @click="getCode" type="button" :value="countText">
            </div>

            <button type="button" class="submit" @click="getCode" v-if="!isCode" :style="model.randCode.length>3?'background: #ff6700;':''">发送手机短信验证码</button>
            <button type="button" class="submit" @click="mobileSubmit" v-if="isCode" :style="mobileCode.length>5?'background: #ff6700;':''">
              <span v-if="bindType ==='cellphone'">确定<span v-if="isAccount">绑定</span></span>
              <span v-else-if="page ==='mobileLogin'||page === 'bind'">登录</span>
              <span v-else-if="page ==='message'">注册</span>
            </button>
          </div>

          <a @click="init" href="javascript:;" class="back-other" v-if="page !=='init' && page !== 'weixin' && page !== 'qq' && !isAccount && host === 'www.jihui88.com'">
            返回<span v-if="page ==='login'||page ==='mobile'||page ==='mobileLogin'">其他</span>登录
          </a>
          <a @click="page = 'login'" href="javascript:;" class="back-other" v-if="agent.openCellphoneRegister === '01' && page === 'register'">登录</a>
          <a @click="page = 'register'" href="javascript:;" class="back-other" v-if="agent.openCellphoneRegister === '01' && page === 'login'">注册</a>
        </div>
        <div class="contact-service" v-if="agent.user.cellphone || agent.user.weixin || agent.user.qq || agent.user.phone">
          联系客服：
          <span v-if="agent.user.cellphone">手机（{{agent.user.cellphone}}）</span>
          <span v-if="agent.user.weixin">，微信（{{agent.user.weixin}}）</span>
          <span v-if="agent.user.qq">，QQ（{{agent.user.qq}}）</span>
          <span v-if="agent.user.phone">，座机（{{agent.user.phone}}）</span>
        </div>
      </div>
    </div>
    <div class="alert" v-if="page === 'qq' && !isMobile" v-drag>
      <div class="head">
        <span>QQ</span>登录<img src="http://www.jihui88.com/member/static/images/f-x.png" alt="" class="close" @click="page='init'">
      </div>
      <div class="container">
        <iframe :src="qqUrl"></iframe>
      </div>
    </div>
    <!-- 微信 -->
    <div class="oAuth__content" v-if="page === 'qq' && isMobile">
      <iframe v-if="page === 'qq'" :src="qqUrl"></iframe>
      <a @click="init" href="javascript:;" class="back-other" v-if="!isAccount">返回其他登录</a>
    </div>
  </div>
</template>
<script>
export default {
  data　() {
    return {
      model: {
        username: '',
        password: '',
        randCode: '',
        type: '0',
        redirectURL: '',
        appId: '',
        state: null,
        lanId: '1'
      },
      checked: false,
      verifyPic: '',
      // 用户绑定页面数据
      openid: '',
      type: '',
      oauthtype: '',
      // 手机
      state: '',
      isCode: false,
      mobileCode: '',
      backURL: '',
      countdown: 60,
      countText: '重新发送短信',
      // 注册
      page: 'init',
      p: {
        phone: '',
        password: ''
      },
      validate: false,
      // qq
      qqUrl: '',
      isAccount: false,
      isMobile: false, // 小于400的窗口
      isAppMobile: false, // 在手机上的窗口
      version: 'v3',
      websiteUlr: 'http://www.jihui88.com/manage_v3/',
      // 头像
      agent: {
        bindUrl: 'www.jihui88.com',
        user: {}
      },
      host: location.host
    }
  },
  beforeCreate() {
    if (location.host === 'www.jihui88.com' || location.host === 'localhost:8072') {
      document.getElementsByTagName("title")[0].innerText = '机汇网会员登录'
      document.querySelector("link[rel*='icon']").href = 'http://cps.jihui88.com/static/img/favicon.ico'
    }
  },
  created () {
    if (location.host !== 'www.jihui88.com') {
      this.getLogo()
      this.page = 'login'
    } else {
      this.agent.loginLogo1 = 'http://www.jihui88.com/member/static/images/logo2.jpg'
      this.agent.loginLogo2 = 'http://www.jihui88.com/member/static/images/bg.png'
      this.agent.loginLogo3 = 'http://www.jihui88.com/member/static/images/f-logo.png'
    }
    this.openid = this.getUrlParam('openid') || ''
    // 登录账号类型， 0为企业账号登录，1为员工账号登录
    this.type = this.getUrlParam('type') || ''
    // 授权类型
    this.oauthtype = this.getUrlParam('oauthtype') || ''
    // 后端控制的重定向（用于授权登录），前端无需关注
    this.model.redirectURL = this.getUrlParam('redirectURL') || ''
    if (location.pathname.indexOf('manage_v4') > -1) {
      this.version = 'v4'
      this.websiteUlr = location.origin + '/manage_v4/'
    }
    // 前端控制的返回页面, 从哪个子项目里进来的授权成功后将返回到backURL
    this.backURL = this.getUrlParam('backURL') ? (this.getUrlParam('backURL') + (location.hash ? location.hash : '')) : (this.websiteUlr + 'index.html#/')
    // 系统控制的授权返回地址
    this.oauthBackURL = this.getUrlParam('oauthBackURL') || (this.websiteUlr + 'qqRedirect.html')
    // 授权类型范围， 比如snsapi_login_pc   网页端授权， snsapi_login_quick  弹出框形式授权
    this.scope = this.getUrlParam('scope') ? this.getUrlParam('scope') : null
    // 系统分配
    this.appId = this.getUrlParam('appId')
    // 为某账号绑定第三方
    this.addBind = this.getUrlParam('addBind')
    // 绑定类型， 手机绑定为cellphone, 微信绑定为weixin ， QQ绑定为qq
    this.bindType = this.getUrlParam('bindType')
    // 注册来源
    this.model.domain = this.getUrlParam('domain')
    // 显示页面
    this.page = this.getUrlParam('page') || this.page
    // 注册
    this.p.phone = this.getUrlParam('phone') || ''
    this.p.password = this.getUrlParam('password') || ''

    if (this.scope === 'snsapi_login_quick'){
      this.isMobile = true
    }
    if(/Android|webOS|iPhone|iPod|BlackBerry/i.test(navigator.userAgent)) {
      this.isMobile = true
      this.isAppMobile = true
    }
    // 重新绑定
    if (this.getUrlParam('isAccount') === '1') {
      this.mobile()
      this.isAccount = true
    }
    // 解决后端请求不到带有#/hash的网址的相关参数问题
    if (this.backURL){
      this.backURL = this.backURL.replace('#', '@**@')
    }
  },
  mounted: function () {
    var ctx = this
    this.$nextTick(function () {
      if (this.bindType === 'weixin') {
        this.wxLogin()
      } else if (this.bindType === 'qq') {
        this.qqLogin()
      } else if (this.bindType === 'cellphone' && this.page !== 'mobile') {
        this.page = 'bind'
      }
      if (this.p.phone.length > 6) {
        this.isUseAccout()
      }
    })
    window.addEventListener('message', function (e) {
      var data = e.data || {}
      if (data.type === 1) {
        window.location.href = ctx.backURL ? ctx.backURL.replace('@**@', '#') : ctx.model.redirectURL ? (ctx.model.redirectURL + (ctx.model.redirectURL.indexOf('?') > -1 ? '&' : '?') + 'code=' + res.attributes.code + '&state=' + res.attributes.state) : (ctx.websiteUlr + 'index.html')
      }
      if (data.type === 'bind') {
        ctx.page = 'bind'
        window.parent.postMessage({type: 'init'}, '*')
      }
    }, false)
    window.onresize = () => {
      return (() => {
        if (document.body.clientWidth < 400 || ctx.scope === 'snsapi_login_quick') {
          ctx.isMobile = true
        } else {
          ctx.isMobile = false
        }
      })()
    }
  },
  directives: {
    drag: {
      inserted (el, bindings) {
        el.onmousedown = function(e){
          var disx = e.pageX - el.offsetLeft;
          var disy = e.pageY - el.offsetTop;
          document.onmousemove = function (e){
            el.style.left = e.pageX - disx + 400 +'px';
            el.style.top = e.pageY - disy + 300 +'px';
          }
          document.onmouseup = function(){
            document.onmousemove = document.onmouseup = null;
          }
        }
      }
    }
  },
  methods: {
    getLogo () {
      var ctx = this
      $.ajax({
        url: '/rest/api/agent/config/getConfigByDomain',
        data: {
          'domain': location.host
        },
        success: function(res) {
          if (res.attributes) {
            let data = res.attributes.data
            if (res.success && data) {
              if (data.loginLogo1) data.loginLogo1 = 'http://img.jihui88.com/' + data.loginLogo1
              if (data.loginLogo2) data.loginLogo2 = 'http://img.jihui88.com/' + data.loginLogo2
              if (data.loginLogo3) data.loginLogo3 = 'http://img.jihui88.com/' + data.loginLogo3
              ctx.agent = data
              // ico
              ctx.initIco(data.ico)
              ctx.initTitle(data.user.enterprise.name)
            } else {
              ctx.initIco()
              console.log('经销商:' + res.msg)
              setTimeout(function() {
                if (location.port !== '8072') window.location.href = 'http://www.jihui88.com/404'
              }, 5000)
            }
          }
        }
      })
    },
    initIco (ico) {
      var link = document.querySelector("link[rel*='icon']") || document.createElement('link')
      link.type = 'image/x-icon'
      link.rel = 'shortcut icon'
      link.href = ico ? ('http://img.jihui88.com/' + ico) : 'http://cps.jihui88.com/static/img/favicon.ico'
      document.getElementsByTagName('head')[0].appendChild(link)
    },
    initTitle (title) {
      if (title) {
        title = title + '会员登录'
        document.getElementsByTagName("title")[0].innerText = title
        document.querySelector("meta[name='generator']").content = title
        document.querySelector("meta[name='author']").content = title
        document.querySelector("meta[name='keywords']").content = title
        document.querySelector("meta[name='description']").content = title
      }
    },
    // 1.手机注册
    registerMobile () {
      var ctx = this
      let test = /^1[3|4|5|7|8][0-9]\d{4,8}$/
      if (!test.test(this.p.phone)) { return alert('不是有效的手机号码！') }
      if (this.page === 'register'){
        if (this.p.password.length < 6) { return alert('登录密码，不少于6位！') }
        // 判断手机是否注册过
        this.isUseAccout()
      } else {
        this.page = 'mobileLogin'
        this.mobile()
      }
      this.model.randCode = ''
    },
    isUseAccout () {
      var ctx = this
      $.ajax({
        url: '/rest/api/user/validate',
        data: {
          'vali-username': this.p.phone
        },
        success: function(res) {
          if (res.success) {
            ctx.page = 'message'
            ctx.validate = false
            ctx.mobile()
          } else {
            ctx.page = 'register'
            if(res.msg === '该用户已经存在'){
              ctx.validate = true
            } else {
              console.log(res.msg)
            }
          }
        }
      })
    },
    // 切换到手机，获取state
    mobile () {
      var ctx = this
      $.ajax({
        url: '/rest/api/user/oauth',
        data: {
          requestType: 'state', // 请求state(必填)
          redirectURL: ctx.model.redirectURL,
          scope: ctx.scope,
          appId: ctx.appId,
          addBind: ctx.addBind,
          backURL: ctx.backURL,   // 登录成功后的跳转地址
          quick: ctx.addBind ? '00' :'01',
          domain: ctx.model.domain
        },
        success: function(res) {
          if (res && res.success) {
            ctx.state = res.attributes.data + '_' + '0' + '_cellphone';
          }
        }
      })
    },
    // 获取验证验
    getCode () {
      var ctx = this
      if (this.model.randCode === '') { return alert('请输入图片验证码') }
      if (!this.isCode) {
        this.isCode = true
      }
      $.ajax({
        type: 'post',
        url: '/rest/api/user/sendCellphone',
        data: {
          agentId: this.agent.agentId,
          cellphone: this.p.phone,   // 手机号码(必填)
          state: this.state,    // 第一步里获取到的state参数(必填)
          randCode: this.model.randCode  // 图像验证码(必填)
        },
        success: function(res){
          if (res.success) {
            ctx.setTime()
          } else {
            if (res.msg !== '请60秒后再刷新') {
              ctx.isCode = false
            } else {
              ctx.countdown = 60
              ctx.refreshCode()
              ctx.model.randCode = ''
            }
            alert(res.msg)
          }
        }
      });
    },
    // 手机登录
    mobileSubmit () {
      var ctx = this
      let test = /^1[3|4|5|7|8][0-9]\d{4,8}$/
      if (this.validate) { return alert('该手机号已注册') }
      if (!test.test(this.p.phone)) { return alert('不是有效的手机号码！') }
      if (this.mobileCode === '') {
        return alert('请填写短信验证码')
      }
      $.ajax({
        url: '/rest/api/user/oauth',
        data: {
          state: this.state,    // 第一步里获取到的state参数(必填)
          code: this.mobileCode,    // 手机验证码(必填)
          redirectURL: this.model.redirectURL,
          scope: ctx.scope,
          quick: ctx.addBind ? '00': '01',
          addBind: ctx.addBind,
          appId: ctx.appId,
          backURL: this.backURL,  // 登录成功后的跳转地址
          username: this.p.phone,
          password: this.p.password,
          domain: this.model.domain,
          agentId: this.agent.agentId
        },
        success: function(res) {
          if (res && res.success) {
            if (res.attributes && res.attributes.data){
              window.location.href=res.attributes.data
              return
            }
            window.location.href = ctx.backURL ? ctx.backURL.replace('@**@', '#') : ctx.model.redirectURL ? (ctx.model.redirectURL + (ctx.model.redirectURL.indexOf('?') > -1 ? '&' : '?') + 'code=' + res.attributes.code + '&state=' + res.attributes.state ) : (ctx.websiteUlr + 'index.html')
          } else if (res && res.msg) {
            alert(res.msg)
            ctx.refreshCode()
          }
        }
      })
    },
    // 2.登录
    refreshCode () {
      this.verifyPic = '?time=' + new Date().getTime()
    },
    check () {
      this.checked = !this.checked
    },
    submit (e) {
      var ctx = this
      if (this.model.username === '') { return alert('请输入账号') }
      if (this.model.randCode === '') { return alert('请输入验证码') }
      if (this.model.password === '') { return alert('请输入密码') }
      if (this.checked) {this.model.type = '1'} else {this.model.type = '0', this.subusername = ''}
      var eTar = e.currentTarget
      $(e.currentTarget).html('登录中...')
      setTimeout(function() {
        $(eTar).html('登录')
      }, 5000)
      $.ajax({
        url: '/rest/api/user/login',
        type: 'post',
        data: {
          model: JSON.stringify(this.model)
        },
        success: function(res) {
          if (res.success) {
            // 判断是否存在backURL? [说明是普通跳转] : [是否存在redirectUrl? 是[[说明是授权登录，需要跳转到redirectURL地址，并带上code与state参数]] : 否 [[跳转到用户后台首页]] ]
            window.location.href =ctx.backURL? ctx.backURL.replace('@**@', '#') : ctx.model.redirectURL? (ctx.model.redirectURL+(ctx.model.redirectURL.indexOf('?') > -1? '&' : '?') + 'code=' + res.attributes.code + '&state=' + res.attributes.state ) : (ctx.websiteUlr + 'index.html')
          } else{
            alert(res.msg)
            ctx.refreshCode()
          }
          $(e.currentTarget).html('登录')
        }
      })
    },
    pcLogin (skey, callback) {
      var url = "http://pc.jihui88.com/pc/skey.html?skey=" + skey;
      if (this.host !== 'www.jihui88.com') {
        url = "http://pc.ykyh.com/pc/skey.html?skey=" + skey
      }
      var $iframe = $('<iframe src=""></iframe>')
      $('body').append($iframe)
      $iframe.attr('src', url)
      setTimeout(function() { $iframe.remove(); if (callback) callback.call(this) }, 1000)
    },
    // 微信登录
    wxLogin () {
      var ctx = this
      var newTab = window.open('about:blank');
      $.ajax({
        url: '/rest/api/user/oauth',
        data: {
          requestType: 'state',
          redirectURL: ctx.model.redirectURL,
          scope: ctx.scope,
          appId: ctx.appId,
          addBind: ctx.addBind,
          backURL: ctx.backURL,
          quick: ctx.addBind ? '00' :'01',
          domain: ctx.model.domain
        },
        success: function(res) {
          if (res.success) {
            var url = "https://open.weixin.qq.com/connect/qrconnect?appid=wx308c58370e47720c&redirect_uri="+encodeURIComponent('http://www.jihui88.com/rest/api/user/oauth?backURL=' + ctx.backURL + '&oauthBackURL=' + ctx.oauthBackURL)+
              "&response_type=code&scope=snsapi_login&state="+res.attributes.data + '_' + ctx.model.type + '_weixin'+"#wechat_redirect"
            if (newTab) {
              newTab.location.href = url;
            } else {
              window.open(url,'newwindow','height=600,width=1000,top=0,left=0,toolbar=no,menubar=no,scrollbars=no, resizable=no,location=no, status=no')
            }
          }
        }
      })
    },
    // qq登录
    qqLogin () {
      var ctx = this;
      this.page = 'qq'
      $.ajax({
        url: '/rest/api/user/oauth',
        data: {
          requestType: 'state',
          redirectURL: ctx.model.redirectURL,
          scope: ctx.scope,
          appId: ctx.appId,
          addBind: ctx.addBind,
          backURL: ctx.backURL,
          quick: ctx.addBind ? '00' :'01',
          domain: ctx.model.domain
        },
        success: function(res) {
          if (res.success) {
            var url = 'https://graph.qq.com/oauth/show?which=ConfirmPage&display=pc&client_id=101370473&response_type=code&state=' + res.attributes.data + '_' + ctx.model.type + '_qq' + "&scope=&display=&redirect_uri=" +
              encodeURIComponent("http://www.jihui88.com/rest/api/user/oauth?backURL=" + ctx.backURL + "&oauthBackURL=" + ctx.oauthBackURL)
              if (ctx.scope === 'snsapi_login_quick') {
                window.parent.postMessage({type: 'qq'}, '*')
              }
              ctx.qqUrl = url
          }
        }
      })
    },
    getUrlParam (name) {
      let url = location.search //获取url中"?"符后的字串
      if (url.indexOf("?") !== -1) {
        let str = url.substr(1)
        let strs = str.split("&")
        for(let i = 0; i < strs.length; i ++) {
          if (strs[i].split("=")[0] === name) {
            return strs[i].split("=")[1]
          }
        }
      }
    },
    setTime () {
      var ctx = this
      if (this.countdown == 0) {
        this.countText = '获取短信验证码'
        this.countdown = 60
        return false
      } else {
        if (this.countdown < 10) {
          this.countText = '00:0' + this.countdown
        } else {
          this.countText = '00:' + this.countdown
        }
        this.countdown --
        setTimeout(function() {
          ctx.setTime()
        },1000)
      }
    },
    init () {
      this.page = 'init'
      window.parent.postMessage({type: 'init'}, '*')
    }
  }
}
</script>
<style lang="less">
  body {
    background: #fff;
    margin: 0;
    font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", "\5FAE\8F6F\96C5\9ED1", Arial, sans-serif;
  }
  a {
    text-decoration: none;
  }
  img{
    border:none;
  }
  .userAgent{
    .header,.banner{
      display: none;
    }
    .content{
      background: #fff;
      .wapper{
        width: 100%;
        background: none;
      }
      .form{
        width: 100%;
        border: none;
        margin-top: 0;
      }
    }

  }
  .header{
    height: 85px;
    .wapper{
      width: 1000px;
      margin: 0 auto;
      position: relative;
      img{
        margin-top: 15px;
        max-height: 53px;
      }
      .version{
        color: #fff;
        font-size: 12px;
        background: #ff5b00;
        padding: 0px 4px;
        border-radius: 3px;
        position: absolute;
        top: 37px;
        margin-left: 10px;
      }
    }
  }
  .content{
    background: #202020;
    .wapper{
      height: 600px;
      width: 1000px;
      margin: 0 auto;
      position: relative;
    }
    .banner{
      margin: 85px 0 0 20px;
      max-width: 599px;
      max-height: 420px;
    }
    // 表单
    .form{
      width: 351px;
      height: 501px;
      box-sizing: border-box;
      border: 1px solid #b9b9b9;
      border-radius: 5px;
      background: #fff;
      float: right;
      margin-top: 28px;
      text-align: center;
      font-family: '微软雅黑';
      position: relative;
      .logo{
        margin: 82px auto 28px auto;
        max-height: 80px;
      }
      .f-btn {
        display: block;
        margin: 8px auto;
        width: 257px;
        height: 40px;
        line-height: 40px;
        border: 1px solid #ff7333;
        border-radius: 40px;
        color: #ff7333;
        font-size: 14px;
      }
      .other{
        padding-top: 80px;
      }
      .other-text{
        text-align: center;
        line-height: 30px;
        height: 30px;
        clear: both;position: relative;
        &:after {
          content: "";
          background: url(http://www.jihui88.com/member/static/images/f-border.png) no-repeat;
          width: 85px;
          height: 3px;
          margin: 14px 0 0 -175px;
          display: inline-block;
          position: absolute;
        }
        &:before {
          content: "";
          background: url(http://www.jihui88.com/member/static/images/f-border.png) no-repeat 0 -3px;
          width: 85px;
          height: 3px;
          margin: 14px 0 0 90px;
          display: inline-block;
          position: absolute;
        }
        span {
          width: 90px;
          display: inline-block;
          background: #fff;
          color: #5b5b5b;
          font-size: 12px;
          box-sizing: border-box;
          position: relative;
        }
      }
      .other-icon{
        padding-top: 5px;
        span{
          display: inline-block;
          width: 50px;
          color: #9b9b9b;
          font-size: 12px;
          margin: 0 5px;
        }
      }
      .f-icon{
        width: 39px;
        height: 39px;
        display: inline-block;
        margin: 0 8px;
        background: url(http://www.jihui88.com/member/static/images/f-icon.png) no-repeat;
      }
      .icon-wx{
        background-position: -39px 0;
      }
      .icon-mobile{
        background-position: -78px 0;
      }
    }

    .back-other:hover{
      text-decoration: underline;
    }
    .contact-service{
      position: absolute;
      bottom: 22px;
      right: 0;
      font-size: 12px;
      color: #ffff;
      opacity: 0.70;
      filter:alpha(opacity=70);
      font-family: 'Microsoft YaHei';
    }
  }
  .close{
    position: absolute;
    padding: 2px;
    right:11px;
    top:13px;
    cursor: pointer;
  }
  .back-other{
    color: #666666;
    font-size: 12px;
    position: absolute;
    bottom: 17px;
    width: 100px;
    margin-left: 125px;
    left: 0;
  }
  // 登录
  .f-login{
    width: 260px;
    margin: 0 auto;
    .f-checkbox{
      padding: 10px 0;
      text-align: left;
      font-size: 12px;
      color: #ccc;
    }
    .veriImg{
      position: absolute;
      right: 14px;
      top: 6px;
      width: 65px;
    }
    .f-checkout-select{
      width: 10px;height: 10px;display: inline-block; margin-right: 5px;
      background: url(http://www.jihui88.com/member/static/images/checkbox-selected.png) no-repeat;
    }
    .f-checkout-select.select{
      background-position: -10px 0;
    }
  }
  // 表单
  .f-input{
    width: 260px;
    box-sizing: border-box;
    border: 1px solid #d9d9d9;
    border-radius: 5px;
    margin: 0 auto;
    position: relative;
    input{
      border-radius: 5px;
      border:none;
      width:228px;
      height: 17px;
      padding: 10px 15px;
      outline: 0 none;
    }
    .username{
      border-bottom: 1px solid #d9d9d9;
      border-bottom-right-radius: 0;
      border-bottom-left-radius: 0;
    }
    .forget-password{
      position: absolute;
      color: #ccc;
      font-size: 12px;
      right: 15px;
      top: 10px;
    }
  }
  .submit{
    background: #ff6700;
    border-radius: 5px;
    border: 0 none;
    cursor: pointer;
    display: block;
    width: 260px;
    margin: 0 auto;
    text-align: center;
    height: 38px;
    color: #fff;
    font-size: 14px;
    transition: all 0.3s ease 0s;
    -moz-transition: all 0.3s ease 0s;
    -webkit-transition: all 0.3s ease 0s;
    -o-transition: all 0.3s ease 0s;
  }
  .submit:hover {
  	background:#ff4800;
  }

  .f-register{
    .submit{
      margin-top: 15px;
    }
  }
  .f-message{
    text-align: left;
    width: 260px;
    margin: 0 auto;
    .tip{
      font-size: 12px;
      color: #666;
      text-align: center;
      padding-bottom: 10px;
    }
    .refresh-btn{
      float: right;
      margin-top: 16px;
      height: 25px;
      width: 16px;
      cursor: pointer;
      background: url(http://pc.jihui88.com/pc/styles/default/img/regVersion2016.png?v=201701061749) no-repeat -675px -650px;
      padding-right: 6px;
    }
    input{
      border: 1px solid #d9d9d9;
      padding: 10px 15px;
      width: 100px;
      outline: 0 none;
      margin-top: 10px;
      border-radius: 3px;
    }
    img{
      vertical-align: middle;height: 36px;
    }
    .mobileCode{
      height: 36px;
      float: right;width: 120px;cursor: pointer;background: #dedede;
      font-size: 12px;
      color: #666;
    }
    .submit{
      margin-top: 15px;
      background: #7b7b7b;
    }
    .submit:hover {
    	background:#ff6700;
    }
  }
  .alert {
    width: 800px;
    height: 600px;
    position: absolute;
    margin: -300px 0 0 -400px;
    left: 50%;
    top: 50%;
    background: #fafafa;
    z-index: 999;
    border-radius: 3px;
    box-shadow: 1px 1px 3px #aaa;
    .head{
      text-align: center;
      font-size: 12px;
      height: 39px;
      line-height: 39px;
      border-top-left-radius: 3px;
      border-top-right-radius: 3px;
      background: linear-gradient(#fff, #ddd);
    }
    .container{
      width: 786px;
      height: 554px;
      margin: 1px auto 0 auto;
      border: 2px solid #a5a5a5;
      background: #fff;
      #wxlogin_container{
        text-align: center;
      }
      iframe{
        width: 100%;height: 554px;border: none;
      }
    }
  }
  .oAuth__content{
    position: absolute;
    top: 0;
    width: 100%;
    background: #fff;
    text-align: center;
    z-index: 1000;
    height: 500px;
    #wxlogin_container2{
      margin-top: 30px;
    }
    iframe{
      width: 100%;height: 554px;border: none;
    }
  }
  input:-webkit-autofill,
  textarea:-webkit-autofill,
  select:-webkit-autofill {
    -webkit-box-shadow: 0 0 0 1000px white inset;
  }
</style>
