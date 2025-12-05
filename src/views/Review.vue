<template>
  <div class="review-container">
    <!-- 页面标题 -->
    <div class="text-center mb-8">
      <div class="inline-flex items-center gap-3 mb-3">
        <div class="w-12 h-12 rounded-2xl bg-gradient-to-br from-green-500 to-green-600 flex items-center justify-center">
          <span class="text-2xl">🔄</span>
        </div>
        <h1 class="text-2xl md:text-3xl font-bold text-gray-900 dark:text-gray-100">
          复习模式
        </h1>
      </div>
      <p class="text-gray-500 dark:text-gray-400">
        根据记忆曲线复习需要巩固的单词
      </p>
    </div>

    <!-- 待复习单词列表 -->
    <div v-if="!reviewMode && dueWords.length > 0" class="max-w-2xl mx-auto space-y-6">
      <div class="card text-center">
        <div class="w-16 h-16 mx-auto mb-4 rounded-2xl bg-gradient-to-br from-amber-100 to-amber-200 dark:from-amber-900/50 dark:to-amber-800/50 flex items-center justify-center">
          <span class="text-3xl">📝</span>
        </div>
        <h2 class="text-2xl font-bold text-gray-900 dark:text-gray-100 mb-2">
          今日待复习
        </h2>
        <p class="text-4xl font-bold text-gradient mb-6">{{ dueWords.length }} 个单词</p>
        <button @click="startReview" class="btn btn-primary btn-lg w-full">
          <span>🚀</span>
          开始复习
        </button>
      </div>

      <div class="card">
        <h3 class="font-bold text-gray-900 dark:text-gray-100 mb-4 flex items-center gap-2">
          <span>📋</span> 待复习列表
        </h3>
        <div class="space-y-2">
          <div
            v-for="item in dueWords.slice(0, 10)"
            :key="item.word"
            class="p-3 rounded-xl bg-gray-50 dark:bg-gray-800/50 flex justify-between items-center hover:bg-gray-100 dark:hover:bg-gray-800 transition-colors"
          >
            <span class="font-medium text-gray-900 dark:text-gray-100">{{ item.word }}</span>
            <span class="text-sm text-gray-500 dark:text-gray-400 badge badge-warning">
              {{ formatNextReview(item.next_review) }}
            </span>
          </div>
          <p v-if="dueWords.length > 10" class="text-sm text-gray-500 dark:text-gray-400 text-center pt-3">
            还有 {{ dueWords.length - 10 }} 个单词...
          </p>
        </div>
      </div>
    </div>

    <!-- 复习模式 -->
    <div v-else-if="reviewMode && currentWord" class="max-w-4xl mx-auto">
      <!-- 进度条 -->
      <div class="mb-8">
        <div class="flex items-center justify-between mb-2">
          <span class="text-sm font-medium text-gray-600 dark:text-gray-400">复习进度</span>
          <span class="badge badge-primary">{{ reviewedCount }} / {{ totalDueWords }}</span>
        </div>
        <div class="progress">
          <div class="progress-bar" :style="{ width: `${(reviewedCount / totalDueWords) * 100}%` }"></div>
        </div>
      </div>

      <!-- 单词卡片 -->
      <div class="card text-center py-12 mb-6 relative overflow-hidden">
        <div class="absolute inset-0 bg-gradient-to-br from-green-50 via-transparent to-primary-50 dark:from-green-900/20 dark:to-primary-900/20"></div>
        
        <div class="relative z-10">
          <div class="text-4xl md:text-5xl font-bold text-gray-900 dark:text-gray-100 mb-4">
            {{ currentWord.word }}
          </div>
          <div v-if="currentPhonetic" class="text-lg text-gray-500 dark:text-gray-400 font-mono mb-6">
            [{{ currentPhonetic }}]
          </div>
          <div v-else-if="phoneticLoading" class="flex items-center justify-center gap-2 text-gray-500 mb-6">
            <span class="animate-spin">⏳</span>
            <span>加载音标中...</span>
          </div>
          
          <!-- 简洁定义 -->
          <div v-if="currentWord.concise_definition" class="max-w-lg mx-auto px-4">
            <div 
              @click="showConciseDefinition = !showConciseDefinition"
              class="p-4 rounded-xl bg-green-50 dark:bg-green-900/30 border border-green-100 dark:border-green-900/50 cursor-pointer transition-all hover:shadow-md"
            >
              <div :class="['text-green-800 dark:text-green-200 leading-relaxed transition-all', showConciseDefinition ? '' : 'blur-sm select-none']">
                {{ currentWord.concise_definition }}
              </div>
              <div class="text-xs text-green-600 dark:text-green-400 mt-2 font-medium">
                {{ showConciseDefinition ? '👆 点击隐藏' : '👆 点击显示释义' }}
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 操作按钮 -->
      <div class="flex items-center justify-center gap-4 mb-8">
        <SpeakerButton :word="currentWord.word" :text="currentWord.word" :lang="'en'" :speed="1.0" />
        <button @click="showDefinition = !showDefinition" class="btn btn-outline">
          {{ showDefinition ? '隐藏释义' : '📖 查看释义' }}
        </button>
      </div>

      <!-- 释义区域 -->
      <transition name="slide-down">
        <div v-if="showDefinition" class="card mb-8">
          <h3 class="font-bold text-gray-900 dark:text-gray-100 mb-4 flex items-center gap-2">
            <span>📚</span> {{ currentWord.word }} 的释义
          </h3>
          <div class="space-y-4">
            <div v-for="(def, index) in currentWord.definitions" :key="index" class="p-4 rounded-xl bg-gray-50 dark:bg-gray-800/50">
              <span class="badge badge-primary mb-3">{{ def.pos }}</span>
              <div class="space-y-2">
                <p class="text-gray-600 dark:text-gray-400 text-sm">{{ def.explanation_en }}</p>
                <p class="text-gray-900 dark:text-gray-100 font-medium">{{ def.explanation_cn }}</p>
                <div v-if="def.example_en" class="p-3 rounded-lg bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700 mt-3">
                  <p class="text-gray-600 dark:text-gray-400 italic text-sm">{{ def.example_en }}</p>
                  <p class="text-gray-800 dark:text-gray-200 text-sm mt-1">{{ def.example_cn }}</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </transition>

      <!-- 掌握程度选择 -->
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
            :class="['quality-btn group', `quality-${option.color}`, submitting && 'opacity-50 cursor-not-allowed']"
          >
            <div class="text-3xl mb-2 group-hover:scale-110 transition-transform">{{ option.icon }}</div>
            <div class="font-bold mb-1">{{ option.label }}</div>
            <div class="text-xs opacity-80">{{ option.description }}</div>
          </button>
        </div>
      </div>

      <!-- 移动端底部栏 -->
      <div class="mobile-quality-bar md:hidden">
        <div class="text-center text-sm font-medium text-gray-700 dark:text-gray-300 mb-3">掌握程度</div>
        <div class="grid grid-cols-3 gap-2">
          <button
            v-for="option in qualityOptions"
            :key="option.value"
            @click="markWord(option.value)"
            :disabled="submitting"
            :class="['mobile-quality-btn', `mobile-${option.color}`, submitting && 'opacity-50']"
          >
            <span class="text-xl">{{ option.icon }}</span>
            <span class="text-xs font-semibold">{{ option.label }}</span>
          </button>
        </div>
      </div>
    </div>

    <!-- 无待复习单词 -->
    <div v-else-if="!loading && dueWords.length === 0" class="card max-w-md mx-auto text-center py-16">
      <div class="w-20 h-20 mx-auto mb-6 rounded-3xl bg-gradient-to-br from-green-100 to-green-200 dark:from-green-900/50 dark:to-green-800/50 flex items-center justify-center">
        <span class="text-4xl">🎉</span>
      </div>
      <h2 class="text-2xl font-bold text-gray-900 dark:text-gray-100 mb-3">太棒了！</h2>
      <p class="text-gray-600 dark:text-gray-400 mb-8">暂时没有需要复习的单词</p>
      <router-link to="/study" class="btn btn-primary">
        <span>📖</span> 去学习新单词
      </router-link>
    </div>

    <!-- 完成复习 -->
    <div v-else-if="reviewMode && !currentWord && reviewedCount > 0" class="card max-w-md mx-auto text-center py-16">
      <div class="w-20 h-20 mx-auto mb-6 rounded-3xl bg-gradient-to-br from-green-400 to-green-500 flex items-center justify-center">
        <span class="text-4xl">✅</span>
      </div>
      <h2 class="text-2xl font-bold text-gray-900 dark:text-gray-100 mb-3">复习完成！</h2>
      <p class="text-gray-600 dark:text-gray-400 mb-2">本次复习了</p>
      <p class="text-4xl font-bold text-gradient mb-8">{{ reviewedCount }} 个单词</p>
      <button @click="exitReview" class="btn btn-primary">返回</button>
    </div>

    <!-- 加载状态 -->
    <div v-else-if="loading" class="card max-w-md mx-auto text-center py-16">
      <span class="text-5xl mb-4 block animate-bounce">⏳</span>
      <p class="text-gray-500 dark:text-gray-400">加载中...</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import SpeakerButton from '@/components/SpeakerButton.vue'
import { useDictionaryStore } from '@/stores/dictionary'
import { useLearningStore } from '@/stores/learning'
import { simpleQualityOptions, formatNextReview } from '@/utils/sm2'
import { getPhonetic } from '@/utils/phonetic'

const dictionaryStore = useDictionaryStore()
const learningStore = useLearningStore()

const reviewMode = ref(false)
const currentWord = ref(null)
const currentIndex = ref(0)
const loading = ref(false)
const submitting = ref(false)
const reviewedCount = ref(0)
const totalDueWords = ref(0)
const showDefinition = ref(false)
const showConciseDefinition = ref(false)
const currentPhonetic = ref('')
const phoneticLoading = ref(false)

const qualityOptions = simpleQualityOptions
const dueWords = computed(() => learningStore.dueWords)

async function fetchPhonetic() {
  if (!currentWord.value) return
  phoneticLoading.value = true
  try {
    const phonetic = await getPhonetic(currentWord.value.word, currentWord.value.pronunciation)
    currentPhonetic.value = phonetic
  } catch (error) {
    console.error('获取音标失败:', error)
    currentPhonetic.value = currentWord.value.pronunciation || ''
  } finally {
    phoneticLoading.value = false
  }
}

async function startReview() {
  reviewMode.value = true
  currentIndex.value = 0
  reviewedCount.value = 0
  totalDueWords.value = dueWords.value.length
  showDefinition.value = false
  showConciseDefinition.value = false
  await loadNextWord()
}

async function loadNextWord() {
  if (currentIndex.value >= dueWords.value.length) {
    currentWord.value = null
    return
  }
  loading.value = true
  showDefinition.value = false
  showConciseDefinition.value = false
  
  try {
    const wordItem = dueWords.value[currentIndex.value]
    const wordData = await dictionaryStore.getWordDetail(wordItem.word)
    currentWord.value = wordData
    await fetchPhonetic()
  } catch (err) {
    console.error('加载单词失败:', err)
  } finally {
    loading.value = false
  }
}

async function markWord(quality) {
  if (!currentWord.value || submitting.value) return
  submitting.value = true
  await learningStore.updateWordProgress(currentWord.value.word, quality)
  reviewedCount.value++
  currentIndex.value++
  submitting.value = false
  setTimeout(() => loadNextWord(), 300)
}

function exitReview() {
  reviewMode.value = false
  currentWord.value = null
  currentIndex.value = 0
  reviewedCount.value = 0
}

onMounted(async () => {
  loading.value = true
  await learningStore.loadDueWords()
  loading.value = false
})
</script>

<style scoped>
.review-container {
  max-width: 1000px;
  margin: 0 auto;
  padding-bottom: 140px;
}

@media (min-width: 768px) {
  .review-container {
    padding-bottom: 2rem;
  }
}

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

.slide-down-enter-active,
.slide-down-leave-active {
  transition: all 0.3s ease;
}

.slide-down-enter-from,
.slide-down-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}
</style>
