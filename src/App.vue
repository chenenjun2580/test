<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

const containerRef = ref<HTMLElement | null>(null)
let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let controls: OrbitControls
const meteorites: THREE.Mesh[] = []
let aurora: THREE.Mesh
let newYearText: THREE.Mesh
let particles: THREE.Points
let animationState = 'meteorite' // meteorite, aurora, transition, newyear
let animationTime = 0

// 初始化场景
function initScene() {
  // 创建场景
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x000011)

  // 创建相机
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.z = 5

  // 创建渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))

  if (containerRef.value) {
    containerRef.value.appendChild(renderer.domElement)
  }

  // 添加轨道控制器
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05

  // 创建流星雨
  createMeteorites()

  // 创建极光
  createAurora()

  // 创建3D新年快乐文字
  createNewYearText()

  // 创建粒子效果
  createParticles()

  // 监听窗口大小变化
  window.addEventListener('resize', onWindowResize)

  // 开始动画
  animate()
}

// 窗口大小变化处理
function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

// 创建流星雨
function createMeteorites() {
  const meteoriteGeometry = new THREE.CylinderGeometry(0.01, 0.1, 1, 8)
  const meteoriteMaterial = new THREE.MeshBasicMaterial({ color: 0xffffff })

  for (let i = 0; i < 50; i++) {
    const meteorite = new THREE.Mesh(meteoriteGeometry, meteoriteMaterial)
    
    // 随机位置
    meteorite.position.x = (Math.random() - 0.5) * 20
    meteorite.position.y = (Math.random() - 0.5) * 20
    meteorite.position.z = (Math.random() - 0.5) * 10
    
    // 随机旋转
    meteorite.rotation.x = Math.random() * Math.PI
    meteorite.rotation.y = Math.random() * Math.PI
    
    // 随机缩放
    meteorite.scale.setScalar(Math.random() * 0.5 + 0.5)
    
    scene.add(meteorite)
    meteorites.push(meteorite)
  }
}

// 创建极光
function createAurora() {
  const auroraGeometry = new THREE.PlaneGeometry(20, 10, 100, 100)
  const auroraMaterial = new THREE.ShaderMaterial({
    uniforms: {
      time: { value: 0 },
      color: { value: new THREE.Color(0x00ffff) }
    },
    vertexShader: `
      uniform float time;
      varying vec2 vUv;
      void main() {
        vUv = uv;
        vec3 pos = position;
        pos.z += sin(pos.x * 0.5 + time) * 0.1;
        pos.z += sin(pos.y * 0.5 + time * 0.5) * 0.1;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
      }
    `,
    fragmentShader: `
      uniform float time;
      uniform vec3 color;
      varying vec2 vUv;
      void main() {
        float strength = sin(vUv.y * 10.0 + time) * 0.5 + 0.5;
        strength *= sin(vUv.x * 5.0 + time * 0.5) * 0.5 + 0.5;
        strength *= 0.5 + 0.5 * sin(time);
        gl_FragColor = vec4(color, strength * 0.3);
      }
    `,
    transparent: true,
    blending: THREE.AdditiveBlending
  })

  aurora = new THREE.Mesh(auroraGeometry, auroraMaterial)
  aurora.position.z = -2
  scene.add(aurora)
  aurora.visible = false
}

// 创建3D新年快乐文字
function createNewYearText() {
  // 使用CanvasTexture创建包含中文字符的纹理
  const canvas = document.createElement('canvas')
  canvas.width = 1024
  canvas.height = 256
  const context = canvas.getContext('2d')
  
  if (context) {
    // 设置字体和样式
    context.fillStyle = '#000000'
    context.fillRect(0, 0, canvas.width, canvas.height)
    context.fillStyle = '#ff0000'
    context.font = 'bold 80px "Microsoft YaHei", "SimHei", sans-serif'
    context.textAlign = 'center'
    context.textBaseline = 'middle'
    
    // 绘制中文文本
    const text = '新年快乐 2026'
    context.fillText(text, canvas.width / 2, canvas.height / 2)
    
    // 创建纹理
    const texture = new THREE.CanvasTexture(canvas)
    texture.needsUpdate = true
    
    // 创建平面几何体
    const geometry = new THREE.PlaneGeometry(3, 0.8)
    const material = new THREE.MeshBasicMaterial({ 
      map: texture,
      transparent: true
    })
    
    newYearText = new THREE.Mesh(geometry, material)
    newYearText.position.z = -5
    scene.add(newYearText)
    newYearText.visible = false
  }

  // 添加灯光
  const ambientLight = new THREE.AmbientLight(0x404040)
  scene.add(ambientLight)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(1, 1, 1)
  scene.add(directionalLight)

  const pointLight = new THREE.PointLight(0xff0000, 1, 100)
  pointLight.position.set(0, 0, 5)
  scene.add(pointLight)
}

// 创建粒子效果
function createParticles() {
  const particleGeometry = new THREE.BufferGeometry()
  const particleCount = 10000

  const positions = new Float32Array(particleCount * 3)
  const colors = new Float32Array(particleCount * 3)

  for (let i = 0; i < particleCount * 3; i += 3) {
    // 随机位置
    positions[i] = (Math.random() - 0.5) * 50
    positions[i + 1] = (Math.random() - 0.5) * 50
    positions[i + 2] = (Math.random() - 0.5) * 50

    // 随机颜色
    colors[i] = Math.random() * 0.5 + 0.5 // 红色
    colors[i + 1] = Math.random() * 0.5 // 绿色
    colors[i + 2] = Math.random() * 0.5 // 蓝色
  }

  particleGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  particleGeometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))

  const particleMaterial = new THREE.PointsMaterial({
    size: 0.05,
    vertexColors: true,
    blending: THREE.AdditiveBlending,
    transparent: true,
    opacity: 0.8
  })

  particles = new THREE.Points(particleGeometry, particleMaterial)
  scene.add(particles)
  particles.visible = false
}

// 动画循环
function animate() {
  requestAnimationFrame(animate)

  animationTime += 0.01

  // 更新轨道控制器
  controls.update()

  // 无论处于哪个状态，都更新流星，这样流星效果会一直存在
  updateMeteorites()

  switch (animationState) {
    case 'meteorite':
      // 3秒后切换到极光效果
      if (animationTime > 3) {
        animationState = 'aurora'
        animationTime = 0
        aurora.visible = true
      }
      break

    case 'aurora':
      // 更新极光
      if (aurora.material instanceof THREE.ShaderMaterial && aurora.material.uniforms.time) {
        aurora.material.uniforms.time.value = animationTime
      }
      
      // 5秒后切换到穿越动画
      if (animationTime > 5) {
        animationState = 'transition'
        animationTime = 0
      }
      break

    case 'transition':
      // 穿越动画：相机快速移动
      camera.position.z -= 0.5
      
      // 2秒后切换到新年快乐效果
      if (animationTime > 2) {
        animationState = 'newyear'
        animationTime = 0
        camera.position.z = 5
        newYearText.visible = true
        particles.visible = true
      }
      break

    case 'newyear':
      // 更新3D文字旋转
      if (newYearText) {
        newYearText.rotation.y += 0.01
      }
      
      // 更新粒子效果
      if (particles) {
        particles.rotation.y += 0.005
      }
      break
  }

  // 渲染场景
  renderer.render(scene, camera)
}

// 更新流星雨
function updateMeteorites() {
  meteorites.forEach((meteorite) => {
    // 流星下落
    meteorite.position.y -= 0.1 + Math.random() * 0.2
    
    // 流星旋转
    meteorite.rotation.x += 0.01
    
    // 流星超出边界后重置
    if (meteorite.position.y < -10) {
      meteorite.position.y = 10
      meteorite.position.x = (Math.random() - 0.5) * 20
      meteorite.position.z = (Math.random() - 0.5) * 10
    }
  })
}

// 挂载时初始化
onMounted(() => {
  initScene()
})

// 卸载时清理
onUnmounted(() => {
  window.removeEventListener('resize', onWindowResize)
  if (containerRef.value && renderer.domElement) {
    containerRef.value.removeChild(renderer.domElement)
  }
  renderer.dispose()
  controls.dispose()
})
</script>

<template>
  <div ref="containerRef" class="container"></div>
</template>

<style scoped>
.container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}
</style>
