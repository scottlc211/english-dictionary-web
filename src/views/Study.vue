<template>
  <div class="study-container">
    <!-- 页面标题 -->
    <div class="text-center mb-8">
      <div class="inline-flex items-center gap-3 mb-3">
        <div class="w-12 h-12 rounded-2xl bg-gradient-to-br from-primary-500 to-purple-500 flex items-center justify-center">
          <span class="text-2xl">📖</span>
        </div>
        <h1 class="text-2xl md:text-3xl font-bold text-gray-900 dark:text-gray-100">
          学习模式
        </h1>
      </div>
      <p class="text-gray-500 dark:text-gray-400">
        选择单词开始学习，标记您的掌握程度
      </p>
    </div>

    <!-- 主要内容区域 -->
    <div v-if="currentWord" class="main-content">
      <!-- 左侧：单词信息区域 -->
      <div class="word-section">
        <!-- 单词卡片 -->
        <div class="card text-center py-12 relative overflow-hidden">
          <!-- 装饰背景 -->
          <div class="absolute inset-0 bg-gradient-to-br from-primary-50 via-transparent to-purple-50 dark:from-primary-900/20 dark:to-purple-900/20"></div>
          
          <div class="relative z-10">
            <div class="text-4xl md:text-5xl font-bold text-gray-900 dark:text-gray-100 mb-4">
              {{ currentWord.word }}
            </div>
            <div v-if="currentPhonetic" class="text-lg text-gray-500 dark:text-gray-400 font-mono mb-6">
              [{{ currentPhonetic }}]
            </div>
            <div v-else-if="phoneticLoading" class="flex items-center justify-center gap-2 text-gray-500 dark:text-gray-400 mb-6">
              <span class="animate-spin">⏳</span>
              <span>加载音标中...</span>
            </div>
            
            <!-- 简洁定义（模糊点击显示） -->
            <div v-if="currentWord.concise_definition" class="max-w-lg mx-auto px-4">
              <div 
                @click="showConciseDefinition = !showConciseDefinition"
                class="p-4 rounded-xl bg-primary-50 dark:bg-primary-900/30 border border-primary-100 dark:border-primary-900/50 cursor-pointer transition-all hover:shadow-md"
              >
                <div :class="['text-primary-800 dark:text-primary-200 leading-relaxed transition-all', showConciseDefinition ? '' : 'blur-sm select-none']">
                  {{ currentWord.concise_definition }}
                </div>
                <div class="text-xs text-primary-600 dark:text-primary-400 mt-2 font-medium">
                  {{ showConciseDefinition ? '👆 点击隐藏' : '👆 点击显示释义' }}
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- 操作按钮 -->
        <div class="flex items-center justify-center gap-4 my-6">
          <SpeakerButton 
            :word="currentWord.word" 
            :text="currentWord.word"
            :lang="'en'"
            :speed="1.0"
          />
          <button
            v-if="userStore.isAuthenticated"
            @click="toggleCollection"
            :class="['btn', isCollected ? 'btn-primary' : 'btn-outline']"
          >
            {{ isCollected ? '⭐ 已收藏' : '☆ 收藏' }}
          </button>
          <span v-else class="text-sm text-gray-500 dark:text-gray-400">
            🔐 登录后可收藏
          </span>
        </div>

        <!-- 掌握程度选择（桌面端） -->
        <div class="hidden md:block">
          <h3 class="text-center text-lg font-semibold text-gray-900 dark:text-gray-100 mb-4">
            您对这个单词的掌握程度是？
          </h3>
          
          <div class="grid grid-cols-3 gap-4">
            <button
              v-for="option in qualityOptions"
              :key="option.value"
              @click="markWord(option.value)"
              :disabled="submitting"
              :class="[
                'quality-btn group',
                option.color === 'red' && 'quality-red',
                option.color === 'yellow' && 'quality-yellow',
                option.color === 'green' && 'quality-green',
                submitting && 'opacity-50 cursor-not-allowed'
              ]"
            >
              <div class="text-3xl mb-2 group-hover:scale-110 transition-transform">{{ option.icon }}</div>
              <div class="font-bold mb-1">{{ option.label }}</div>
              <div class="text-xs opacity-80">{{ option.description }}</div>
            </button>
          </div>
        </div>
      </div>

      <!-- 右侧：释义区域 -->
      <div class="definition-section card">
        <div v-if="!showDefinition" class="h-full flex items-center justify-center py-16">
          <button @click="showDefinition = true" class="btn btn-primary btn-lg">
            <span>📖</span>
            查看详细释义
          </button>
        </div>
        <div v-else class="h-full flex flex-col">
          <div class="flex items-center justify-between pb-4 border-b border-gray-100 dark:border-gray-700">
            <h3 class="text-lg font-bold text-gray-900 dark:text-gray-100 flex items-center gap-2">
              <span>📚</span>
              {{ currentWord.word }} 的释义
            </h3>
            <button @click="showDefinition = false" class="btn btn-ghost btn-sm">
              隐藏
            </button>
          </div>
          <div class="flex-1 overflow-y-auto py-4 space-y-4">
            <div v-for="(def, index) in currentWord.definitions" :key="index" class="p-4 rounded-xl bg-gray-50 dark:bg-gray-800/50">
              <span class="badge badge-primary mb-3">{{ def.pos }}</span>
              <div class="space-y-3">
                <div>
                  <div class="text-xs text-gray-500 dark:text-gray-400 mb-1">英文解释</div>
                  <div class="text-gray-600 dark:text-gray-300">{{ def.explanation_en }}</div>
                </div>
                <div>
                  <div class="text-xs text-gray-500 dark:text-gray-400 mb-1">中文解释</div>
                  <div class="text-gray-900 dark:text-gray-100 font-medium">{{ def.explanation_cn }}</div>
                </div>
                <div v-if="def.example_en" class="p-3 rounded-lg bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700">
                  <div class="text-gray-600 dark:text-gray-400 italic text-sm mb-1">{{ def.example_en }}</div>
                  <div class="text-gray-800 dark:text-gray-200 text-sm">{{ def.example_cn }}</div>
                </div>
              </div>
            </div>
            
            <!-- 词形变化 -->
            <div v-if="currentWord.forms && Object.keys(currentWord.forms).length > 0" class="pt-4 border-t border-gray-100 dark:border-gray-700">
              <h4 class="font-bold text-gray-900 dark:text-gray-100 mb-3 flex items-center gap-2">
                <span>🔄</span> 词形变化
              </h4>
              <div class="flex flex-wrap gap-2">
                <div v-for="(value, key) in currentWord.forms" :key="key" class="badge badge-primary">
                  {{ key }}: {{ value }}
                </div>
              </div>
            </div>
            
            <!-- 相似词辨析 -->
            <div v-if="currentWord.comparison && currentWord.comparison.length > 0" class="pt-4 border-t border-gray-100 dark:border-gray-700">
              <h4 class="font-bold text-gray-900 dark:text-gray-100 mb-3 flex items-center gap-2">
                <span>⚖️</span> 相似词辨析
              </h4>
              <div class="space-y-3">
                <div v-for="(comp, index) in currentWord.comparison" :key="index" class="p-3 rounded-lg bg-gray-50 dark:bg-gray-800/50">
                  <div class="flex items-center gap-2 mb-2 font-medium">
                    <span class="text-primary-600 dark:text-primary-400">{{ currentWord.word }}</span>
                    <span class="text-gray-400">vs</span>
                    <span class="text-purple-600 dark:text-purple-400">{{ comp.word_to_compare }}</span>
                  </div>
                  <div class="text-sm text-gray-600 dark:text-gray-400">{{ comp.analysis }}</div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 移动端固定底部掌握程度选择 -->
    <div v-if="currentWord" class="mobile-quality-bar md:hidden">
      <div class="text-center text-sm font-medium text-gray-700 dark:text-gray-300 mb-3">
        掌握程度
      </div>
      <div class="grid grid-cols-3 gap-2">
        <button
          v-for="option in qualityOptions"
          :key="option.value"
          @click="markWord(option.value)"
          :disabled="submitting"
          :class="[
            'mobile-quality-btn',
            option.color === 'red' && 'mobile-red',
            option.color === 'yellow' && 'mobile-yellow',
            option.color === 'green' && 'mobile-green',
            submitting && 'opacity-50'
          ]"
        >
          <span class="text-xl">{{ option.icon }}</span>
          <span class="text-xs font-semibold">{{ option.label }}</span>
        </button>
      </div>
    </div>

    <!-- 加载状态 -->
    <div v-else-if="loading" class="card text-center py-20">
      <span class="text-5xl mb-4 block animate-bounce">⏳</span>
      <p class="text-gray-500 dark:text-gray-400">加载中...</p>
    </div>

    <!-- 错误提示 -->
    <div v-else-if="error" class="card text-center py-20">
      <span class="text-5xl mb-4 block">❌</span>
      <p class="text-red-600 dark:text-red-400 mb-6">{{ error }}</p>
      <button @click="loadRandomWord" class="btn btn-primary">
        重试
      </button>
    </div>

    <!-- 学习进度 -->
    <div v-if="learnedCount > 0" class="card text-center mt-8">
      <p class="text-gray-600 dark:text-gray-400">
        本次已学习 
        <span class="text-3xl font-bold text-gradient mx-2">{{ learnedCount }}</span> 
        个单词
      </p>
      <button @click="loadRandomWord" class="btn btn-primary mt-4">
        <span>📖</span>
        继续学习
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import SpeakerButton from '@/components/SpeakerButton.vue'
import { useDictionaryStore } from '@/stores/dictionary'
import { useLearningStore } from '@/stores/learning'
import { useUserStore } from '@/stores/user'
import { simpleQualityOptions } from '@/utils/sm2'
import { getPhonetic } from '@/utils/phonetic'

const dictionaryStore = useDictionaryStore()
const learningStore = useLearningStore()
const userStore = useUserStore()

const currentWord = ref(null)
const loading = ref(false)
const error = ref(null)
const submitting = ref(false)
const learnedCount = ref(0)
const showDefinition = ref(false)
const showConciseDefinition = ref(false)
const currentPhonetic = ref('')
const phoneticLoading = ref(false)

const qualityOptions = simpleQualityOptions

const isCollected = computed(() => {
  if (!currentWord.value) return false
  return learningStore.isCollected(currentWord.value.word)
})

async function fetchPhonetic() {
  if (!currentWord.value) return
  phoneticLoading.value = true
  try {
    const phonetic = await getPhonetic(currentWord.value.word, currentWord.value.pronunciation)
    currentPhonetic.value = phonetic
  } catch (err) {
    console.error('获取音标失败:', err)
    currentPhonetic.value = currentWord.value.pronunciation || ''
  } finally {
    phoneticLoading.value = false
  }
}

async function loadRandomWord() {
  loading.value = true
  error.value = null
  showDefinition.value = false
  showConciseDefinition.value = false
  
  try {
    const words = await dictionaryStore.loadRandomWords(1)
    if (words && words.length > 0) {
      const wordData = await dictionaryStore.getWordDetail(words[0].word)
      currentWord.value = wordData
      await fetchPhonetic()
    } else {
      error.value = '无法加载单词'
    }
  } catch (err) {
    error.value = '加载单词失败: ' + err.message
    console.error(err)
  } finally {
    loading.value = false
  }
}

async function markWord(quality) {
  if (!currentWord.value || submitting.value) return
  submitting.value = true

  if (userStore.isAuthenticated) {
    const result = await learningStore.updateWordProgress(currentWord.value.word, quality)
    if (!result.success) {
      console.error('保存学习进度失败')
    }
  }

  learnedCount.value++
  submitting.value = false

  setTimeout(() => {
    loadRandomWord()
  }, 300)
}

async function toggleCollection() {
  if (!currentWord.value || !userStore.isAuthenticated) return
  if (isCollected.value) {
    await learningStore.removeCollection(currentWord.value.word)
  } else {
    await learningStore.addCollection(currentWord.value.word)
  }
}

function handleKeyPress(e) {
  if (submitting.value) return
  if (e.key >= '1' && e.key <= '3') {
    e.preventDefault()
    const qualityIndex = parseInt(e.key) - 1
    if (qualityIndex < qualityOptions.length) {
      markWord(qualityOptions[qualityIndex].value)
    }
  }
}

onMounted(() => {
  loadRandomWord()
  window.addEventListener('keydown', handleKeyPress)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyPress)
})
</script>

<style scoped>
.study-container {
  max-width: 1200px;
  margin: 0 auto;
  padding-bottom: 140px;
}

@media (min-width: 768px) {
  .study-container {
    padding-bottom: 2rem;
  }
}

.main-content {
  display: grid;
  grid-template-columns: 1fr;
  gap: 2rem;
}

@media (min-width: 768px) {
  .main-content {
    grid-template-columns: 1fr 1fr;
  }
}

.word-section {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.definition-section {
  min-height: 400px;
  max-height: 600px;
  overflow: hidden;
}

/* 掌握程度按钮 */
.quality-btn {
  @apply p-5 rounded-2xl text-center transition-all duration-200;
  @apply border-2 hover:-translate-y-1 hover:shadow-lg;
}

.quality-red {
  @apply bg-red-50 dark:bg-red-900/30 border-red-200 dark:border-red-800;
  @apply text-red-700 dark:text-red-300;
  @apply hover:bg-red-100 dark:hover:bg-red-900/50;
}

.quality-yellow {
  @apply bg-amber-50 dark:bg-amber-900/30 border-amber-200 dark:border-amber-800;
  @apply text-amber-700 dark:text-amber-300;
  @apply hover:bg-amber-100 dark:hover:bg-amber-900/50;
}

.quality-green {
  @apply bg-green-50 dark:bg-green-900/30 border-green-200 dark:border-green-800;
  @apply text-green-700 dark:text-green-300;
  @apply hover:bg-green-100 dark:hover:bg-green-900/50;
}

/* 移动端底部栏 */
.mobile-quality-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 1rem;
  background: var(--glass-bg);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-top: 1px solid var(--glass-border);
  z-index: 40;
}

.mobile-quality-btn {
  @apply flex flex-col items-center gap-1 p-3 rounded-xl transition-all;
  @apply border-2 active:scale-95;
}

.mobile-red {
  @apply bg-red-50 dark:bg-red-900/50 border-red-200 dark:border-red-800 text-red-600 dark:text-red-400;
}

.mobile-yellow {
  @apply bg-amber-50 dark:bg-amber-900/50 border-amber-200 dark:border-amber-800 text-amber-600 dark:text-amber-400;
}

.mobile-green {
  @apply bg-green-50 dark:bg-green-900/50 border-green-200 dark:border-green-800 text-green-600 dark:text-green-400;
}
</style>
