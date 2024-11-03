<template>
  <div class="pomodoro-container">
    <div class="timer">
      <div class="visibility-toggle" @click="toggleSettings">
          <svg width="20px" height="20px" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M22 6.5H16" stroke="#292D32" stroke-width="1.5" stroke-miterlimit="10" stroke-linecap="round" stroke-linejoin="round"/>
            <g opacity="0.4">
            <path d="M6 6.5H2" stroke="#292D32" stroke-width="1.5" stroke-miterlimit="10" stroke-linecap="round" stroke-linejoin="round"/>
            <path d="M10 10C11.933 10 13.5 8.433 13.5 6.5C13.5 4.567 11.933 3 10 3C8.067 3 6.5 4.567 6.5 6.5C6.5 8.433 8.067 10 10 10Z" stroke="#292D32" stroke-width="1.5" stroke-miterlimit="10" stroke-linecap="round" stroke-linejoin="round"/>
            </g>
            <path d="M8 17.5H2" stroke="#292D32" stroke-width="1.5" stroke-miterlimit="10" stroke-linecap="round" stroke-linejoin="round"/>
            <g opacity="0.4">
            <path d="M22 17.5H18" stroke="#292D32" stroke-width="1.5" stroke-miterlimit="10" stroke-linecap="round" stroke-linejoin="round"/>
            <path d="M14 21C15.933 21 17.5 19.433 17.5 17.5C17.5 15.567 15.933 14 14 14C12.067 14 10.5 15.567 10.5 17.5C10.5 19.433 12.067 21 14 21Z" stroke="#292D32" stroke-width="1.5" stroke-miterlimit="10" stroke-linecap="round" stroke-linejoin="round"/>
          </g>
          </svg>
      </div>

      <div class="status-label" :class="{ 'break': isBreak }">
        {{ isBreak ? 'Break Time' : 'Work Time' }}
      </div>

      <div class="time-display" :class="{ 'break': isBreak }">
        {{ countdownActive ? countdown : formatTime(timeLeft) }}
      </div>

      <div class="cycles">
        <div 
          v-for="index in 4" 
          :key="index"
          class="cycle-dot"
          :class="{ 'completed': completedCycles >= index }"
        ></div>
      </div>
      
      <div class="controls">
        <button @click="startTimer" :disabled="isRunning || countdownActive">Start</button>
        <button @click="pauseTimer" :disabled="!isRunning">Stop</button>
      </div>

      <div class="settings" v-show="showSettings">
        <div class="input-group">
          <label>Work Time (minutes):</label>
          <input type="number" v-model="workTime" :disabled="isRunning" min="1">
        </div>
        <div class="input-group">
          <label>Break Time (minutes):</label>
          <input type="number" v-model="breakTime" :disabled="isRunning" min="1">
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onUnmounted, watch } from 'vue'

// Định nghĩa các biến state
const workTime = ref<number>(25)
const breakTime = ref<number>(5)
const timeLeft = ref<number>(25 * 60)
const isRunning = ref<boolean>(false)
const isBreak = ref<boolean>(false)
const timer = ref<number | null>(null)
const completedCycles = ref<number>(0)
const showSettings = ref<boolean>(true)

// Thêm biến cho đếm ngược
const countdown = ref<number>(5)
const countdownActive = ref<boolean>(false)
const countdownTimer = ref<number | null>(null)

// Định nghĩa các âm thanh với file local
const startWorkSound = new Audio('/sounds/alarm.wav')
const startBreakSound = new Audio('/sounds/alarm.wav')
const endBreakSound = new Audio('/sounds/alarm.wav')

// Điều chỉnh volume nếu cần
startWorkSound.volume = 0.7
startBreakSound.volume = 0.7
endBreakSound.volume = 0.7

// Thêm watch để theo dõi sự thay đổi của workTime
watch(workTime, (newValue) => {
  // Chỉ cập nhật timeLeft khi không trong thời gian nghỉ và không đang chạy
  if (!isBreak.value && !isRunning.value) {
    timeLeft.value = newValue * 60
  }
})

// Thêm watch để theo dõi sự thay đổi của breakTime
watch(breakTime, (newValue) => {
  // Chỉ cập nhật timeLeft khi đang trong thời gian nghỉ và không đang chạy
  if (isBreak.value && !isRunning.value) {
    timeLeft.value = newValue * 60
  }
})

// Các methods
const startTimer = () => {
  if (!isRunning.value && !countdownActive.value) {
    countdownActive.value = true
    countdown.value = 5

    // Bắt đầu đếm ngược 5s
    countdownTimer.value = setInterval(() => {
      countdown.value--

      if (countdown.value === 0) {
        clearInterval(countdownTimer.value!)
        countdownActive.value = false

        // Bắt đầu timer chính
        startMainTimer()
      }
    }, 1000)
  }
}

const startMainTimer = () => {
  isRunning.value = true

  // Phát âm thanh khi bắt đầu
  if (isBreak.value) {
    startBreakSound.play()
  } else {
    startWorkSound.play()
  }

  timer.value = setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value--
    } else {
      if (!isBreak.value) {
        // Kết thúc thời gian làm việc, bắt đầu nghỉ
        startBreakSound.play()
        timeLeft.value = breakTime.value * 60
        isBreak.value = true
      } else {
        // Kết thúc thời gian nghỉ, bắt đầu chu kỳ mới
        endBreakSound.play()
        timeLeft.value = workTime.value * 60
        isBreak.value = false
        completedCycles.value = (completedCycles.value + 1) % 5
      }
    }
  }, 1000)
}

const pauseTimer = () => {
  // Dừng cả countdown timer và main timer
  if (countdownTimer.value) {
    clearInterval(countdownTimer.value)
    countdownTimer.value = null
    countdownActive.value = false
  }

  if (timer.value) {
    clearInterval(timer.value)
    timer.value = null
  }
  isRunning.value = false
}

const formatTime = (seconds: number): string => {
  const minutes = Math.floor(seconds / 60)
  const remainingSeconds = seconds % 60
  return `${minutes.toString().padStart(2, '0')}:${remainingSeconds.toString().padStart(2, '0')}`
}

const toggleSettings = () => {
  showSettings.value = !showSettings.value
}

// Cleanup khi component bị hủy
onUnmounted(() => {
  if (timer.value) {
    clearInterval(timer.value)
  }
  if (countdownTimer.value) {
    clearInterval(countdownTimer.value)
  }
})
</script>

<style >
body {
  background: green;
}

.pomodoro-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  background: green;
}

.status-label {
  font-size: 2rem;
  font-weight: 600;
  margin: 1rem 0;
  color: gray;
}

.timer {
  background-color: white;
  padding: 1rem 2rem;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  text-align: center;
  position: relative;
  width: 320px;
}

.time-display {
  font-size: 4rem;
  font-weight: bold;
  margin: 1rem 0;
  font-family: monospace;
  transition: color 0.3s ease;
}

.controls {
  margin-bottom: 2rem;
}

button {
  padding: 0.5rem 1rem;
  margin: 0 0.5rem;
  font-size: 1.1rem;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  background-color: #4CAF50;
  color: white;
}

button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.settings {
  text-align: left;
}

.input-group {
  margin: 1rem 0;
}

input {
  padding: 0.5rem;
  margin-left: 1rem;
  width: 60px;
}

label {
  font-size: 1rem;
}

.cycles {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin: 1.5rem 0;
}

.cycle-dot {
  width: 15px;
  height: 15px;
  border-radius: 50%;
  background-color: #ddd;
  transition: background-color 0.3s ease;
}

.cycle-dot.completed {
  background-color: #ff4444;
}

.visibility-toggle {
  position: absolute;
  top: 1rem;
  right: 1rem;
  cursor: pointer;
  font-size: 1.5rem;
  color: #666;
  transition: color 0.3s ease;
}

.visibility-toggle:hover {
  color: #333;
}

/* Thêm style cho countdown nếu muốn */
.time-display.countdown {
  color: #ff4444;
}
</style>
