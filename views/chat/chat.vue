<template>
  <div class="chat-container">
    <!-- <el-button type="primary" @click="startWebSocket">连接</el-button>  -->
    <el-card class="chatList">
      <div class="student-list">
        <div class="list-header">
          <h3>用户列表</h3>
        </div>
        <!-- 学生列表 -->
        <div v-infinite-scroll="load" class="infinite-list" style="overflow: auto">
          <el-button class="infinite-list-item" v-for="(item, index) in students" :key="index" @click="buttonClick(item)">{{
            item.realName
          }}</el-button>
        </div>
      </div>
    </el-card>

    <!--  -->
    <el-card class="chat-card">
      <template #header>
        <div class="card-header">
          <span>{{ people.receiverName }}</span>
          <el-tag :type="connectionStatus === 'Connected' ? 'success' : 'danger'">
            {{ connectionStatus }}
          </el-tag>
        </div>
      </template>
      <!--  -->
      <div class="chat-messages" @scroll="handleScroll" ref="chatMessagesRef">
        <div v-for="(message, index) in messages" :key="index" :class="['message', message.type]">
          {{ message.content }}
        </div>
      </div>
      <!--  -->
      <template #footer>
        <div class="chat-input">
          <el-input v-model="inputMessage" placeholder="在此输入文本" @keyup.enter="sendMessage">
            <template #append>
              <el-button type="primary" @click="sendMessage" :disabled="!isConnected"> Send </el-button>
            </template>
          </el-input>
        </div>
      </template>
    </el-card>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, nextTick } from "vue";
import { ElCard, ElTag, ElInput, ElButton } from "element-plus";
import service from "@/api/request.js";

const ws = ref(null);
const messages = ref([]);
const inputMessage = ref("");
const connectionStatus = ref("Disconnected");
const isConnected = ref(false);
const chatMessagesRef = ref(null);

let people = ref({
  receiverId: null,
  receiverName: null
});

const startWebSocket = () => {
  connectWebSocket();
};
const connectWebSocket = () => {
  const t = localStorage.getItem("token");
  ws.value = new WebSocket(`ws://172.22.127.134:1314/ws/chat?receiverId=${people.value.receiverId}&t=${t} `);

  ws.value.onopen = () => {
    connectionStatus.value = "Connected";
    isConnected.value = true;
    messages.value.push({ type: "system", content: "已连接到chat服务器" });
  };

  ws.value.onmessage = event => {
    messages.value.push({ type: "received", content: event.data });
    scrollToBottom();
  };

  ws.value.onclose = () => {
    connectionStatus.value = "Disconnected";
    isConnected.value = false;
    messages.value.push({ type: "system", content: "与 Chat 服务器断开连接" });
  };

  ws.value.onerror = error => {
    console.error("WebSocket error:", error);
    messages.value.push({ type: "system", content: "错误:无法连接到Chat服务器" });
  };
};

const sendMessage = () => {
  if (inputMessage.value.trim() && isConnected.value) {
    ws.value.send(inputMessage.value);
    messages.value.push({ type: "sent", content: inputMessage.value });
    inputMessage.value = "";
    scrollToBottom();
  }
};

const scrollToBottom = () => {
  nextTick(() => {
    if (chatMessagesRef.value) {
      chatMessagesRef.value.scrollTop = chatMessagesRef.value.scrollHeight;
    }
  });
};

//list
const page = ref({
  current: 1,
  total: 1
});
const count = ref(0);
const students = ref([{}]);
const buttonClick = async item => {
  clearChat();

  people.value.receiverId = item.id;
  people.value.receiverName = item.realName;
  // console.log(item.realName);
  page.value.current = 1;
  await getMoreMessages();
  startWebSocket();
};
const getStudentsinfo = async () => {
  const res = await service.get(`info/stu/all`);
  // console.log(res)
  students.value = res.data;
};
const load = () => {
  count.value += 2;
};
onMounted(async () => {
  await getStudentsinfo();
});
//chatview
const clearChat = () => {
  messages.value = [];
};
const oldmessages = ref([]);
const moreF = ref(0);
const handleScroll = event => {
  const element = event.target;
  if (element.scrollTop === 0 && page.value.current <= page.value.total) {
    getMoreMessages();
    moreF.value = 0;
  } else if (page.value.current > page.value.total && moreF.value == 0 && element.scrollTop === 0) {
    moreF.value = 1;
    oldmessages.value.push({ type: "system", content: "无更多聊天记录" });
    messages.value = [...oldmessages.value, ...messages.value];
    oldmessages.value = [];
  }
};
const getMoreMessages = async () => {
  const res = await service.get(`chat/history/${people.value.receiverId}/${page.value.current}`);
  console.log(res);

  page.value.current = page.value.current + 1;
  page.value.total = res.data.page.total;

  for (let i of res.data.history) {
    if (i.sender == "me") {
      oldmessages.value.push({ type: "sent", content: i.content });
    } else if (i.sender == "other") {
      oldmessages.value.push({ type: "received", content: i.content });
    }
  }
  //messages.value.push({ type: 'received', content: event.data })
  //messages.value.push({ type: 'sent', content: inputMessage.value })
  messages.value = [...oldmessages.value, ...messages.value];
  oldmessages.value = [];
};
//xxxxxxx
onUnmounted(() => {
  if (ws.value) {
    ws.value.close();
  }
});
</script>

<style scoped>
.chat-container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
  display: flex;
}

.chatList {
  width: 25%;
  /* Set the width of the chat list */
  margin-right: 20px;
  display: flex;
  flex-direction: column;
  /* 设置子元素沿竖轴排列 */
  align-items: flex-start;
}

.chat-card {
  height: 500px;
  display: flex;
  flex-direction: column;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.chat-messages {
  flex-grow: 1;
  overflow-y: auto;
  padding: 10px;
  display: flex;
  flex-direction: column;
  height: 300px;
  width: 600px;
}

.message {
  max-width: 70%;
  padding: 8px 12px;
  margin-bottom: 10px;
  border-radius: 20px;
  word-wrap: break-word;
}

.sent {
  align-self: flex-end;
  background-color: #dcf8c6;
}

.received {
  align-self: flex-start;
  background-color: #f2f2f2;
}

.system {
  align-self: center;
  background-color: #e1e1e1;
  font-style: italic;
}

.chat-input {
  margin-top: 20px;
}

/* 其他样式与之前相同 */
.student-list {
  width: 100%;
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

.list-header h3 {
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

.infinite-list {
  height: 300px;
  padding: 0;
  margin: 0;
  list-style: none;
}

.infinite-list .infinite-list-item {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 30px;
  background: var(--el-color-primary-light-9);
  margin: 10px;
  color: var(--el-color-primary);
}

.infinite-list .infinite-list-item + .list-item {
  margin-top: 10px;
}
</style>
