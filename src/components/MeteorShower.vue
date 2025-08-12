<template>
  <div ref="container" class="three-container"></div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import * as THREE from 'three'

const container = ref(null)
let scene, camera, renderer, meteors = []

onMounted(() => {
  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(60, container.value.clientWidth / container.value.clientHeight, 0.1, 1000)
  camera.position.z = 5

  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(container.value.clientWidth, container.value.clientHeight)
  container.value.appendChild(renderer.domElement)

  // 灯光
  const light = new THREE.AmbientLight(0xffffff, 1)
  scene.add(light)

  animate()
  setInterval(spawnMeteor, 1000)
})

function spawnMeteor() {
  const geometry = new THREE.SphereGeometry(0.05, 8, 8)
  const material = new THREE.MeshBasicMaterial({ color: 0x88ccff })
  const meteor = new THREE.Mesh(geometry, material)
  meteor.position.set(Math.random() * 8 - 4, Math.random() * 4, -5)
  meteor.userData.velocity = new THREE.Vector3(-0.1, -0.1, 0.1)
  scene.add(meteor)
  meteors.push(meteor)
}

function animate() {
  requestAnimationFrame(animate)
  meteors.forEach(m => {
    m.position.add(m.userData.velocity)
  })
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
