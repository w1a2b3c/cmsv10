<template>
  <div class="snow-page">
    <div class="home-page">
      <!-- 常用功能 -->
      <section class="section">
        <h2>心理安全常用功能</h2>
        <div class="card-container">
          <div class="card">
            <h3>心理健康自测</h3>
            <p>通过自测，评估你的心理健康状态，及时发现潜在问题。</p>
          </div>
          <div class="card">
            <h3>员工心理支持</h3>
            <p>为员工提供专业的心理支持和咨询，帮助他们应对工作中的挑战。</p>
          </div>
          <div class="card">
            <h3>心理安全培训课程</h3>
            <p>参与培训，学习如何提升团队的心理安全感，避免潜在的心理危机。</p>
          </div>
          <div class="card">
            <h3>心理安全资源库</h3>
            <p>查阅丰富的心理安全资源，提升你的工作环境心理健康。</p>
          </div>
        </div>
      </section>

      <!-- 心理安全话题 -->
      <section class="section">
        <h2>心理安全话题</h2>
        <div class="topic-list">
          <div class="topic-item">
            <h4>如何增强团队的心理安全感</h4>
            <p>创建一个支持性环境，鼓励开放的交流和反馈，增强团队成员的信任感。</p>
          </div>
          <div class="topic-item">
            <h4>识别和应对职场压力</h4>
            <p>学会识别职场压力的信号，并采取有效措施来应对。</p>
          </div>
          <div class="topic-item">
            <h4>心理安全在企业文化中的重要性</h4>
            <p>心理安全感是健康企业文化的基石，有助于员工创新和团队合作。</p>
          </div>
          <div class="topic-item">
            <h4>如何有效沟通以建立信任</h4>
            <p>通过良好的沟通建立信任，减少误解，增加工作中的心理安全感。</p>
          </div>
        </div>
      </section>

      <!-- 心理健康建议 -->
      <section class="section">
        <h2>心理健康建议</h2>
        <p>保持积极的心态和良好的心理健康非常重要。以下是一些实用建议：</p>
        <ul>
          <li>每天花时间进行冥想或深呼吸练习，放松身心。</li>
          <li>保持规律的运动，运动有助于缓解压力。</li>
          <li>与朋友和家人保持良好的沟通，分享情感。</li>
          <li>在工作中设定合理的目标，避免过度负荷。</li>
        </ul>
      </section>

      <!-- 心理安全指数 -->
      <section class="section">
        <h2>心理安全指数</h2>
        <div class="safety-index">
          <p>团队的心理安全指数是衡量员工在工作环境中是否感到心理安全的一个重要指标。</p>
          <div class="index-bar">
            <div class="progress" style="width: 85%"></div>
          </div>
          <p>当前心理安全指数：85%</p>
        </div>
      </section>
    </div>
  </div>
</template>

<script setup lang="ts"></script>

<style lang="scss" scoped>
.home-page {
  padding: 30px;
  background: linear-gradient(to bottom, #f9f9f9, #e9ecef);
}

.section {
  margin-bottom: 30px;
  padding: 20px;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

h2 {
  font-size: 1.8rem;
  color: #2d3a3b;
  font-weight: bold;
  margin-bottom: 20px;
}

.card-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
}

.card {
  background: #fafafa;
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  text-align: center;
  transition: transform 0.3s ease;
}

.card:hover {
  transform: translateY(-10px);
}

.card h3 {
  font-size: 1.2rem;
  color: #007b5e;
  margin-bottom: 10px;
}

.card p {
  font-size: 1rem;
  color: #555;
}

.topic-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.topic-item {
  background: #f7f7f7;
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.topic-item h4 {
  font-size: 1.1rem;
  color: #2d3a3b;
  margin-bottom: 10px;
}

.topic-item p {
  font-size: 1rem;
  color: #666;
}

.safety-index {
  text-align: center;
}

.index-bar {
  width: 100%;
  height: 20px;
  background-color: #e0e0e0;
  border-radius: 10px;
  margin: 10px 0;
}

.progress {
  height: 100%;
  background-color: #4caf50;
  border-radius: 10px;
}

p {
  font-size: 1rem;
  color: #555;
}
</style>
