<template>
  <div id="app">
    <IntroLoading v-if="showIntro" @done="onIntroDone" />
    <RiverScene v-else-if="showRiver" :lang="lang" @done="onRiverDone" />
    <div v-else>
      <router-view />
    </div>
    <CustomCursor />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import IntroLoading from './components/IntroLoading.vue'
import CustomCursor from './components/CustomCursor.vue'

const showIntro = ref(true)
const showRiver = ref(false)
const lang = ref('id')

function onIntroDone(selectedLang) {
  lang.value = selectedLang || 'id'
  showIntro.value = false
  showRiver.value = true
}

function onRiverDone() {
  showRiver.value = false
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body, #app {
  width: 100%;
  height: 100%;
}
</style>