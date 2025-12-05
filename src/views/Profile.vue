<template>
  <div class="max-w-4xl mx-auto">
    <!-- 页面头部 -->
    <div class="card mb-8 relative overflow-hidden">
      <!-- 装饰背景 -->
      <div class="absolute inset-0 bg-gradient-to-br from-primary-500/10 via-purple-500/10 to-transparent"></div>
      
      <div class="relative flex items-center gap-6">
        <div class="w-20 h-20 rounded-2xl bg-gradient-to-br from-primary-500 to-purple-500 flex items-center justify-center shadow-lg">
          <span class="text-4xl">👤</span>
        </div>
        <div>
          <h1 class="text-3xl font-bold text-gray-900 dark:text-gray-100 mb-1">
            个人中心
          </h1>
          <p class="text-gray-500 dark:text-gray-400 flex items-center gap-2">
            <span>📧</span>
            {{ userStore.user?.email }}
          </p>
        </div>
      </div>
    </div>

    <!-- 学习统计 -->
    <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-8">
      <div class="stat-card">
        <div class="stat-value">{{ stats.totalWords }}</div>
        <div class="stat-label flex items-center justify-center gap-1">
          <span>📚</span> 总学习
        </div>
      </div>

      <div class="stat-card">
        <div class="stat-value !bg-gradient-to-r !from-green-600 !to-green-400">
          {{ stats.masteredWords }}
        </div>
        <div class="stat-label flex items-center justify-center gap-1">
          <span>✅</span> 已掌握
        </div>
      </div>

      <div class="stat-card">
        <div class="stat-value !bg-gradient-to-r !from-primary-600 !to-purple-400">
          {{ stats.learnedToday }}
        </div>
        <div class="stat-label flex items-center justify-center gap-1">
          <span>📅</span> 今日学习
        </div>
      </div>

      <div class="stat-card">
        <div class="stat-value !bg-gradient-to-r !from-amber-600 !to-amber-400">
          {{ stats.dueWords }}
        </div>
        <div class="stat-label flex items-center justify-center gap-1">
          <span>🔔</span> 待复习
        </div>
      </div>
    </div>

    <!-- 学习进度图表 -->
    <div v-if="chartData.length > 0" class="card mb-8">
      <h2 class="text-xl font-bold text-gray-900 dark:text-gray-100 mb-6 flex items-center gap-3">
        <span class="text-2xl">📊</span>
        学习趋势
      </h2>
      <ProgressChart :data="chartData" />
    </div>

    <!-- 最近学习 -->
    <div class="card mb-8">
      <h2 class="text-xl font-bold text-gray-900 dark:text-gray-100 mb-6 flex items-center gap-3">
        <span class="text-2xl">📝</span>
        最近学习
      </h2>

      <div v-if="recentWords.length > 0" class="space-y-3">
        <div
          v-for="item in recentWords"
          :key="item.word"
          class="flex items-center justify-between p-4 rounded-xl bg-gray-50 dark:bg-gray-800/50 hover:bg-gray-100 dark:hover:bg-gray-800 transition-colors group"
        >
          <router-link
            :to="{ name: 'WordDetail', params: { word: item.word } }"
            class="font-semibold text-gray-900 dark:text-gray-100 group-hover:text-primary-600 dark:group-hover:text-primary-400 transition-colors"
          >
            {{ item.word }}
          </router-link>
          <div class="text-right">
            <div class="text-sm text-gray-600 dark:text-gray-400 flex items-center gap-1">
              <span>🔄</span>
              复习 {{ item.repetitions }} 次
            </div>
            <div class="text-xs text-gray-500 dark:text-gray-500 mt-0.5">
              {{ formatNextReview(item.next_review) }}
            </div>
          </div>
        </div>
      </div>

      <div v-else class="text-center py-12 text-gray-500 dark:text-gray-400">
        <span class="text-4xl mb-4 block">📭</span>
        暂无学习记录
      </div>
    </div>

    <!-- 数据同步状态 -->
    <div class="card mb-8">
      <h2 class="text-xl font-bold text-gray-900 dark:text-gray-100 mb-6 flex items-center gap-3">
        <span class="text-2xl">☁️</span>
        数据同步
      </h2>

      <div class="space-y-4">
        <div class="flex items-center justify-between p-4 rounded-xl bg-gray-50 dark:bg-gray-800/50">
          <div class="flex items-center gap-4">
            <div class="w-12 h-12 rounded-xl flex items-center justify-center" 
                 :class="learningStore.isSyncing ? 'bg-primary-100 dark:bg-primary-900/30' : 'bg-green-100 dark:bg-green-900/30'">
              <span class="text-2xl" :class="{ 'animate-spin': learningStore.isSyncing }">
                {{ learningStore.isSyncing ? '🔄' : '✅' }}
              </span>
            </div>
            <div>
              <div class="font-semibold text-gray-900 dark:text-gray-100">
                {{ learningStore.isSyncing ? '正在同步...' : '数据已同步' }}
              </div>
              <div class="text-sm text-gray-500 dark:text-gray-400 mt-0.5">
                使用本地缓存，减少网络请求
              </div>
            </div>
          </div>
          <button
            @click="handleManualSync"
            :disabled="learningStore.isSyncing"
            class="btn btn-outline btn-sm"
          >
            {{ learningStore.isSyncing ? '同步中' : '手动同步' }}
          </button>
        </div>

        <div class="alert alert-info">
          <span class="text-2xl">💡</span>
          <div class="flex-1">
            <div class="font-semibold mb-2">智能缓存机制</div>
            <ul class="space-y-1 text-sm opacity-90">
              <li class="flex items-center gap-2">
                <span>•</span> 用户数据：本地优先，自动同步
              </li>
              <li class="flex items-center gap-2">
                <span>•</span> 词典数据：按需加载，永久缓存
              </li>
              <li class="flex items-center gap-2">
                <span>•</span> 已缓存单词：<span class="font-semibold">{{ cacheStats.total }}</span> 个
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>

    <!-- 账号操作 -->
    <div class="card">
      <h2 class="text-xl font-bold text-gray-900 dark:text-gray-100 mb-6 flex items-center gap-3">
        <span class="text-2xl">⚙️</span>
        账号设置
      </h2>

      <div class="space-y-4">
        <div class="flex items-center justify-between p-4 rounded-xl bg-gray-50 dark:bg-gray-800/50">
          <div class="flex items-center gap-4">
            <div class="w-10 h-10 rounded-xl bg-primary-100 dark:bg-primary-900/30 flex items-center justify-center">
              <span class="text-xl">📧</span>
            </div>
            <div>
              <div class="font-semibold text-gray-900 dark:text-gray-100">邮箱地址</div>
              <div class="text-sm text-gray-500 dark:text-gray-400 mt-0.5">
                {{ userStore.user?.email }}
              </div>
            </div>
          </div>
        </div>

        <button
          @click="handleClearCache"
          class="w-full btn btn-secondary justify-start gap-3"
        >
          <span class="text-xl">🗑️</span>
          清除本地缓存
        </button>

        <button
          @click="handleSignOut"
          class="w-full btn btn-danger justify-start gap-3"
        >
          <span class="text-xl">🚪</span>
          退出登录
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'
import { useLearningStore } from '@/stores/learning'
import ProgressChart from '@/components/ProgressChart.vue'
import { formatNextReview } from '@/utils/sm2'
import { clearCache as clearDictCache, clearIndexedDB, getCacheStats } from '@/utils/dictionary'

const router = useRouter()
const userStore = useUserStore()
const learningStore = useLearningStore()

const cacheStats = ref({ memoryCache: 0, indexedDBCache: 0, total: 0 })

const stats = computed(() => learningStore.stats)

const recentWords = computed(() => {
  return [...learningStore.progressList]
    .sort((a, b) => new Date(b.last_reviewed) - new Date(a.last_reviewed))
    .slice(0, 10)
})

const chartData = ref([])

function generateChartData() {
  const data = []
  const today = new Date()
  
  for (let i = 6; i >= 0; i--) {
    const date = new Date(today)
    date.setDate(date.getDate() - i)
    
    const dateString = date.toLocaleDateString('zh-CN', { month: 'short', day: 'numeric' })
    const startOfDay = new Date(date.getFullYear(), date.getMonth(), date.getDate())
    const endOfDay = new Date(date.getFullYear(), date.getMonth(), date.getDate() + 1)
    
    const count = learningStore.progressList.filter(p => {
      const reviewDate = new Date(p.last_reviewed)
      return reviewDate >= startOfDay && reviewDate < endOfDay
    }).length
    
    data.push({ date: dateString, count })
  }
  
  chartData.value = data
}

async function handleSignOut() {
  if (confirm('确定要退出登录吗？')) {
    await userStore.signOut()
    router.push('/')
  }
}

async function handleManualSync() {
  await learningStore.syncWithServer()
}

async function handleClearCache() {
  if (confirm('清除缓存后将从服务器重新加载数据，确定继续吗？')) {
    try {
      learningStore.clearUserData()
      clearDictCache()
      await clearIndexedDB()
      
      await learningStore.loadCollections()
      await learningStore.loadProgress()
      await learningStore.loadStats()
      
      cacheStats.value = await getCacheStats()
      
      alert('缓存已清除！')
    } catch (err) {
      console.error('清除缓存失败:', err)
      alert('清除缓存失败，请刷新页面重试')
    }
  }
}

onMounted(async () => {
  await learningStore.loadProgress()
  await learningStore.loadStats()
  generateChartData()
  cacheStats.value = await getCacheStats()
})
</script>
