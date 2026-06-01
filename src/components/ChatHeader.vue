<template>
  <div class="chat-header">
    <span class="header-title">{{ sceneName }}</span>
    <span class="header-model-badge">{{ modelName }}</span>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  currentScene: {
    type: Object,
    default: () => ({ name: '默认场景' })
  },
  currentModel: {
    type: String,
    default: 'v4'
  }
})

const sceneName = computed(() => {
  return props.currentScene?.name || '默认场景'
})

const modelName = computed(() => {
  return props.currentModel === 'flash' ? 'DeepSeek V4 Flash' : 'DeepSeek V4 Pro'
})
</script>
