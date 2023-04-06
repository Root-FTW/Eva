<script setup lang="ts">
import { NButton, NCheckbox, NDivider, useMessage } from 'naive-ui'
import { useFirebaseAuth } from 'vuefire'
import { ref, watch } from 'vue'
import { browserLocalPersistence, browserSessionPersistence } from '@firebase/auth'
import GoogleSignInButton from './components/GoogleSignInButton.vue'
import EmailSignIn from './components/EmailSignIn.vue'
import PhoneSignIn from './components/PhoneSignIn/index.vue'
import AuthLayout from '@/components/custom/AuthLayout.vue'
import { useAppStore, useAuthStore } from '@/store'
import { t } from '@/locales'
import { router } from '@/router'
import { verifyIdToken } from '@/api'

interface Props {
  visible: boolean
}

defineProps<Props>()

const ms = useMessage()

const isUsingEmail = ref(false)
const rememberMe = ref(false)

const { language } = useAppStore()

const authStore = useAuthStore()

const auth = useFirebaseAuth()
if (auth)
  auth.languageCode = language

watch(() => rememberMe.value, (newRememberMe) => {
  if (auth)
    auth.setPersistence(newRememberMe ? browserLocalPersistence : browserSessionPersistence)
})

async function setToken(token: string) {
  try {
    const resp = await verifyIdToken(token)
    authStore.setAuthState({
      ...resp.data,
      token,
    })
    router.replace('/')
  }
  catch {
    ms.error(t('auth.noPermissionToSignIn'), {
      duration: 900000,
      closable: true,
    })
  }
}

const appName = import.meta.env.VITE_APP_NAME
</script>

<template>
  <AuthLayout>
    <div class="px-4 rounded w-full max-w-lg">
      <header class="mb-8 text-center">
        <h1 class="text-4xl md:text-5xl font-bold text-slate-700 dark:text-slate-100">
          {{ t('auth.welcomeBack') }}
        </h1>
      </header>
      <div class="flex flex-col gap-4">
        <p class="text-slate-500 dark:text-slate-200 font-light text-xl text-center">
          {{ t("auth.signInTips", { appName }) }}
        </p>
        <GoogleSignInButton :on-success="setToken" />
				<NDivider class="my-2">
</NDivider>
				<div class="text-center my-4">
  <span class="text-white">Si no tienes acceso,</span>
  <a href="https://www.facebook.com/Jonathan.RootByte/" target="_blank"><NButton text type="info" size="small" class="ml-2">solicítalo aquí al administrador.</NButton></a>
</div>

<form action="https://www.paypal.com/cgi-bin/webscr" method="post" target="_top">
<input type="hidden" name="cmd" value="_s-xclick">
<input type="hidden" name="hosted_button_id" value="9VTY3873PGME8">
<table>
<tr><td><input type="hidden" name="on0" value="Tu correo de Gmail:">Tu correo de Gmail:</td></tr><tr><td><input type="text" name="os0" maxlength="200"></td></tr>
</table>
<input type="image" src="https://www.paypalobjects.com/en_US/i/btn/btn_subscribe_SM.gif" border="0" name="submit" alt="PayPal - The safer, easier way to pay online!">
<img alt="" border="0" src="https://www.paypalobjects.com/en_US/i/scr/pixel.gif" width="1" height="1">
</form>


				        <NDivider class="my-2">
          <span class="text-slate-600 dark:text-slate-200 font-semibold">
            {{ t("auth.or") }}
          </span>
        </NDivider>
        <PhoneSignIn v-if="!isUsingEmail" :on-success="setToken" />
        <EmailSignIn v-else />
        <div class="w-full flex justify-between">
          <NCheckbox :checked="rememberMe" @update:checked="rememberMe = $event">
            {{ t('auth.rememberMe') }}
          </NCheckbox>
          <NButton text type="primary" @click="isUsingEmail = !isUsingEmail">
            {{ isUsingEmail ? t('auth.signInWithPhone') : t('auth.signInWithEmail') }}
          </NButton>
        </div>

			 </div>
    </div>
  </AuthLayout>
</template>
