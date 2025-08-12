<template>
  <div class="scene">
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
