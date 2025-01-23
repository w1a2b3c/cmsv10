<template>
  <div class="article-management">
    <h1>文章管理系统</h1>

    <!-- 查询文章部分 -->
    <div class="article-search">
      <div class="search-bar">
        <input v-model="searchQuery" placeholder="输入关键词进行搜索" @keyup.enter="handleSearchArticles" class="input-field" />
        <button @click="handleSearchArticles" class="btn btn-primary">搜索</button>
      </div>
    </div>

    <!-- 文章列表 -->
    <ul v-if="articles.length" class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h2>{{ article.topic }}</h2>
        <p>发布时间: {{ formatDate(article.createTime) }}</p>
        <p v-if="article.introduction">简介: {{ article.introduction }}</p>
        <button @click="handleDeleteArticle(article.id)" class="btn btn-danger">删除</button>
      </li>
    </ul>
    <p v-else class="no-articles">没有找到相关的文章</p>

    <!-- 分页部分 -->
    <div class="pagination">
      <button @click="changePage(pageNo - 1)" :disabled="pageNo === 1" class="btn btn-primary">上一页</button>
      <span>第 {{ pageNo }} 页 / 共 {{ totalPages }} 页</span>
      <button @click="changePage(pageNo + 1)" :disabled="pageNo === totalPages" class="btn btn-primary">下一页</button>
    </div>

    <!-- 创建文章按钮 -->
    <!--    <button @click="isCreateArticleModalVisible = true" class="btn btn-success">创建文章</button>-->

    <!-- 创建文章模态框 -->
    <div v-if="isCreateArticleModalVisible" class="modal">
      <div class="modal-content">
        <span class="close" @click="isCreateArticleModalVisible = false"> &times; </span>
        <h2>创建文章</h2>

        <input v-model="article.topic" placeholder="文章标题" class="input-field" />
        <textarea v-model="article.content" placeholder="文章内容" class="textarea-field"></textarea>

        <!--        <div class="upload-section">-->
        <!--          <label for="contentImage">内容图片：</label>-->
        <!--          <input type="file" id="contentImage" @change="handleImageUpload('imgUrl', $event)" />-->
        <!--        </div>-->

        <!--        <div class="upload-section">-->
        <!--          <label for="topicImage">主题图片：</label>-->
        <!--          <input type="file" id="topicImage" @change="handleImageUpload('topicImg', $event)" />-->
        <!--        </div>-->

        <button @click="handleCreateArticle" class="btn btn-primary">创建</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      searchQuery: "",
      pageNo: 1,
      pageSize: 5,
      totalArticles: 0,
      totalPages: 1,
      articles: [],
      article: {
        topic: "",
        content: ""
        //imgUrl: "", // 内容图片
        //topicImg: "" // 主题图片
        //introduction: ""
      },
      isCreateArticleModalVisible: false
    };
  },
  created() {
    this.handleSearchArticles();
  },
  methods: {
    validateArticle() {
      // 确保所有字段都有值，避免 undefined 错误
      const topic = this.article.topic || "";
      const content = this.article.content || "";
      // const imgUrl = this.article.imgUrl || "";
      // const topicImg = this.article.topicImg || "";
      //const cont = this.article.content || "";

      if (!topic.trim()) {
        alert("文章标题不能为空！");
        return false;
      }
      if (!content.trim()) {
        alert("文章内容不能为空！");
        return false;
      }
      // if (!imgUrl.trim()) {
      //   alert("请上传内容图片！");
      //   return false;
      // }
      // if (!topicImg.trim()) {
      //   alert("请上传主题图片！");
      //   return false;
      // }
      return true;
    },
    async handleSearchArticles() {
      try {
        const response = await axios.get("http://localhost:1314/article/all/main", {
          params: {
            pageNo: this.pageNo,
            pageSize: this.pageSize,
            keyword: this.searchQuery
          }
        });
        if (response.data.code === 200) {
          this.articles = response.data.data.list;
          this.totalArticles = response.data.data.total;
          this.totalPages = Math.ceil(this.totalArticles / this.pageSize);
        } else {
          console.error("Error fetching articles:", response.data.message);
        }
      } catch (error) {
        console.error("Error fetching articles:", error);
      }
    },
    changePage(newPage) {
      if (newPage >= 1 && newPage <= this.totalPages) {
        this.pageNo = newPage;
        this.handleSearchArticles();
      }
    },
    async handleDeleteArticle(id) {
      try {
        const response = await axios.delete(`http://localhost:1314/article/teacher/${id}`);
        if (response.data.code === 200) {
          this.handleSearchArticles();
        } else {
          console.error("Error deleting article:", response.data.message);
        }
      } catch (error) {
        console.error("Error deleting article:", error);
      }
    },
    async handleCreateArticle() {
      // 验证文章数据是否合法
      if (this.validateArticle()) {
        try {
          // 从本地存储获取 token
          const token = localStorage.getItem("token");
          console.log("token" + token);

          // 如果没有 token，提示用户登录
          if (!token) {
            alert("请先登录！");
            return;
          }

          // 调用后端接口创建文章，并在请求头中携带 token
          const response = await axios.post(
            "http://localhost:1314/article/teacher",
            {
              topic: this.article.topic,
              content: this.article.content
              // imgUrl: this.article.imgUrl,
              // topicImg: this.article.topicImg
            },
            {
              headers: {
                token: `${token}` // 设置 Authorization 请求头
              }
            }
          );

          if (response.data.code === 200) {
            // 重新加载文章列表
            this.handleSearchArticles();

            // 清空创建表单并关闭模态框
            this.article = {
              topic: "",
              content: ""
              // imgUrl: "",
              // topicImg: ""
            };
            this.isCreateArticleModalVisible = false;
          } else {
            console.error("创建文章失败:", response.data.message);
          }
        } catch (error) {
          console.error("创建文章出错:", error);
        }
      }
    },
    handleImageUpload(type, event) {
      const file = event.target.files[0];
      if (file) {
        const formData = new FormData();
        formData.append("file", file);

        // 假设后端返回图片URL
        axios
          .post("http://localhost:1314/common/upload", formData)
          .then(response => {
            if (response.data.code === 200) {
              this.article[type] = response.data.url; // 返回的图片URL
            } else {
              alert("上传失败，请重试！");
            }
          })
          .catch(error => {
            console.error("图片上传失败:", error);
            alert("图片上传失败！");
          });
      }
    },
    formatDate(dateString) {
      const options = { year: "numeric", month: "2-digit", day: "2-digit" };
      return new Date(dateString).toLocaleDateString(undefined, options);
    }
  }
};
</script>

<style scoped>
.article-management {
  font-family: Arial, sans-serif;
  padding: 20px;
}

.search-bar {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.pagination {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
}

.article-list {
  list-style: none;
  padding: 0;
}

.article-item {
  border: 1px solid #ddd;
  padding: 10px;
  margin-bottom: 10px;
  border-radius: 5px;
}

.article-img {
  max-width: 100%;
  margin-top: 10px;
}

.no-articles {
  color: #777;
}

.modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: white;
  padding: 20px;
  border-radius: 5px;
  z-index: 1000;
  width: 80%;
  max-width: 600px;
}

.modal-content {
  position: relative;
}

.close {
  position: absolute;
  top: 10px;
  right: 10px;
  cursor: pointer;
}

.upload-section {
  margin: 10px 0;
}

.upload-section input {
  width: 100%;
}

.btn {
  padding: 8px 16px;
  cursor: pointer;
  border: none;
  border-radius: 4px;
}

.btn-primary {
  background-color: #007bff;
  color: white;
}

.btn-success {
  background-color: #28a745;
  color: white;
}

.btn-danger {
  background-color: #dc3545;
  color: white;
}
</style>
