<template>
  <div class="modal-overlay" :class="{ show: visible }">
    <div class="modal">
      <div class="modal-title">编辑场景</div>
      <input type="text" class="modal-input" :disabled="scene.id === 'default'" v-model="sceneName" placeholder="请输入场景名称" />
      <textarea class="modal-input" v-model="scenePrompt" placeholder="请输入系统提示词（可选）" rows="5"></textarea>
      
      <div class="modal-buttons">
        <button class="modal-btn modal-btn-cancel" @click="close">取消</button>
        <button class="modal-btn modal-btn-confirm" @click="save">确定</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  visible: {
    type: Boolean,
    default: false
  },
  scene: {
    type: Object,
    default: () => ({ name: '', systemPrompt: '', enableRAG: false, knowledgeBaseId: '', ragTopK: 3 })
  }
})

const emit = defineEmits(['close', 'save'])

const sceneName = ref('')
const scenePrompt = ref('')
const enableRAG = ref(false)
const knowledgeBaseId = ref('')
const ragTopK = ref(3)

watch(() => props.scene, (newScene) => {
  if (newScene) {
    sceneName.value = newScene.name || ''
    scenePrompt.value = newScene.systemPrompt || ''
    enableRAG.value = newScene.enableRAG || false
    knowledgeBaseId.value = newScene.knowledgeBaseId || ''
    ragTopK.value = newScene.ragTopK || 3
  }
}, { immediate: true })

function close() {
  emit('close')
}

function save() {
  emit('save', {
    name: sceneName.value.trim(),
    systemPrompt: scenePrompt.value.trim(),
    enableRAG: enableRAG.value,
    knowledgeBaseId: knowledgeBaseId.value.trim(),
    ragTopK: ragTopK.value
  })
  close()
}
</script>

<style scoped>
.toggle-label {
  font-size: 14px;
  color: #374151;
}
</style>
