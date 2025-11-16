<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import { ref, onMounted } from 'vue'
import { isAuthenticated } from '@/utils/request/api'

const year = new Date().getFullYear()
const isLoggedIn = ref(false)

onMounted(() => {
  isLoggedIn.value = isAuthenticated()
  
  // Re-check auth status on storage changes
  window.addEventListener('storage', () => {
    isLoggedIn.value = isAuthenticated()
  })
})
</script>

<template>
  <div class="layout">
    <header class="navbar bg-base-100 shadow-md">
      <div class="navbar-start">
        <div class="flex-1">
          <h1 class="btn btn-ghost normal-case text-2xl font-bold">EasyTierMC Uptime</h1>
        </div>
      </div>
      <div class="navbar-end lg:flex">
        <ul class="menu menu-horizontal px-1 gap-2">
          <li><RouterLink to="/" class="btn btn-ghost">主页</RouterLink></li>
          <li><RouterLink to="/monitor" class="btn btn-ghost">节点监控</RouterLink></li>
          <li><RouterLink to="/submit" class="btn btn-ghost">提交节点</RouterLink></li>
          <li v-if="isLoggedIn"><RouterLink to="/dashboard" class="btn btn-ghost">管理控制台</RouterLink></li>
          <li v-else><RouterLink to="/login" class="btn btn-ghost">登录</RouterLink></li>
        </ul>
      </div>
    </header>
    <main class="flex-1">
      <RouterView />
    </main>
    <footer class="footer footer-center bg-base-200 text-base-content p-4 border-t">
      <div>
        <small>© {{ year }} EasyTierMC Uptime</small>
      </div>
    </footer>
  </div>
</template>

<style scoped>
.layout {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

header {
  flex-shrink: 0;
}

main {
  flex: 1;
  padding: 1.25rem 1.5rem 2rem;
}

footer {
  flex-shrink: 0;
}

@media (min-width: 1024px) {
  main { 
    padding: 1.75rem 2rem 2.5rem; 
  }
}
</style>
