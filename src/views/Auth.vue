<template>
  <div class="min-h-[70vh] flex items-center justify-center">
    <div class="w-full max-w-md">
      <!-- 装饰背景 -->
      <div class="absolute inset-0 -z-10 overflow-hidden">
        <div class="absolute top-1/4 left-1/4 w-72 h-72 bg-primary-400/20 rounded-full blur-3xl"></div>
        <div class="absolute bottom-1/4 right-1/4 w-96 h-96 bg-purple-400/20 rounded-full blur-3xl"></div>
      </div>

      <div class="card shadow-soft-lg">
        <!-- 头部 -->
        <div class="text-center mb-8">
          <div class="inline-flex items-center justify-center w-16 h-16 rounded-2xl bg-gradient-to-br from-primary-500 to-purple-500 mb-4">
            <span class="text-3xl">📚</span>
          </div>
          <h1 class="text-2xl font-bold text-gray-900 dark:text-gray-100">
            {{ isLogin ? '欢迎回来' : '创建账号' }}
          </h1>
          <p class="text-gray-500 dark:text-gray-400 mt-2">
            {{ isLogin ? '登录以同步您的学习进度' : '注册开始您的学习之旅' }}
          </p>
        </div>

        <form @submit.prevent="handleSubmit" class="space-y-5">
          <div>
            <label class="block text-sm font-semibold text-gray-700 dark:text-gray-300 mb-2">
              邮箱地址
            </label>
            <input
              v-model="email"
              type="email"
              required
              class="input"
              placeholder="your@email.com"
            />
          </div>

          <div>
            <label class="block text-sm font-semibold text-gray-700 dark:text-gray-300 mb-2">
              密码
            </label>
            <input
              v-model="password"
              type="password"
              required
              minlength="6"
              class="input"
              placeholder="至少 6 位字符"
            />
          </div>

          <transition name="fade">
            <div v-if="error" class="alert alert-danger">
              <span class="text-lg">⚠️</span>
              <span>{{ error }}</span>
            </div>
          </transition>

          <button
            type="submit"
            :disabled="loading"
            class="w-full btn btn-primary btn-lg"
          >
            <span v-if="loading" class="animate-spin">⏳</span>
            <span v-else>{{ isLogin ? '🔐' : '✨' }}</span>
            {{ loading ? '处理中...' : (isLogin ? '登录' : '注册') }}
          </button>
        </form>

        <!-- 分割线 -->
        <div class="divider my-8">
          <span class="divider-text">或</span>
        </div>

        <!-- 第三方登录 -->
        <button
          @click="handleGoogleSignIn"
          :disabled="loading"
          class="w-full btn btn-secondary btn-lg group"
        >
          <span class="text-xl group-hover:scale-110 transition-transform">🔐</span>
          <span>使用 Google 登录</span>
        </button>

        <!-- 切换登录/注册 -->
        <div class="mt-8 text-center">
          <button
            @click="isLogin = !isLogin"
            class="text-primary-600 dark:text-primary-400 hover:underline font-medium"
          >
            {{ isLogin ? '没有账号？立即注册' : '已有账号？立即登录' }}
          </button>
        </div>

        <!-- 服务条款 -->
        <div class="mt-6 text-center text-xs text-gray-500 dark:text-gray-400">
          <p>登录即表示您同意我们的服务条款和隐私政策</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { useUserStore } from '@/stores/user'

const router = useRouter()
const route = useRoute()
const userStore = useUserStore()

const isLogin = ref(true)
const email = ref('')
const password = ref('')
const loading = ref(false)
const error = ref(null)

async function handleSubmit() {
  error.value = null
  loading.value = true

  try {
    let result
    if (isLogin.value) {
      result = await userStore.signIn(email.value, password.value)
    } else {
      result = await userStore.signUp(email.value, password.value)
    }

    if (result.success) {
      const redirect = route.query.redirect || '/'
      router.push(redirect)
    } else {
      error.value = result.error || '操作失败，请重试'
    }
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

async function handleGoogleSignIn() {
  error.value = null
  loading.value = true

  try {
    const result = await userStore.signInWithGoogle()
    if (!result.success) {
      error.value = result.error || 'Google 登录失败'
    }
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}
</script>
