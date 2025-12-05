<template>
  <button
    @click="handleSpeak"
    :disabled="isSpeaking"
    :class="[
      'sentence-speaker group',
      isSpeaking && 'speaking'
    ]"
    :title="isSpeaking ? '朗读中...' : '朗读句子'"
  >
    <svg
      v-if="!isSpeaking"
      xmlns="http://www.w3.org/2000/svg"
      class="h-3.5 w-3.5"
      viewBox="0 0 20 20"
      fill="currentColor"
    >
      <path d="M9.383 2.058a1 1 0 011.234 0l4.25 2.833A1 1 0 0115 6v8a1 1 0 01-.383.792l-4.25 2.833a1 1 0 01-1.234 0L4.75 14.792A1 1 0 014 14V6a1 1 0 01.75-.792l4.633-3.084z"/>
    </svg>
    <div
      v-else
      class="w-3.5 h-3.5 border-2 border-current border-t-transparent rounded-full animate-spin"
    ></div>
    <span class="hidden sm:inline text-xs font-medium">{{ isSpeaking ? '朗读中' : '朗读' }}</span>
  </button>
</template>

<script setup>
import { ref } from 'vue'
import { speakSentence } from '@/utils/tts'

const props = defineProps({
  text: {
    type: String,
    required: true
  },
  lang: {
    type: String,
    default: 'en-US'
  },
  speed: {
    type: Number,
    default: 0.8
  }
})

const isSpeaking = ref(false)

const handleSpeak = async () => {
  if (isSpeaking.value || !props.text) return
  
  isSpeaking.value = true
  
  try {
    setTimeout(async () => {
      try {
        await speakSentence(props.text, {
          lang: props.lang,
          speed: props.speed
        })
      } catch (error) {
        console.warn('朗读句子失败:', error)
      } finally {
        const estimatedTime = Math.max(props.text.length * 50, 1000)
        setTimeout(() => {
          isSpeaking.value = false
        }, estimatedTime)
      }
    }, 50)
  } catch (error) {
    console.warn('朗读句子失败:', error)
    isSpeaking.value = false
  }
}
</script>

<style scoped>
.sentence-speaker {
  @apply inline-flex items-center gap-1.5 px-3 py-1.5 rounded-lg;
  @apply text-gray-600 dark:text-gray-400;
  @apply bg-gray-100 dark:bg-gray-800;
  @apply hover:bg-primary-100 dark:hover:bg-primary-900/50;
  @apply hover:text-primary-600 dark:hover:text-primary-400;
  @apply transition-all duration-200;
  @apply disabled:opacity-50 disabled:cursor-not-allowed;
}

.sentence-speaker.speaking {
  @apply bg-primary-100 dark:bg-primary-900/50;
  @apply text-primary-600 dark:text-primary-400;
}
</style>
