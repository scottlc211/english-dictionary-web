<template>
  <div class="max-w-3xl mx-auto">
    <!-- 页面头部 -->
    <div class="mb-8">
      <div class="flex items-center gap-4 mb-2">
        <div class="w-12 h-12 rounded-2xl bg-gradient-to-br from-gray-400 to-gray-500 flex items-center justify-center">
          <span class="text-2xl">⚙️</span>
        </div>
        <div>
          <h1 class="text-3xl font-bold text-gray-900 dark:text-gray-100">个人设置</h1>
          <p class="text-gray-500 dark:text-gray-400 mt-1">自定义您的学习体验</p>
        </div>
      </div>
    </div>

    <div class="space-y-6">
      <!-- 朗读设置 -->
      <div class="card">
        <h2 class="text-xl font-bold text-gray-900 dark:text-gray-100 mb-6 flex items-center gap-3">
          <span class="text-2xl">🔊</span>
          朗读设置
        </h2>
        
        <!-- 自动朗读总开关 -->
        <div class="setting-item">
          <div class="flex-1">
            <h3 class="font-semibold text-gray-900 dark:text-gray-100">自动朗读</h3>
            <p class="text-sm text-gray-500 dark:text-gray-400 mt-0.5">开启后，查看单词时会自动朗读</p>
          </div>
          <label class="toggle">
            <input v-model="settings.autoSpeak" @change="updateSettings" type="checkbox" class="sr-only peer">
            <div class="toggle-track peer-checked:bg-primary-500"></div>
          </label>
        </div>

        <!-- 朗读场景设置 -->
        <transition name="slide-down">
          <div v-if="settings.autoSpeak" class="mt-4 ml-4 pl-4 border-l-2 border-primary-200 dark:border-primary-800 space-y-4">
            <div class="setting-item-sm">
              <div class="flex-1">
                <h4 class="font-medium text-gray-900 dark:text-gray-100">单词详情页</h4>
                <p class="text-xs text-gray-500 dark:text-gray-400">进入单词详情时自动朗读</p>
              </div>
              <label class="toggle">
                <input v-model="settings.speakOnWordDetail" @change="updateSettings" type="checkbox" class="sr-only peer">
                <div class="toggle-track peer-checked:bg-primary-500"></div>
              </label>
            </div>

            <div class="setting-item-sm">
              <div class="flex-1">
                <h4 class="font-medium text-gray-900 dark:text-gray-100">学习模式</h4>
                <p class="text-xs text-gray-500 dark:text-gray-400">学习新单词时自动朗读</p>
              </div>
              <label class="toggle">
                <input v-model="settings.speakOnStudy" @change="updateSettings" type="checkbox" class="sr-only peer">
                <div class="toggle-track peer-checked:bg-primary-500"></div>
              </label>
            </div>

            <div class="setting-item-sm">
              <div class="flex-1">
                <h4 class="font-medium text-gray-900 dark:text-gray-100">搜索结果</h4>
                <p class="text-xs text-gray-500 dark:text-gray-400">搜索单词时自动朗读</p>
              </div>
              <label class="toggle">
                <input v-model="settings.speakOnSearch" @change="updateSettings" type="checkbox" class="sr-only peer">
                <div class="toggle-track peer-checked:bg-primary-500"></div>
              </label>
            </div>

            <!-- 朗读参数 -->
            <div class="pt-4 border-t border-gray-100 dark:border-gray-700 space-y-4">
              <div>
                <label class="flex items-center justify-between mb-3">
                  <span class="font-medium text-gray-900 dark:text-gray-100">朗读语速</span>
                  <span class="badge badge-primary">{{ settings.speakSpeed }}x</span>
                </label>
                <input
                  v-model="settings.speakSpeed"
                  @input="updateSettings"
                  type="range"
                  min="0.5"
                  max="2.0"
                  step="0.1"
                  class="range-slider"
                >
                <div class="flex justify-between text-xs text-gray-400 mt-1">
                  <span>慢 0.5x</span>
                  <span>正常 1.0x</span>
                  <span>快 2.0x</span>
                </div>
              </div>

              <div>
                <label class="block font-medium text-gray-900 dark:text-gray-100 mb-2">朗读语言</label>
                <select v-model="settings.speakLanguage" @change="updateSettings" class="input">
                  <option value="en-US">🇺🇸 英语 (美式)</option>
                  <option value="en-GB">🇬🇧 英语 (英式)</option>
                  <option value="en-AU">🇦🇺 英语 (澳式)</option>
                  <option value="zh-CN">🇨🇳 中文 (普通话)</option>
                </select>
              </div>
            </div>
          </div>
        </transition>
      </div>

      <!-- 显示设置 -->
      <div class="card">
        <h2 class="text-xl font-bold text-gray-900 dark:text-gray-100 mb-6 flex items-center gap-3">
          <span class="text-2xl">👁️</span>
          显示设置
        </h2>
        
        <div class="space-y-4">
          <div class="setting-item">
            <div class="flex-1">
              <h3 class="font-semibold text-gray-900 dark:text-gray-100">显示音标</h3>
              <p class="text-sm text-gray-500 dark:text-gray-400 mt-0.5">在单词旁边显示音标</p>
            </div>
            <label class="toggle">
              <input v-model="settings.showPhonetic" @change="updateSettings" type="checkbox" class="sr-only peer">
              <div class="toggle-track peer-checked:bg-primary-500"></div>
            </label>
          </div>

          <div class="setting-item">
            <div class="flex-1">
              <h3 class="font-semibold text-gray-900 dark:text-gray-100">显示例句</h3>
              <p class="text-sm text-gray-500 dark:text-gray-400 mt-0.5">显示单词的例句</p>
            </div>
            <label class="toggle">
              <input v-model="settings.showExamples" @change="updateSettings" type="checkbox" class="sr-only peer">
              <div class="toggle-track peer-checked:bg-primary-500"></div>
            </label>
          </div>
        </div>
      </div>

      <!-- 操作按钮 -->
      <div class="card">
        <div class="flex items-center justify-between">
          <button @click="resetSettings" class="btn btn-secondary">
            <span>🔄</span>
            重置设置
          </button>
          
          <div class="flex items-center gap-2 text-sm text-green-600 dark:text-green-400">
            <span>✅</span>
            设置已自动保存
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { getSettings, saveSettings, resetSettings as resetAllSettings } from '@/utils/settings'

const settings = ref({})

function loadSettings() {
  settings.value = getSettings()
}

function updateSettings() {
  saveSettings(settings.value)
}

function resetSettings() {
  if (confirm('确定要重置所有设置吗？')) {
    settings.value = resetAllSettings()
  }
}

onMounted(() => {
  loadSettings()
})
</script>

<style scoped>
.setting-item {
  @apply flex items-center justify-between py-4 border-b border-gray-100 dark:border-gray-700 last:border-b-0;
}

.setting-item-sm {
  @apply flex items-center justify-between py-2;
}

.toggle {
  @apply relative inline-flex items-center cursor-pointer;
}

.toggle-track {
  @apply w-12 h-6 bg-gray-200 dark:bg-gray-700 rounded-full transition-colors;
  @apply after:content-[''] after:absolute after:top-0.5 after:left-0.5;
  @apply after:bg-white after:rounded-full after:h-5 after:w-5 after:transition-transform;
  @apply peer-checked:after:translate-x-6;
}

.range-slider {
  @apply w-full h-2 bg-gray-200 dark:bg-gray-700 rounded-full appearance-none cursor-pointer;
}

.range-slider::-webkit-slider-thumb {
  @apply appearance-none w-5 h-5 bg-primary-500 rounded-full cursor-pointer shadow-md;
  @apply hover:bg-primary-600 transition-colors;
}

.range-slider::-moz-range-thumb {
  @apply w-5 h-5 bg-primary-500 rounded-full cursor-pointer shadow-md border-0;
  @apply hover:bg-primary-600 transition-colors;
}

.slide-down-enter-active,
.slide-down-leave-active {
  transition: all 0.3s ease;
}

.slide-down-enter-from,
.slide-down-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}
</style>
