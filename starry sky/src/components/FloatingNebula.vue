<template>
  <div ref="container" class="three-container"></div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import * as THREE from 'three'

const container = ref(null)
let scene, camera, renderer, nebula

onMounted(() => {
  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(75, container.value.clientWidth / container.value.clientHeight, 0.1, 1000)
  camera.position.z = 5

  renderer = new THREE.WebGLRenderer({ alpha: true })
  renderer.setSize(container.value.clientWidth, container.value.clientHeight)
  container.value.appendChild(renderer.domElement)

  const textureLoader = new THREE.TextureLoader()
  textureLoader.load('/textures/nebula.png', texture => {
    const material = new THREE.SpriteMaterial({ map: texture, transparent: true, opacity: 0.8 })
    nebula = new THREE.Sprite(material)
    nebula.scale.set(10, 10, 1)
    scene.add(nebula)
  })

  animate()
})

function animate() {
  requestAnimationFrame(animate)
  if (nebula) nebula.rotation += 0.0005
  renderer.render(scene, camera)
}

onBeforeUnmount(() => {
  renderer.dispose()
})
</script>

<style scoped>
.three-container {
  width: 100%;
  height: 100%;
}
</style>
