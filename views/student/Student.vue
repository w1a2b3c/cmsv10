<template>
  <div class="student-management">
    <!-- 左侧学生目录 -->
    <div class="student-list">
      <div class="list-header">
        <h2>学生管理系统</h2>
        <!-- 搜索和操作按钮 -->
        <div class="search-box">
          <i class="search-icon">🔍</i>
          <input v-model="searchQuery" type="text" placeholder="搜索学生姓名" />
        </div>
        <div class="action-buttons">
          <div class="import-export-buttons">
            <button @click="handleImport" class="import-btn">导入数据</button>
            <button @click="exportToExcel" class="export-btn">导出数据</button>
          </div>
          <!-- 隐藏的文件输入框 -->
          <input type="file" ref="fileInput" style="display: none" accept=".xlsx, .xls" @change="onFileChange" />
        </div>
      </div>

      <!-- 学生列表 -->
      <div class="list-container">
        <ul class="student-items">
          <li
            v-for="student in filteredStudents"
            :key="student.realName"
            :class="{ active: selectedStudent && selectedStudent.realName === student.realName }"
            @click="selectStudent(student)"
          >
            <div class="student-item-content">
              <div class="avatar">{{ student.realName[0] }}</div>
              <div class="student-info">
                <span class="student-name">{{ student.realName }}</span>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>

    <!-- 中间学生详情 -->
    <div class="student-detail-container" v-if="selectedStudent">
      <div class="detail-header">
        <h2>学生详细信息</h2>
        <div class="detail-avatar">{{ selectedStudent.realName[0] }}</div>
      </div>

      <div class="detail-form">
        <div class="form-group">
          <label>姓名：</label>
          <input v-model="editingStudent.realName" type="text" class="form-input" />
        </div>
        <div class="form-group">
          <label>性别：</label>
          <select v-model="editingStudent.sex" class="form-input">
            <option value="男">男</option>
            <option value="女">女</option>
          </select>
        </div>
        <div class="form-group">
          <label>学号：</label>
          <input v-model="editingStudent.number" type="text" class="form-input" disabled />
        </div>
        <div class="form-group">
          <label>电子邮件：</label>
          <input v-model="editingStudent.email" type="text" class="form-input" disabled />
        </div>
        <div class="form-group">
          <label>家庭住址：</label>
          <input v-model="editingStudent.province" type="text" class="form-input" />
        </div>
        <div class="form-group">
          <label>个性签名：</label>
          <input v-model="editingStudent.profile" type="text" class="form-input" />
        </div>
        <div class="form-group">
          <label>学校：</label>
          <input v-model="editingStudent.school" type="text" class="form-input" />
        </div>

        <button @click="updateStudentsinfo" class="submit-btn">保存修改</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import service from "@/api/request.js";
import { ElMessage } from "element-plus";
import { ref, computed, onMounted } from "vue";
import * as XLSX from "xlsx";

onMounted(async () => {
  await getStudentsinfo();
});
// 学生数据
const students = ref([
  {
    email: "",
    number: "",
    realName: "",
    avatar: "",
    profile: "",
    sex: "",
    school: "",
    province: ""
  }
]);

// 响应式状态
const selectedStudent = ref(null);
const editingStudent = ref(null);
const searchQuery = ref("");
const fileInput = ref(null);

//查询所有学生
const getStudentsinfo = async () => {
  const res = await service.get(`info/stu/all`);
  console.log(res);
  students.value = res.data;
};

//修改学生信息
const updateStudentsinfo = async () => {
  const res = await service.post(`info/stu/update`, editingStudent.value);
  console.log(res);
  students.value = res.data;
  window.location.reload();
};

// 计算属性：过滤后的学生列表
const filteredStudents = computed(() => {
  return students.value.filter(student => student.realName.toLowerCase().includes(searchQuery.value.toLowerCase()));
});

// 导入导出相关方法
const handleImport = () => {
  fileInput.value.click();
};

const onFileChange = event => {
  const file = event.target.files[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = async e => {
    const data = new Uint8Array(e.target.result);
    const workbook = XLSX.read(data, { type: "array" });
    const firstSheetName = workbook.SheetNames[0];
    const worksheet = workbook.Sheets[firstSheetName];
    const jsonData = XLSX.utils.sheet_to_json(worksheet);
    console.log("转换后的 JSON 数据:", jsonData);
    for (let i of jsonData) {
      console.log(i);
      const res = await service.post(`account/add`, i);
      if (res.code === 200) {
        ElMessage({
          message: `插入成功--${i.realName}`,
          type: "success",
          duration: 2000 // 消息显示的毫秒数，默认是3000ms，这里设置为2000ms
        });
      }
      window.location.reload();
    }
    return;
  };
  reader.readAsArrayBuffer(file);
};

const exportToExcel = () => {
  // 准备导出数据
  const exportData = students.value.map(student => ({
    姓名: student.realName,
    性别: student.sex,
    学号: student.number,
    电子邮件: student.email,
    家庭住址: student.province,
    个性签名: student.profile,
    学校: student.school
  }));

  // 创建工作簿
  const ws = XLSX.utils.json_to_sheet(exportData);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "学生信息");

  // 导出文件
  XLSX.writeFile(wb, "学生信息表.xlsx");
};

// 其他方法
const selectStudent = student => {
  selectedStudent.value = student;
  editingStudent.value = { ...student };
};
</script>

<style scoped>
/* 基础样式 */
.student-management {
  display: flex;
  height: 100vh;
  gap: 24px;
  padding: 24px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
}

/* 导入导出按钮样式 */
.action-buttons {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.import-export-buttons {
  display: flex;
  gap: 8px;
}

.import-btn,
.export-btn {
  flex: 1;
  padding: 8px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s;
}

.import-btn {
  background-color: #4caf50;
  color: white;
}

.export-btn {
  background-color: #ff9800;
  color: white;
}

.import-btn:hover {
  background-color: #43a047;
}

.export-btn:hover {
  background-color: #f57c00;
}

/* 其他样式与之前相同 */
.student-list {
  width: 320px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
  display: flex;
  flex-direction: column;
}

.list-header {
  padding: 20px;
  border-bottom: 1px solid #eee;
}

.list-header h2 {
  margin: 0 0 20px 0;
  color: #2c3e50;
  font-size: 1.5rem;
}

.search-box {
  position: relative;
  margin-bottom: 15px;
}

.search-icon {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: #999;
}

.search-box input {
  width: 100%;
  padding: 10px 10px 10px 35px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 14px;
  transition: all 0.3s;
}

.search-box input:focus {
  outline: none;
  border-color: #2196f3;
  box-shadow: 0 0 0 2px rgba(33, 150, 243, 0.1);
}

.add-btn {
  width: 100%;
  padding: 10px;
  background-color: #2196f3;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  transition: background-color 0.3s;
  margin-bottom: 10px;
}

.add-btn:hover {
  background-color: #1976d2;
}

.list-container {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

.student-items {
  list-style: none;
  padding: 0;
  margin: 0;
}

.student-items li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px;
  margin-bottom: 8px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
}

.student-items li:hover {
  background-color: #f5f7fa;
}

.student-items li.active {
  background-color: #e3f2fd;
}

.student-item-content {
  display: flex;
  align-items: center;
  gap: 12px;
}

.avatar {
  width: 40px;
  height: 40px;
  background-color: #2196f3;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  font-weight: bold;
}

.student-info {
  display: flex;
  flex-direction: column;
}

.student-name {
  font-weight: 500;
  color: #2c3e50;
}

.student-class {
  font-size: 12px;
  color: #666;
}
.delete-btn {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background-color: #ff4444;
  color: white;
  border: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  opacity: 0;
  transition: all 0.3s;
}

.student-items li:hover .delete-btn {
  opacity: 1;
}

.student-detail-container {
  flex: 1;
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
  padding: 24px;
  position: relative;
}

.detail-header {
  display: flex;
  align-items: center;
  gap: 20px;
  margin-bottom: 30px;
}

.detail-avatar {
  width: 60px;
  height: 60px;
  background-color: #2196f3;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  font-weight: bold;
}

.detail-form {
  max-width: 500px;
}

.form-group {
  margin-bottom: 20px;
  display: flex;
  align-items: center;
}

.form-group label {
  width: 100px;
  color: #666;
  font-size: 14px;
}

.form-input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 14px;
  transition: all 0.3s;
}

.form-input:focus {
  outline: none;
  border-color: #2196f3;
  box-shadow: 0 0 0 2px rgba(33, 150, 243, 0.1);
}

.submit-btn {
  position: absolute;
  bottom: 24px;
  right: 24px;
  padding: 12px 24px;
  background-color: #2196f3;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s;
}

.submit-btn:hover {
  background-color: #1976d2;
}

.dialog {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.dialog-content {
  background: white;
  border-radius: 12px;
  width: 500px;
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.15);
}

.dialog-header {
  padding: 20px;
  border-bottom: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.dialog-header h3 {
  margin: 0;
  color: #2c3e50;
}

.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  color: #999;
  cursor: pointer;
}

.dialog-body {
  padding: 20px;
}

.dialog-footer {
  padding: 20px;
  border-top: 1px solid #eee;
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

.cancel-btn,
.confirm-btn {
  padding: 10px 20px;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;
}

.cancel-btn {
  background-color: #f5f7fa;
  border: 1px solid #ddd;
  color: #666;
}

.confirm-btn {
  background-color: #2196f3;
  border: none;
  color: white;
}

.cancel-btn:hover {
  background-color: #eee;
}

.confirm-btn:hover {
  background-color: #1976d2;
}

/* 滚动条美化 */
::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb {
  background: #ccc;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: #999;
}
</style>
