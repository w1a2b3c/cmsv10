<template>
  <div class="questions-container">
    <h1 class="page-title">试题列表</h1>

    <button @click="openAddQuestionModal" class="btn btn-primary">新增试题</button>

    <div v-if="questions.length === 0" class="no-questions">
      <p>没有找到试题数据</p>
    </div>
    <div v-else class="questions-list">
      <div v-for="(question, index) in questions" :key="index" class="question-item">
        <div class="question-header">
          <span class="question-index">问题 {{ index + 1 }}:</span>
          <p class="question-text">{{ question.question }}</p>
        </div>

        <!-- 显示选项和对应的分值 -->
        <ul class="options-list">
          <li v-for="(option, i) in formatOptions(question.options)" :key="i" class="option-item">
            <span>{{ option }} </span>
            <span v-html="'(分值: <span class=\'option-value\'>' + formatValue(question.val)[i] + '</span>)'"></span>
          </li>
        </ul>
        <p v-if="question.blank" class="fill-in">这是一个填空题。</p>
        <button @click="openEditQuestionModal(index)" class="btn btn-secondary">编辑</button>
      </div>
    </div>

    <div v-if="showModal" class="modal-overlay">
      <div class="modal-content">
        <h2>{{ isEdit ? "编辑试题" : "新增试题" }}</h2>
        <form @submit.prevent="submitQuestionForm">
          <label for="question">问题内容：</label>
          <input type="text" v-model="currentQuestion.question" id="question" required />

          <label for="options">选项：</label>
          <input type="text" v-model="currentQuestion.options" id="options" required placeholder="用空格分隔选项" />

          <label for="val">选项分值：</label>
          <input type="text" v-model="currentQuestion.val" id="val" required placeholder="用空格分隔分值" />

          <label for="status">状态：</label>
          <input type="checkbox" v-model="currentQuestion.status" id="status" />

          <label for="blank">填空题：</label>
          <input type="checkbox" v-model="currentQuestion.blank" id="blank" />

          <button type="submit">{{ isEdit ? "保存" : "新增" }}</button>
          <button type="button" @click="closeModal">取消</button>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import { addQuestion, updateQuestion } from "@/api/modules/question/index.ts";

export default {
  data() {
    return {
      questions: [], // 存储所有问题数据
      showModal: false, // 控制弹窗显示与隐藏
      isEdit: false, // 判断是否为编辑状态
      currentQuestion: {
        id: "",
        question: "",
        options: "",
        val: "",
        status: true,
        blank: false, // 新增的填空题字段
        testId: "", // 确保 testId 可以通过后端传递
        questionNumber: null
      }
    };
  },
  created() {
    const questionsParam = this.$route.query.questions;

    if (questionsParam) {
      try {
        this.questions = JSON.parse(decodeURIComponent(questionsParam));
      } catch (error) {
        console.error("解析试题数据失败:", error);
        alert("加载试题数据失败，请稍后重试");
      }
    }
  },
  methods: {
    formatOptions(options) {
      // 将选项字符串按照空格分隔成数组
      return typeof options === "string" ? options.split(" ") : [];
    },

    formatValue(val) {
      // 将分值字符串按照空格分隔成数组
      return val ? val.split(" ") : [];
    },

    openAddQuestionModal() {
      this.isEdit = false;
      this.currentQuestion = {
        id: "",
        question: "",
        options: "",
        val: "",
        status: true,
        blank: false, // 确保填空题字段为空
        testId: this.getTestIdFromUrl(), // 从 URL 或其他地方获取 testId
        questionNumber: 1 // 自动编号下一个问题
      };
      this.showModal = true;
    },

    openEditQuestionModal(index) {
      this.isEdit = true;
      this.currentQuestion = { ...this.questions[index], questionNumber: index + 1 };
      this.showModal = true;
    },

    getTestIdFromUrl() {
      // 假设 testId 是通过 URL 参数传递的，或者通过其他方式获取
      const testId = this.$route.query.testId || 12; // 默认 testId 为 12
      return testId;
    },

    async submitQuestionForm() {
      if (this.isEdit) {
        await this.updateQuestion();
      } else {
        await this.addQuestion();
      }
      this.closeModal();
    },

    async addQuestion() {
      try {
        // 确保 testId 不为 undefined 或 null
        if (!this.currentQuestion.testId) {
          console.error("未找到模块ID");
          alert("未找到模块ID");
          return;
        }

        console.log("发送到后端的 testId:", this.currentQuestion.testId); // 打印 testId 值

        // 格式化问题数据，确保按照要求的格式
        const questionData = {
          question: this.currentQuestion.question, // 问题内容
          testId: this.currentQuestion.testId, // 测试ID
          questionNumber: 1, // 问题编号
          options: this.currentQuestion.options, // 选项
          val: this.currentQuestion.val, // 答案的正确值
          status: this.currentQuestion.status // 问题状态
        };

        // 打印数据以便调试
        console.log("发送到后端的数据:", [questionData]);

        // 将数据包裹在数组中，确保后端可以正确解析
        const response = await addQuestion([questionData]); // 包裹成数组发送

        // 检查后端返回的结果
        if (response.data === "SUCCESS") {
          this.questions.push(questionData); // 添加新问题到列表
          alert("新增试题成功");
        } else {
          alert("新增试题失败，请稍后重试");
        }
      } catch (error) {
        alert("新增试题失败，请稍后重试");
        console.error(error);
      }
    },

    async updateQuestion() {
      try {
        // 校验 testId
        if (!this.currentQuestion.testId) {
          console.error("testId is missing");
          alert("testId is missing, please check.");
          return;
        }

        // 格式化当前问题数据，确保按照后端要求的格式
        const formattedCurrentQuestion = {
          question: this.currentQuestion.question, // 问题内容
          testId: this.currentQuestion.testId, // 测试ID
          questionNumber: this.currentQuestion.questionNumber, // 问题编号
          options: this.currentQuestion.options, // 选项（字符串）
          val: this.currentQuestion.val, // 答案的正确值（字符串）
          status: this.currentQuestion.status // 问题状态
        };

        // 打印格式化后的数据
        console.log("格式化后的数据:", formattedCurrentQuestion);

        // 构建更新后的问题列表，使用新的数据替换旧数据
        const updatedQuestions = this.questions.map(q =>
          q.questionNumber === this.currentQuestion.questionNumber ? formattedCurrentQuestion : q
        );

        // 打印完整数据以便调试
        console.log("发送到后端的完整数据:", updatedQuestions);

        // 调用 API 发送数据
        const response = await updateQuestion(updatedQuestions); // 传递完整问题列表
        console.log("API 响应:", response);

        // 根据响应结果更新前端状态
        if (response.data === "SUCCESS") {
          this.questions = updatedQuestions; // 用后端同步后的数据更新前端
          alert("试题修改成功");
        } else {
          console.warn("后端返回的非预期结果:", response);
          alert("修改试题失败，请稍后重试");
        }
      } catch (error) {
        alert("修改试题失败，请稍后重试");
        console.error("错误信息:", error);
      }
    },

    formatQuestionData(question) {
      // 确保 testId 不为 undefined
      if (!question.testId) {
        throw new Error("testId is missing");
      }

      // 格式化问题数据，符合规定格式
      return {
        id: question.id, // 只需要 id
        question: question.question,
        testId: question.testId, // 保证有 testId
        questionNumber: question.questionNumber,
        options: question.options.split(" "), // 选项分割
        val: question.val.split(" "), // 分值拆分
        status: question.status
      };
    },

    closeModal() {
      this.showModal = false; // 关闭模态框
    }
  }
};
</script>

<style scoped>
.questions-container {
  padding: 20px;
  background-color: #f9f9f9;
  min-height: 100vh;
  box-sizing: border-box;
  overflow: hidden;
}

.page-title {
  font-size: 24px;
  text-align: center;
  margin-bottom: 20px;
  font-weight: bold;
  color: #333;
}

.no-questions {
  text-align: center;
  font-size: 18px;
  color: #888;
}

.questions-list {
  max-height: 80vh;
  overflow-y: auto;
  padding-right: 10px;
}

.question-item {
  background-color: #fff;
  padding: 15px;
  margin-bottom: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.question-item:hover {
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
}

.question-header {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.question-index {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin-right: 10px;
}

.question-text {
  font-size: 16px;
  color: #555;
}

.options-list {
  list-style-type: none;
  padding: 0;
  margin: 10px 0;
}

.option-item {
  background-color: #f0f0f0;
  padding: 8px;
  border-radius: 4px;
  margin-bottom: 8px;
  font-size: 14px;
  color: #555;
}

.option-value {
  color: #007bff;
}

.fill-in {
  margin-top: 10px;
  color: #e91e63;
  font-style: italic;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  width: 400px;
}

.modal-content h2 {
  text-align: center;
  margin-bottom: 20px;
}

.modal-content form label {
  display: block;
  margin: 10px 0 5px;
}

.modal-content input {
  width: 100%;
  padding: 8px;
  margin-bottom: 15px;
  border-radius: 4px;
}

.modal-content button {
  width: 100%;
  padding: 10px;
  margin-top: 10px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
}

.modal-content button:hover {
  background-color: #0056b3;
}
</style>
