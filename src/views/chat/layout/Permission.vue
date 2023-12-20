<script setup lang='ts'>
import { computed, ref } from 'vue'
import { NButton, NInput, NModal, useMessage } from 'naive-ui'
import { fetchVerify } from '@/api'
import { useAuthStore } from '@/store'
import Icon403 from '@/icons/403.vue'
import AV from 'leancloud-storage';
// 替换以下信息为你的 LeanCloud 应用程序信息
const appId = 'nKTP00LEDmPVi5WoVQ2wphHv-gzGzoHsz';
const appKey = 'QZrVfCMSB3DpjEkdsO9lAAZe';
AV.init({
  appId,
  appKey,
});
// 定义查询
const query = new AV.Query('chatxyz_user');

// 添加查询条件
query.equalTo('user_name', 'test');

// 执行查询
query.find().then(
  (results) => {
    // 查询成功，results 包含满足条件的数据数组
    console.log('Query results:', results);
  },
  (error) => {
    // 查询失败，error 包含错误信息
    console.error('Query error:', error);
  }
);

interface Props {
  visible: boolean
}

defineProps<Props>()

const authStore = useAuthStore()

const ms = useMessage()

const loading = ref(false)
const token = ref('')
const password = ref('')

const disabled = computed(() => !token.value.trim() || !password.value.trim() || loading.value)

async function handleVerify() {
  const secretKey = token.value.trim()
  const enteredPassword = password.value.trim()

  if (!secretKey || enteredPassword !== 'Akira.Pipixia') {
    ms.error('密码错误或未输入密钥')
    return
  }

  try {
    loading.value = true
    await fetchVerify(secretKey)
    authStore.setToken(secretKey)
    ms.success('验证成功')
    window.location.reload()
  }
  catch (error: any) {
    ms.error(error.message ?? '验证失败')
    authStore.removeToken()
    token.value = ''
    password.value = ''  // 清除密码输入框
  }
  finally {
    loading.value = false
  }
}

function handlePress(event: KeyboardEvent) {
  if (event.key === 'Enter' && !event.shiftKey) {
    event.preventDefault()
    handleVerify()
  }
}
</script>

<template>
  <NModal :show="visible" style="width: 90%; max-width: 640px">
    <div class="p-10 bg-white rounded dark:bg-slate-800">
      <div class="space-y-4">
        <header class="space-y-2">
          <h2 class="text-2xl font-bold text-center text-slate-800 dark:text-neutral-200">
            403
          </h2>
          <p class="text-base text-center text-slate-500 dark:text-slate-500">
            {{ $t('common.unauthorizedTips') }}
          </p>
          <Icon403 class="w-[200px] m-auto" />
        </header>
        <NInput v-model:value="token" type="password" placeholder="请输入密钥" />
        <NInput v-model:value="password" type="password" placeholder="请输入密码" @keypress="handlePress" />
        <NButton
          block
          type="primary"
          :disabled="disabled"
          :loading="loading"
          @click="handleVerify"
        >
          {{ $t('common.verify') }}
        </NButton>
      </div>
    </div>
  </NModal>
</template>
