<template>
  <button
    @click="handleSpeak"
    :disabled="isLoading"
    :class="[
      'speaker-button group',
      isLoading && 'animate-pulse',
      isPlaying && 'playing'
    ]"
    :title="isPlaying ? '停止发音' : '播放发音'"
  >
    <div class="relative">
      <!-- 播放图标 -->
      <svg
        v-if="!isPlaying"
        class="w-5 h-5 text-gray-500 dark:text-gray-400 group-hover:text-primary-600 dark:group-hover:text-primary-400 transition-colors"
        fill="currentColor"
        viewBox="0 0 24 24"
      >
        <path d="M3 9v6h4l5 5V4L7 9H3zm13.5 3c0-1.77-1.02-3.29-2.5-4.03v8.05c1.48-.73 2.5-2.25 2.5-4.02zM14 3.23v2.06c2.89.86 5 3.54 5 6.71s-2.11 5.85-5 6.71v2.06c4.01-.91 7-4.49 7-8.77s-2.99-7.86-7-8.77z"/>
      </svg>
      
      <!-- 停止图标 -->
      <svg
        v-else
        class="w-5 h-5 text-primary-600 dark:text-primary-400"
        fill="currentColor"
        viewBox="0 0 24 24"
      >
        <path d="M6 6h12v12H6z"/>
      </svg>
      
      <!-- 加载动画 -->
      <div
        v-if="isLoading"
        class="absolute inset-0 flex items-center justify-center"
      >
        <div class="w-4 h-4 border-2 border-primary-500 border-t-transparent rounded-full animate-spin"></div>
      </div>
    </div>
  </button>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { speakText, stopSpeaking, isSpeaking } from '@/utils/tts'

const props = defineProps({
  text: {
    type: String,
    required: true
  },
  lang: {
    type: String,
    default: 'en'
  },
  speed: {
    type: Number,
    default: 1.0
  }
})

const isLoading = ref(false)
const isPlaying = ref(false)

let statusCheckInterval = null

onMounted(() => {
  statusCheckInterval = setInterval(() => {
    isPlaying.value = isSpeaking()
  }, 100)
})

onUnmounted(() => {
  if (statusCheckInterval) {
    clearInterval(statusCheckInterval)
  }
  stopSpeaking()
})

async function handleSpeak() {
  if (isPlaying.value) {
    stopSpeaking()
    isPlaying.value = false
    return
  }

  if (!props.text) {
    console.warn('没有要播放的文本')
    return
  }

  isLoading.value = true
  
  try {
    const success = await speakText(props.text, props.lang, props.speed)
    if (success) {
      isPlaying.value = true
    } else {
      console.error('发音播放失败')
    }
  } catch (error) {
    console.error('发音播放出错:', error)
  } finally {
    isLoading.value = false
  }
}
</script>

<style scoped>
.speaker-button {
  @apply p-2.5 rounded-xl transition-all duration-200;
  @apply bg-gray-100 dark:bg-gray-800;
  @apply hover:bg-primary-100 dark:hover:bg-primary-900/50;
  @apply hover:shadow-md hover:-translate-y-0.5;
  @apply focus:outline-none focus-visible:ring-2 focus-visible:ring-primary-500 focus-visible:ring-offset-2;
  @apply disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:transform-none;
  min-width: 2.5rem;
  min-height: 2.5rem;
}

.speaker-button.playing {
  @apply bg-primary-100 dark:bg-primary-900/50;
  @apply ring-2 ring-primary-500/50;
}
</style>
