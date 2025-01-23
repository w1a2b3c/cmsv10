<template>
  <div class="module-list">
    <h2>试题模块管理</h2>

    <!-- 添加模块按钮 -->
    <button @click="showAddModuleForm" class="add-module-button">添加模块</button>

    <!-- 显示试题模块列表 -->
    <div class="module-container">
      <div v-for="module in modules" :key="module.id" class="module-item">
        <div class="module-image-container">
          <img :src="module.src" alt="模块图片" class="module-image" />
        </div>
        <div class="module-info">
          <h3>{{ module.title }}</h3>
          <p class="module-introductions">{{ truncateIntroduction(module.introductions) }}</p>

          <!-- 始终显示的查看详情按钮，且颜色不同 -->
          <button @click="viewDetails(module)" class="view-details-button">查看详情</button>

          <!-- 新增“查看试题”按钮 -->
          <button @click="viewQuestions(module)" class="view-questions-button">查看试题</button>

          <div v-if="module.status" class="module-status">
            <p>模块状态：启用</p>
          </div>
        </div>
        <!-- 操作按钮放到模块项的右侧 -->
        <div class="module-actions">
          <button @click="editModule(module)" class="edit-button">编辑</button>
          <button @click="deleteModule(module.id)" class="delete-button">删除</button>
        </div>
      </div>
    </div>

    <div class="pagination">
      <button @click="prevPage">上一页</button>
      <span>第 {{ pageNo }} 页，共 {{ totalPages }} 页</span>
      <button @click="nextPage">下一页</button>
    </div>

    <!-- 弹出框: 添加/编辑模块表单 -->
    <div v-if="showForm" class="modal">
      <div class="modal-content">
        <h3>{{ formType === "edit" ? "编辑模块" : "添加模块" }}</h3>
        <input v-model="formModule.title" placeholder="模块标题" />
        <input v-model="formModule.introductions" placeholder="模块介绍" />
        <input v-model="formModule.src" placeholder="模块图片链接" />
        <textarea v-model="formModule.resultDescription" placeholder="测试结果描述"></textarea>
        <label>
          <input type="checkbox" v-model="formModule.status" />
          启用此模块
        </label>
        <button @click="submitForm">{{ formType === "edit" ? "更新模块" : "提交模块" }}</button>
        <button @click="closeForm" class="cancel-button">取消</button>
      </div>
    </div>

    <!-- 弹出框: 查看详情 -->
    <div v-if="showDetails" class="modal">
      <div class="modal-content">
        <h3>{{ details.title }}</h3>
        <p>{{ details.introductions }}</p>
        <p>{{ details.resultDescription }}</p>
        <button @click="closeDetails" class="cancel-button">关闭</button>
      </div>
    </div>
  </div>
</template>

<script>
import { fetchModules, addModule, updateModule, deleteModule, fetchQuestions } from "@/api/modules/module/index.ts"; // 导入接口函数

export default {
  data() {
    return {
      modules: [], // 存储试题模块数据
      pageNo: 1, // 当前页码
      pageSize: 6, // 每页显示条数
      totalPages: 1, // 总页数
      showForm: false, // 是否显示表单弹出框
      showDetails: false, // 是否显示查看详情弹出框
      formType: "", // 'add' 或 'edit'，表示添加或编辑操作
      formModule: { title: "", introductions: "", resultDescription: "", src: "", status: false, id: null }, // 表单模块数据
      details: {} // 存储查看详情的模块数据
    };
  },
  created() {
    this.fetchModulesList(); // 初始化加载模块列表
  },
  methods: {
    // 获取试题模块列表
    async fetchModulesList() {
      try {
        const token = localStorage.getItem("token"); // 从本地存储获取 token
        if (!token) {
          alert("未登录，请先登录");
          this.$router.push({ name: "login" });
          return;
        }

        const { totalPages, modules } = await fetchModules(this.pageNo, this.pageSize);
        this.modules = modules;
        this.totalPages = totalPages;
      } catch (error) {
        console.error(error);
        alert(error.message || "请求失败，请稍后再试");
      }
    },

    // 上一页
    prevPage() {
      if (this.pageNo > 1) {
        this.pageNo -= 1;
        this.fetchModulesList();
      } else {
        alert("已经是第一页了");
      }
    },

    // 下一页
    nextPage() {
      if (this.pageNo < this.totalPages) {
        this.pageNo += 1;
        this.fetchModulesList();
      } else {
        alert("已经是最后一页了");
      }
    },

    // 显示添加模块表单
    showAddModuleForm() {
      this.formType = "add";
      this.formModule = { title: "", introductions: "", resultDescription: "", src: "", status: false, id: null };
      this.showForm = true;
    },

    // 显示编辑模块表单
    editModule(module) {
      this.formType = "edit";
      this.formModule = { ...module }; // 克隆当前模块数据到编辑对象
      this.showForm = true;
    },

    // 关闭表单弹出框
    closeForm() {
      this.showForm = false;
      this.formModule = { title: "", introductions: "", resultDescription: "", src: "", status: false, id: null };
    },

    // 提交表单（添加或更新模块）
    async submitForm() {
      try {
        const token = localStorage.getItem("token"); // 从本地存储获取 token
        if (!token) {
          alert("未登录，请先登录");
          return;
        }

        if (this.formType === "add") {
          await addModule(this.formModule);
          this.fetchModulesList(); // 刷新模块列表
        } else if (this.formType === "edit") {
          await updateModule(this.formModule);
        }
        this.fetchModulesList(); // 刷新模块列表
        this.closeForm(); // 关闭表单弹出框
      } catch (error) {
        console.error(error);
        alert(error.message || "操作失败");
      }
    },

    // 删除模块
    async deleteModule(moduleId) {
      try {
        const response = await deleteModule(moduleId);
        if (response) {
          this.fetchModulesList(); // 刷新模块列表
        }
      } catch (error) {
        console.error(error);
        alert(error.message || "删除模块失败");
      }
    },

    // 截取简介内容
    truncateIntroduction(introduction) {
      if (introduction.length > 100) {
        return introduction.substring(0, 100) + "...";
      }
      return introduction;
    },

    // 查看详情
    viewDetails(module) {
      this.details = module;
      this.showDetails = true;
    },

    // 关闭查看详情弹出框
    closeDetails() {
      this.showDetails = false;
    },

    // 查看试题
    async viewQuestions(module) {
      const token = localStorage.getItem("token"); // 获取用户的 token

      if (!token) {
        alert("未登录，请先登录");
        return;
      }

      try {
        const questions = await fetchQuestions(module.id); // 获取试题数据

        // 跳转到试题页面，并传递试题数据
        this.$router.push({
          name: "questions",
          params: { moduleId: module.id },
          query: { questions: encodeURIComponent(JSON.stringify(questions)) } // 安全地传递查询参数
        });
      } catch (error) {
        console.error("获取试题失败:", error);
        alert("获取试题失败，请稍后重试"); // 显示用户友好的错误提示
      }
    }
  }
};
</script>

<style scoped>
/* 保证页面可上下滑动 */
.module-list {
  overflow-y: auto;
  max-height: 80vh; /* 控制内容区域的最大高度，超过时出现滚动条 */
}

/* 弹出框样式 */
.modal {
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
  background-color: #f0f8ff; /* 浅蓝色背景 */
  padding: 20px;
  border-radius: 12px;
  width: 450px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  color: #333;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

h2 {
  font-size: 24px;
  margin-bottom: 20px;
}

.add-module-button {
  padding: 10px 20px;
  background-color: #5ca0d3; /* 浅蓝色 */
  color: white;
  border: none;
  cursor: pointer;
  margin-bottom: 20px;
  border-radius: 5px;
  font-weight: bold;
}

.add-module-button:hover {
  background-color: #4292bb;
}

.module-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(47%, 1fr)); /* 每行两个模块 */
  gap: 20px;
}

.module-item {
  border: 1px solid #ccc;
  padding: 15px;
  background-color: #f9f9f9;
  border-radius: 10px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease-in-out;
  display: flex;
}

.module-item:hover {
  transform: scale(1.05);
}

.module-image-container {
  flex: 0 0 80px;
  margin-right: 20px;
}

.module-image {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 8px;
}

.module-info {
  flex-grow: 1;
}

h3 {
  font-size: 18px;
  margin: 10px 0;
}

.module-introductions {
  font-size: 14px;
  color: #666;
}

.view-details-button {
  margin-top: 10px;
  padding: 8px 12px;
  background-color: #007bff; /* 蓝色按钮 */
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.view-details-button:hover {
  background-color: #0056b3;
}

.view-questions-button {
  margin-top: 10px;
  padding: 8px 12px;
  background-color: #28a745; /* 绿色按钮 */
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.view-questions-button:hover {
  background-color: #218838;
}

.module-actions {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
}

.edit-button,
.delete-button {
  padding: 8px 12px;
  margin-top: 10px;
  font-size: 14px;
}

.edit-button {
  background-color: #17a2b8;
  color: white;
}

.delete-button {
  background-color: #dc3545;
  color: white;
}

.edit-button:hover {
  background-color: #138496;
}

.delete-button:hover {
  background-color: #c82333;
}

.pagination {
  margin-top: 20px;
  text-align: center;
}

.pagination button {
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin: 0 10px;
}

.pagination button:disabled {
  background-color: #ccc;
}

.cancel-button {
  padding: 10px 20px;
  background-color: #f44336; /* 红色 */
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.cancel-button:hover {
  background-color: #d32f2f;
}
</style>
