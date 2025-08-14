<template>
  <div class="scene">
    <div style="position: absolute; top: 20px; left: 20px; color: white; font-size: 24px; z-index: 1000;">hello world</div>
    <StarfieldSkybox />
   <StarScene />
  </div>
</template>

<script setup>

import StarfieldSkybox from './components/StarfieldSkybox.vue'
import StarScene from './components/StarScene.vue';
</script>

<style>
.scene {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  overflow: hidden;
  background-color: #000020; /* 微蓝色背景 */
}
</style>
