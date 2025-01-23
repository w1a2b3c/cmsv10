<template>
  <div class="diary-container">
    <!-- 日记列表 -->
    <div class="diary-list">
      <ul>
        <li v-for="diary in filteredDiaries" :key="diary.id" class="diary-item">
          <h3>{{ diary.title }}</h3>
          <p>{{ diary.content ? diary.content.substring(0, 50) : "" }}...</p>
          <button @click="viewDiary(diary.id)">查看详情</button>
          <button @click="editDiary(diary)">编辑</button>
          <button @click="deleteDiary(diary.id)">删除</button>
        </li>
      </ul>
    </div>

    <!-- 创建日记按钮 -->
    <div class="create-diary">
      <button @click="openCreateDialog">创建日记</button>
    </div>

    <!-- 编辑日记模态框 -->
    <div v-if="showEditDialog" class="edit-dialog">
      <h3>编辑日记</h3>
      <input v-model="currentDiary.title" placeholder="日记标题" />
      <textarea v-model="currentDiary.content" placeholder="日记内容"></textarea>
      <button @click="saveDiary">保存</button>
      <button @click="closeDialog">取消</button>
    </div>

    <!-- 创建日记模态框 -->
    <div v-if="showCreateDialog" class="create-dialog">
      <h3>创建日记</h3>
      <input v-model="newDiary.title" placeholder="日记标题" />
      <textarea v-model="newDiary.content" placeholder="日记内容"></textarea>
      <button @click="createDiary">创建</button>
      <button @click="closeDialog">取消</button>
    </div>
  </div>
</template>

<script>
import { getDiaries, createDiary, updateDiary, deleteDiary } from "@/api/modules/diary"; // 导入接口请求函数

export default {
  data() {
    return {
      diaries: [], // 日记列表
      newDiary: { title: "", content: "" }, // 新建日记的数据
      currentDiary: { id: null, title: "", content: "" }, // 当前编辑的日记
      showCreateDialog: false, // 是否显示创建日记的模态框
      showEditDialog: false // 是否显示编辑日记的模态框
    };
  },
  created() {
    this.loadDiaries(); // 初始化时加载日记
  },
  computed: {
    // 计算属性：返回过滤后的日记列表
    filteredDiaries() {
      return this.diaries.filter(diary => !diary.isDeleted); // 过滤掉已删除的日记
    }
  },
  methods: {
    // 获取当前用户的日记列表
    async loadDiaries() {
      try {
        const token = localStorage.getItem("token"); // 获取 JWT token
        console.log(token);
        const data = await getDiaries(token);
        console.log(data);
        this.diaries = data || []; // 确保数据是数组，如果为空则设为[]
      } catch (error) {
        console.error("加载日记失败：", error);
      }
    },

    // 查看日记详情
    viewDiary(diaryId) {
      this.$router.push(`/diary/${diaryId}`); // 跳转到详情页
    },

    // 打开创建日记的模态框
    openCreateDialog() {
      this.showCreateDialog = true;
    },

    // 创建日记
    async createDiary() {
      try {
        const token = localStorage.getItem("token"); // 获取 JWT token
        const response = await createDiary(this.newDiary, token);
        if (response === "日记创建成功") {
          this.loadDiaries(); // 重新加载日记列表
          this.closeDialog(); // 关闭模态框
        } else {
          console.error("日记创建失败");
        }
      } catch (error) {
        console.error("创建日记失败：", error);
      }
    },

    // 打开编辑日记的模态框
    editDiary(diary) {
      this.currentDiary = { ...diary }; // 设置当前编辑的日记数据
      this.showEditDialog = true; // 显示编辑模态框
    },

    // 保存编辑的日记
    async saveDiary() {
      try {
        const token = localStorage.getItem("token"); // 获取 JWT token
        const response = await updateDiary(this.currentDiary.id, this.currentDiary, token);
        if (response === "日记更新成功") {
          this.loadDiaries(); // 重新加载日记列表
          this.closeDialog(); // 关闭模态框
        } else {
          console.error("日记更新失败");
        }
      } catch (error) {
        console.error("保存日记失败：", error);
      }
    },

    // 删除日记
    async deleteDiary(diaryId) {
      try {
        const token = localStorage.getItem("token"); // 获取 JWT token
        const response = await deleteDiary(diaryId, token);
        if (response === "日记删除成功") {
          this.loadDiaries(); // 重新加载日记列表
        } else {
          console.error("日记删除失败");
        }
      } catch (error) {
        console.error("删除日记失败：", error);
      }
    },

    // 关闭模态框
    closeDialog() {
      this.showCreateDialog = false;
      this.showEditDialog = false;
      this.newDiary = { title: "", content: "" }; // 清空新建日记的数据
      this.currentDiary = { id: null, title: "", content: "" }; // 清空当前编辑的日记数据
    }
  }
};
</script>

<style scoped>
/* 样式与原来保持一致 */
.diary-item {
  border-bottom: 1px solid #ddd;
  padding: 10px 0;
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

.create-dialog,
.edit-dialog {
  position: fixed;
  top: 20%;
  left: 50%;
  transform: translateX(-50%);
  background-color: white;
  padding: 20px;
  border: 1px solid #ddd;
  z-index: 100;
}

.create-dialog input,
.edit-dialog input,
.create-dialog textarea,
.edit-dialog textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
}

.create-dialog button,
.edit-dialog button {
  background-color: #28a745;
}

.create-dialog button:hover,
.edit-dialog button:hover {
  background-color: #218838;
}
</style>
