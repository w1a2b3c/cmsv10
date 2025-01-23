<template>
  <div>
    <div class="login_form_box">
      <a-form :rules="rules" :model="form" layout="vertical" @submit="onSubmit">
        <a-form-item field="number" :hide-asterisk="true">
          <a-input v-model="form.number" allow-clear placeholder="请输入账号">
            <template #prefix>
              <icon-user />
            </template>
          </a-input>
        </a-form-item>
        <a-form-item field="pwd" :hide-asterisk="true">
          <a-input-password v-model="form.pwd" allow-clear placeholder="请输入密码">
            <template #prefix>
              <icon-lock />
            </template>
          </a-input-password>
        </a-form-item>
        <a-form-item field="verifyCode" :hide-asterisk="true">
          <div class="verifyCode">
            <a-input style="width: 160px" v-model="form.verifyCode" allow-clear placeholder="请输入验证码" />
            <VerifyCode :content-height="30" :font-size-max="30" :content-width="110" @verify-code-change="verifyCodeChange" />
          </div>
        </a-form-item>
        <a-form-item field="remember">
          <div class="remember">
            <a-checkbox v-model="form.remember">记住密码</a-checkbox>
            <div class="forgot-password">忘记密码</div>
          </div>
        </a-form-item>
        <a-form-item>
          <a-button long type="primary" html-type="submit">登录</a-button>
        </a-form-item>
      </a-form>
    </div>
    <!-- 注册账号部分，添加点击事件 -->
    <div class="register" @click="goToRegister">注册账号</div>
  </div>
</template>

<script setup lang="ts">
import { Message } from "@arco-design/web-vue";
import { useRouter } from "vue-router";
import { loginAPI } from "@/api/modules/user/index";
import { useUserInfoStore } from "@/store/modules/user-info";

// 路由
const router = useRouter();

// 获取 Pinia store 实例
const userStore = useUserInfoStore();

// 表单数据
const form = ref({
  number: "",
  pwd: "",
  verifyCode: "",
  remember: true
});

const goToRegister = () => {
  router.push("../register"); // 例如从当前页面相对路径跳转到注册页面
};

// 表单校验规则
const rules = ref({
  number: [
    {
      required: true,
      message: "请输入账号"
    }
  ],
  pwd: [
    {
      required: true,
      message: "请输入密码"
    }
  ],
  verifyCode: [
    {
      required: true,
      message: "请输入验证码"
    },
    {
      validator: (value: string, cb: any) => {
        if (value !== verifyCode.value) {
          cb("请输入正确的验证码");
        } else {
          cb();
        }
      }
    }
  ]
});

// 验证码处理
const verifyCode = ref("");
const verifyCodeChange = (code: string) => (verifyCode.value = code);

// 登录提交函数
const onSubmit = async (formData: { errors: any }) => {
  if (formData.errors) return;

  try {
    const res = await loginAPI({
      number: form.value.number,
      pwd: form.value.pwd
    });

    if (res.success && "token" in res) {
      Message.success("登录成功");
      const token = res.token;
      if (typeof token === "string") {
        await userStore.setToken(token);
        localStorage.setItem("token", token);

        router.replace("/home");
      }
    } else {
      Message.error(res.message || "登录失败");
    }
  } catch (error: unknown) {
    if (error instanceof Error) {
      Message.error(`错误：${error.message}`);
    } else {
      Message.error("请求失败，请稍后重试");
    }
  }
};

// 页面加载时检查是否已有token
onMounted(() => {
  const token = localStorage.getItem("token");
  if (token) {
    router.replace("/home");
  }
});
</script>

<style lang="scss" scoped>
.login_form_box {
  margin-top: 28px;
  .verifyCode {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
  }
  .remember {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    .forgot-password {
      color: $color-primary;
      cursor: pointer;
    }
  }
}
.register {
  font-size: $font-size-body-1;
  color: $color-text-3;
  text-align: center;
  cursor: pointer;
}
</style>
