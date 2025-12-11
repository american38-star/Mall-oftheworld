<template>
  <div class='topbar'><div class='app-title'>Alarms</div>
    <div class='topbar-actions'>
      <div class='lang-wrapper'>
        <button class='icon-btn' @click='toggleLang'>{{ currentLangLabel }}</button>
        <div v-if='showLang' class='lang-menu'>
          <div class='lang-item' @click='setLang("ar")'>العربية</div>
          <div class='lang-item' @click='setLang("en")'>English</div>
          <div class='lang-item' @click='setLang("tr")'>Türkçe</div>
        </div>
      </div>
      <button class='support-btn' @click='goSupport'>الدعم</button>
    </div>
  </div>
</template>
<script setup>
import { ref, computed } from 'vue'
import { useAuthStore } from '../store/auth'
const store=useAuthStore()
const showLang=ref(false)
const toggleLang=()=>showLang.value=!showLang.value
const setLang=(c)=>{store.setLang(c);showLang.value=false}
const goSupport=()=>window.open('https://t.me/American_38X','_blank')
const currentLangLabel=computed(()=>store.lang==='ar'?'العربية':store.lang==='en'?'English':'Türkçe')
</script>
<style scoped>
.topbar{display:flex;justify-content:space-between;padding:12px;background:linear-gradient(90deg,#0f67d7,#79b7ff);color:white}
</style>