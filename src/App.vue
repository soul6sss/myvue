
<template>
  <div class="app">
   <HelloWorld></HelloWorld>
  </div>
</template>
<script  lang="ts">
import HelloWorld from './components/HelloWorld.vue'
export default{
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style scoped>
.app {
background-color: aqua;
}
</style>
