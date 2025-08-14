<template>
  <!-- 直接用全屏 canvas -->
  <div ref="container" class="three-fullscreen"></div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import * as THREE from 'three'

const container = ref(null)
let scene, camera, renderer

function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

onMounted(() => {
  // 场景
  scene = new THREE.Scene()

  // 相机 - 减小FOV以缩小视角，让背景图片更完整地显示
  camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.set(0, 0, 2) // 调整相机位置，使背景更加突出

  // 渲染器（直接全屏）
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(window.devicePixelRatio)
  renderer.setClearColor(0x000020, 1) // 设置微蓝色背景
  renderer.domElement.style.position = 'absolute'
  renderer.domElement.style.top = '0'
  renderer.domElement.style.left = '0'
  renderer.domElement.style.width = '100%'
  renderer.domElement.style.height = '100%'
  container.value.appendChild(renderer.domElement)

  // 加载背景贴图
  const loader = new THREE.TextureLoader()
  loader.load('/texturees/milkyway.png', texture => {
    texture.mapping = THREE.EquirectangularReflectionMapping
    
    scene.background = texture
    
    animate()
  })

  // 监听窗口变化
  window.addEventListener('resize', onWindowResize)
})

function animate() {
  requestAnimationFrame(animate)
  renderer.render(scene, camera)
}

onBeforeUnmount(() => {
  renderer.dispose()
  window.removeEventListener('resize', onWindowResize)
})
</script>

<style>
.three-fullscreen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 0; /* 确保在底层 */
}
</style>
