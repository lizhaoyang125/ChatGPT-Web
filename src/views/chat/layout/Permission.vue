<script setup lang='ts'>
import { computed, ref } from 'vue'
import { NButton, NInput, NModal, useMessage } from 'naive-ui'
import { fetchVerify } from '@/api'
import { useAuthStore } from '@/store'
import Icon403 from '@/icons/403.vue'


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

async function requestBackend(name:string, password:string) {
    var formData = new URLSearchParams();
    formData.append('nm', name);
    formData.append('pw', password);
    
    try {//    fetch('https://www.guitarslice.cn:5000/login', {
        const response = await fetch('https://www.guitarslice.cn:5000/login', {
            method: 'POST',
            headers: { 'Content-Type': 'application/x-www-form-urlencoded',},
            body: formData,
        });
        const data = await response.json();
        console.log(data["value"]);
        localStorage.setItem('valid_date', data["message"]);
        return data["value"]
        //if(data["value"]==0){return 1;}
        //else { return 0; }
    } catch (error) {
        console.error('Error:', error);
        return 404;
    }
}


async function handleVerify() {
  //const secretKey = token.value.trim()
  const secretKey= "Zhaoyang.pipixia"
  const enteredUsername = token.value.trim()
  const enteredPassword = password.value.trim()
  // 将数据存储到localStorage中
  localStorage.setItem('username', enteredUsername);
  localStorage.setItem('passwd', enteredPassword);

  // if (!secretKey || enteredPassword !== 'Akira.Pipixia') {
  //   ms.error('密码错误或未输入密钥')
  //   return
  // }
  const result = await requestBackend(enteredUsername, enteredPassword);
  switch(result)
  {
    case 0: 
      break;
    case 1:
      alert('您的账号已过期！')
      break;
    case 2:
      alert('用户名登录次数已经用完')
      break;
    case 3:
      alert('用户名不存在或密码错误')
      break;
    default:
      break;
  }
  //alert("登录成功,剩余登录次数-1");
  ms.success('登录成功,剩余登录次数-1')
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
        <NInput v-model:value="token" type="password" placeholder="请输入用户名" />
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
