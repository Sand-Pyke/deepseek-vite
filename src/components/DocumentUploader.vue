<template>
  <div class="document-uploader">
    <div 
      class="upload-area" 
      :class="{ dragging: isDragging, 'has-files': uploadedFiles.length > 0 }"
      @dragover.prevent="handleDragOver"
      @dragleave="handleDragLeave"
      @drop.prevent="handleDrop"
      @click="triggerFileInput"
    >
      <input 
        ref="fileInput"
        type="file" 
        multiple 
        accept=".pdf,.txt,.md,.docx,.doc"
        class="file-input"
        @change="handleFileSelect"
      />
      
      <div v-if="uploadedFiles.length === 0" class="upload-hint">
        <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
          <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
          <polyline points="17 8 12 3 7 8"></polyline>
          <line x1="12" y1="3" x2="12" y2="15"></line>
        </svg>
        <p>点击或拖拽文档到此处上传</p>
        <p class="supported-formats">支持 PDF、TXT、MD、DOCX、DOC 格式</p>
      </div>
      
      <div v-else class="uploaded-files">
        <div v-for="(file, index) in uploadedFiles" :key="index" class="file-item">
          <div class="file-icon">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
              <polyline points="14 2 14 8 20 8"></polyline>
              <line x1="16" y1="13" x2="8" y2="13"></line>
              <line x1="16" y1="17" x2="8" y2="17"></line>
              <polyline points="10 9 9 9 8 9"></polyline>
            </svg>
          </div>
          <div class="file-info">
            <span class="file-name">{{ file.name }}</span>
            <span class="file-size">{{ formatFileSize(file.size) }}</span>
          </div>
          <button class="remove-btn" @click.stop="removeFile(index)">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="18" y1="6" x2="6" y2="18"></line>
              <line x1="6" y1="6" x2="18" y2="18"></line>
            </svg>
          </button>
        </div>
      </div>
      
      <div v-if="uploadProgress > 0" class="upload-progress">
        <div class="progress-bar" :style="{ width: uploadProgress + '%' }"></div>
      </div>
    </div>
    
    <button 
      v-if="uploadedFiles.length > 0"
      class="upload-btn" 
      :disabled="isUploading"
      @click="handleUpload"
    >
      <svg v-if="!isUploading" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
        <polyline points="17 8 12 3 7 8"></polyline>
        <line x1="12" y1="3" x2="12" y2="15"></line>
      </svg>
      <span>{{ isUploading ? '上传中...' : '上传到知识库' }}</span>
    </button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  knowledgeBaseId: {
    type: String,
    default: ''
  }
})

const emit = defineEmits(['upload-success', 'upload-error'])

const fileInput = ref(null)
const isDragging = ref(false)
const uploadedFiles = ref([])
const isUploading = ref(false)
const uploadProgress = ref(0)

function triggerFileInput() {
  if (!isUploading.value) {
    fileInput.value?.click()
  }
}

function handleDragOver() {
  isDragging.value = true
}

function handleDragLeave() {
  isDragging.value = false
}

function handleDrop(e) {
  isDragging.value = false
  const files = Array.from(e.dataTransfer.files)
  addFiles(files)
}

function handleFileSelect(e) {
  const files = Array.from(e.target.files)
  addFiles(files)
  e.target.value = ''
}

function addFiles(files) {
  const validFiles = files.filter(file => {
    const validExtensions = ['.pdf', '.txt', '.md', '.docx', '.doc']
    const ext = file.name.toLowerCase().substring(file.name.lastIndexOf('.'))
    return validExtensions.includes(ext)
  })
  uploadedFiles.value = [...uploadedFiles.value, ...validFiles]
}

function removeFile(index) {
  uploadedFiles.value.splice(index, 1)
}

function formatFileSize(bytes) {
  if (bytes < 1024) return bytes + ' B'
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB'
  return (bytes / (1024 * 1024)).toFixed(1) + ' MB'
}

async function handleUpload() {
  if (!props.knowledgeBaseId) {
    emit('upload-error', '请先创建知识库')
    return
  }
  
  isUploading.value = true
  uploadProgress.value = 0
  
  try {
    const formData = new FormData()
    uploadedFiles.value.forEach(file => {
      formData.append('files', file)
    })
    formData.append('knowledge_base_id', props.knowledgeBaseId)
    
    const response = await fetch('https://api.deepseek.com/v1/rag/documents/upload', {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${import.meta.env.VITE_DEEPSEEK_API_KEY || 'sk-1d5741c1a35a40a785f4afd4e7165547'}`
      },
      body: formData
    })
    
    if (!response.ok) {
      const errorData = await response.json().catch(() => ({ message: '上传失败' }))
      throw new Error(errorData.message || `HTTP ${response.status}`)
    }
    
    const result = await response.json()
    emit('upload-success', result)
    uploadedFiles.value = []
    uploadProgress.value = 100
  } catch (error) {
    emit('upload-error', error.message)
  } finally {
    isUploading.value = false
    uploadProgress.value = 0
  }
}

defineExpose({
  clearFiles: () => { uploadedFiles.value = [] }
})
</script>

<style scoped>
.document-uploader {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.upload-area {
  border: 2px dashed #cbd5e1;
  border-radius: 12px;
  padding: 24px;
  cursor: pointer;
  transition: all 0.2s;
  position: relative;
  overflow: hidden;
}

.upload-area:hover {
  border-color: #3b82f6;
  background-color: #eff6ff;
}

.upload-area.dragging {
  border-color: #3b82f6;
  background-color: #eff6ff;
}

.upload-area.has-files {
  border-style: solid;
  border-color: #e2e8f0;
}

.file-input {
  display: none;
}

.upload-hint {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  color: #64748b;
}

.upload-hint svg {
  color: #94a3b8;
}

.upload-hint p {
  margin: 0;
  font-size: 14px;
}

.supported-formats {
  font-size: 12px !important;
  color: #94a3b8;
}

.uploaded-files {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.file-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 12px;
  background: #f8fafc;
  border-radius: 8px;
}

.file-icon {
  color: #3b82f6;
  flex-shrink: 0;
}

.file-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.file-name {
  font-size: 14px;
  color: #1e293b;
}

.file-size {
  font-size: 12px;
  color: #64748b;
}

.remove-btn {
  background: transparent;
  border: none;
  color: #94a3b8;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
  transition: all 0.2s;
}

.remove-btn:hover {
  background: #f1f5f9;
  color: #ef4444;
}

.upload-progress {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: #e2e8f0;
}

.progress-bar {
  height: 100%;
  background: #3b82f6;
  transition: width 0.3s;
}

.upload-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 12px 20px;
  background: #3b82f6;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.2s;
}

.upload-btn:hover:not(:disabled) {
  background: #2563eb;
}

.upload-btn:disabled {
  background: #64748b;
  cursor: not-allowed;
}
</style>