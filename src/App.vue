<template>
  <div id="app" :class="{ 'dark': isDark }">
    <div class="min-h-screen">
      <!-- 顶部导航栏 -->
      <nav class="sticky top-0 z-50 glass border-b border-white/20 dark:border-gray-700/50">
        <div class="container mx-auto px-4">
          <div class="flex items-center justify-between h-16">
            <!-- Logo -->
            <router-link to="/" class="flex items-center space-x-3 group">
              <span class="text-2xl group-hover:animate-bounce-soft transition-transform">📚</span>
              <span class="text-xl font-bold text-gradient">英汉词典</span>
            </router-link>

            <!-- 导航链接 -->
            <div class="hidden md:flex items-center space-x-2">
              <router-link to="/" class="nav-link flex items-center gap-2" :class="{ 'active': $route.name === 'Home' }">
                <span class="text-lg">🏠</span>首页
              </router-link>
              <router-link to="/study" class="nav-link flex items-center gap-2" :class="{ 'active': $route.name === 'Study' }">
                <span class="text-lg">📖</span>学习
              </router-link>
              <router-link v-if="userStore.isAuthenticated" to="/review" class="nav-link flex items-center gap-2" :class="{ 'active': $route.name === 'Review' }">
                <span class="text-lg">🔄</span>复习
                <span v-if="learningStore.dueWords.length > 0" class="ml-1 px-2 py-0.5 text-xs bg-gradient-to-r from-red-500 to-red-400 text-white rounded-full font-semibold shadow-sm">
                  {{ learningStore.dueWords.length }}
                </span>
              </router-link>
              <router-link v-if="userStore.isAuthenticated" to="/collection" class="nav-link flex items-center gap-2" :class="{ 'active': $route.name === 'Collection' }">
                <span class="text-lg">⭐</span>收藏
              </router-link>
            </div>

            <!-- 右侧按钮 -->
            <div class="flex items-center space-x-2">
              <!-- 主题切换 -->
              <button @click="toggleTheme" class="btn-icon btn-ghost" title="切换主题">
                <span v-if="isDark" class="text-xl">🌞</span>
                <span v-else class="text-xl">🌙</span>
              </button>

              <!-- 用户菜单 -->
              <div v-if="userStore.isAuthenticated" class="flex items-center space-x-1">
                <router-link to="/settings" class="btn-icon btn-ghost" title="设置">
                  <span class="text-xl">⚙️</span>
                </router-link>
                <router-link to="/profile" class="btn-icon btn-ghost" title="个人中心">
                  <span class="text-xl">👤</span>
                </router-link>
              </div>
              <div v-else>
                <router-link to="/auth" class="btn btn-primary btn-sm">
                  <span>🔐</span>登录
                </router-link>
              </div>

              <!-- 移动端菜单按钮 -->
              <button @click="mobileMenuOpen = !mobileMenuOpen" class="md:hidden btn-icon btn-ghost">
                <span class="text-xl">{{ mobileMenuOpen ? '✕' : '☰' }}</span>
              </button>
            </div>
          </div>

          <!-- 移动端菜单 -->
          <transition name="slide-down">
            <div v-show="mobileMenuOpen" class="md:hidden py-4 space-y-1 border-t border-gray-200/50 dark:border-gray-700/50">
              <router-link to="/" class="flex items-center gap-3 px-4 py-3 rounded-xl hover:bg-primary-50 dark:hover:bg-primary-900/50 transition-colors" @click="mobileMenuOpen = false">
                <span class="text-xl">🏠</span><span class="font-medium">首页</span>
              </router-link>
              <router-link to="/study" class="flex items-center gap-3 px-4 py-3 rounded-xl hover:bg-primary-50 dark:hover:bg-primary-900/50 transition-colors" @click="mobileMenuOpen = false">
                <span class="text-xl">📖</span><span class="font-medium">学习</span>
              </router-link>
              <router-link v-if="userStore.isAuthenticated" to="/review" class="flex items-center gap-3 px-4 py-3 rounded-xl hover:bg-primary-50 dark:hover:bg-primary-900/50 transition-colors" @click="mobileMenuOpen = false">
                <span class="text-xl">🔄</span><span class="font-medium">复习</span>
                <span v-if="learningStore.dueWords.length > 0" class="ml-auto px-2 py-0.5 text-xs bg-red-500 text-white rounded-full">{{ learningStore.dueWords.length }}</span>
              </router-link>
              <router-link v-if="userStore.isAuthenticated" to="/collection" class="flex items-center gap-3 px-4 py-3 rounded-xl hover:bg-primary-50 dark:hover:bg-primary-900/50 transition-colors" @click="mobileMenuOpen = false">
                <span class="text-xl">⭐</span><span class="font-medium">收藏</span>
              </router-link>
              <router-link v-if="userStore.isAuthenticated" to="/settings" class="flex items-center gap-3 px-4 py-3 rounded-xl hover:bg-primary-50 dark:hover:bg-primary-900/50 transition-colors" @click="mobileMenuOpen = false">
                <span class="text-xl">⚙️</span><span class="font-medium">设置</span>
              </router-link>
              <router-link v-if="userStore.isAuthenticated" to="/profile" class="flex items-center gap-3 px-4 py-3 rounded-xl hover:bg-primary-50 dark:hover:bg-primary-900/50 transition-colors" @click="mobileMenuOpen = false">
                <span class="text-xl">👤</span><span class="font-medium">个人中心</span>
              </router-link>
            </div>
          </transition>
        </div>
      </nav>

      <!-- 主内容区 -->
      <main class="container mx-auto px-4 py-8">
        <router-view v-slot="{ Component }">
          <transition name="fade" mode="out-in">
            <component :is="Component" />
          </transition>
        </router-view>
      </main>

      <!-- 页脚 -->
      <footer class="mt-16 py-10 glass border-t border-white/20 dark:border-gray-700/50">
        <div class="container mx-auto px-4 text-center">
          <div class="flex items-center justify-center gap-2 mb-3">
            <span class="text-2xl">📚</span>
            <span class="text-lg font-bold text-gradient">英汉词典</span>
          </div>
          <p class="text-gray-600 dark:text-gray-400">
            基于 <a href="https://github.com/ahpxex/open-english-dictionary" target="_blank" class="text-primary-600 dark:text-primary-400 hover:underline font-medium">Open English Dictionary</a> 数据构建
          </p>
          <p class="mt-2 text-sm text-gray-500 dark:text-gray-500">开源项目 · 自由部署</p>
        </div>
      </footer>
    </div>

    <!-- 网络状态提示 -->
    <NetworkStatus />
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { useUserStore } from '@/stores/user'
import { useLearningStore } from '@/stores/learning'
import { useDictionaryStore } from '@/stores/dictionary'
import NetworkStatus from '@/components/NetworkStatus.vue'
import { auth } from '@/utils/supabase'

const userStore = useUserStore()
const learningStore = useLearningStore()
const dictionaryStore = useDictionaryStore()

const isDark = ref(false)
const mobileMenuOpen = ref(false)

function toggleTheme() {
  isDark.value = !isDark.value
  localStorage.setItem('theme', isDark.value ? 'dark' : 'light')
}

onMounted(async () => {
  const savedTheme = localStorage.getItem('theme')
  if (savedTheme) {
    isDark.value = savedTheme === 'dark'
  } else {
    isDark.value = window.matchMedia('(prefers-color-scheme: dark)').matches
  }

  await userStore.initUser()
  await dictionaryStore.initIndex()

  if (userStore.isAuthenticated) {
    learningStore.loadCollections()
    learningStore.loadProgress()
    learningStore.loadStats()
  }

  auth.onAuthStateChange((event, session) => {
    if (event === 'SIGNED_IN') {
      userStore.user = session?.user || null
      learningStore.loadCollections()
      learningStore.loadProgress()
      learningStore.loadStats()
    } else if (event === 'SIGNED_OUT') {
      userStore.user = null
      learningStore.clearUserData()
    }
  })

  let syncInterval = null
  if (userStore.isAuthenticated) {
    syncInterval = setInterval(() => {
      if (userStore.isAuthenticated && !learningStore.isSyncing) {
        learningStore.syncWithServer()
      }
    }, 5 * 60 * 1000)
  }

  onUnmounted(() => {
    if (syncInterval) {
      clearInterval(syncInterval)
    }
  })

  document.addEventListener('visibilitychange', () => {
    if (!document.hidden && userStore.isAuthenticated && !learningStore.isSyncing) {
      learningStore.syncWithServer()
    }
  })
})
</script>

<style scoped>
.slide-down-enter-active,
.slide-down-leave-active {
  transition: all 0.3s ease;
  transform-origin: top;
}

.slide-down-enter-from,
.slide-down-leave-to {
  opacity: 0;
  transform: scaleY(0.95);
}

.btn-icon {
  @apply p-2.5 rounded-xl transition-all duration-200;
}

.btn-ghost {
  @apply text-gray-600 dark:text-gray-400;
  @apply hover:bg-gray-100 dark:hover:bg-gray-800;
  @apply hover:text-gray-900 dark:hover:text-gray-100;
}
</style>
