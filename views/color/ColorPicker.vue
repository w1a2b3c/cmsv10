<template>
  <div class="demo-color-block fixed-top-right">
    <el-color-picker v-model="color1" />
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from "vue";

const color1 = ref("#409EFF");
watch(color1, newColor => {
  document.body.style.backgroundColor = newColor;
});
</script>

<style>
/* 定位调色板到右上角 */
.fixed-top-right {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1000; /* 确保它位于其他内容之上 */
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1); /*添加阴影*/
  background-color: white; /* 背景色 */
  border-radius: 4px;
}

.demo-color-block {
  display: flex;
  align-items: center;
}
</style>
