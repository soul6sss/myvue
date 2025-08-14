<template>
  <div ref="container" class="three-fullscreen"></div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import * as THREE from 'three'

const container = ref(null)

let scene, camera, renderer, stars, clock
// nebula变量已移除

let raycaster = new THREE.Raycaster()
let mouse = new THREE.Vector2()
let particles = []

function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

// 创建圆形粒子纹理
function createCircleTexture() {
  const canvas = document.createElement('canvas')
  canvas.width = 64
  canvas.height = 64
  
  const context = canvas.getContext('2d')
  
  // 创建径向渐变，从中心到边缘
  const gradient = context.createRadialGradient(
    canvas.width / 2, canvas.height / 2, 0,
    canvas.width / 2, canvas.height / 2, canvas.width / 2
  )
  
  // 设置渐变颜色
  gradient.addColorStop(0, 'rgba(255, 255, 255, 1)')
  gradient.addColorStop(0.2, 'rgba(255, 255, 255, 0.8)')
  gradient.addColorStop(0.5, 'rgba(255, 255, 255, 0.4)')
  gradient.addColorStop(1, 'rgba(255, 255, 255, 0)')
  
  // 填充圆形
  context.fillStyle = gradient
  context.beginPath()
  context.arc(canvas.width / 2, canvas.height / 2, canvas.width / 2, 0, Math.PI * 2, false)
  context.fill()
  
  // 创建纹理
  const texture = new THREE.Texture(canvas)
  texture.needsUpdate = true
  
  // 保存到全局变量以便复用
  window.circleTexture = texture
  return texture
}

onMounted(() => {
  scene = new THREE.Scene()

  camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.set(0, 0, 5)

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

  clock = new THREE.Clock()

  const loader = new THREE.TextureLoader()

  // 创建更暗的背景
  loader.load('/texturees/milkyway.png', texture => {
    texture.mapping = THREE.EquirectangularReflectionMapping
    
    // 创建一个暗色滤镜使背景更暗
    const darkFilter = new THREE.Color(0.3, 0.3, 0.4) // 暗蓝色滤镜
    scene.background = texture
    scene.fog = new THREE.FogExp2(0x000020, 0.001) // 添加雾效果，使远处更暗
  })

  // 星云纹理已移除
  // 不再加载星云纹理，以去除中间的蓝色方框
  
  // 创建圆形粒子纹理
  createCircleTexture()

  addStars()
  
  // 流星效果已移除

  container.value.addEventListener('click', onClick)
  window.addEventListener('resize', onWindowResize)

  animate()
})

onBeforeUnmount(() => {
  renderer.dispose()
  container.value.removeEventListener('click', onClick)
  window.removeEventListener('resize', onWindowResize)
})

function addStars() {
  const geometry = new THREE.BufferGeometry()
  const positions = []
  const sizes = []
  const colors = []
  
  // 增加星星数量到2000
  for (let i = 0; i < 2000; i++) {
    positions.push((Math.random() - 0.5) * 50)
    positions.push((Math.random() - 0.5) * 50)
    positions.push((Math.random() - 0.5) * 50)
    
    // 随机大小，让星空更有层次感
    sizes.push(0.05 + Math.random() * 0.15)
    
    // 随机颜色，增加一些蓝色和金色的星星
    const colorChoice = Math.random()
    if (colorChoice > 0.8) {
      // 蓝白色星星
      colors.push(0.8 + Math.random() * 0.2)
      colors.push(0.8 + Math.random() * 0.2)
      colors.push(1)
    } else if (colorChoice > 0.6) {
      // 金黄色星星
      colors.push(1)
      colors.push(0.8 + Math.random() * 0.2)
      colors.push(0.4 + Math.random() * 0.2)
    } else {
      // 白色星星
      const shade = 0.8 + Math.random() * 0.2
      colors.push(shade)
      colors.push(shade)
      colors.push(shade)
    }
  }
  
  geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3))
  geometry.setAttribute('size', new THREE.Float32BufferAttribute(sizes, 1))
  geometry.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3))

  // 使用我们创建的圆形粒子纹理
  const starTexture = window.circleTexture
  
  const material = new THREE.PointsMaterial({
    color: 0xffffff,
    size: 0.15,
    transparent: true,
    opacity: 0.8,
    map: starTexture,
    vertexColors: true,
    depthWrite: false,
    blending: THREE.AdditiveBlending
  })
  
  stars = new THREE.Points(geometry, material)
  scene.add(stars)
}

// 流星生成函数已移除

function onClick(event) {
  const rect = renderer.domElement.getBoundingClientRect()
  mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1
  mouse.y = - ((event.clientY - rect.top) / rect.height) * 2 + 1

  raycaster.setFromCamera(mouse, camera)

  const intersects = raycaster.intersectObject(stars, false)

  if (intersects.length > 0) {
    const intersect = intersects[0]
    spawnExplosion(intersect.point)
  }
}

// 樱花散开效果，散开的粒子逐渐变小直到消失
function spawnExplosion(position) {
  const particleCount = 200 // 增加粒子数量
  const geometry = new THREE.BufferGeometry()
  const positions = new Float32Array(particleCount * 3)
  const velocities = new Float32Array(particleCount * 3)
  const colors = new Float32Array(particleCount * 3)
  const sizes = new Float32Array(particleCount)
  const lifeTimes = new Float32Array(particleCount) // 每个粒子的生命周期
  const initialSizes = new Float32Array(particleCount) // 初始大小

  // 樱花颜色范围
  const colorPalette = [
    new THREE.Color('#ffd7e5'), // 浅粉
    new THREE.Color('#ffb7c5'), // 中粉
    new THREE.Color('#ff8fab'), // 深粉
    new THREE.Color('#ff69b4'), // 亮粉
    new THREE.Color('#fff0f5')  // 白粉
  ]

  for (let i = 0; i < particleCount; i++) {
    positions[i * 3] = position.x
    positions[i * 3 + 1] = position.y
    positions[i * 3 + 2] = position.z

    // 初始大小更大，更有樱花感
    const initialSize = 0.15 + Math.random() * 0.2
    sizes[i] = initialSize
    initialSizes[i] = initialSize

    // 随机生命周期，使粒子消失时间不同
    lifeTimes[i] = 0.8 + Math.random() * 1.2

    // 樱花散开效果，速度更慢更优雅
    const speed = 0.05 + Math.random() * 0.08
    
    // 更倾向于水平方向散开的樱花
    const theta = Math.random() * 2 * Math.PI
    const phi = (Math.random() * 0.5 + 0.25) * Math.PI // 更集中在水平面附近

    velocities[i * 3] = speed * Math.sin(phi) * Math.cos(theta)
    velocities[i * 3 + 1] = speed * Math.sin(phi) * Math.sin(theta) - 0.01 // 轻微下落
    velocities[i * 3 + 2] = speed * Math.cos(phi) * 0.5 // z方向速度减半

    // 随机选择颜色
    const colorIndex = Math.floor(Math.random() * colorPalette.length)
    const color = colorPalette[colorIndex]
    colors[i * 3] = color.r
    colors[i * 3 + 1] = color.g
    colors[i * 3 + 2] = color.b
  }

  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  geometry.setAttribute('velocity', new THREE.BufferAttribute(velocities, 3))
  geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))
  geometry.setAttribute('size', new THREE.BufferAttribute(sizes, 1))

  // 使用圆形纹理
  const material = new THREE.PointsMaterial({
    vertexColors: true,
    size: 0.2,
    transparent: true,
    opacity: 0.9,
    depthWrite: false,
    blending: THREE.AdditiveBlending,
    map: window.circleTexture // 使用圆形纹理
  })

  const particleSystem = new THREE.Points(geometry, material)
  particleSystem.userData = {
    life: 1.0,
    lifeTimes: lifeTimes,
    initialSizes: initialSizes
  }
  scene.add(particleSystem)
  particles.push(particleSystem)
}



function animate() {
  requestAnimationFrame(animate)
  const elapsed = clock.getElapsedTime()
  const delta = clock.getDelta()

  if (stars) {
    // 星星闪烁效果
    stars.material.opacity = 0.7 + 0.3 * Math.sin(elapsed * 1.5)
    
    // 让星星微微旋转
    stars.rotation.y = elapsed * 0.02
    stars.rotation.x = elapsed * 0.01
  }

  // 流星动画已移除

  // 星云动画已移除
  // if (nebula) {
  //   nebula.rotation.z += 0.0005
  // }

  // 粒子动画，只处理樱花粒子
  for (let i = particles.length - 1; i >= 0; i--) {
    const ps = particles[i]
    
    // 确保粒子系统有必要的属性
    if (!ps.geometry || !ps.geometry.attributes || !ps.geometry.attributes.position || !ps.userData) {
      // 如果缺少必要属性，移除粒子系统
      scene.remove(ps)
      particles.splice(i, 1)
      continue
    }
    
    const positions = ps.geometry.attributes.position.array
    
    // 确保size属性存在
    if (!ps.geometry.attributes.size) {
      scene.remove(ps)
      particles.splice(i, 1)
      continue
    }
    
    const sizes = ps.geometry.attributes.size.array
    
    // 更新樱花粒子
    if (ps.geometry.attributes.velocity) {
      const velocities = ps.geometry.attributes.velocity.array
      const lifeTimes = ps.userData.lifeTimes || [1.0] // 提供默认值
      const initialSizes = ps.userData.initialSizes || [0.1] // 提供默认值

      // 更新樱花粒子位置
      for (let j = 0; j < positions.length; j += 3) {
        // 添加一些随机摆动，模拟樱花飘落
        const wobble = Math.sin(elapsed * 3 + j) * 0.002
        positions[j] += velocities[j] + wobble
        positions[j + 1] += velocities[j + 1] - 0.001 // 轻微下落
        positions[j + 2] += velocities[j + 2] + wobble
        
        // 计算每个粒子的生命周期比例
        const particleIndex = j / 3
        const lifeTime = lifeTimes[particleIndex] || lifeTimes[0] || 1.0
        const lifeRatio = Math.max(0, ps.userData.life / lifeTime)
        
        // 粒子逐渐变小
        const initialSize = initialSizes[particleIndex] || initialSizes[0] || 0.1
        sizes[particleIndex] = initialSize * lifeRatio
      }
    }
    
    ps.geometry.attributes.position.needsUpdate = true
    ps.geometry.attributes.size.needsUpdate = true

    // 整体生命周期递减
    ps.userData.life -= delta * 0.8
    ps.material.opacity = Math.max(ps.userData.life * 0.8, 0) // 透明度随生命周期变化

    // 生命周期结束，移除粒子系统
    if (ps.userData.life <= 0) {
      scene.remove(ps)
      particles.splice(i, 1)
    }
  }

  renderer.render(scene, camera)
}
</script>

<style>
.three-fullscreen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 1;
  background-color: #000020; /* 微蓝色背景 */
}
</style>
