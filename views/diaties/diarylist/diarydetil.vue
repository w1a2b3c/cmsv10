<template>
  <div class="diary-detail" v-if="diary">
    <h2>{{ diary.title }}</h2>
    <p>{{ diary.content }}</p>
    <button @click="goBack">返回列表</button>
  </div>
</template>

<script>
import { fetchDiaryDetail } from "http://localhost:1314";

export default {
  data() {
    return {
      diary: null
    };
  },
  created() {
    this.loadDiaryDetail();
  },
  methods: {
    async loadDiaryDetail() {
      const diaryId = this.$route.params.diaryId;
      try {
        this.diary = await fetchDiaryDetail(diaryId);
      } catch (error) {
        console.error("加载日记详情失败：", error);
      }
    },
    goBack() {
      this.$router.push("/diaries");
    }
  }
};
</script>

<style scoped>
.diary-detail {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}
button {
  color: #fff;
  background-color: #007bff;
  border: none;
  padding: 5px 10px;
  border-radius: 4px;
}
button:hover {
  background-color: #0056b3;
}
</style>
