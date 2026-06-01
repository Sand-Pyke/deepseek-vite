<template>
  <div class="chat-messages" ref="chatMessagesDiv" @scroll="handleScroll">
    <div v-if="messages.length === 0" class="welcome-message">
      <h2>欢迎使用 DeepSeek</h2>
      <p>有什么我可以帮助你的吗？选择左侧的模型开始对话。</p>
    </div>

    <div
      v-for="(msg, index) in messages"
      :key="index"
      class="message"
      :class="msg.role === 'user' ? 'user-message' : 'assistant-message'"
    >
      <div class="avatar" :class="msg.role === 'user' ? 'user-avatar' : 'assistant-avatar'"></div>
      <div class="message-content">
        <div class="message-bubble" style="position: relative;">
          <div class="reply-content" v-html="msg.role === 'user' ? escapeHtml(msg.content) : renderMarkdown(msg.content)"></div>
          <button v-if="msg.role === 'assistant'" class="message-copy-btn" @click="copyMessage(msg.content, $event)" title="复制全文">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <rect width="14" height="14" x="8" y="8" rx="2" ry="2"></rect>
              <path d="M4 16c-1.1 0-2-.9-2-2V4c0-1.1.9-2 2-2h10c1.1 0 2 .9 2 2v1"></path>
            </svg>
          </button>
        </div>
      </div>
    </div>

    <!-- 正在生成的消息 -->
    <div v-if="isGenerating" class="message assistant-message">
      <div class="avatar assistant-avatar"></div>
      <div class="message-content">
        <div class="message-bubble">
          <div class="reply-content">
            <div v-if="!accumulatedContent" class="thinking-indicator">
              <span class="thinking-dot"></span>
              <span class="thinking-dot"></span>
              <span class="thinking-dot"></span>
            </div>
            <div v-else v-html="renderMarkdown(accumulatedContent)"></div>
          </div>
        </div>
      </div>
    </div>

    <button v-if="isUserScrolledUp" class="scroll-to-bottom-btn" @click="scrollToBottom">
      <svg width="14" height="14" viewBox="0 0 14 14" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M11.8486 5.5L11.4238 5.92383L8.69727 8.65137C8.44157 8.90706 8.21562 9.13382 8.01172 9.29785C7.79912 9.46883 7.55595 9.61756 7.25 9.66602C7.08435 9.69222 6.91565 9.69222 6.75 9.66602C6.44405 9.61756 6.20088 9.46883 5.98828 9.29785C5.78438 9.13382 5.55843 8.90706 5.30273 8.65137L2.57617 5.92383L2.15137 5.5L3 4.65137L3.42383 5.07617L6.15137 7.80273C6.42595 8.07732 6.59876 8.24849 6.74023 8.3623C6.87291 8.46904 6.92272 8.47813 6.9375 8.48047C6.97895 8.48703 7.02105 8.48703 7.0625 8.48047C7.07728 8.47813 7.12709 8.46904 7.25977 8.3623C7.40124 8.24849 7.57405 8.07732 7.84863 7.80273L10.5762 5.07617L11 4.65137L11.8486 5.5Z" fill="currentColor"></path>
      </svg>
    </button>
  </div>
</template>

<script setup>
import { ref, nextTick } from 'vue'
import { marked } from 'marked'
import DOMPurify from 'dompurify'

const props = defineProps({
  messages: {
    type: Array,
    default: () => []
  },
  isGenerating: {
    type: Boolean,
    default: false
  },
  accumulatedContent: {
    type: String,
    default: ''
  },
  autoScroll: {
    type: Boolean,
    default: true
  }
})

const emit = defineEmits(['scroll-change', 'regenerate'])

function regenerateMessage(index) {
  emit('regenerate', index)
}

const chatMessagesDiv = ref(null)
const isUserScrolledUp = ref(false)

// Markdown 渲染
function renderMarkdown(markdownText) {
  if (!markdownText) return ''
  try {
    let rawHtml = marked.parse(markdownText)
    rawHtml = rawHtml.replace(/<table([^>]*)>/g, '<div class="table-container"><table$1>')
    rawHtml = rawHtml.replace(/<\/table>/g, '</table></div>')
    rawHtml = rawHtml.replace(/<a([^>]*)>/g, '<a$1 target="_blank" rel="noopener noreferrer">')
    rawHtml = rawHtml.replace(/<pre>/g, '<pre><button class="copy-btn" onclick="copyCode(this)">\n      <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">\n        <rect width="14" height="14" x="8" y="8" rx="2" ry="2"></rect>\n        <path d="M4 16c-1.1 0-2-.9-2-2V4c0-1.1.9-2 2-2h10c1.1 0 2 .9 2 2v1"></path>\n      </svg>\n     \n    </button>')
    return DOMPurify.sanitize(rawHtml, { ADD_ATTR: ['target', 'rel', 'onclick'] })
  } catch (e) {
    console.warn('Markdown render error', e)
    return markdownText.replace(/</g, '&lt;').replace(/>/g, '&gt;')
  }
}

// 复制代码
function copyCode(btn) {
  const pre = btn.parentElement
  const code = pre.querySelector('code')
  if (code) {
    navigator.clipboard.writeText(code.textContent).then(() => {
      btn.innerHTML = `\n      <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">\n        <polyline points="20 6 9 17 4 12"></polyline>\n      </svg>\n      \n    `
      btn.classList.add('copied')
      setTimeout(() => {
        btn.innerHTML = `\n      <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">\n        <rect width="14" height="14" x="8" y="8" rx="2" ry="2"></rect>\n        <path d="M4 16c-1.1 0-2-.9-2-2V4c0-1.1.9-2 2-2h10c1.1 0 2 .9 2 2v1"></path>\n      </svg>\n      \n    `
        btn.classList.remove('copied')
      }, 2000)
    }).catch(err => {
      console.error('复制失败:', err)
    })
  }
}

// 复制整条消息
function copyMessage(content, event) {
  navigator.clipboard.writeText(content).then(() => {
    const btn = event.currentTarget
    btn.innerHTML = `
      <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <polyline points="20 6 9 17 4 12"></polyline>
      </svg>
    `
    btn.classList.add('copied')
    setTimeout(() => {
      btn.innerHTML = `
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <rect width="14" height="14" x="8" y="8" rx="2" ry="2"></rect>
          <path d="M4 16c-1.1 0-2-.9-2-2V4c0-1.1.9-2 2-2h10c1.1 0 2 .9 2 2v1"></path>
        </svg>
      `
      btn.classList.remove('copied')
    }, 2000)
  }).catch(err => {
    console.error('复制失败:', err)
  })
}

// 将复制函数挂载到全局
window.copyCode = copyCode

// HTML 转义
function escapeHtml(str) {
  return str.replace(/[&<>]/g, (m) => {
    if (m === '&') return '&amp;'
    if (m === '<') return '&lt;'
    if (m === '>') return '&gt;'
    return m
  }).replace(/\n/g, '<br>')
}

// 检查滚动位置
function handleScroll() {
  if (!chatMessagesDiv.value) return
  const { scrollTop, scrollHeight, clientHeight } = chatMessagesDiv.value
  const scrollDistanceFromBottom = scrollHeight - scrollTop - clientHeight
  isUserScrolledUp.value = scrollDistanceFromBottom > 50
  emit('scroll-change', isUserScrolledUp.value)
}

// 滚动到底部（只有在用户没有主动向上滚动时才执行）
function scrollToBottom(force = false) {
  if (!chatMessagesDiv.value) return
  
  // 如果用户已经向上滚动超过50px，且不是强制滚动，则不执行自动滚动
  if (!force && isUserScrolledUp.value) {
    return
  }
  
  chatMessagesDiv.value.scrollTop = chatMessagesDiv.value.scrollHeight
  isUserScrolledUp.value = false
  emit('scroll-change', false)
}

// 暴露方法给父组件
defineExpose({
  scrollToBottom,
  handleScroll
})
</script>
