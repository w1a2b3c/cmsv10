<template>
  <div class="doctor-popover" style="position: absolute; left: 16px; top: 16px">
    <el-popover
      :width="300"
      trigger="click"
      popper-style="box-shadow: rgb(14 18 22 / 35%) 0px 10px 38px -10px, rgb(14 18 22 / 20%) 0px 10px 20px -15px; padding: 20px;"
    >
      <template #reference>
        <el-avatar
          :size="50"
          :src="doctorInfo.photo"
          style="cursor: pointer; border: 2px solid var(--el-color-primary-light-3)"
        />
      </template>

      <template #default>
        <div class="doctor-info" style="display: flex; gap: 16px; flex-direction: column">
          <div style="display: flex; align-items: start; gap: 16px">
            <div>
              <el-input v-model="doctorInfo.doctorName" placeholder="医生姓名"></el-input>
              <el-input v-model="doctorInfo.title" placeholder="职称"></el-input>
              <el-input v-model="doctorInfo.gender" placeholder="性别"></el-input>
              <el-input v-model="doctorInfo.rating" placeholder="评分" disabled></el-input>
            </div>
          </div>

          <el-descriptions :column="1" border>
            <el-descriptions-item label="专业领域">
              <el-input v-model="doctorInfo.specializedFields" placeholder="专业领域"></el-input>
            </el-descriptions-item>
            <el-descriptions-item label="背景">
              <el-input v-model="doctorInfo.background" placeholder="背景"></el-input>
            </el-descriptions-item>
            <el-descriptions-item label="其他信息">
              <el-input v-model="doctorInfo.otherInformation" placeholder="其他信息"></el-input>
            </el-descriptions-item>
          </el-descriptions>

          <div style="margin-top: 20px">
            <el-button type="primary" @click="saveInfo">保存</el-button>
            <el-button @click="cancelEdit">取消</el-button>
          </div>
        </div>
      </template>
    </el-popover>
  </div>
</template>

<script lang="ts" setup>
import { ref } from "vue";

const doctorInfo = ref({
  background: "毕业于北京协和医学院，临床经验15年以上",
  doctorName: "陈医生",
  title: "主任医师",
  gender: "女",
  rating: 99,
  specializedFields: "心血管疾病，介入性心脏病学",
  otherInformation: "中国医师协会会员，曾获多项医疗成就奖",
  photo: "https://avatars.githubusercontent.com/u/72015883?v=4"
});

const saveInfo = () => {
  // 在这里添加保存逻辑，比如发送到后端服务器
  console.log("保存信息:", doctorInfo.value);
};

const cancelEdit = () => {
  doctorInfo.value = {
    background: "毕业于北京协和医学院，临床经验15年以上",
    doctorName: "陈医生",
    title: "主任医师",
    gender: "女",
    rating: 4.8,
    specializedFields: "心血管疾病，介入性心脏病学",
    otherInformation: "中国医师协会会员，曾获多项医疗成就奖",
    photo: "https://avatars.githubusercontent.com/u/72015883?v=4"
  };
};
</script>

<style scoped>
.doctor-popover :deep(.el-avatar) {
  border: 2px solid transparent;
  transition: border-color 0.3s ease;
}

.doctor-popover :deep(.el-avatar):hover {
  border-color: var(--el-color-primary);
}

.doctor-info :deep(.el-descriptions__label) {
  width: 80px;
}

.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}

.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}

.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
