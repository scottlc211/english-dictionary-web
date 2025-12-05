<template>
  <div class="relative">
    <div class="relative group">
      <input
        v-model="searchQuery"
        @input="handleSearch"
        @focus="showResults = true"
        type="text"
        placeholder="搜索单词..."
        class="input input-lg pr-14 shadow-soft group-hover:shadow-soft-lg transition-shadow"
      />
      <span class="absolute right-4 top-1/2 transform -translate-y-1/2 text-2xl opacity-50 group-hover:opacity-100 transition-opacity">
        🔍
      </span>
    </div>

    <!-- 搜索结果下拉框 -->
    <transition name="dropdown">
      <div 
        v-if="showResults && (results.length > 0 || loading)"
        class="absolute z-20 w-full mt-3 glass rounded-2xl shadow-soft-lg max-h-96 overflow-y-auto border border-white/30 dark:border-gray-700/50"
      >
        <!-- 加载中 -->
        <div v-if="loading" class="p-6 text-center">
          <div class="inline-flex items-center gap-3 text-gray-500 dark:text-gray-400">
            <span class="animate-spin text-xl">⏳</span>
            <span>搜索中...</span>
          </div>
        </div>

        <!-- 搜索结果 -->
        <div v-else>
          <button
            v-for="(word, index) in results"
            :key="word.word"
            @click="selectWord(word.word)"
            class="w-full px-5 py-4 text-left hover:bg-primary-50 dark:hover:bg-primary-900/50 transition-colors border-b border-gray-100/50 dark:border-gray-700/50 last:border-b-0 group"
            :class="{ 'rounded-t-2xl': index === 0, 'rounded-b-2xl': index === results.length - 1 }"
          >
            <div class="flex items-center justify-between">
              <div class="flex-1">
                <div class="font-semibold text-gray-900 dark:text-gray-100 group-hover:text-primary-600 dark:group-hover:text-primary-400 transition-colors">
                  {{ word.word }}
                  <span v-if="word.pronunciation" class="ml-2 text-sm text-gray-500 dark:text-gray-400 font-mono">
                    [{{ word.pronunciation }}]
                  </span>
                </div>
                <div class="text-sm text-gray-600 dark:text-gray-400 mt-1.5 line-clamp-1">
                  {{ word.concise_definition }}
                </div>
              </div>
              <span class="text-lg text-gray-400 group-hover:text-primary-500 group-hover:translate-x-1 transition-all opacity-0 group-hover:opacity-100">
                →
              </span>
            </div>
          </button>
        </div>
      </div>
    </transition>

    <!-- 点击外部关闭 -->
    <div 
      v-if="showResults"
      @click="showResults = false"
      class="fixed inset-0 z-10"
    ></div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import { useRouter } from 'vue-router'
import { useDictionaryStore } from '@/stores/dictionary'

const router = useRouter()
const dictionaryStore = useDictionaryStore()

const searchQuery = ref('')
const showResults = ref(false)
const loading = ref(false)

const results = ref([])

let searchTimeout = null

// 搜索处理（防抖）
function handleSearch() {
  if (searchTimeout) {
    clearTimeout(searchTimeout)
  }

  if (!searchQuery.value.trim()) {
    results.value = []
    return
  }

  loading.value = true
  searchTimeout = setTimeout(async () => {
    await dictionaryStore.search(searchQuery.value)
    results.value = dictionaryStore.searchResults
    loading.value = false
  }, 300)
}

// 选择单词
function selectWord(word) {
  showResults.value = false
  searchQuery.value = ''
  results.value = []
  router.push({ name: 'WordDetail', params: { word } })
}

// 监听搜索结果变化
watch(() => dictionaryStore.searchResults, (newResults) => {
  results.value = newResults
})
</script>

<style scoped>
.line-clamp-1 {
  display: -webkit-box;
  -webkit-line-clamp: 1;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.dropdown-enter-active,
.dropdown-leave-active {
  transition: all 0.2s ease;
}

.dropdown-enter-from,
.dropdown-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}
</style>
