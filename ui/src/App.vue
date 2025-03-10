<script setup>
import { computed, onMounted, onBeforeUnmount } from 'vue';
import TheWorkView from './components/TheWorkView.vue';
import TheSettings from './components/TheSettings.vue';
import TheOctocat from './components/TheOctocat.vue';
import TheNotification from './components/TheNotification.vue';
import TheNameDisplay from './components/TheNameDisplay.vue';
import TheGraphView from './components/TheGraphView.vue';
import { wakeLock } from './pwa.js';
import { useAppStore } from './stores/appstore.js';
const store = useAppStore();
const info = computed(() => store.info);
const appInfo = computed(() => store.appInfo);
let awakeLock = null;
onMounted(async () => {
  awakeLock = await wakeLock.request();
  await store.init();
});
onBeforeUnmount(() => {
  wakeLock.release(awakeLock);
});
</script>

<template>
  <the-octocat :version="appInfo?.app_version"/>
  <div class="container p-2 no-click">
    <the-notification v-model="store.error" />
    <div class="columns">
      <div class="column is-half is-full-mobile">
        <the-name-display :name="info.name" :is-new-available="appInfo?.is_new_available"/>
      </div>
      <div v-show="store.isBusy" class="spinner"><div></div></div>
      <div class="column is-half is-full-mobile has-text-right-tablet has-text-left"></div>
    </div>
    <div class="columns is-multiline is-justify-content-center mb-5">
      <div class="column is-hidden-portrait is-6 m-0 is-8-widescreen">
        <the-graph-view/>
      </div>
      <div class="column is-flex is-6 m-0 is-4-widescreen is-justify-content-center">
        <the-work-view />
      </div>
    </div>
    <the-settings/>
  </div>
</template>
<style scoped>
@media (orientation: portrait) {
  .is-hidden-portrait {
    display: none;
  }
}
@keyframes spinner {
  to {
    transform: rotate(360deg);
  }
}
.spinner {
  position: fixed;
  top: 30px;
  left: calc(50% - 10px);
  z-index: 2;
}
.spinner div:before {
  content: '';
  box-sizing: border-box;
  position: absolute;
  top: 50%;
  left: 50%;
  width: 20px;
  height: 20px;
  margin-top: -10px;
  margin-left: -10px;
  border-radius: 50%;
  border: 2px solid #ccc;
  border-top-color: #000;
  animation: spinner 0.6s linear infinite;
}
</style>
