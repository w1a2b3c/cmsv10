<template>
  <div class="doctor-registration">
    <h1>医生注册页面</h1>
    <form @submit.prevent="handleSubmit">
      <div>
        <label for="background">背景信息:</label>
        <textarea v-model="formData.background" id="background" required></textarea>
      </div>
      <div>
        <label for="title">职称:</label>
        <input type="text" v-model="formData.title" id="title" required />
      </div>
      <div>
        <label for="gender">性别:</label>
        <select v-model="formData.gender" id="gender" required>
          <option value="male">男</option>
          <option value="female">女</option>
          <option value="other">其他</option>
        </select>
      </div>
      <div>
        <label for="rating">评分 (1-5):</label>
        <input type="number" v-model.number="formData.rating" id="rating" min="1" max="5" required />
      </div>
      <div>
        <label for="specializedFields">专业领域:</label>
        <textarea v-model="formData.specializedFields" id="specializedFields" required></textarea>
      </div>
      <div>
        <label for="otherInformation">其他信息:</label>
        <textarea v-model="formData.otherInformation" id="otherInformation" required></textarea>
      </div>
      <div>
        <label for="photo">照片URL:</label>
        <input type="text" v-model="formData.photo" id="photo" required />
      </div>
      <button type="submit">注册</button>
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      formData: {
        background: "",
        title: "",
        gender: "",
        rating: "",
        specializedFields: "",
        otherInformation: "",
        photo: ""
      }
    };
  },
  methods: {
    async handleSubmit() {
      try {
        // 发送POST请求到API端点
        const response = await fetch("http://localhost:1314/doctor/teacher", {
          method: "POST",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify(this.formData)
        });

        // 检查响应状态
        if (!response.ok) {
          throw new Error("网络响应不是ok " + response.statusText);
        }

        // 解析JSON响应（假设服务器返回JSON）
        const data = await response.json();

        // 根据响应数据做相应的处理
        console.log("成功:", data);
        alert("医生注册信息已成功提交！");
      } catch (error) {
        console.error("提交失败:", error);
        alert("提交医生注册信息时出错，请稍后再试。");
      }
    }
  }
};
</script>

<style scoped>
.doctor-registration {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
}

.doctor-registration label {
  display: block;
  margin-bottom: 8px;
}

.doctor-registration input,
.doctor-registration textarea {
  width: 100%;
  padding: 8px;
  margin-bottom: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.doctor-registration button {
  width: 100%;
  padding: 10px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.doctor-registration button:hover {
  background-color: #369a6f;
}
</style>
