<template>
  <div class="flip-card" :class="{ 'flipped': isFlipped }">
    <div class="flip-card-inner">
      <!-- 正面：单词 -->
      <div class="flip-card-front">
        <div class="card h-full flex flex-col items-center justify-center relative overflow-hidden">
          <!-- 装饰背景 -->
          <div class="absolute inset-0 bg-gradient-to-br from-primary-50 via-transparent to-purple-50 dark:from-primary-900/30 dark:to-purple-900/30 opacity-50"></div>
          
          <div class="relative z-10 text-center px-6">
            <h2 class="text-4xl md:text-5xl font-bold text-gray-900 dark:text-gray-100 mb-8">
              {{ word }}
            </h2>
            
            <button
              @click="isFlipped = true"
              class="btn btn-primary btn-lg group"
            >
              <span class="group-hover:scale-110 transition-transform">📖</span>
              查看释义
            </button>

            <div class="mt-6 text-sm text-gray-500 dark:text-gray-400 flex items-center justify-center gap-2">
              <kbd>空格</kbd>
              <span>翻转卡片</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 背面：释义 -->
      <div class="flip-card-back">
        <div class="card word-detail-card h-full">
          <!-- 紧凑头部 -->
          <div class="word-header">
            <div class="flex items-center justify-center gap-4">
              <h2 class="text-2xl md:text-3xl font-bold text-gray-900 dark:text-gray-100">
                {{ wordData?.word }}
              </h2>
              <SpeakerButton 
                :text="wordData?.word || ''"
                :lang="'en'"
                :speed="0.8"
              />
            </div>
            <div v-if="currentPhonetic || phoneticLoading" class="mt-2 text-center">
              <div v-if="phoneticLoading" class="inline-flex items-center gap-2 text-gray-500 dark:text-gray-400 text-sm">
                <span class="animate-spin">⏳</span>
                获取音标中...
              </div>
              <p v-else-if="currentPhonetic" class="text-gray-600 dark:text-gray-400 font-mono">
                [{{ currentPhonetic }}]
              </p>
            </div>
          </div>

          <!-- 可滚动内容区域 -->
          <div class="word-content">
            <!-- 简明释义 -->
            <div v-if="wordData?.concise_definition" class="mb-5 p-4 rounded-xl bg-gradient-to-r from-primary-50 to-purple-50 dark:from-primary-900/30 dark:to-purple-900/30 border border-primary-100 dark:border-primary-900/50">
              <h3 class="text-xs font-bold text-primary-600 dark:text-primary-400 mb-2 flex items-center gap-2">
                <span>💡</span> 简明释义
              </h3>
              <p class="text-gray-800 dark:text-gray-200 leading-relaxed">
                {{ wordData.concise_definition }}
              </p>
            </div>

            <!-- 详细释义 -->
            <div v-if="wordData?.definitions && wordData.definitions.length > 0" class="space-y-4">
              <h3 class="text-sm font-bold text-gray-900 dark:text-gray-100 flex items-center gap-2">
                <span>📚</span> 详细释义
              </h3>
              <div
                v-for="(def, index) in wordData.definitions"
                :key="index"
                class="p-4 rounded-xl bg-gray-50 dark:bg-gray-800/50 border-l-4 border-primary-500"
              >
                <span class="badge badge-primary mb-3">{{ def.pos }}</span>
                <p class="text-gray-700 dark:text-gray-300 mb-3 leading-relaxed">
                  {{ def.explanation_cn }}
                </p>
                <div v-if="def.example_en" class="p-3 rounded-lg bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700">
                  <div class="flex items-start justify-between gap-3 mb-2">
                    <p class="text-gray-600 dark:text-gray-400 italic text-sm leading-relaxed flex-1" v-html="highlightWord(def.example_en, wordData?.word)"></p>
                    <SentenceSpeaker 
                      :text="def.example_en"
                      :lang="'en'"
                      :speed="0.8"
                    />
                  </div>
                  <p class="text-gray-700 dark:text-gray-300 text-sm leading-relaxed" v-html="highlightWord(def.example_cn, wordData?.word)"></p>
                </div>
              </div>
            </div>
          </div>

          <!-- 底部按钮 -->
          <div class="word-footer">
            <button
              @click="isFlipped = false"
              class="w-full btn btn-secondary"
            >
              <span>↩️</span>
              返回单词
              <kbd class="ml-2">Esc</kbd>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue'
import SpeakerButton from './SpeakerButton.vue'
import SentenceSpeaker from './SentenceSpeaker.vue'
import { getPhonetic } from '@/utils/phonetic'
import { autoSpeak } from '@/utils/tts'

const props = defineProps({
  word: {
    type: String,
    required: true
  },
  wordData: {
    type: Object,
    default: null
  }
})

const isFlipped = ref(false)
const currentPhonetic = ref('')
const phoneticLoading = ref(false)

function highlightWord(text, word) {
  if (!text || !word) return text
  const regex = new RegExp(`\\b(${word.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')})\\b`, 'gi')
  return text.replace(regex, '<span class="highlight-word">$1</span>')
}

async function fetchPhonetic() {
  if (!props.word) return
  
  phoneticLoading.value = true
  
  try {
    const phonetic = await getPhonetic(props.word, props.wordData?.pronunciation)
    currentPhonetic.value = phonetic || ''
  } catch (error) {
    console.warn('获取音标失败:', error)
    currentPhonetic.value = props.wordData?.pronunciation || ''
  } finally {
    phoneticLoading.value = false
  }
}

function handleKeyPress(e) {
  if (e.code === 'Space') {
    e.preventDefault()
    isFlipped.value = !isFlipped.value
  } else if (e.code === 'Escape' && isFlipped.value) {
    e.preventDefault()
    isFlipped.value = false
  }
}

watch(() => props.word, () => {
  isFlipped.value = false
  fetchPhonetic()
})

watch(() => props.wordData, async () => {
  if (props.wordData) {
    fetchPhonetic()
    setTimeout(async () => {
      await autoSpeak(props.wordData.word, 'study')
    }, 100)
  }
})

onMounted(() => {
  window.addEventListener('keydown', handleKeyPress)
  if (props.word) {
    fetchPhonetic()
  }
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyPress)
})
</script>

<style scoped>
.flip-card {
  background-color: transparent;
  width: 100%;
  min-height: 400px;
  max-height: 600px;
  height: 550px;
  perspective: 1000px;
  margin-bottom: 2rem;
}

@media (min-width: 768px) {
  .flip-card {
    min-height: 450px;
    max-height: 650px;
    height: 600px;
    margin-bottom: 3rem;
  }
}

.flip-card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  text-align: center;
  transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
  transform-style: preserve-3d;
}

.flip-card.flipped .flip-card-inner {
  transform: rotateY(180deg);
}

.flip-card-front, .flip-card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  -webkit-backface-visibility: hidden;
  backface-visibility: hidden;
}

.flip-card-back {
  transform: rotateY(180deg);
}

.flip-card-front .card,
.flip-card-back .card {
  height: 100%;
}

.word-detail-card {
  display: grid;
  grid-template-rows: auto 1fr auto;
  height: 100%;
}

.word-header {
  padding: 16px 20px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
  background: linear-gradient(to bottom, rgba(255,255,255,0.8), rgba(255,255,255,0));
}

.dark .word-header {
  border-bottom-color: rgba(255, 255, 255, 0.05);
  background: linear-gradient(to bottom, rgba(31,41,55,0.8), rgba(31,41,55,0));
}

.word-content {
  overflow-y: auto;
  padding: 20px;
  min-height: 0;
  text-align: left;
}

.word-footer {
  padding: 16px 20px;
  border-top: 1px solid rgba(0, 0, 0, 0.05);
  background: linear-gradient(to top, rgba(255,255,255,0.8), rgba(255,255,255,0));
}

.dark .word-footer {
  border-top-color: rgba(255, 255, 255, 0.05);
  background: linear-gradient(to top, rgba(31,41,55,0.8), rgba(31,41,55,0));
}

.highlight-word {
  background: linear-gradient(120deg, #fbbf24 0%, #f59e0b 100%);
  color: #1f2937;
  font-weight: 600;
  padding: 2px 6px;
  border-radius: 6px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.dark .highlight-word {
  background: linear-gradient(120deg, #fbbf24 0%, #f59e0b 100%);
  color: #111827;
}

.word-content::-webkit-scrollbar {
  width: 6px;
}

.word-content::-webkit-scrollbar-track {
  background: transparent;
  border-radius: 3px;
}

.word-content::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 3px;
}

.word-content::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}

.dark .word-content::-webkit-scrollbar-thumb {
  background: #4b5563;
}

.dark .word-content::-webkit-scrollbar-thumb:hover {
  background: #6b7280;
}
</style>
