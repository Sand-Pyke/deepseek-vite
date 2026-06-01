<template>
  <div class="app-container">
    <!-- 侧边栏 -->
    <ChatSidebar
      :expanded="sidebarExpanded"
      :current-model="currentModel"
      :scenes="scenes"
      :current-scene-id="currentSceneId"
      @set-model="setActiveModel"
      @switch-scene="switchScene"
      @add-scene="showAddSceneModal"
      @edit-scene="showEditSceneModal"
      @delete-scene="showDeleteSceneModal"
      @clear-chat="showClearChatModal"
    />

    <!-- 侧边栏切换按钮 -->
    <button
      class="sidebar-toggle"
      :class="{ collapsed: !sidebarExpanded, expanded: sidebarExpanded }"
      @click="toggleSidebar"
    >
      <svg
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2"
      >
        <polyline points="15 18 9 12 15 6"></polyline>
      </svg>
    </button>

    <!-- 主内容区 -->
    <div class="main-content" :class="{ 'full-width': !sidebarExpanded }">
      <ChatHeader :current-scene="currentScene" :current-model="currentModel" />

      <ChatMessages
        ref="chatMessagesRef"
        :messages="messages"
        :is-generating="isGenerating"
        :accumulated-content="accumulatedContent"
        @scroll-change="handleScrollChange"
      />

      <ChatInput
        ref="chatInputRef"
        :is-generating="isGenerating"
        :is-sending="isSending"
        @send="handleSendMessage"
        @stop="handleStopGenerating"
      />
    </div>

    <!-- 模态框 -->
    <AddSceneModal
      :visible="addSceneModalVisible"
      @close="closeAddSceneModal"
      @confirm="handleAddScene"
    />

    <EditSceneModal
      :visible="editSceneModalVisible"
      :scene="editingScene"
      @close="closeEditSceneModal"
      @save="handleEditScene"
    />

    <DeleteSceneModal
      :visible="deleteSceneModalVisible"
      :scene-name="sceneToDeleteName"
      @close="closeDeleteSceneModal"
      @confirm="handleDeleteScene"
    />

    <ClearChatModal
      :visible="clearChatModalVisible"
      @close="closeClearChatModal"
      @confirm="handleClearChat"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, nextTick } from "vue";
import ChatSidebar from "./components/ChatSidebar.vue";
import ChatHeader from "./components/ChatHeader.vue";
import ChatMessages from "./components/ChatMessages.vue";
import ChatInput from "./components/ChatInput.vue";
import AddSceneModal from "./components/modals/AddSceneModal.vue";
import EditSceneModal from "./components/modals/EditSceneModal.vue";
import DeleteSceneModal from "./components/modals/DeleteSceneModal.vue";
import ClearChatModal from "./components/modals/ClearChatModal.vue";

const HARDCODED_API_KEY = import.meta.env.VITE_API_KEY;

// 状态
const currentModel = ref("v4");
const sidebarExpanded = ref(true);
const isGenerating = ref(false);
const isSending = ref(false);
const accumulatedContent = ref("");
const statusMessage = ref("");
const messages = ref([]);

// 场景相关
const scenes = ref([
  {
    id: "default",
    name: "默认场景",
    systemPrompt: "",
    enableRAG: false,
    knowledgeBaseId: "",
    ragTopK: 3,
  },
]);
const currentSceneId = ref("default");
const conversationHistory = ref({});

// 模态框状态
const addSceneModalVisible = ref(false);
const editSceneModalVisible = ref(false);
const deleteSceneModalVisible = ref(false);
const clearChatModalVisible = ref(false);
const editingSceneId = ref(null);
const sceneToDeleteId = ref(null);
const sceneToDeleteName = ref("");

// Refs
const chatMessagesRef = ref(null);
const chatInputRef = ref(null);
const abortControllerRef = ref(null);

// 计算属性
const currentScene = computed(() => {
  return (
    scenes.value.find((s) => s.id === currentSceneId.value) || {
      name: "默认场景",
    }
  );
});

// 存储键
const STORAGE_KEYS = {
  MODEL_PREF: "deepseek_model_pref",
  SCENES: "deepseek_scenes",
  CONVERSATIONS: "deepseek_conversations",
  CURRENT_SCENE: "deepseek_current_scene",
};

// 切换模型
function setActiveModel(model) {
  currentModel.value = model;
  localStorage.setItem(STORAGE_KEYS.MODEL_PREF, model);
}

// 切换侧边栏
function toggleSidebar() {
  sidebarExpanded.value = !sidebarExpanded.value;
}

// 切换场景
function switchScene(sceneId) {
  if (currentSceneId.value === sceneId) return;
  if (isGenerating.value) {
    return;
  }

  currentSceneId.value = sceneId;
  loadConversationHistory();
  localStorage.setItem(STORAGE_KEYS.CURRENT_SCENE, sceneId);
}

// 加载对话历史
function loadConversationHistory() {
  const saved = localStorage.getItem(STORAGE_KEYS.CONVERSATIONS);
  if (saved) {
    try {
      const history = JSON.parse(saved);
      if (history && history[currentSceneId.value]) {
        conversationHistory.value = history;
        messages.value = history[currentSceneId.value];
      } else {
        messages.value = [];
      }
    } catch (e) {
      console.warn("历史解析失败", e);
      messages.value = [];
    }
  } else {
    messages.value = [];
  }
  nextTick(() => chatMessagesRef.value?.scrollToBottom());
}

// 保存对话
function saveConversations() {
  localStorage.setItem(
    STORAGE_KEYS.CONVERSATIONS,
    JSON.stringify(conversationHistory.value),
  );
}

// 保存场景
function saveScenes() {
  localStorage.setItem(STORAGE_KEYS.SCENES, JSON.stringify(scenes.value));
}

// 添加场景
function addScene(name, systemPrompt = "") {
  const sceneId = "scene_" + Date.now();
  const newScene = {
    id: sceneId,
    name,
    systemPrompt,
  };
  scenes.value.push(newScene);
  if (!conversationHistory.value[sceneId]) {
    conversationHistory.value[sceneId] = [];
  }
  switchScene(sceneId);
  saveScenes();
  saveConversations();
}

// 模态框操作
function showAddSceneModal() {
  addSceneModalVisible.value = true;
}

function closeAddSceneModal() {
  addSceneModalVisible.value = false;
}

function handleAddScene({ name, prompt }) {
  addScene(name, prompt);
}

function showEditSceneModal(sceneId) {
  editingSceneId.value = sceneId;
  editSceneModalVisible.value = true;
}

function closeEditSceneModal() {
  editingSceneId.value = null;
  editSceneModalVisible.value = false;
}

function handleEditScene({
  name,
  systemPrompt,
  enableRAG,
  knowledgeBaseId,
  ragTopK,
}) {
  if (!editingSceneId.value) return;
  const scene = scenes.value.find((s) => s.id === editingSceneId.value);
  if (scene) {
    scene.name = name || scene.name;
    scene.systemPrompt = systemPrompt;
    scene.enableRAG = enableRAG || false;
    scene.knowledgeBaseId = knowledgeBaseId || "";
    scene.ragTopK = ragTopK || 3;
    saveScenes();
  }
  closeEditSceneModal();
}

function showDeleteSceneModal(sceneId, sceneName) {
  sceneToDeleteId.value = sceneId;
  sceneToDeleteName.value = sceneName;
  deleteSceneModalVisible.value = true;
}

function closeDeleteSceneModal() {
  sceneToDeleteId.value = null;
  deleteSceneModalVisible.value = false;
}

function handleDeleteScene() {
  if (!sceneToDeleteId.value || sceneToDeleteId.value === "default") return;
  const index = scenes.value.findIndex((s) => s.id === sceneToDeleteId.value);
  if (index !== -1) {
    scenes.value.splice(index, 1);
    delete conversationHistory.value[sceneToDeleteId.value];
    if (currentSceneId.value === sceneToDeleteId.value) {
      switchScene("default");
    }
    saveScenes();
    saveConversations();
  }
  closeDeleteSceneModal();
}

function showClearChatModal() {
  clearChatModalVisible.value = true;
}

function closeClearChatModal() {
  clearChatModalVisible.value = false;
}

function handleClearChat() {
  messages.value = [];
  conversationHistory.value[currentSceneId.value] = [];
  saveConversations();
  closeClearChatModal();
}

// 停止生成
function handleStopGenerating() {
  if (abortControllerRef.value) {
    abortControllerRef.value.abort();
  }
}

// 处理滚动变化
function handleScrollChange(isScrolledUp) {
  // 可以在这里处理滚动状态
}

// 发送消息
async function handleSendMessage(text) {
  if (!text || isGenerating.value || isSending.value) return;
  isSending.value = true;

  // 确保历史数组存在
  if (!conversationHistory.value[currentSceneId.value]) {
    conversationHistory.value[currentSceneId.value] = [];
  }
  const currentHistory = conversationHistory.value[currentSceneId.value];
  currentHistory.push({ role: "user", content: text });
  // messages 指向同一数组，自动更新，无需再次 push
  saveConversations();

  isGenerating.value = true;
  accumulatedContent.value = "";

  nextTick(() => chatMessagesRef.value?.scrollToBottom(true));

  const currentScene = scenes.value.find((s) => s.id === currentSceneId.value);

  try {
    await callChatAPI(currentScene);
  } catch (error) {
    // 错误处理保持不变
  } finally {
    isGenerating.value = false;
    isSending.value = false;
    nextTick(() => chatMessagesRef.value?.scrollToBottom());
  }
}

// 调用普通Chat API
async function callChatAPI(currentScene) {
  const chatMessages = [];
  const systemPrompt = currentScene?.systemPrompt || "";
  if (systemPrompt) {
    chatMessages.push({ role: "system", content: systemPrompt });
  }
  for (const turn of conversationHistory.value[currentSceneId.value]) {
    if (turn.role === "user" || turn.role === "assistant") {
      chatMessages.push({ role: turn.role, content: turn.content });
    }
  }

  const apiModel =
    currentModel.value === "flash" ? "deepseek-v4-flash" : "deepseek-v4-pro";

  abortControllerRef.value = new AbortController();
  const response = await fetch("https://api.deepseek.com/v1/chat/completions", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${HARDCODED_API_KEY}`,
    },
    body: JSON.stringify({
      model: apiModel,
      messages: chatMessages,
      stream: true,
      temperature: 0.7,
      max_tokens: 8192,
    }),
    signal: abortControllerRef.value.signal,
  });

  if (!response.ok) {
    const errorText = await response.text();
    throw new Error(
      `API错误 ${response.status}: ${errorText.substring(0, 200)}`,
    );
  }

  const reader = response.body.getReader();
  const decoder = new TextDecoder("utf-8");
  let buffer = "";

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    buffer += decoder.decode(value, { stream: true });
    const lines = buffer.split("\n");
    buffer = lines.pop() || "";

    for (const line of lines) {
      const trimmed = line.trim();
      if (!trimmed || trimmed === "data: [DONE]") continue;
      if (trimmed.startsWith("data: ")) {
        const jsonStr = trimmed.slice(6);
        try {
          const chunk = JSON.parse(jsonStr);
          const delta = chunk.choices?.[0]?.delta;
          if (!delta) continue;

          const content = delta.content || null;
          if (content) {
            accumulatedContent.value += content;
            nextTick(() => chatMessagesRef.value?.scrollToBottom(false));
          }
        } catch (e) {}
      }
    }
  }

  if (accumulatedContent.value.trim()) {
    conversationHistory.value[currentSceneId.value].push({
      role: "assistant",
      content: accumulatedContent.value,
    });
    saveConversations();
    messages.value.push({
      role: "assistant",
      content: accumulatedContent.value,
    });
    accumulatedContent.value = "";
  } else {
    statusMessage.value = "模型返回空内容";
  }
}

// 计算属性
const editingScene = computed(() => {
  if (!editingSceneId.value)
    return {
      name: "",
      systemPrompt: "",
      enableRAG: false,
      knowledgeBaseId: "",
      ragTopK: 3,
    };
  return (
    scenes.value.find((s) => s.id === editingSceneId.value) || {
      name: "",
      systemPrompt: "",
      enableRAG: false,
      knowledgeBaseId: "",
      ragTopK: 3,
    }
  );
});

// 初始化
onMounted(() => {
  // 加载保存的场景
  const savedScenes = localStorage.getItem(STORAGE_KEYS.SCENES);
  if (savedScenes) {
    try {
      const loadedScenes = JSON.parse(savedScenes);
      if (Array.isArray(loadedScenes) && loadedScenes.length > 0) {
        scenes.value = loadedScenes;
      }
    } catch (e) {
      console.warn("场景加载失败", e);
    }
  }

  // 加载当前场景
  const savedCurrentScene = localStorage.getItem(STORAGE_KEYS.CURRENT_SCENE);
  if (savedCurrentScene) {
    const sceneExists = scenes.value.find((s) => s.id === savedCurrentScene);
    if (sceneExists) {
      currentSceneId.value = savedCurrentScene;
    }
  }

  // 加载模型偏好
  const savedModel = localStorage.getItem(STORAGE_KEYS.MODEL_PREF);
  if (savedModel && (savedModel === "flash" || savedModel === "v4")) {
    currentModel.value = savedModel;
  }

  // 加载对话历史
  loadConversationHistory();
});
</script>

<style scoped>
.app-container {
  display: flex;
  width: 100%;
}
</style>
