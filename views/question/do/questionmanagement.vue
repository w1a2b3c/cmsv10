<template>
  <v-container>
    <!-- 新增试题 -->
    <v-row>
      <v-col cols="12" md="6">
        <v-card class="elevation-2">
          <v-card-title class="headline text-center">新增试题</v-card-title>
          <v-form @submit.prevent="addQuestions">
            <v-text-field
              v-for="(question, index) in questions"
              :key="index"
              v-model="question.question"
              label="试题标题"
              outlined
              dense
              required
            />
            <v-textarea
              v-for="(question, index) in questions"
              :key="index"
              v-model="question.options"
              label="选项（格式：A.hh B.ss C.dd）"
              outlined
              dense
              required
            />
            <v-text-field
              v-for="(question, index) in questions"
              :key="index"
              v-model="question.val"
              label="答案（格式：0 1 0）"
              outlined
              dense
              required
            />
            <v-checkbox
              v-for="(question, index) in questions"
              :key="index"
              v-model="question.status"
              label="启用此问题"
              class="mt-3"
            />
            <v-btn type="submit" color="primary" class="mt-4" block>提交试题</v-btn>
          </v-form>
        </v-card>
      </v-col>
    </v-row>

    <!-- 查询试题 -->
    <v-row>
      <v-col cols="12" md="6">
        <v-card class="elevation-2">
          <v-card-title class="headline text-center">查询试题</v-card-title>
          <v-btn @click="getQuestions(moduleId)" color="success" block>查询试题</v-btn>
          <v-list class="mt-4">
            <v-list-item v-for="(question, index) in questionList" :key="index">
              <v-list-item-content>
                <v-list-item-title class="font-weight-bold">{{ question.question }}</v-list-item-title>
                <v-list-item-subtitle>{{ question.options }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list>
        </v-card>
      </v-col>
    </v-row>

    <!-- 批量修改试题 -->
    <v-row>
      <v-col cols="12" md="6">
        <v-card class="elevation-2">
          <v-card-title class="headline text-center">批量修改试题</v-card-title>
          <v-form @submit.prevent="updateQuestions">
            <v-text-field
              v-for="(question, index) in questionsToUpdate"
              :key="index"
              v-model="question.question"
              label="试题标题"
              outlined
              dense
              required
            />
            <v-textarea
              v-for="(question, index) in questionsToUpdate"
              :key="index"
              v-model="question.options"
              label="选项"
              outlined
              dense
              required
            />
            <v-text-field
              v-for="(question, index) in questionsToUpdate"
              :key="index"
              v-model="question.val"
              label="答案"
              outlined
              dense
              required
            />
            <v-checkbox
              v-for="(question, index) in questionsToUpdate"
              :key="index"
              v-model="question.status"
              label="启用此问题"
              class="mt-3"
            />
            <v-btn type="submit" color="primary" class="mt-4" block>提交更新</v-btn>
          </v-form>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { addQuestions, doTest, getQuestions, updateQuestions } from "@/api/modules/question";

interface Question {
  question: string;
  options: string;
  val: string;
  status: boolean;
}

export default defineComponent({
  name: "QuestionManagement",
  data() {
    return {
      questions: [{ question: "", options: "", val: "", status: true }] as Question[],
      questionList: [] as Question[],
      questionsToUpdate: [] as Question[],
      quizAnswers: [] as string[],
      moduleId: 12, // 模块ID
      token: "your-jwt-token" // 假设你有一个有效的JWT token
    };
  },
  methods: {
    async addQuestions() {
      try {
        const formattedQuestions = this.questions.map(q => ({
          question: q.question,
          testId: this.moduleId,
          questionNumber: this.questions.indexOf(q) + 1,
          options: q.options,
          val: q.val,
          status: q.status
        }));

        const response = await addQuestions(formattedQuestions, this.token);
        console.log("新增试题成功", response);
      } catch (error: any) {
        console.error("新增试题失败", error.message);
      }
    },

    async getQuestions(moduleId: number) {
      try {
        const response = await getQuestions(moduleId);
        this.questionList = response.data;
      } catch (error: any) {
        console.error("查询试题失败", error.message);
      }
    },

    async updateQuestions() {
      try {
        const formattedQuestions = this.questionsToUpdate.map(q => ({
          question: q.question,
          testId: this.moduleId,
          questionNumber: this.questionsToUpdate.indexOf(q) + 1,
          options: q.options,
          val: q.val,
          status: q.status
        }));

        const response = await updateQuestions(formattedQuestions, this.token);
        console.log("批量修改试题成功", response);
      } catch (error: any) {
        console.error("批量修改试题失败", error.message);
      }
    },
    async doTest() {
      try {
        const quizDTO = { testId: this.moduleId, answers: this.quizAnswers };
        const response = await doTest(quizDTO, this.token);
        console.log("做题成功", response);
      } catch (error: any) {
        console.error("做题失败", error.message);
      }
    }
  }
});
</script>

<style scoped>
/* 美化样式 */
.v-container {
  margin-top: 20px;
}

.v-card {
  background-color: #f4f7fb;
}

.v-btn {
  font-weight: bold;
  letter-spacing: 0.5px;
}

.v-text-field,
.v-textarea {
  margin-bottom: 15px;
}

.v-card-title {
  background-color: #6200ea;
  color: white;
  text-align: center;
  padding: 15px;
}

.v-list-item {
  background-color: #eaeaea;
  margin-bottom: 5px;
  border-radius: 5px;
}

.v-list-item-title {
  font-weight: 600;
}

.v-list-item-subtitle {
  font-size: 14px;
  color: #757575;
}
</style>
