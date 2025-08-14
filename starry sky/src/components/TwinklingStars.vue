<template>
  <div ref="container" class="three-container"></div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import * as THREE from 'three'

const container = ref(null)
let scene, camera, renderer, stars, clock

onMounted(() => {
  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(75, container.value.clientWidth / container.value.clientHeight, 0.1, 1000)
  camera.position.z = 5

  renderer = new THREE.WebGLRenderer()
  renderer.setSize(container.value.clientWidth, container.value.clientHeight)
  container.value.appendChild(renderer.domElement)

  clock = new THREE.Clock()

  // 创建星点
  const geometry = new THREE.BufferGeometry()
  const positions = []
  for (let i = 0; i < 500; i++) {
    positions.push((Math.random() - 0.5) * 50)
    positions.push((Math.random() - 0.5) * 50)
    positions.push((Math.random() - 0.5) * 50)
  }
  geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3))

  const material = new THREE.PointsMaterial({ color: 0xffffff, size: 0.1, transparent: true })
  stars = new THREE.Points(geometry, material)
  scene.add(stars)

  animate()
})

function animate() {
  requestAnimationFrame(animate)
  const time = clock.getElapsedTime()
  stars.material.opacity = 0.5 + 0.5 * Math.sin(time * 2)
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
