<template>
  <div class="chat-container">
    <div class="message-group">
      <TransitionGroup name="message">
        <div 
          v-for="(message, index) in messages" 
          :key="message.id" 
          class="message"
        >
          {{ message.text }}
        </div>
        <div class="input-wrapper" style="margin-top: 8px;" :key="'input'">
          <div class="input-container">
            <div class="avatar">
              <img src="/avatar.jpg" alt="avatar">
            </div>
            <input
              ref="inputRef"
              v-model="newMessage"
              @input="adjustWidth"
              @keydown.enter="sendMessage"
              type="text"
              placeholder="Nhập tin nhắn..."
              class="auto-input"
            >
          </div>
        </div>
      </TransitionGroup>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const messages = ref([
])

const newMessage = ref('')
const inputRef = ref(null)

const sendMessage = () => {
  if (newMessage.value.trim()) {
    messages.value.push({
      id: messages.value.length + 1,
      text: newMessage.value
    })
    newMessage.value = ''
    adjustWidth()
  }
}

// Tạo một element ẩn để tính toán độ rộng
const calculateWidth = (text) => {
  const span = document.createElement('span')
  span.style.visibility = 'hidden'
  span.style.position = 'absolute'
  span.style.whiteSpace = 'pre'
  span.style.font = window.getComputedStyle(inputRef.value).font
  span.textContent = text || ' '
  document.body.appendChild(span)
  const width = span.getBoundingClientRect().width
  document.body.removeChild(span)
  return width
}

const adjustWidth = () => {
  if (inputRef.value) {
    const minWidth = 60 // Độ rộng tối thiểu
    const padding = 32 // Tổng padding left và right
    const width = calculateWidth(newMessage.value)
    inputRef.value.style.width = `${Math.max(minWidth, width + padding)}px`
  }
}

onMounted(() => {
  if (inputRef.value) {
    inputRef.value.focus()
    adjustWidth()
  }
})
</script>

<style >
body {
  background: rgb(0, 82, 0);
}
.chat-container {
  padding: 20px;
  width: 800px;
  height: 400px; /* Chiều cao cố định */
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  overflow-y: auto; /* Cho phép cuộn khi có nhiều tin nhắn */
}


.message-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-top: auto;
}

/* Animation cho tin nhắn mới */
.message-enter-active,
.message-leave-active {
  transition: all 0.3s ease;
}

.message-enter-from {
  opacity: 0;
  transform: translateY(20px);
}

.message-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}

/* Đảm bảo các tin nhắn không bị nhảy */
.message-move {
  transition: transform 0.3s ease;
}

.message {
  background-color: white;
  padding: 12px 16px;
  border-radius: 20px;
  max-width: 80%;
  width: fit-content;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
  font-size: 15px;
  color: #333;
}

.input-container {
  display: flex;
  align-items: center;
  gap: 8px;
}

.avatar {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  overflow: hidden;
  flex-shrink: 0;
}

.avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover; /* Đảm bảo ảnh luôn fit và căn giữa */
  object-position: center;
}

.auto-input {
  background-color: white;
  padding: 12px 16px;
  border-radius: 20px;
  min-width: 60px;
  border: none;
  outline: none;
  font-size: 15px;
  color: #333;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
  transition: width 0.1s;
}

.auto-input::placeholder {
  color: #999;
}
</style>