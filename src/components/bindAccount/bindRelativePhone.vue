<template>
  <div class="page">
    <div class="bindPhone flex1 overflow-y-auto overflow-touch">
      <div class="bindPhoneCenter">
        <div class="bigMiddle">
          <p>
            <img src="../../../static/img/填写手机号.png" alt="">
            <span>请先填写您的手机号</span>
          </p>
        </div>
        <div class="aboutNumber">
          <div class="formContent" ref="formContent">
            <div class="form phone border-1px">
              <label for="" class="phoneLabel"> <img src="../../../static/img/手机号.png" alt=""> </label>
              <input type="number" placeholder="请输入手机号" @focus="focus()" @blur="blur()"
                     v-model.trim="phone"
                     @input="$v.phone.$touch()" class="numberInput">
            </div>
            <div class="form verifyCode border-1px">
              <label for="" class="codeLabel"> <img src="../../../static/img/验证码.png" alt=""> </label>
              <input type="number" @focus="focus()" @blur="blur()" placeholder="请输入验证码" class="codeInput"
                     v-model="code">
              <span @click="getCode" v-if="countdown == 60 || countdown == 0">获取验证码</span>
              <span v-else>{{ countdown }}s后重新获取</span>
            </div>
          </div>
          <div class="buttonWrap">
            <button class="bottom" @touchend="verifyCode()">这是我的手机号</button>
          </div>
        </div>
      </div>
      <div class="verifyCenter">
        <verify v-if="showVerify" :verifyTips="verifyTips"></verify>
      </div>
    </div>
    <msg ref="msg"></msg>
  </div>
</template>
<script>
  import Msg from '../../plugins/msg'
  import header from '../../base/header'
  import api from '../../lib/api.js'
  import mask from '../../base/mask'
  import verify from '../../base/verify'
  import weuijs from 'weui.js'
  import {openidCache} from "../../lib/cache";
  import {getENV} from "../../lib/util";
  import {isBindMixin, isLoginMixin, scrollIntoViewMixin} from "../../lib/mixin"
  import {between, minLength, required} from 'vuelidate/lib/validators'

  export default {
    mixins: [isLoginMixin, scrollIntoViewMixin, isBindMixin],
    data() {
      return {
        phone: "",
        code: "",
        cid: "",
        codeValue: "",
        showMask: false,
        regStatus: "",
        showVerify: false,
        verifyTips: "手机号不能为空",
        countdown: 60,
        a: "",
        backPath: "",
        docId: ""
      }
    },
    validations: {
      phone: {
        required,
        minLength: minLength(4)
      },
      age: {
        between: between(20, 30)
      }
    },
    async created() {
      this.backPath = this.$route.query.backPath
      this.docId = this.$route.query.docId
      if (this.docId) {
        this.$router.replace({
          path: "/onlineDoctorCard",
          query: {docId: this.docId}
        });
      } else {
        let ret = await this._isBind();
        if (ret) {
          this.$router.replace({
            path: "/Profile",
          });
        }
      }
    },
    methods: {
      async bind() {
        let data = await api("nethos.pat.wechat.bind", {
          captcha: this.code,
          cid: this.cid,
          openId: openidCache.get()
        });
        if (data.code == 0) {
          this.$router.replace({
            path: '/login',
            query: {backPath: this.backPath}
          })
        } else {
          this.$refs.msg.show(data.msg || "接口错误" + data.code);
        }
      },
      async getCode() {
        if (this.phone == "") {
          this.$refs.msg.show('手机号不能为空');
          return
        }
        let loading = weuijs.loading("加载中...");
        let ret = await api("nethos.system.captcha.pat.wechat.bind", {
          mobile: this.phone,
        });
        if (ret.code == 0) {
          this.regStatus = ret.regStatus
          this.cid = ret.obj.cid
          this.codeValue = ret.obj.value;

          let env = getENV();
          (env.plat !== 'pro') && (this.codeValue) && (this.code = this.codeValue);

          this.a = setInterval(() => {
            this.countdown--
          }, 1000)
        } else {
          this.$refs.msg.show(ret.msg || "接口错误" + ret.code);
        }
        loading.hide();
      },
      async verifyCode() {
        if (this.phone == '') {
          this.$refs.msg.show("手机号不能为空");
        }
        else if (this.code == '') {
          this.$refs.msg.show("验证码不能为空");
        }
        else if (!this.cid) {
          this.$refs.msg.show("验证码不正确");
        }
        else {
          let ret = await api('nethos.system.captcha.checkcaptcha.v2', {
            captcha: this.code,
            cid: this.cid
          })
          if (ret.code != 0) {
            this.$refs.msg.show("验证码不正确");
            return false;
          }

          if (this.regStatus == 'REGISTER') {
            this.$router.replace({
              path: '/register',
              query: {cid: this.cid, codeValue: this.code, backPath: this.backPath}
            })
          } else if (this.regStatus == 'BIND') {
            this.bind()
          }
        }
      },
      focus() {
      },
      blur() {
      }
    },
    components: {
      "VHeader": header,
      "VMask": mask,
      verify, Msg
    },
    watch: {
      phone() {
        if (!this.$v.phone.required) {
          this.showVerify = true
          setTimeout(() => {
            this.showVerify = false
          }, 1000)
        }
      },
      countdown() {
        if (this.countdown == 0) {
          clearInterval(this.a)
          this.countdown = 60
        }
      }
    }
  }
</script>
<style scoped lang="scss">
  @import '../../common/public.scss';

  .page {
    flex-direction: column;
  }

  .bindPhone {
    .verifyCenter {
    }
    .bindPhoneCenter {
      width: 690rem/$rem;
      margin: 0 auto;
      .bigMiddle {
        margin-top: 100rem/$rem;
        font-size: 32rem/$rem;
        color: #333333;
        display: flex;
        align-items: center;
        justify-content: center;
        p {
          display: flex;
          flex-direction: column;
          align-items: center;
          img {
            width: 319rem/$rem;
            height: 246rem/$rem;
            margin-bottom: 85rem/$rem;
          }
          span {
            font-size: 50rem/$rem;
            color: #333333;
          }
        }
      }
      .verifyTips {
        margin-top: 130rem/$rem;
        position: relative;
        text-align: center;
      }
      .aboutNumber {
        background-color: white;
        border-radius: 7px;
        /*<!--margin-top: 18rem/$rem;-->*/
        /*<!--height:500rem/$rem;-->*/
        .formContent {
          width: 690rem/$rem;
          margin: 0 auto;
          margin-top: 10px;
          .form {
            width: 100%;
            height: 40px;
            line-height: 40px;
            display: flex;
            margin-top: 1px;
            label {
              width: 60px;
              padding-left: 1rem;
              /*<!--background-color: $bgColor2;-->*/
            }
            label.phoneLabel {
              border-top-left-radius: 7px;
              display: flex;
              align-items: center;
              justify-content: center;
              img {
                height: 50rem/$rem;
                width: 30rem/$rem;
              }
            }
            label.codeLabel {
              border-bottom-left-radius: 7px;
              display: flex;
              align-items: center;
              justify-content: center;
              img {
                height: 45rem/$rem;
                width: 40rem/$rem;
              }
            }
            input {
              flex: 1;
              text-align: left;
              border: none;
              outline: medium;
              font-size: 32rem/$rem;
              color: #999999;
              /*<!--background-color: $bgColor2;-->*/
            }
            input.codeInput {
              border-bottom-right-radius: 7px;
            }
            p {
              flex: 1;
              font-size: 32rem/$rem;
              color: #999999;
              /*<!--background-color: $bgColor2;-->*/
              border-top-right-radius: 7px;
            }
          }
          .phone {
            /*margin-top: 20px;*/
          }
          .verifyCode {
            position: relative;
            height: 109rem/$rem
          }
          .verifyCode span {
            position: absolute;
            display: inline-block;
            top: 30rem/$rem;
            text-align: center;
            right: 0;
            height: 57rem/$rem;
            line-height: 57rem/$rem;
            width: 190rem/$rem;
            font-size: 26rem/$rem;
            border: 1px solid #3Dccc2;
            border-radius: 5px;
            color: #3Dccc2;
          }
        }
        .buttonWrap {
          width: 690rem/$rem;
          height: 90rem/$rem;
          margin-top: 69rem/$rem;
          text-align: center;
        }
        button.bottom {
          width: 650rem/$rem;
          height: 90rem/$rem;
          border: none;
          outline: medium;
          border-radius: 22px;
          color: white;
          font-size: 36rem/$rem;
          background-color: $buttonColor;
        }
      }
    }
  }
</style>
