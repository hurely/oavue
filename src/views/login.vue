<template>
    <Row type="flex" justify="center" align="middle" class="login" @keydown.enter.native="submitLogin">
        <Col :xs="{span:22}" style="width: 368px;">
            <Row class="header">
                <img src="../images/xboot.png" width="220px"/>
                <div class="description">KELINGTON</div>
            </Row>
            <Alert type="error" show-icon v-if="error">{{errorMsg}}</Alert>
            <Row class="login-form">

             <Form ref="usernameLoginForm" :model="form" :rules="rules" class="form">
                <FormItem prop="username">
                    <Input v-model="form.username" size="large" clearable placeholder="请输入用户名">
                        <span slot="prepend">
                            <Icon :size="16" type="person"></Icon>
                        </span>
                    </Input>
                </FormItem>
                <FormItem prop="password">
                    <Input type="password" v-model="form.password" size="large" clearable  placeholder="请输入密码">
                        <span slot="prepend">
                            <Icon :size="14" type="locked"></Icon>
                        </span>
                    </Input>
                </FormItem>
            </Form>




                <Row type="flex" justify="space-between" class="code-row-bg">
                    <Checkbox v-model="saveLogin" size="large">自动登录</Checkbox>
                    <router-link to="/regist"><a class="forget-pass">注册账户</a></router-link>
                </Row>

                <Row>
                    <Button class="login-btn" type="primary" size="large" :loading="loading" @click="submitLogin" long>
                        <span v-if="!loading">登录</span>
                        <span v-else>登录中...</span>
                    </Button>
                </Row>

            </Row>


        </Col>
    </Row>
</template>

<script>
import Cookies from "js-cookie";
import { setStore } from "@/utils/storage";
import router from "@/router/index";
import util from "@/libs/util.js";
export default {
  data() {
    const validateMobile = (rule, value, callback) => {
      var reg = /^[1][3,4,5,7,8][0-9]{9}$/;
      if (!reg.test(value)) {
        callback(new Error("手机号格式错误"));
      } else {
        callback();
      }
    };
    return {
      error: false,
      errorMsg: "",
      saveLogin: true,
      loading: false,
      sended: false,
      count: 60,
      countButton: "60 s",
      maxLength: 4,
      errorCode: "",
      form: {
        username: "",
        password: "",
        mobile: "",
        verifyCode: ""
      },
      rules: {
        username: [
          { required: true, message: "账号不能为空", trigger: "blur" }
        ],
        password: [
          { required: true, message: "密码不能为空", trigger: "blur" }
        ],
        mobile: [
          { required: true, message: "手机号不能为空", trigger: "blur" },
          { validator: validateMobile, trigger: "blur" }
        ]
      }
    };
  },
  methods: {
    showErrorMsg(msg) {
      this.error = true;
      this.errorMsg = msg;
    },
    sendVerify() {
      this.$refs.mobileLoginForm.validate(valid => {
        if (valid) {
          this.sended = true;
          this.countDown();
        }
      });
    },
    countDown() {
      let that = this;
      if (this.count === 0) {
        this.sended = false;
        this.count = 60;
        return;
      } else {
        this.countButton = this.count + " s";
        this.count--;
      }
      setTimeout(function() {
        that.countDown();
      }, 1000);
    },
    submitLogin() {
        this.$refs.usernameLoginForm.validate(valid => {
          if (valid) {
            this.loading = true;
            this.postRequest("/login", {
              username: this.form.username,
              password: this.form.password,
              saveLogin: this.saveLogin
            }).then(res => {
              if (res.success === true) {
                setStore("accessToken", res.result);
                // 获取用户信息
                this.getRequest("/user/info").then(res => {
                  if (res.success === true) {
                    // 避免超过大小限制
                    delete res.result.permissions;
                    if (this.saveLogin) {
                      // 保存7天
                      Cookies.set("userInfo", JSON.stringify(res.result), {
                        expires: 7
                      });
                    } else {
                      Cookies.set("userInfo", JSON.stringify(res.result));
                    }
                    setStore("userInfo", res.result);
                    this.$store.commit("setAvatarPath", res.result.avatar);
                    // 加载菜单
                    util.initRouter(this);
                    this.$router.push({
                      name: "home_index"
                    });
                  } else {
                    this.loading = false;
                  }
                });
              } else {
                this.loading = false;
              }
            });
          }
        });

    }
  }
};
</script>

<style lang="less">
@import "./login.less";
</style>
