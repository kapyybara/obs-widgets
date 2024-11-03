<template>
  <div class="container">
    <!-- Status Label -->
    <div class="status-label" :class="{ 'is-break': isBreak }">
      <div class="status-text">{{ isBreak ? 'Break Time' : 'Working Time' }}</div>
      <div class="timer">{{ formatTime(timer) }}</div>
    </div>

    <!-- Keyboard Display (giữ nguyên code cũ) -->
    <div class="keystroke-display">
      <div class="keyboard-layout">
        <div v-for="(row, rowIndex) in [row1, row2, row3, row4, row5]" :key="rowIndex" class="row">
          <div 
            v-for="key in row" 
            :key="key.code"
            class="key" 
            :class="{ active: pressedKeys.includes(key.code) }"
            :style="{ width: `${key.width ? key.width * 40 : 40}px` }"
          >
            {{ key.label }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'

const pressedKeys = ref<string[]>([])

const row1 = [
  { label: '`', code: 'Backquote'},
  { label: 'ESC', code: 'Escape' },
  { label: '1', code: 'Digit1' },
  { label: '2', code: 'Digit2' },
  { label: '3', code: 'Digit3' },
  { label: '4', code: 'Digit4' },
  { label: '5', code: 'Digit5' },
  { label: '6', code: 'Digit6' },
  { label: '7', code: 'Digit7' },
  { label: '8', code: 'Digit8' },
  { label: '9', code: 'Digit9' },
  { label: '0', code: 'Digit0' },
  { label: '-', code: 'Minus' },
  { label: '=', code: 'Equal' },
  { label: 'Backspace', code: 'Backspace', width: 2 },
]

const row2 = [
  { label: 'Tab', code: 'Tab', width: 1.5 },
  { label: 'Q', code: 'KeyQ' },
  { label: 'W', code: 'KeyW' },
  { label: 'E', code: 'KeyE' },
  { label: 'R', code: 'KeyR' },
  { label: 'T', code: 'KeyT' },
  { label: 'Y', code: 'KeyY' },
  { label: 'U', code: 'KeyU' },
  { label: 'I', code: 'KeyI' },
  { label: 'O', code: 'KeyO' },
  { label: 'P', code: 'KeyP' },
  { label: '[', code: 'BracketLeft' },
  { label: ']', code: 'BracketRight' },
  { label: '\\', code: 'Backslash', width: 1.5 },
  { label: 'Del', code: 'Delete' }
]

const row3 = [
  { label: 'Caps', code: 'CapsLock', width: 1.75 },
  { label: 'A', code: 'KeyA' },
  { label: 'S', code: 'KeyS' },
  { label: 'D', code: 'KeyD' },
  { label: 'F', code: 'KeyF' },
  { label: 'G', code: 'KeyG' },
  { label: 'H', code: 'KeyH' },
  { label: 'J', code: 'KeyJ' },
  { label: 'K', code: 'KeyK' },
  { label: 'L', code: 'KeyL' },
  { label: ';', code: 'Semicolon' },
  { label: "'", code: 'Quote' },
  { label: 'Enter', code: 'Enter', width: 2.25 },
  { label: 'PgUp', code: 'PageUp' }
]

const row4 = [
  { label: 'Shift', code: 'ShiftLeft', width: 2.25 },
  { label: 'Z', code: 'KeyZ' },
  { label: 'X', code: 'KeyX' },
  { label: 'C', code: 'KeyC' },
  { label: 'V', code: 'KeyV' },
  { label: 'B', code: 'KeyB' },
  { label: 'N', code: 'KeyN' },
  { label: 'M', code: 'KeyM' },
  { label: ',', code: 'Comma' },
  { label: '.', code: 'Period' },
  { label: '/', code: 'Slash' },
  { label: 'Shift', code: 'ShiftRight', width: 1.75 },
  { label: '↑', code: 'ArrowUp' },
  { label: 'PgDn', code: 'PageDown' }
]

const row5 = [
  { label: 'Ctrl', code: 'ControlLeft', width: 1.25 },
  { label: 'Win', code: 'MetaLeft', width: 1.25 },
  { label: 'Alt', code: 'AltLeft', width: 1.25 },
  { label: 'Space', code: 'Space', width: 6.25 },
  { label: 'Alt', code: 'AltRight', width: 1.25 },
  { label: 'Fn', code: 'Fn', width: 1.25 },
  { label: '←', code: 'ArrowLeft' },
  { label: '↓', code: 'ArrowDown' },
  { label: '→', code: 'ArrowRight' }
]

const handleKeyDown = (event: KeyboardEvent) => {
  event.preventDefault()
  if (!pressedKeys.value.includes(event.code)) {
    pressedKeys.value.push(event.code)
  }
}

const handleKeyUp = (event: KeyboardEvent) => {
  event.preventDefault()
  pressedKeys.value = pressedKeys.value.filter(key => key !== event.code)
}

const isBreak = ref(false)
const timer = ref(0)
const workDuration = 25 * 60 // 25 phút làm việc
const breakDuration = 5 * 60 // 5 phút nghỉ
let intervalId: number | null = null

// Format thời gian từ giây sang mm:ss
const formatTime = (seconds: number) => {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
}

// Xử lý timer
const startTimer = () => {
  timer.value = isBreak.value ? breakDuration : workDuration
  
  intervalId = window.setInterval(() => {
    if (timer.value > 0) {
      timer.value--
    } else {
      // Chuyển đổi giữa work/break
      isBreak.value = !isBreak.value
      timer.value = isBreak.value ? breakDuration : workDuration
    }
  }, 1000)
}

onMounted(() => {
  document.body.setAttribute('tabindex', '-1')
  document.body.focus()
  document.body.addEventListener('blur', () => {
    document.body.focus()
  })
  document.addEventListener('keydown', handleKeyDown, true)
  document.addEventListener('keyup', handleKeyUp, true)
  startTimer()
})

onBeforeUnmount(() => {
  document.body.removeAttribute('tabindex')
  document.removeEventListener('keydown', handleKeyDown, true)
  document.removeEventListener('keyup', handleKeyUp, true)
  document.body.removeEventListener('blur', () => {
    document.body.focus()
  })
  if (intervalId) {
    clearInterval(intervalId)
  }
})
</script>

<style scoped>
:global(body:focus) {
  outline: none;
}

.container {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.status-label {
  background: #2d2d2d;
  padding: 15px 25px;
  border-radius: 8px;
  color: #4CAF50;
  font-weight: bold;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: all 0.3s ease;
}

.status-label.is-break {
  color: #FF5722;
}

.status-text {
  font-size: 24px;
}

.timer {
  font-size: 28px;
  font-family: monospace;
}

.keystroke-display {
  padding: 20px;
  background: #1a1a1a;
  border-radius: 8px;
  width: fit-content;
}

.keyboard-layout {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.row {
  display: flex;
  gap: 4px;
}

.key {
  height: 40px;
  background: #2d2d2d;
  border: 1px solid #3d3d3d;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #ffffff;
  font-size: 12px;
  user-select: none;
  transition: all 0.2s ease;
}

.key.active {
  background: #4d4d4d;
  border-color: #5d5d5d;
  box-shadow: 0 0 5px rgba(255, 255, 255, 0.2);
}
</style>
