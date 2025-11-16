<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { login } from '@/utils/request/api'

const router = useRouter()

const username = ref('admin')
const password = ref('admin123')
const loading = ref(false)
const error = ref<string | null>(null)
const success = ref(false)

async function onSubmit() {
  if (!username.value || !password.value) {
    error.value = '请输入用户名和密码'
    return
  }
  loading.value = true
  error.value = null
  success.value = false
  try {
    await login(username.value, password.value)
    success.value = true
    // 登录成功后跳转到管理控制台
    setTimeout(() => router.push('/dashboard'), 600)
  } catch (e: any) {
    error.value = e?.message || '登录失败，请重试'
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div class="min-h-[calc(100vh-200px)] flex items-center justify-center">
    <div class="max-w-md w-full mx-auto px-4">
      <div class="card bg-base-100 shadow-lg">
        <div class="card-body">
          <h2 class="card-title text-2xl justify-center mb-4">登录</h2>

          <form @submit.prevent="onSubmit" class="flex flex-col gap-4">
            <label class="form-control w-full">
              <div class="label">
                <span class="label-text font-medium">用户账户</span>
              </div>
              <input v-model="username" type="text" class="input input-bordered w-full" autocomplete="username" />
            </label>
            <label class="form-control w-full">
              <div class="label">
                <span class="label-text font-medium">登陆密码</span>
              </div>
              <input v-model="password" type="password" class="input input-bordered w-full" autocomplete="current-password" />
            </label>
            
            <div v-if="error" class="alert alert-error">
              <span>{{ error }}</span>
            </div>
            <button class="btn btn-primary mt-2 w-full" type="submit" :disabled="loading">
              <span>{{loading ? '登录中...' : '登录' }}</span>
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
</style>
