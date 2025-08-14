<template>
  <div ref="container" class="three-container"></div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import * as THREE from 'three'

const container = ref(null)
let scene, camera, renderer, raycaster, mouse, stars

onMounted(() => {
  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(75, container.value.clientWidth / container.value.clientHeight, 0.1, 1000)
  camera.position.z = 5

  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(container.value.clientWidth, container.value.clientHeight)
  container.value.appendChild(renderer.domElement)

  raycaster = new THREE.Raycaster()
  mouse = new THREE.Vector2()

  const geometry = new THREE.SphereGeometry(0.1, 8, 8)
  const material = new THREE.MeshBasicMaterial({ color: 0xffff00 })
  stars = []
  for (let i = 0; i < 20; i++) {
    const star = new THREE.Mesh(geometry, material)
    star.position.set((Math.random() - 0.5) * 10, (Math.random() - 0.5) * 5, 0)
    scene.add(star)
    stars.push(star)
  }

  container.value.addEventListener('click', onClick)
  animate()
})

function onClick(event) {
  const rect = container.value.getBoundingClientRect()
  mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1
  mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1

  raycaster.setFromCamera(mouse, camera)
  const intersects = raycaster.intersectObjects(stars)
  if (intersects.length > 0) {
    const star = intersects[0].object
    star.material.color.set(0xff0000)
    // 简单爆炸效果：放大后移除
    star.scale.set(2, 2, 2)
    setTimeout(() => {
      scene.remove(star)
    }, 300)
  }
}

function animate() {
  requestAnimationFrame(animate)
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
