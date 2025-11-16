<script setup lang="ts">
import { onMounted, ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import {
  getMe,
  listAdmins,
  deleteAdmin,
  listApiKeys,
  createApiKey,
  toggleApiKeyStatus,
  deleteApiKey,
  listNodes,
  deleteNode,
  updateNodeStatus,
  logout
} from '@/utils/request/api'

const router = useRouter()

// User info
const currentUser = ref<any>(null)
const loading = ref(true)
const error = ref<string | null>(null)

// Active tab
const activeTab = ref<'nodes' | 'apiKeys' | 'users'>('nodes')

// Nodes
const nodeList = ref<any[]>([])
const loadingNodes = ref(false)

// API Keys
const apiKeys = ref<any[]>([])
const loadingApiKeys = ref(false)
const showApiKeyForm = ref(false)
const apiKeyForm = ref({ name: '', rateLimit: 100 })

// Admins
const userList = ref<any[]>([])
const loadingAdmins = ref(false)

const isAdmin = computed(() => currentUser.value?.username === 'admin')

async function fetchUserInfo() {
  try {
    const { data } = await getMe()
    currentUser.value = data
  } catch (e: any) {
    console.error(e);
    
    error.value = '未授权，请先登录'
    logout()
    setTimeout(() => router.push('/login'), 1500)
  } finally {
    loading.value = false
  }
}

// Nodes management
async function fetchNodes() {
  loadingNodes.value = true
  try {
    const {data} = await listNodes()
    nodeList.value = data
  } catch (e: any) {
    console.error('获取节点失败:', e)
  } finally {
    loadingNodes.value = false
  }
}

async function handleDeleteNode(id: string | number) {
  if (!confirm('确定删除此节点？')) return
  try {
    await deleteNode(id)
    await fetchNodes()
  } catch (e: any) {
    alert('删除失败: ' + (e.message || e))
  }
}

async function handleToggleNodeStatus(node: any) {
  try {
    const newStatus = node.status === 'active' ? 'inactive' : 'active'
    await updateNodeStatus(node.id, newStatus)
    await fetchNodes()
  } catch (e: any) {
    alert('更新状态失败: ' + (e.message || e))
  }
}

// API Keys management
async function fetchApiKeys() {
  loadingApiKeys.value = true
  try {
    const {data} = await listApiKeys()
    
    apiKeys.value = data
  } catch (e: any) {
    console.error('获取 API Keys 失败:', e)
  } finally {
    loadingApiKeys.value = false
  }
}

async function handleCreateApiKey() {
  if (!apiKeyForm.value.name) return
  try {
    await createApiKey(apiKeyForm.value)
    apiKeyForm.value = { name: '', rateLimit: 100 }
    showApiKeyForm.value = false
    await fetchApiKeys()
  } catch (e: any) {
    alert('创建 API Key 失败: ' + (e.message || e))
  }
}

async function handleToggleApiKeyStatus(key: any) {
  try {
    const newStatus = key.is_active === 'active' ? 'inactive' : 'active'
    await toggleApiKeyStatus(key.id, newStatus)
    await fetchApiKeys()
  } catch (e: any) {
    alert('更新状态失败: ' + (e.message || e))
  }
}

async function handleDeleteApiKey(id: string | number) {
  if (!confirm('确定删除此 API Key')) return
  try {
    await deleteApiKey(id)
    await fetchApiKeys()
  } catch (e: any) {
    alert('删除失败: ' + (e.message || e))
  }
}

// Admins management
async function fetchAdmins() {
  if (!isAdmin.value) return
  loadingAdmins.value = true
  try {
    const { data } = await listAdmins()
    
    userList.value = data
  } catch (e: any) {
    console.error('获取管理员列表失败:', e)
  } finally {
    loadingAdmins.value = false
  }
}

async function handleDeleteAdmin(id: string | number) {
  if (!confirm('确定删除此管理员？')) return
  try {
    await deleteAdmin(id)
    await fetchAdmins()
  } catch (e: any) {
    alert('删除失败: ' + (e.message || e))
  }
}

function handleLogout() {
  logout()
  router.push('/login')
}

// Tab switching
function switchTab(tab: 'nodes' | 'apiKeys' | 'users') {
  activeTab.value = tab
  if (tab === 'nodes' && nodeList.value.length === 0) fetchNodes()
  if (tab === 'apiKeys' && apiKeys.value.length === 0) fetchApiKeys()
  if (tab === 'users' && userList.value.length === 0) fetchAdmins()
}

onMounted(async () => {
  await fetchUserInfo()
  
  if (!error.value) await fetchNodes()
})
</script>

<template>
  <div class="max-w-7xl mx-auto">
    <div v-if="loading" class="text-center py-8">加载中...</div>
    <div v-else-if="error" class="alert alert-error">{{ error }}</div>
    <div v-else>
      <!-- Header -->
      <div class="flex justify-between items-center mb-6">
        <div>
          <h1 class="text-3xl font-bold">管理控制台</h1>
          <p class="text-base-content/70 mt-1">
            欢迎，{{ currentUser?.username }}
            <span v-if="currentUser?.role" class="badge badge-sm ml-2">{{ currentUser.role }}</span>
          </p>
        </div>
        <button class="btn btn-ghost btn-sm" @click="handleLogout">退出登录</button>
      </div>

      <!-- Tabs -->
      <div role="tablist" class="tabs tabs-bordered mb-4">
        <a role="tab" class="tab" :class="{ 'tab-active': activeTab === 'nodes' }"
          @click="switchTab('nodes')">
          节点管理
        </a>
        <a role="tab" class="tab" :class="{ 'tab-active': activeTab === 'apiKeys' }"
          @click="switchTab('apiKeys')">
          API Keys
        </a>
        <a v-if="isAdmin" role="tab" class="tab" :class="{ 'tab-active': activeTab === 'users' }"
          @click="switchTab('users')">
          管理员
        </a>
      </div>

      <!-- Nodes Tab -->
      <div v-show="activeTab === 'nodes'" class="space-y-4">
        <div class="flex justify-between items-center">
          <h2 class="text-xl font-semibold">节点列表</h2>
        </div>

        <!-- Nodes Table -->
        <div class="overflow-x-auto">
          <table class="table table-zebra w-full">
            <thead>
              <tr>
                <th>ID</th>
                <th>名称</th>
                <th>地址</th>
                <th>协议</th>
                <th>最大连接</th>
                <th>中继</th>
                <th>区域</th>
                <th>运营商</th>
                <th>状态</th>
                <th>响应时间</th>
                <th>操作</th>
              </tr>
            </thead>
            <tbody>
              <tr v-if="loadingNodes">
                <td colspan="11" class="text-center">加载中...</td>
              </tr>
              <tr v-else-if="nodeList.length === 0">
                <td colspan="11" class="text-center text-base-content/60">暂无节点</td>
              </tr>
              <tr v-else v-for="node in nodeList" :key="node.id">
                <td>{{ node.id }}</td>
                <td>
                  <div class="font-medium">{{ node.name }}</div>
                  <div v-if="node.description" class="text-xs text-base-content/60">{{ node.description }}</div>
                </td>
                <td>
                  <div class="font-mono text-sm">{{ node.host }}:{{ node.port }}</div>
                  <div v-if="node.network_name" class="text-xs text-base-content/60">网络: {{ node.network_name }}</div>
                </td>
                <td>
                  <span class="badge badge-sm">{{ node.protocol?.toUpperCase() }}</span>
                </td>
                <td>{{ node.max_connections }}</td>
                <td>
                  <span v-if="node.allow_relay" class="badge badge-success badge-sm">允许</span>
                  <span v-else class="badge badge-ghost badge-sm">禁止</span>
                </td>
                <td>{{ node.region || '-' }}</td>
                <td>{{ node.ISP || '-' }}</td>
                <td>
                  <span
                    class="badge badge-sm"
                    :class="{
                      'badge-success': node.status === 'Online',
                      'badge-error': node.status === 'Offline',
                      'badge-warning': node.status === 'Checking',
                      'badge-ghost': !node.status
                    }"
                  >
                    {{ node.status || 'Unknown' }}
                  </span>
                  <div v-if="node.last_status_update" class="text-xs text-base-content/60 mt-1">
                    {{ new Date(node.last_status_update).toLocaleString() }}
                  </div>
                </td>
                <td>
                  <span v-if="node.response_time">{{ node.response_time }}ms</span>
                  <span v-else class="text-base-content/60">-</span>
                </td>
                <td>
                  <div class="flex gap-1">
                    <button
                      class="btn btn-xs btn-ghost"
                      @click="handleToggleNodeStatus(node)"
                      :title="node.status === 'Online' ? '设为离线' : '设为在线'"
                    >
                      切换
                    </button>
                    <button
                      class="btn btn-xs btn-error"
                      @click="handleDeleteNode(node.id)"
                    >
                      删除
                    </button>
                  </div>
                  <div v-if="node.qq_number || node.mail" class="text-xs text-base-content/60 mt-1">
                    <div v-if="node.qq_number">QQ: {{ node.qq_number }}</div>
                    <div v-if="node.mail">{{ node.mail }}</div>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- API Keys Tab -->
      <div v-show="activeTab === 'apiKeys'" class="space-y-4">
        <div class="flex justify-between items-center">
          <h2 class="text-xl font-semibold">API Key 列表</h2>
          <button class="btn btn-primary btn-sm" @click="showApiKeyForm = !showApiKeyForm">
            {{ showApiKeyForm ? '取消' : '+ 创建 API Key' }}
          </button>
        </div>

        <!-- API Key Form -->
        <div v-if="showApiKeyForm" class="card bg-base-200 shadow">
          <div class="card-body">
            <h3 class="card-title text-lg">创建新 API Key</h3>
            <form @submit.prevent="handleCreateApiKey" class="grid gap-3">
              <input v-model="apiKeyForm.name" type="text" placeholder="API Key 名称"
                class="input input-bordered" required />
              <input v-model.number="apiKeyForm.rateLimit" type="number" placeholder="速率限制（次/分钟）"
                class="input input-bordered" min="1" />
              <button type="submit" class="btn btn-primary">创建</button>
            </form>
          </div>
        </div>

        <!-- API Keys Table -->
        <div class="overflow-x-auto">
          <table class="table table-zebra w-full">
            <thead>
              <tr>
                <th>ID</th>
                <th>Key</th>
                <th>描述</th>
                <th>速率限制</th>
                <th>状态</th>
                <th>操作</th>
              </tr>
            </thead>
            <tbody>
              <tr v-if="loadingApiKeys">
                <td colspan="6" class="text-center">加载中...</td>
              </tr>
              <tr v-else-if="apiKeys.length === 0">
                <td colspan="6" class="text-center text-base-content/60">暂无 API Key</td>
              </tr>
              <tr v-else v-for="key in apiKeys" :key="key.id">
                <td>{{ key.id }}</td>
                <td class="font-mono text-xs">{{ key.key }}</td>
                <td>{{ key.description }}</td>
                <td>{{ key.rate_limit}}</td>
                <td>
                  <span class="badge" :class="key.status === 'active' ? 'badge-success' : 'badge-ghost'">
                    {{ key.is_active }}
                  </span>
                </td>
                <td>
                  <button class="btn btn-xs btn-ghost" @click="handleToggleApiKeyStatus(key)" >
                    切换状态
                  </button>
                  <button class="btn btn-xs btn-error ml-1" @click="handleDeleteApiKey(key.id)"> 
                    删除
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- Admins Tab -->
      <div v-if="isAdmin" v-show="activeTab === 'users'" class="space-y-4">
        <div class="flex justify-between items-center">
          <h2 class="text-xl font-semibold">管理员列表</h2>
        </div>

        <div class="overflow-x-auto">
          <table class="table table-zebra w-full">
            <thead>
              <tr>
                <th>ID</th>
                <th>用户名</th>
                <th>Email</th>
                <th>创建时间</th>
                <th>更新时间</th>
                <th>操作</th>
              </tr>
            </thead>
            <tbody>
              <tr v-if="loadingAdmins">
                <td colspan="6" class="text-center">加载中...</td>
              </tr>
              <tr v-else-if="userList.length === 0">
                <td colspan="6" class="text-center text-base-content/60">暂无管理员</td>
              </tr>
              <tr v-else v-for="user in userList" :key="user.id">
                <td>{{ user.id }}</td>
                <td>{{ user.username }}</td>
                <td>{{ user.email }}</td>
                <td>{{ new Date(user.created_at).toLocaleDateString()}}</td>
                <td>{{ new Date(user.updated_at).toLocaleDateString()}}</td>
                <td>
                  <button v-if="user.id !== currentUser?.id" class="btn btn-xs btn-error" @click="handleDeleteAdmin(user.id)">
                    删除
                  </button>
                  <span v-else class="text-xs text-base-content/60">当前用户</span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      
    </div>
  </div>
</template>

<style scoped>
.tabs {
  margin-bottom: 1.5rem;
}
</style>
