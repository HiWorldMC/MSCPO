<script setup lang="ts">
import { onMounted, onUnmounted, watch, nextTick, computed } from 'vue'
import DefaultTheme from 'vitepress/theme'
import { useData, useRoute } from 'vitepress'
import {
  useFlash,
  useBackground,
  useFeatures,
  useHeroMove,
} from '../composables/home.js'

import zhCN from '@arco-design/web-vue/lib/locale/lang/zh-cn';
import enUS from '@arco-design/web-vue/lib/locale/lang/en-us';

const { Layout } = DefaultTheme
// const route = useRoute()
const { site, theme, isDark, localeIndex } = useData()
const { base } = site.value
const { logoImg } = theme.value
const { flashEnable, flashStyle } = useFlash()
const { bgEnable, bgStyle } = useBackground()
const { ftStyle } = useFeatures()
const { parallaxEnable, heroMove } = useHeroMove()

const localeMap = {
  root: zhCN,
  zh_CN: zhCN,
};

const arcoLocale = computed(() => {
  return localeMap[localeIndex.value] || enUS
});

const pageBgEnable = theme.value.pageBgEnable || true
const pageBgOpacity = theme.value.pageBgOpacity || 0.8

const logoSrc = `${base}${logoImg}`.replaceAll('//', '/')

const renderHomeBg = () => {
  const domStyle = document.documentElement.style
  // 首页闪烁动画样式设置
  if (flashEnable && flashStyle) {
    domStyle.setProperty('--vt-bg-light', flashStyle)
  }
  // 文章背景图设置
  if (pageBgEnable && pageBgOpacity) {
    domStyle.setProperty('--vt-bg-doc', `rgba(var(--vt-c-bg-rgb), ${pageBgOpacity})`)
  }
  // 首页背景图设置
  if (bgEnable && bgStyle) {
    domStyle.setProperty('--vt-bg-content', bgStyle)
  }
  if (ftStyle) {
    const vpFeat: any = document.getElementsByClassName('VPFeatures')[0]
    const fs = vpFeat?.style
    if (fs) fs.background = ftStyle
  }
}

const getBrowserWidth = function () {
  if (window.innerWidth < 768) {
    return false;
  } else {
    return true;
  }

};

onMounted(() => {
  watch(isDark, async () => {
    if (isDark.value) {
      document.body.setAttribute('arco-theme', 'dark')
    } else {
      document.body.removeAttribute('arco-theme')
    }
  }, { immediate: true })
  renderHomeBg()
  if (getBrowserWidth()) {
    if (parallaxEnable) window.addEventListener('mousemove', heroMove)
  }
})

onUnmounted(() => {
  if (parallaxEnable) window.removeEventListener('mousemove', heroMove)
})
</script>

<template>
  <a-config-provider :locale="arcoLocale">
    <Layout>
      <template #home-hero-before>
        <img class="VPHeroLogo" :src="logoSrc" />
        <slot name="home-hero-before" />
      </template>
    </Layout>
  </a-config-provider>
</template>
