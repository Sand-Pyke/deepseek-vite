<template>
  <div class="input-area">
    <div class="input-container">
      <textarea
        id="userTextInput"
        class="input-textarea"
        placeholder="输入消息..."
        autocomplete="off"
        v-model="inputText"
        @keydown="handleKeyDown"
        ref="inputTextarea"
      ></textarea>
      <button id="stopBtn" class="stop-btn" v-if="isGenerating" @click="stopGenerating">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <rect x="6" y="6" width="12" height="12" rx="2" ry="2"></rect>
        </svg>
      </button>
      <button id="sendMsgBtn" class="send-btn" @click="sendMessage" :disabled="isGenerating">
        <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M8.3125 0.981587C8.66767 1.0545 8.97902 1.20558 9.2627 1.43374C9.48724 1.61438 9.73029 1.85933 9.97949 2.10854L14.707 6.83608L13.293 8.25014L9 3.95717V15.0431H7V3.95717L2.70703 8.25014L1.29297 6.83608L6.02051 2.10854C6.26971 1.85933 6.51277 1.61438 6.7373 1.43374C6.97662 1.24126 7.28445 1.04542 7.6875 0.981587C7.8973 0.94841 8.1031 0.956564 8.3125 0.981587Z" fill="currentColor"></path>
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, nextTick } from 'vue'

const props = defineProps({
  isGenerating: {
    type: Boolean,
    default: false
  },
  isSending: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['send', 'stop'])

const inputText = ref('')
const inputTextarea = ref(null)
const isSendingLocal = ref(false)

// 发送消息
function sendMessage() {
  const text = inputText.value.trim()
  if (!text || props.isGenerating || props.isSending || isSendingLocal.value) return
  
  isSendingLocal.value = true

  emit('send', text)
  inputText.value = ''
  
  // 重置输入框高度
  if (inputTextarea.value) {
    inputTextarea.value.style.height = 'auto'
  }
  
  setTimeout(() => {
    isSendingLocal.value = false
  }, 500)
}

// 停止生成
function stopGenerating() {
  emit('stop')
}

// 处理键盘事件
function handleKeyDown(e) {
  if (e.key === 'Enter' && !e.shiftKey) {
    e.preventDefault()
    sendMessage()
  }
}

// 自动调整输入框高度
watch(inputText, () => {
  if (inputTextarea.value) {
    inputTextarea.value.style.height = 'auto'
    inputTextarea.value.style.height = Math.max(100, Math.min(inputTextarea.value.scrollHeight, 300)) + 'px'
  }
})

// 暴露方法给父组件
defineExpose({
  focus: () => inputTextarea.value?.focus()
})
</script>
