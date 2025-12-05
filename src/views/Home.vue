<template>
  <div class="max-w-5xl mx-auto">
    <!-- Hero Section -->
    <div class="text-center mb-16 pt-8 relative">
      <!-- 装饰背景 -->
      <div class="absolute inset-0 -z-10 overflow-hidden">
        <div class="absolute top-20 left-1/4 w-72 h-72 bg-primary-400/20 rounded-full blur-3xl"></div>
        <div class="absolute top-40 right-1/4 w-96 h-96 bg-purple-400/20 rounded-full blur-3xl"></div>
      </div>

      <div class="inline-flex items-center gap-2 px-4 py-2 rounded-full bg-primary-100 dark:bg-primary-900/30 text-primary-700 dark:text-primary-300 text-sm font-medium mb-6">
        <span>🎯</span>
        <span>科学记忆，高效学习</span>
      </div>

      <h1 class="text-4xl md:text-6xl font-bold mb-6">
        <span class="text-gray-900 dark:text-gray-100">智能</span>
        <span class="text-gradient">英汉词典</span>
      </h1>
      <p class="text-xl text-gray-600 dark:text-gray-400 mb-10 max-w-2xl mx-auto leading-relaxed">
        基于记忆曲线的科学学习方法，让每一个单词都深深印在脑海
      </p>

      <!-- 搜索栏 -->
      <div class="max-w-2xl mx-auto mb-12">
        <SearchBar />
      </div>

      <!-- 统计信息 -->
      <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-16">
        <div class="stat-card group">
          <div class="stat-value group-hover:scale-110 transition-transform">
            {{ dictionaryStore.totalWords.toLocaleString() }}
          </div>
          <div class="stat-label flex items-center justify-center gap-1">
            <span>📚</span> 词汇总量
          </div>
        </div>
        <div class="stat-card group">
          <div class="stat-value !bg-gradient-to-r !from-green-600 !to-green-400 group-hover:scale-110 transition-transform">
            {{ learningStore.stats.totalWords }}
          </div>
          <div class="stat-label flex items-center justify-center gap-1">
            <span>✅</span> 已学单词
          </div>
        </div>
        <div class="stat-card group">
          <div class="stat-value !bg-gradient-to-r !from-primary-600 !to-purple-400 group-hover:scale-110 transition-transform">
            {{ learningStore.stats.learnedToday }}
          </div>
          <div class="stat-label flex items-center justify-center gap-1">
            <span>📅</span> 今日学习
          </div>
        </div>
        <div class="stat-card group">
          <div class="stat-value !bg-gradient-to-r !from-amber-600 !to-amber-400 group-hover:scale-110 transition-transform">
            {{ learningStore.dueWords.length }}
          </div>
          <div class="stat-label flex items-center justify-center gap-1">
            <span>🔔</span> 待复习
          </div>
        </div>
      </div>
    </div>

    <!-- 快速开始 -->
    <div class="grid md:grid-cols-2 gap-6 mb-16">
      <router-link to="/study" class="card card-interactive card-gradient group">
        <div class="flex items-center gap-6">
          <div class="text-5xl group-hover:animate-bounce-soft">📖</div>
          <div class="flex-1">
            <h3 class="text-xl font-bold text-gray-900 dark:text-gray-100 group-hover:text-primary-600 dark:group-hover:text-primary-400 transition-colors mb-2">
              开始学习
            </h3>
            <p class="text-gray-600 dark:text-gray-400">浏览词汇库，学习新单词</p>
          </div>
          <span class="text-2xl text-gray-400 group-hover:text-primary-500 group-hover:translate-x-1 transition-all">→</span>
        </div>
      </router-link>

      <router-link v-if="userStore.isAuthenticated" to="/review" class="card card-interactive card-gradient group">
        <div class="flex items-center gap-6">
          <div class="text-5xl group-hover:animate-bounce-soft">🔄</div>
          <div class="flex-1">
            <h3 class="text-xl font-bold text-gray-900 dark:text-gray-100 group-hover:text-primary-600 dark:group-hover:text-primary-400 transition-colors mb-2">
              复习单词
            </h3>
            <p class="text-gray-600 dark:text-gray-400">根据记忆曲线复习</p>
          </div>
          <span class="text-2xl text-gray-400 group-hover:text-primary-500 group-hover:translate-x-1 transition-all">→</span>
        </div>
      </router-link>

      <div v-else class="card opacity-60 cursor-not-allowed">
        <div class="flex items-center gap-6">
          <div class="text-5xl grayscale">🔄</div>
          <div class="flex-1">
            <h3 class="text-xl font-bold text-gray-500 dark:text-gray-400 mb-2">复习单词</h3>
            <p class="text-gray-400 dark:text-gray-500 flex items-center gap-2">
              <span>🔐</span> 请先登录
            </p>
          </div>
        </div>
      </div>
    </div>

    <!-- 每日推荐 -->
    <div class="card mb-16">
      <div class="flex items-center justify-between mb-8">
        <div>
          <h2 class="text-2xl font-bold text-gray-900 dark:text-gray-100 flex items-center gap-3">
            <span class="text-3xl">✨</span> 随机单词
          </h2>
          <p class="text-gray-500 dark:text-gray-400 mt-1">发现新词汇，拓展词汇量</p>
        </div>
        <button @click="loadRandomWords" :disabled="loading" class="btn btn-outline">
          <span :class="{ 'animate-spin': loading }">🔄</span>
          {{ loading ? '加载中' : '换一批' }}
        </button>
      </div>

      <div v-if="loading" class="grid md:grid-cols-2 gap-4">
        <div v-for="i in 6" :key="i" class="skeleton h-28"></div>
      </div>

      <div v-else class="grid md:grid-cols-2 gap-4">
        <router-link v-for="word in randomWords" :key="word.word" :to="{ name: 'WordDetail', params: { word: word.word } }"
          class="group p-5 rounded-xl border-2 border-gray-100 dark:border-gray-700 hover:border-primary-400 dark:hover:border-primary-500 bg-gray-50/50 dark:bg-gray-800/50 hover:bg-white dark:hover:bg-gray-800 transition-all duration-300 hover:shadow-soft">
          <div class="flex items-start justify-between">
            <div class="flex-1">
              <div class="font-bold text-lg text-gray-900 dark:text-gray-100 group-hover:text-primary-600 dark:group-hover:text-primary-400 transition-colors">
                {{ word.word }}
              </div>
              <div v-if="word.pronunciation" class="text-sm text-gray-500 dark:text-gray-400 font-mono mt-1">
                [{{ word.pronunciation }}]
              </div>
            </div>
            <span class="text-xl opacity-0 group-hover:opacity-100 transition-opacity">→</span>
          </div>
          <div class="text-sm text-gray-600 dark:text-gray-400 mt-3 line-clamp-2 leading-relaxed">
            {{ word.concise_definition }}
          </div>
        </router-link>
      </div>
    </div>

    <!-- 使用帮助 -->
    <div class="card mb-16">
      <div class="flex items-center justify-between mb-6">
        <h2 class="text-2xl font-bold text-gray-900 dark:text-gray-100 flex items-center gap-3">
          <span class="text-3xl">💡</span> 使用帮助
        </h2>
        <button @click="showHelp = !showHelp" class="btn btn-ghost btn-sm">
          {{ showHelp ? '收起 ↑' : '展开 ↓' }}
        </button>
      </div>
      
      <div v-if="showHelp" class="space-y-6">
        <div class="alert alert-info">
          <span class="text-2xl">⌨️</span>
          <div class="flex-1">
            <h3 class="font-bold mb-3">学习模式快捷键</h3>
            <div class="grid md:grid-cols-2 gap-4 text-sm">
              <div class="space-y-2">
                <div class="flex items-center gap-3"><kbd>空格</kbd><span>翻转单词卡片</span></div>
                <div class="flex items-center gap-3"><kbd>Esc</kbd><span>返回单词面</span></div>
              </div>
              <div class="space-y-2">
                <div class="flex items-center gap-3"><kbd>1</kbd><span>不认识</span></div>
                <div class="flex items-center gap-3"><kbd>2</kbd><span>有点难</span></div>
                <div class="flex items-center gap-3"><kbd>3</kbd><span>很简单</span></div>
              </div>
            </div>
          </div>
        </div>

        <div class="alert alert-success">
          <span class="text-2xl">🎯</span>
          <div class="flex-1">
            <h3 class="font-bold mb-3">学习建议</h3>
            <ul class="space-y-2 text-sm">
              <li class="flex items-start gap-2"><span class="text-green-500 mt-0.5">✓</span><span>每天坚持学习 15-30 分钟，效果最佳</span></li>
              <li class="flex items-start gap-2"><span class="text-green-500 mt-0.5">✓</span><span>诚实评估掌握程度，系统会据此调整复习间隔</span></li>
              <li class="flex items-start gap-2"><span class="text-green-500 mt-0.5">✓</span><span>收藏重要单词，方便重点复习</span></li>
              <li class="flex items-start gap-2"><span class="text-green-500 mt-0.5">✓</span><span>定期查看复习页面，巩固已学单词</span></li>
            </ul>
          </div>
        </div>
      </div>
    </div>

    <!-- 功能特性 -->
    <div class="mb-16">
      <div class="text-center mb-12">
        <h2 class="text-3xl font-bold text-gray-900 dark:text-gray-100 mb-4">为什么选择我们</h2>
        <p class="text-gray-600 dark:text-gray-400">科学方法 + 优质内容 = 高效学习</p>
      </div>
      <div class="grid md:grid-cols-3 gap-8">
        <div class="card card-hover text-center group">
          <div class="w-16 h-16 mx-auto mb-6 rounded-2xl bg-gradient-to-br from-primary-100 to-primary-200 dark:from-primary-900/50 dark:to-primary-800/50 flex items-center justify-center group-hover:scale-110 transition-transform">
            <span class="text-3xl">🧠</span>
          </div>
          <h3 class="text-xl font-bold text-gray-900 dark:text-gray-100 mb-3">科学记忆</h3>
          <p class="text-gray-600 dark:text-gray-400 leading-relaxed">基于 SM-2 算法的记忆曲线，让复习更高效</p>
        </div>
        <div class="card card-hover text-center group">
          <div class="w-16 h-16 mx-auto mb-6 rounded-2xl bg-gradient-to-br from-purple-100 to-purple-200 dark:from-purple-900/50 dark:to-purple-800/50 flex items-center justify-center group-hover:scale-110 transition-transform">
            <span class="text-3xl">☁️</span>
          </div>
          <h3 class="text-xl font-bold text-gray-900 dark:text-gray-100 mb-3">跨端同步</h3>
          <p class="text-gray-600 dark:text-gray-400 leading-relaxed">学习进度云端同步，随时随地继续学习</p>
        </div>
        <div class="card card-hover text-center group">
          <div class="w-16 h-16 mx-auto mb-6 rounded-2xl bg-gradient-to-br from-green-100 to-green-200 dark:from-green-900/50 dark:to-green-800/50 flex items-center justify-center group-hover:scale-110 transition-transform">
            <span class="text-3xl">🎯</span>
          </div>
          <h3 class="text-xl font-bold text-gray-900 dark:text-gray-100 mb-3">详细释义</h3>
          <p class="text-gray-600 dark:text-gray-400 leading-relaxed">详尽的双语释义和例句，深入理解每个单词</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import SearchBar from '@/components/SearchBar.vue'
import { useDictionaryStore } from '@/stores/dictionary'
import { useLearningStore } from '@/stores/learning'
import { useUserStore } from '@/stores/user'

const dictionaryStore = useDictionaryStore()
const learningStore = useLearningStore()
const userStore = useUserStore()

const randomWords = ref([])
const loading = ref(false)
const showHelp = ref(false)

async function loadRandomWords() {
  loading.value = true
  await dictionaryStore.loadRandomWords(6)
  randomWords.value = dictionaryStore.randomWords
  loading.value = false
}

onMounted(() => {
  loadRandomWords()
})
</script>

<style scoped>
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>
