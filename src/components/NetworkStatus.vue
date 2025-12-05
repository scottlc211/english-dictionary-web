<template>
  <teleport to="body">
    <transition name="toast">
      <div 
        v-if="showOffline" 
        class="fixed bottom-6 left-4 right-4 md:left-auto md:right-6 md:w-96 z-50"
      >
        <div class="glass rounded-2xl shadow-soft-lg border border-amber-200 dark:border-amber-800 p-4">
          <div class="flex items-center gap-4">
            <div class="w-12 h-12 rounded-xl bg-amber-100 dark:bg-amber-900/50 flex items-center justify-center flex-shrink-0">
              <span class="text-2xl">📡</span>
            </div>
            <div class="flex-1 min-w-0">
              <div class="font-semibold text-amber-800 dark:text-amber-200">
                离线模式
              </div>
              <div class="text-sm text-amber-600 dark:text-amber-400 mt-0.5">
                数据将在恢复连接后自动同步
              </div>
            </div>
          </div>
        </div>
      </div>
    </transition>

    <transition name="toast">
      <div 
        v-if="showSyncing" 
        class="fixed bottom-6 left-4 right-4 md:left-auto md:right-6 md:w-96 z-50"
      >
        <div class="glass rounded-2xl shadow-soft-lg border border-primary-200 dark:border-primary-800 p-4">
          <div class="flex items-center gap-4">
            <div class="w-12 h-12 rounded-xl bg-primary-100 dark:bg-primary-900/50 flex items-center justify-center flex-shrink-0">
              <span class="text-2xl animate-spin">🔄</span>
            </div>
            <div class="flex-1 min-w-0">
              <div class="font-semibold text-primary-800 dark:text-primary-200">
                正在同步...
              </div>
              <div class="text-sm text-primary-600 dark:text-primary-400 mt-0.5">
                正在保存您的学习进度
              </div>
            </div>
          </div>
          <!-- 进度条 -->
          <div class="mt-3 progress">
            <div class="progress-bar animate-pulse" style="width: 60%"></div>
          </div>
        </div>
      </div>
    </transition>
  </teleport>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import { useLearningStore } from '@/stores/learning'

const learningStore = useLearningStore()

const isOnline = ref(navigator.onLine)
const showOffline = ref(false)
const showSyncing = ref(false)

let offlineTimer = null
let syncingTimer = null

function updateOnlineStatus() {
  isOnline.value = navigator.onLine
  
  if (!isOnline.value) {
    showOffline.value = true
  } else {
    showOffline.value = false
    if (learningStore && !learningStore.isSyncing) {
      learningStore.syncWithServer()
    }
  }
}

watch(() => learningStore.isSyncing, (syncing) => {
  if (syncing) {
    showSyncing.value = true
  } else {
    syncingTimer = setTimeout(() => {
      showSyncing.value = false
    }, 1500)
  }
})

onMounted(() => {
  window.addEventListener('online', updateOnlineStatus)
  window.addEventListener('offline', updateOnlineStatus)
})

onUnmounted(() => {
  window.removeEventListener('online', updateOnlineStatus)
  window.removeEventListener('offline', updateOnlineStatus)
  if (offlineTimer) clearTimeout(offlineTimer)
  if (syncingTimer) clearTimeout(syncingTimer)
})
</script>

<style scoped>
.toast-enter-active,
.toast-leave-active {
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.toast-enter-from,
.toast-leave-to {
  opacity: 0;
  transform: translateY(20px) scale(0.95);
}
</style>
