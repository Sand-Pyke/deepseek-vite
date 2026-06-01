<template>
  <div class="modal-overlay" :class="{ show: visible }">
    <div class="modal">
      <div class="modal-title delete-title">
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M3 6h18M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6m3 0V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"></path>
        </svg>
        确认清空
      </div>
      <div class="modal-content">
        确定要清空当前场景的所有对话记录吗？此操作不可撤销。
      </div>
      <div class="modal-buttons">
        <button class="modal-btn modal-btn-cancel" @click="close">取消</button>
        <button class="modal-btn modal-btn-confirm modal-btn-danger" @click="confirm">确定清空</button>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  visible: {
    type: Boolean,
    default: false
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
