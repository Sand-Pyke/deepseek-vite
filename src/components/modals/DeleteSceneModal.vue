<template>
  <div class="modal-overlay" :class="{ show: visible }">
    <div class="modal delete-modal">
      <div class="modal-title delete-title">
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="12" cy="12" r="10"></circle>
          <line x1="15" y1="9" x2="9" y2="15"></line>
          <line x1="9" y1="9" x2="15" y2="15"></line>
        </svg>
        确认删除
      </div>
      <div class="modal-content">
        确定要删除场景 "{{ sceneName }}" 吗？所有对话记录将被删除。
      </div>
      <div class="modal-buttons">
        <button class="modal-btn modal-btn-cancel" @click="close">取消</button>
        <button class="modal-btn modal-btn-confirm modal-btn-danger" @click="confirm">确定删除</button>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  visible: {
    type: Boolean,
    default: false
  },
  sceneName: {
    type: String,
    default: ''
  }
})

const emit = defineEmits(['close', 'confirm'])

function close() {
  emit('close')
}

function confirm() {
  emit('confirm')
  close()
}
</script>
