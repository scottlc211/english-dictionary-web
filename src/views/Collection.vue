<template>
  <div class="max-w-4xl mx-auto">
    <!-- 页面头部 -->
    <div class="mb-8">
      <div class="flex items-center gap-4 mb-2">
        <div class="w-12 h-12 rounded-2xl bg-gradient-to-br from-amber-400 to-amber-500 flex items-center justify-center">
          <span class="text-2xl">⭐</span>
        </div>
        <div>
          <h1 class="text-3xl font-bold text-gray-900 dark:text-gray-100">
            我的收藏
          </h1>
          <p class="text-gray-500 dark:text-gray-400 mt-1">
            共收藏 <span class="font-semibold text-primary-600 dark:text-primary-400">{{ collectionList.length }}</span> 个单词
          </p>
        </div>
      </div>
    </div>

    <!-- 搜索框 -->
    <div class="card mb-6">
      <div class="relative">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="搜索收藏的单词..."
          class="input pr-12"
        />
        <span class="absolute right-4 top-1/2 -translate-y-1/2 text-xl opacity-50">🔍</span>
      </div>
    </div>

    <!-- 收藏列表 -->
    <div v-if="!loading && filteredCollections.length > 0" class="space-y-4">
      <transition-group name="list">
        <div
          v-for="item in filteredCollections"
          :key="item.word"
          class="card card-hover group"
        >
          <div class="flex items-center justify-between">
            <router-link
              :to="{ name: 'WordDetail', params: { word: item.word } }"
              class="flex-1 flex items-center gap-4"
            >
              <div class="w-10 h-10 rounded-xl bg-primary-100 dark:bg-primary-900/30 flex items-center justify-center text-primary-600 dark:text-primary-400 font-bold text-lg group-hover:scale-110 transition-transform">
                {{ item.word.charAt(0).toUpperCase() }}
              </div>
              <div class="flex-1">
                <h3 class="text-lg font-semibold text-gray-900 dark:text-gray-100 group-hover:text-primary-600 dark:group-hover:text-primary-400 transition-colors">
                  {{ item.word }}
                </h3>
                <p class="text-sm text-gray-500 dark:text-gray-400 mt-0.5 flex items-center gap-2">
                  <span>📅</span>
                  收藏于 {{ formatDate(item.created_at) }}
                </p>
              </div>
              <span class="text-xl text-gray-400 group-hover:text-primary-500 group-hover:translate-x-1 transition-all opacity-0 group-hover:opacity-100">→</span>
            </router-link>

            <button
              @click.stop="removeFromCollection(item.word)"
              class="ml-4 p-2.5 text-gray-400 hover:text-red-500 hover:bg-red-50 dark:hover:bg-red-900/50 rounded-xl transition-all"
              title="取消收藏"
            >
              <span class="text-lg">🗑️</span>
            </button>
          </div>
        </div>
      </transition-group>
    </div>

    <!-- 空状态 -->
    <div v-else-if="!loading && collectionList.length === 0" class="card text-center py-16">
      <div class="w-20 h-20 mx-auto mb-6 rounded-3xl bg-gradient-to-br from-gray-100 to-gray-200 dark:from-gray-800 dark:to-gray-700 flex items-center justify-center">
        <span class="text-4xl">📚</span>
      </div>
      <h2 class="text-2xl font-bold text-gray-900 dark:text-gray-100 mb-3">
        还没有收藏
      </h2>
      <p class="text-gray-600 dark:text-gray-400 mb-8 max-w-sm mx-auto">
        在学习时点击收藏按钮，将重要单词添加到这里
      </p>
      <router-link to="/study" class="btn btn-primary">
        <span>📖</span>
        去学习
      </router-link>
    </div>

    <!-- 搜索无结果 -->
    <div v-else-if="!loading && searchQuery && filteredCollections.length === 0" class="card text-center py-16">
      <div class="text-5xl mb-4">🔍</div>
      <p class="text-gray-600 dark:text-gray-400">
        未找到包含 "<span class="font-semibold text-primary-600 dark:text-primary-400">{{ searchQuery }}</span>" 的单词
      </p>
    </div>

    <!-- 加载状态 -->
    <div v-else-if="loading" class="space-y-4">
      <div v-for="i in 5" :key="i" class="skeleton h-20 rounded-2xl"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useLearningStore } from '@/stores/learning'

const learningStore = useLearningStore()

const searchQuery = ref('')
const loading = ref(false)

const collectionList = computed(() => learningStore.collectionList)

const filteredCollections = computed(() => {
  if (!searchQuery.value) {
    return collectionList.value
  }
  const query = searchQuery.value.toLowerCase()
  return collectionList.value.filter(item =>
    item.word.toLowerCase().includes(query)
  )
})

function formatDate(dateString) {
  const date = new Date(dateString)
  const now = new Date()
  const diffMs = now - date
  const diffDays = Math.floor(diffMs / (1000 * 60 * 60 * 24))

  if (diffDays === 0) {
    return '今天'
  } else if (diffDays === 1) {
    return '昨天'
  } else if (diffDays < 7) {
    return `${diffDays} 天前`
  } else {
    return date.toLocaleDateString('zh-CN')
  }
}

async function removeFromCollection(word) {
  if (confirm(`确定要取消收藏"${word}"吗？`)) {
    await learningStore.removeCollection(word)
  }
}

onMounted(async () => {
  loading.value = true
  await learningStore.loadCollections()
  loading.value = false
})
</script>

<style scoped>
.list-enter-active,
.list-leave-active {
  transition: all 0.3s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(-20px);
}

.list-move {
  transition: transform 0.3s ease;
}
</style>
