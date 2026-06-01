<template>
  <div class="modal-overlay" :class="{ show: visible }">
    <div class="modal">
      <div class="modal-title">添加场景</div>
      <div class="scene-tags">
        <button
          v-for="tag in sceneTags"
          :key="tag.value"
          type="button"
          class="scene-tag"
          :class="{ active: selectedTag === tag.value }"
          @click="selectedTag = tag.value"
        >
          {{ tag.label }}
        </button>
      </div>
      <div v-if="selectedTag === 'other'" style="display: block;">
        <input
          type="text"
          class="modal-input"
          v-model="sceneName"
          placeholder="请输入场景名称"
          style="margin-top: 12px"
        />
        <textarea
          class="modal-input"
          v-model="scenePrompt"
          placeholder="请输入系统提示词（可选）"
          rows="3"
          style="margin-top: 12px"
        ></textarea>
      </div>
      <div class="modal-buttons">
        <button class="modal-btn modal-btn-cancel" @click="close">取消</button>
        <button class="modal-btn modal-btn-confirm" @click="confirm">确定</button>
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
  }
})

const emit = defineEmits(['close', 'confirm'])

const selectedTag = ref('chef')
const sceneName = ref('')
const scenePrompt = ref('')

const sceneTags = [
  { label: '厨师', value: 'chef' },
  { label: '医生', value: 'doctor' },
  { label: '生活百科', value: 'life_expert' },
  { label: '其他', value: 'other' }
]

const presetScenes = {
  chef: {
    name: '厨师',
    systemPrompt: '你是一位全能的厨师，擅长任何菜系'
  },
  doctor: {
    name: '医生',
    systemPrompt: '你是一位全能的医生，通晓任何疑难杂症'
  },
  life_expert: {
    name: '生活百科',
    systemPrompt: '你是一位生活专家，了解日常生活的方方面面'
  }
}

watch(() => props.visible, (newVal) => {
  if (newVal) {
    selectedTag.value = 'chef'
    sceneName.value = ''
    scenePrompt.value = ''
  }
})

function close() {
  emit('close')
}

function confirm() {
  let name, prompt
  
  if (selectedTag.value === 'other') {
    name = sceneName.value.trim()
    prompt = scenePrompt.value.trim() || name
  } else {
    const preset = presetScenes[selectedTag.value]
    name = preset.name
    prompt = preset.systemPrompt
  }
  
  if (name) {
    emit('confirm', { name, prompt })
    close()
  }
}
</script>
