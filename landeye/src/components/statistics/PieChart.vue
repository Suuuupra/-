<template>
    <div class="Container">
      <div class="data_title">数据处理情况</div>
      <div v-for="(item, index) in this.data" :key="item.id" class="item" v-if="this.data.length > 0">
        <div class="comment-item">
          <div class="comment-header">
            <img class="comment-avatar" src="/img/logo.png" alt="Avatar">
            <div class="comment-info">
              <div class="comment-author">{{ item.name }}</div>
              <div class="comment-date">{{ item.time.toString().replace("T", " ").replace("Z", "").replace(".000", "") }}</div>
            </div>
          </div>
          <div class="comment-content">
            {{ item.detail }}
          </div>
          <div class="comment-actions">
            <span class="comment-action" @click="showDetails(item.name, item.detail)">Detail</span>
          </div>
          <!-- 模态框 -->
          <div v-if="showModal" class="modal">
            <div class="modal-content">
              <span class="close" @click="hideModal">&times;</span>
              <div>处理人：{{ currentName }}</div>
              <div class="comment-content-full">处理详情：{{ currentLog }}</div> <!-- 显示完整的评论内容 -->
            </div>
          </div>
        </div>
      </div>
    </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      data: '',
      showModal: false,
      currentLog: null,
      currentName: null,
    }
  },
  methods:{
    fetchData() {
      axios.get('http://8.148.10.46:3050/api/GetLog')
          .then((response) => {
            this.data = response.data;
          })
          .catch((error) => {
            console.error('Error fetching data:', error);
          });
    },
    showDetails(name, detail) {
      this.showModal = !this.showModal // 点击详情按钮显示模态框
      this.currentName = name
      this.currentLog = detail
    },
    hideModal() {
      this.showModal = false; // 关闭模态框
    }
  },
  mounted() {
    this.fetchData()
  }
}
</script>
<style>
.Container{
  position:relative;
  width: 48%;
  height: 100%;
  background: #f1f8f0;
  border-radius: 40px;
  overflow-y: auto;
}
.data_title {
  height: 10%;
}

.comment-item {
  border-bottom: 1px dashed #ccc;
  padding: 20px;

}

.comment-header {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.comment-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  margin-right: 10px;
}

.comment-author {
  font-weight: bold;
  color: #1c1c1c;
}

.comment-date {
  margin-top:1vh;
  font-size: 0.8em;
  color: #888;
}

.comment-content {
  margin-bottom: 10px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 300px;
  color: #4b4a4a;
  font-weight: bold;
}

.comment-actions {
  display: flex;
}

.comment-action {
  margin-right: 10px;
  color: #888;
  cursor: pointer;
}

.comment-action:hover {
  color: #333;
}

.modal {
  position: fixed;
  z-index: 1;
  left: 50%;
  top: 50%;
  width: 30%;
  height: 30%;
  overflow: auto;
  background-color: rgb(238, 246, 234);
  transform: translate(-50%, -50%);
  border-radius: 30px;
}

.modal-content {
  background-color: #f9f9f9;
  margin: 15% auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  width: 70%;
  max-width: 500px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

.close {
  color: #aaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
  transition: color 0.3s ease;
}
.close:hover,
.close:focus {
  color: black;
  text-decoration: none;
  cursor: pointer;
}
</style>

