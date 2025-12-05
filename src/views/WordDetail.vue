<template>
  <div class="max-w-4xl mx-auto">
    <!-- Toast 提示 -->
    <teleport to="body">
      <transition name="toast">
        <div
          v-if="showMessage"
          class="fixed top-6 right-6 z-50 glass rounded-2xl shadow-soft-lg border border-green-200 dark:border-green-800 p-4 max-w-sm"
        >
          <div class="flex items-center gap-3">
            <div class="w-10 h-10 rounded-xl bg-green-100 dark:bg-green-900/50 flex items-center justify-center flex-shrink-0">
              <span class="text-xl">✅</span>
            </div>
            <p class="text-green-800 dark:text-green-200 font-medium">{{ messageText }}</p>
          </div>
        </div>
      </transition>
    </teleport>

    <!-- 加载状态 -->
    <div v-if="loading" class="card text-center py-20">
      <span class="text-5xl mb-4 block animate-bounce">⏳</span>
      <p class="text-gray-500 dark:text-gray-400">加载中...</p>
    </div>

    <!-- 单词详情 -->
    <div v-else-if="wordData" class="space-y-6">
      <!-- 标题卡片 -->
      <div class="card relative overflow-hidden">
        <!-- 装饰背景 -->
        <div class="absolute inset-0 bg-gradient-to-br from-primary-50 via-transparent to-purple-50 dark:from-primary-900/20 dark:to-purple-900/20"></div>
        
        <div class="relative z-10">
          <div class="flex flex-col md:flex-row md:items-start md:justify-between gap-6">
            <div class="flex-1">
              <div class="flex items-center gap-4 mb-4">
                <h1 class="text-4xl md:text-5xl font-bold text-gray-900 dark:text-gray-100">
                  {{ wordData.word }}
                </h1>
                <SpeakerButton :text="wordData.word" :lang="'en'" :speed="0.8" />
              </div>
              
              <div v-if="currentPhonetic || phoneticLoading" class="mb-4">
                <div v-if="phoneticLoading" class="inline-flex items-center gap-2 text-gray-500 dark:text-gray-400">
                  <span class="animate-spin">⏳</span>
                  获取音标中...
                </div>
                <p v-else-if="currentPhonetic" class="text-xl text-gray-500 dark:text-gray-400 font-mono">
                  [{{ currentPhonetic }}]
                </p>
              </div>
              
              <div class="p-4 rounded-xl bg-primary-50 dark:bg-primary-900/30 border border-primary-100 dark:border-primary-900/50">
                <p class="text-primary-800 dark:text-primary-200 leading-relaxed">
                  {{ wordData.concise_definition }}
                </p>
              </div>
            </div>

            <div v-if="userStore.isAuthenticated" class="flex flex-row md:flex-col gap-3">
              <button @click="addToLearning" class="btn btn-primary flex-1 md:flex-none">
                <span>📚</span>
                加入学习
              </button>
              <button
                @click="toggleCollection"
                :class="['btn flex-1 md:flex-none', isCollected ? 'btn-primary' : 'btn-outline']"
              >
                {{ isCollected ? '⭐ 已收藏' : '☆ 收藏' }}
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- 词形变化 -->
      <div v-if="wordData.forms && Object.keys(wordData.forms).length > 0" class="card">
        <h2 class="text-xl font-bold text-gray-900 dark:text-gray-100 mb-4 flex items-center gap-3">
          <span class="text-2xl">🔄</span>
          词形变化
        </h2>
        <div class="flex flex-wrap gap-3">
          <div
            v-for="(value, key) in wordData.forms"
            :key="key"
            class="badge badge-primary text-sm py-2 px-4"
          >
            <span class="opacity-70 mr-2">{{ key }}</span>
            <span class="font-semibold">{{ value }}</span>
          </div>
        </div>
      </div>

      <!-- 详细释义 -->
      <div v-if="wordData.definitions" class="card">
        <h2 class="text-xl font-bold text-gray-900 dark:text-gray-100 mb-6 flex items-center gap-3">
          <span class="text-2xl">📚</span>
          详细释义
        </h2>
        <div class="space-y-6">
          <div
            v-for="(def, index) in wordData.definitions"
            :key="index"
            class="p-5 rounded-xl bg-gray-50 dark:bg-gray-800/50 border-l-4 border-primary-500"
          >
            <span class="badge badge-primary mb-4">{{ def.pos }}</span>
            
            <div class="space-y-4">
              <div>
                <div class="text-xs font-medium text-gray-500 dark:text-gray-400 mb-1 uppercase tracking-wide">英文解释</div>
                <p class="text-gray-600 dark:text-gray-300 leading-relaxed">{{ def.explanation_en }}</p>
              </div>
              
              <div>
                <div class="text-xs font-medium text-gray-500 dark:text-gray-400 mb-1 uppercase tracking-wide">中文解释</div>
                <p class="text-gray-900 dark:text-gray-100 font-medium leading-relaxed">{{ def.explanation_cn }}</p>
              </div>
              
              <div v-if="def.example_en" class="p-4 rounded-xl bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700">
                <div class="flex items-center justify-between mb-3">
                  <div class="text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wide">例句</div>
                  <SentenceSpeaker :text="def.example_en" :lang="'en'" :speed="0.8" />
                </div>
                <p class="text-gray-600 dark:text-gray-400 italic mb-2 leading-relaxed">{{ def.example_en }}</p>
                <p class="text-gray-800 dark:text-gray-200 leading-relaxed">{{ def.example_cn }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 近义词对比 -->
      <div v-if="wordData.comparison && wordData.comparison.length > 0" class="card">
        <h2 class="text-xl font-bold text-gray-900 dark:text-gray-100 mb-6 flex items-center gap-3">
          <span class="text-2xl">⚖️</span>
          相似词辨析
        </h2>
        <div class="space-y-4">
          <div
            v-for="(comp, index) in wordData.comparison"
            :key="index"
            class="p-5 rounded-xl bg-gray-50 dark:bg-gray-800/50"
          >
            <div class="flex items-center gap-3 mb-3">
              <span class="font-bold text-lg text-primary-600 dark:text-primary-400">{{ wordData.word }}</span>
              <span class="text-gray-400 font-medium">vs</span>
              <router-link
                :to="{ name: 'WordDetail', params: { word: comp.word_to_compare } }"
                class="font-bold text-lg text-purple-600 dark:text-purple-400 hover:underline"
              >
                {{ comp.word_to_compare }}
              </router-link>
            </div>
            <p class="text-gray-600 dark:text-gray-300 leading-relaxed">{{ comp.analysis }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- 错误提示 -->
    <div v-else class="card max-w-md mx-auto text-center py-16">
      <div class="w-20 h-20 mx-auto mb-6 rounded-3xl bg-gradient-to-br from-red-100 to-red-200 dark:from-red-900/50 dark:to-red-800/50 flex items-center justify-center">
        <span class="text-4xl">❌</span>
      </div>
      <h2 class="text-2xl font-bold text-gray-900 dark:text-gray-100 mb-3">未找到单词</h2>
      <p class="text-red-600 dark:text-red-400 mb-8">{{ error || '该单词不存在或加载失败' }}</p>
      <button @click="$router.back()" class="btn btn-primary">
        <span>←</span>
        返回
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { useRoute } from 'vue-router'
import { useDictionaryStore } from '@/stores/dictionary'
import { useLearningStore } from '@/stores/learning'
import { useUserStore } from '@/stores/user'
import SpeakerButton from '@/components/SpeakerButton.vue'
import SentenceSpeaker from '@/components/SentenceSpeaker.vue'
import { getPhonetic } from '@/utils/phonetic'
import { autoSpeak } from '@/utils/tts'

const route = useRoute()
const dictionaryStore = useDictionaryStore()
const learningStore = useLearningStore()
const userStore = useUserStore()

const wordData = ref(null)
const loading = ref(false)
const error = ref(null)
const currentPhonetic = ref('')
const phoneticLoading = ref(false)
const showMessage = ref(false)
const messageText = ref('')

const isCollected = computed(() => {
  if (!wordData.value) return false
  return learningStore.isCollected(wordData.value.word)
})

async function fetchPhonetic() {
  if (!wordData.value?.word) return
  phoneticLoading.value = true
  try {
    const phonetic = await getPhonetic(wordData.value.word, wordData.value.pronunciation)
    currentPhonetic.value = phonetic || ''
  } catch (err) {
    console.warn('获取音标失败:', err)
    currentPhonetic.value = wordData.value.pronunciation || ''
  } finally {
    phoneticLoading.value = false
  }
}

async function loadWord() {
  const word = route.params.word
  if (!word) return

  loading.value = true
  error.value = null

  try {
    wordData.value = await dictionaryStore.getWordDetail(word)
    if (wordData.value) {
      await fetchPhonetic()
      setTimeout(async () => {
        await autoSpeak(wordData.value.word, 'wordDetail')
      }, 200)
    }
  } catch (err) {
    error.value = '加载单词详情失败: ' + err.message
    console.error(err)
  } finally {
    loading.value = false
  }
}

async function toggleCollection() {
  if (!wordData.value || !userStore.isAuthenticated) return
  if (isCollected.value) {
    await learningStore.removeCollection(wordData.value.word)
  } else {
    await learningStore.addCollection(wordData.value.word)
  }
}

function showMessageFunc(text) {
  messageText.value = text
  showMessage.value = true
  setTimeout(() => {
    showMessage.value = false
  }, 3000)
}

async function addToLearning() {
  if (!wordData.value || !userStore.isAuthenticated) return
  try {
    const result = await learningStore.updateWordProgress(wordData.value.word, 1)
    if (result.success) {
      showMessageFunc('已加入学习！系统会定期提醒你复习。')
    } else {
      showMessageFunc('加入学习失败，请稍后重试。')
    }
  } catch (err) {
    console.error('加入学习失败:', err)
    showMessageFunc('加入学习失败，请稍后重试。')
  }
}

watch(() => route.params.word, () => loadWord())
onMounted(() => loadWord())
</script>

<style scoped>
.toast-enter-active,
.toast-leave-active {
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.toast-enter-from,
.toast-leave-to {
  opacity: 0;
  transform: translateX(20px);
}
</style>
