<template>
  <aside class="sidebar" :class="{ collapsed: !expanded, expanded: expanded }">
    <div class="model-selector-section">
      <div class="logo">
        <svg width="36" height="36" viewBox="0 0 32 32" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M22.5 11C22.5 11 22.125 14.375 19.25 16.5C16.375 18.625 13.5 18.25 13.5 18.25C13.5 18.25 11.375 17.75 10 19.25C8.625 20.75 8.25 22.5 9.5 24C10.75 25.5 13.5 25.75 13.5 25.75" stroke="#3b82f6" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path>
          <path d="M18 25C18 25 18.375 23.5 19.75 22.75C21.125 22 23.25 22.25 23.25 22.25" stroke="#3b82f6" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path>
          <path d="M14 13C14 13 14.5 11.5 16.25 11C18 10.5 20 11.25 20 11.25" stroke="#3b82f6" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path>
          <path d="M26 15C26 15 27.5 14.75 28.25 15.5C29 16.25 28.75 17.5 28.75 17.5" stroke="#3b82f6" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path>
          <path d="M10 14C10 14 9 15 8.5 16.5C8 18 8.75 19.25 10 19.25" stroke="#3b82f6" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path>
        </svg>
        <span>deepseek</span>
      </div>
      <div class="model-option" :class="{ active: currentModel === 'flash' }" data-model="flash" @click="$emit('set-model', 'flash')">
        <span>DeepSeek V4 Flash</span>
        <svg class="check-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <polyline points="20 6 9 17 4 12"></polyline>
        </svg>
      </div>
      <div class="model-option" :class="{ active: currentModel === 'v4' }" data-model="v4" @click="$emit('set-model', 'v4')">
        <span>DeepSeek V4 Pro</span>
        <svg class="check-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <polyline points="20 6 9 17 4 12"></polyline>
        </svg>
      </div>
    </div>

    <div class="system-prompt-section">
      <div class="model-selector-title">场景列表</div>
      <div class="scene-list">
        <div
          v-for="scene in scenes"
          :key="scene.id"
          class="scene-item"
          :class="{ active: scene.id === currentSceneId }"
          @click="$emit('switch-scene', scene.id)"
        >
          <span class="scene-name">{{ scene.name }}</span>
          <span class="scene-prompt">{{ scene.systemPrompt || '无提示词' }}</span>
          <div class="scene-item-actions">
            <button class="scene-action-btn" @click.stop="$emit('edit-scene', scene.id)">编辑</button>
            <button
              class="scene-action-btn"
              v-if="scene.id !== 'default'"
              @click.stop="$emit('delete-scene', scene.id, scene.name)"
            >
              删除
            </button>
          </div>
        </div>
      </div>
    </div>

    <div class="sidebar-divider"></div>
    <button class="sidebar-btn" @click="$emit('add-scene')">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <path d="M12 5v14M5 12h14"></path>
      </svg>
      添加场景
    </button>
    <button class="sidebar-btn" @click="$emit('clear-chat')">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M3 6h18M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6m3 0V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"></path>
      </svg>
      清空对话
    </button>
  </aside>
</template>

<script setup>
defineProps({
  expanded: {
    type: Boolean,
    default: true
  },
  currentModel: {
    type: String,
    default: 'v4'
  },
  scenes: {
    type: Array,
    default: () => []
  },
  currentSceneId: {
    type: String,
    default: 'default'
  }
})

defineEmits([
  'toggle',
  'set-model',
  'switch-scene',
  'add-scene',
  'edit-scene',
  'delete-scene',
  'clear-chat'
])
</script>
