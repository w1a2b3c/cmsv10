<template>
  <div class="register_page">
    <div class="register_form_box">
      <a-form :rules="rules" :model="form" layout="vertical" @submit="onSubmit">
        <!-- 账号输入框 -->
        <a-form-item field="number" :hide-asterisk="true">
          <a-input v-model="form.number" allow-clear placeholder="请输入账号">
            <template #prefix>
              <icon-user />
            </template>
          </a-input>
        </a-form-item>

        <!-- 邮箱输入框 -->
        <a-form-item field="email" :hide-asterisk="true">
          <a-input v-model="form.email" allow-clear placeholder="请输入电子邮件">
            <template #prefix>
              <icon-mail />
            </template>
          </a-input>
        </a-form-item>

        <!-- 获取验证码按钮 -->
        <a-form-item>
          <a-button type="default" @click="getCaptcha" :disabled="isCaptchaRequested">
            {{ isCaptchaRequested ? countdown + "秒后重发" : "获取验证码" }}
          </a-button>
        </a-form-item>

        <!-- 邮箱验证码输入框 -->
        <a-form-item field="emailCaptcha" :hide-asterisk="true">
          <a-input v-model="form.emailCaptcha" allow-clear placeholder="请输入邮箱验证码" />
        </a-form-item>

        <!-- 密码输入框 -->
        <a-form-item field="pwd" :hide-asterisk="true">
          <a-input-password v-model="form.pwd" allow-clear placeholder="请输入密码">
            <template #prefix>
              <icon-lock />
            </template>
          </a-input-password>
        </a-form-item>

        <!-- 确认密码输入框 -->
        <a-form-item field="confirmPwd" :hide-asterisk="true">
          <a-input-password v-model="form.confirmPwd" allow-clear placeholder="确认密码">
            <template #prefix>
              <icon-lock />
            </template>
          </a-input-password>
        </a-form-item>

        <!-- 用户角色选择 -->
        <a-form-item field="role_id" :hide-asterisk="true">
          <a-radio-group v-model="form.role_id">
            <a-radio :value="0">管理员</a-radio>
            <a-radio :value="1">学生</a-radio>
          </a-radio-group>
        </a-form-item>

        <a-form-item>
          <a-button long type="primary" html-type="submit" class="register-button">注册</a-button>
        </a-form-item>
      </a-form>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { useRouter } from "vue-router";
import { Message } from "@arco-design/web-vue";
import { registerAPI, getCaptchaAPI } from "@/api/modules/user/index";

const router = useRouter();

// 表单数据
const form = ref({
  number: "",
  email: "",
  emailCaptcha: "",
  pwd: "",
  confirmPwd: "",
  role_id: 1 // 默认角色为学生
});

// 表单校验规则
const rules = ref({
  number: [{ required: true, message: "请输入账号" }],
  email: [
    { required: true, message: "请输入电子邮件" },
    { type: "email", message: "请输入有效的电子邮件地址" }
  ],
  emailCaptcha: [{ required: true, message: "请输入邮箱验证码" }],
  pwd: [{ required: true, message: "请输入密码" }],
  confirmPwd: [
    { required: true, message: "请确认密码" },
    {
      validator: (value: string, cb: any) => {
        if (value !== form.value.pwd) {
          cb("两次密码输入不一致");
        } else {
          cb();
        }
      }
    }
  ],
  role_id: [{ required: true, message: "请选择角色" }]
});

// 验证码请求状态
const isCaptchaRequested = ref(false);
const countdown = ref(60);

// 获取验证码
const getCaptcha = async () => {
  const { number, email } = form.value;

  if (!number || !email) {
    Message.error("账号和邮箱不能为空");
    return;
  }

  if (isCaptchaRequested.value) return; // 防止重复请求

  try {
    isCaptchaRequested.value = true; // 加锁防止重复请求
    const res = await getCaptchaAPI({ number, email });

    if (res.success) {
      Message.success("验证码已发送到邮箱！");
      countdown.value = 60;

      const interval = setInterval(() => {
        countdown.value -= 1;
        if (countdown.value <= 0) {
          clearInterval(interval);
          isCaptchaRequested.value = false;
        }
      }, 1000);
    } else {
      isCaptchaRequested.value = false;
      Message.error(res.data || "验证码请求失败");
    }
  } catch (error: unknown) {
    isCaptchaRequested.value = false;
    if (error instanceof Error) {
      Message.error(`错误：${error.message}`);
    } else {
      Message.error("请求失败，请稍后重试");
    }
  }
};

// 提交表单
const onSubmit = async (formData: { errors: any }) => {
  if (formData.errors) return;

  try {
    const res = await registerAPI({
      number: form.value.number,
      email: form.value.email,
      emailCaptcha: form.value.emailCaptcha,
      pwd: form.value.pwd,
      roleId: form.value.role_id
    });

    if (res) {
      localStorage.setItem("role_id", form.value.role_id.toString());
      Message.success(res.message || "注册成功");
      router.push("/login");
    } else {
      Message.error("注册失败");
    }
  } catch (error: unknown) {
    if (error instanceof Error) {
      Message.error(`错误：${error.message}`);
    } else {
      Message.error("请求失败，请稍后重试");
    }
  }
};
</script>

<style lang="scss" scoped>
.register_page {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f5f5f5;
  font-family: Arial, sans-serif;

  .register_form_box {
    background-color: #fff;
    padding: 40px;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    width: 100%;
    max-width: 400px;

    .a-form-item {
      margin-bottom: 24px;
    }

    .register-button {
      width: 100%;
      font-size: 16px;
      padding: 10px;
      border-radius: 4px;
      background-color: #1890ff;
      color: white;
      font-weight: bold;

      &:hover {
        background-color: #40a9ff;
      }
    }
  }
}
</style>
