<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
const year = new Date().getFullYear()
</script>

<template>
  <div class="layout">
    <header class="navbar bg-base-100 shadow-md">
      <div class="navbar-start">
        <div class="dropdown">
          <div tabindex="0" role="button" class="btn btn-ghost lg:hidden">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h8m-8 6h16" />
            </svg>
          </div>
          <ul tabindex="0" class="dropdown-content menu p-2 shadow bg-base-100 rounded-box w-52">
            <li><RouterLink to="/">主页</RouterLink></li>
            <li><RouterLink to="/monitor">节点监控</RouterLink></li>
            <li><RouterLink to="/submit">提交节点</RouterLink></li>
          </ul>
        </div>
        <div class="flex-1">
          <h1 class="btn btn-ghost normal-case text-2xl font-bold">EasyTierMC Uptime</h1>
        </div>
      </div>
      <div class="navbar-end hidden lg:flex">
        <ul class="menu menu-horizontal px-1 gap-2">
          <li><RouterLink to="/" class="btn btn-ghost">主页</RouterLink></li>
          <li><RouterLink to="/monitor" class="btn btn-ghost">节点监控</RouterLink></li>
          <li><RouterLink to="/submit" class="btn btn-ghost">提交节点</RouterLink></li>
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
