<template>
  <div id="app">
    <div>
      <el-form :model="form" label-width="auto" style="max-width: 600px">
        <el-form-item v-for="(item, index) in questions" :key="index" :label="item.question">
          <el-radio-group v-model="form[`resource${index}`]">
            <el-radio v-for="option in item.options" :key="option" :label="option">{{ option }}</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submit">提交</el-button>
          <el-button @click="goto('/')">返回</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script setup>
import { useRouter } from "vue-router";
import service from "@/api/request.js";
import { ref, onMounted, reactive } from "vue";
const questions = ref([]);
// const ans = ref([]);
// const val = ref([]);
const fetchQuestion = async () => {
  const res = await service.get("question/all/12");
  console.log(res);
  // 处理每个问题的选项字符串，将其分割成数组
  questions.value = res.data.map(item => ({
    ...item,
    options: splitOptions(item.options),
    correctAnswer: parseCorrectAnswer(item.val)
  }));
  questions.value.map(item => {
    let q = 0;
    for (let i of item.correctAnswer) {
      if (i == 1) {
        if (q == 0) item.answer = "A";
        else if (q == 1) item.answer = "B";
        else if (q == 2) item.answer = "C";
        else if (q == 3) item.answer = "D";
      }
      q++;
    }
  });
};

// 自定义的分割函数，不使用正则表达式
const splitOptions = optionsString => {
  if (optionsString.includes(",")) {
    return optionsString.split(",").map(option => option.trim());
  } else if (optionsString.includes(" ")) {
    return optionsString.split(" ").map(option => option.trim());
  } else {
    return [optionsString]; // 如果既没有逗号也没有空格，直接返回原字符串作为单个选项
  }
};
const parseCorrectAnswer = valString => {
  console.log(valString.split(" ").map(Number));
  return valString.split(" ").map(Number);
};

const router = useRouter();

function goto(url) {
  router.push(url);
}

onMounted(() => {
  fetchQuestion();
});

const form = reactive({
  name: "",
  region: "",
  date1: "",
  date2: "",
  delivery: false,
  type: [],
  resource: "",
  desc: ""
});
const getIndex = option => {
  if (option == "A") return 0;
  else if (option == "B") return 1;
  else if (option == "C") return 2;
  else if (option == "D") return 3;
};
const submit = () => {
  const ans = [];
  for (let i = 0; i < questions.value.length; i++) {
    //selectedOptionIndex是选项的内容
    const selectedOptionIndex = form[`resource${i}`];
    console.log(selectedOptionIndex);
    let yourAns = selectedOptionIndex.split(".")[0];
    console.log(yourAns);
    // if(yourAns==questions.value[i].answer){
    //   ans.push(1)
    // }else{
    //   ans.push(0)
    // }

    ans.push(questions.value[i].correctAnswer[getIndex(yourAns)]);

    // if (selectedOptionIndex !== undefined &&
    //  selectedOptionIndex == questions.value[i].correctAnswer[selectedOptionIndex]) {
    //   ans.push(questions.value[i].correctAnswer[selectedOptionIndex]);
    // } else {
    //   ans.push(0);
    // }
  }
  const lc = {
    testId: 12,
    ans
  };
  console.log("Ans", ans);
  try {
    service.post("question/student/doTest", lc).then(response => {
      console.log("Response from server:", response);
      alert("Your score has been submitted!");
    });
  } catch (error) {
    console.error("Error submitting score:", error);
  }
};
</script>

<style scoped>
#app {
  display: flex; /* 使用 flexbox 布局 */
  justify-content: center; /* 水平居中 */
  height: 100vh; /* 高度设置为视口高度 */
  margin-top: 50px; /* 设置上边距 */
}
</style>
